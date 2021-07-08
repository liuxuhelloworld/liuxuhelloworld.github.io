# Spring Data
Spring Data主要处理data persistence with a variety of different database types，包含很多子项目，比如：
- Spring Data JPA: JPA persistence against a relational database
- Spring Data MongoDB: persistence to a Mongo document database
- Spring Data Neo4j: persistence to a Neo4j graph database
- Spring Data Redis: persistence to a Redis key-value store
- Spring Data Cassandra: persistence to a Cassandra database

The major objective of the project is to provide useful extensions on top of Spring's core data access functionality to interact with databases other than traditional RDBMSs.

Spring Data最突出的特点就是，the ability to automatically create repositories based on a repository specification interface

# Spring Data JPA
Spring Data JPA是Spring Data的一部分，the main objective of Spring Data JPA is to provide additional features for simplifying application development with JPA.

## JPA是什么？
JPA(Java Persistence API)是JCP(Java Community Process)制定的标准，it is the Java API for the management of persistence and object/relational mapping in Java EE and Java SE environments.

The JPA specification is to standardize the ORM programming model in both the JSE and JEE environments. It defines a common set of concepts, annotations, interfaces, and other services that a JPA persistence provider should implement.

JPA is a complete and powerful ORM data access standard, and with the help of third-party libraries such as Spring Data JPA and Hibernate, you can implement various crosscutting concerns relatively easily.

## 常用JPA实现
常用的JPA实现有以下几种：
- Hibernate
- EclipseLink
- Oracle TopLink
- Apache OpenJPA

## 使用Spring Data JPA的步骤
1. add spring-boot-starter-data-jpa dependency
2. annotating your domain objects with JPA mapping annotations
3. 声明JPA repositories：继承CrudRepository即可，不需要显式实现CrudRepository的几十个方法，when the application starts, Spring Data JPA automatically generates an implementation on the fly
4. 定制化JPA repositories：CrudRepository提供的方法不满足使用？你可以自定义声明新的方法。依据verb + optional subject + By + predicate的模式添加自定义方法，比如：
- findByZip(String zip)
- getByZipAndCreatedAtBetween(String zip, Date start, Date end)
- findByStateAndCityAllIgnoresCase(String state, String city)
- findByCityOrderByState(String city)

	需要实现这个方法吗？不需要，when generating the repository implementation, Spring Data examines any methods in the repository interface, parses the method name, and attempts to understand the method's purpose in the context of the persisted object.

5. 对于更复杂的情形，name the method anything you want and annotate it with @Query to explicitly specify the query to be performed when the method is called.
@Query("Order o where o.city = 'Seattle'") List\<Order\> readOrdersDeliveredInSeattle

## EntityManager and EntityManagerFactory
EntityManager是JPA的核心概念，the main job of EntityManager is to maintain a persistence context, in which all the entity instances managed by it will be stored.

## JPA配置示例
```
	@Bean
	public EntityManagerFactory entityManagerFactory() {
		LocalContainerEntityManagerFactoryBean factoryBean = new LocalContainerEntityManagerFactoryBean();
		factoryBean.setDataSource(dataSource());
		factoryBean.setPackagesToScan("jpa.example.entities");
		factoryBean.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
		factoryBean.setJpaProperties(hibernateProperties());
		factoryBean.afterPropertiesSet();

		return factoryBean.getNativeEntityManagerFactory();
	}

	private Properties hibernateProperties() {
		Properties  hibernateProperties = new Properties();
		hibernateProperties.put("hibernate.dialect", "org.hibernate.dialect.MySQLDialect");
		hibernateProperties.put("hibernate.format_sql", true);
		hibernateProperties.put("hibernate.user_sql_comments", true);
		hibernateProperties.put("hibernate.show_sql", true);
		hibernateProperties.put("hibernate.max_fetch_depth", 3);
		hibernateProperties.put("hibernate.jdbc.batch_size", 10);
		hibernateProperties.put("hibernate.jdbc.fetch_size", 50);

		return hibernateProperties;
	}

	@Bean
	public PlatformTransactionManager transactionManager() {
		return new JpaTransactionManager(entityManagerFactory());
	}

```

## JPA Mapping示例
```
@Entity
@Table(name = "singer")
@NamedQueries({
	@NamedQuery(name = "Singer.findAll",
		query = "select s from Singer s"),
	@NamedQuery(name = "Singer.findAllWithAlbum",
		query = "select distinct s from Singer s " +
			"left join fetch s.albums a " +
			"left join fetch s.instruments i"),
	@NamedQuery(name = "Singer.findById",
		query = "select distinct s from Singer s " +
			"left join fetch s.albums a " +
			"left join fetch s.instruments i " +
			"where s.id = :id")
})
public class Singer implements Serializable {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	@Column(name = "id")
	private Long id;

	@Column(name = "first_name")
	private String firstName;

	@Column(name = "last_name")
	private String lastName;

	@Temporal(TemporalType.DATE)
	@Column(name = "birth_date")
	private Date birthDate;

	@Version
	@Column(name = "version")
	private Integer version;

	@OneToMany(mappedBy = "singer", cascade = CascadeType.ALL, orphanRemoval = true)
	private Set<Album> albums = new HashSet<>();

	@ManyToMany
	@JoinTable(name = "singer_instrument",
		joinColumns = @JoinColumn(name = "singer_id"),
		inverseJoinColumns = @JoinColumn(name = "instrument_id"))
	private Set<Instrument> instruments = new HashSet<>();

	public void setId(Long id) {
		this.id = id;
	}

	public Long getId() {
		return id;
	}

	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}

	public String getFirstName() {
		return firstName;
	}

	public void setLastName(String lastName) {
		this.lastName = lastName;
	}

	public String getLastName() {
		return lastName;
	}

	public void setBirthDate(Date birthDate) {
		this.birthDate = birthDate;
	}

	public Date getBirthDate() {
		return birthDate;
	}

	public void setVersion(Integer version) {
		this.version = version;
	}

	public Integer getVersion() {
		return version;
	}

	public void setAlbums(Set<Album> albums) {
		this.albums = albums;
	}

	public Set<Album> getAlbums() {
		return albums;
	}

	public boolean addAlbum(Album album) {
		album.setSinger(this);
		return getAlbums().add(album);
	}

	public boolean removeAlbum(Album album) {
		return getAlbums().remove(album);
	}

	public void setInstruments(Set<Instrument> instruments) {
		this.instruments = instruments;
	}

	public Set<Instrument> getInstruments() {
		return instruments;
	}

	@Override
	public String toString() {
		return "Singer[" +
			"id" + "[" + id + "]" + "," +
			"firstName" + "[" + firstName + "]" + ", " +
			"lastName" + "[" + lastName + "]" + "," +
			"birthDate" + "[" + birthDate + "]" +
			"]";
	}
}
```

## JPA增删改查示例
```
@Service("jpaSingerService")
@Repository
@Transactional
public class SingerServiceImpl implements SingerService {

	@PersistenceContext
	private EntityManager entityManager;

	@Transactional(readOnly = true)
	@Override
	public List<Singer> findAll() {
		return entityManager.createNamedQuery("Singer.findAll", Singer.class)
			.getResultList();
	}

	@Transactional(readOnly = true)
	@Override
	public List<Singer> findAllByNativeQuery() {
		return entityManager.createNativeQuery("SELECT id, first_name, last_name, birth_date, version " +
			"FROM singer", Singer.class)
			.getResultList();
	}

	@Transactional(readOnly = true)
	@Override
	public List<Singer> findByCriteriaQuery(String firstName, String lastName) {
		CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();

		CriteriaQuery<Singer> criteriaQuery = criteriaBuilder.createQuery(Singer.class);

		Root<Singer> singerRoot = criteriaQuery.from(Singer.class);
		singerRoot.fetch(Singer_.albums, JoinType.LEFT);
		singerRoot.fetch(Singer_.instruments, JoinType.LEFT);

		criteriaQuery.select(singerRoot).distinct(true);

		Predicate predicate = criteriaBuilder.conjunction();

		if (firstName != null) {
			Predicate p = criteriaBuilder.equal(singerRoot.get(Singer_.firstName), firstName);
			predicate = criteriaBuilder.and(predicate, p);
		}

		if (lastName != null) {
			Predicate p = criteriaBuilder.equal(singerRoot.get(Singer_.lastName), lastName);
			predicate = criteriaBuilder.and(predicate, p);
		}

		criteriaQuery.where(predicate);

		return entityManager.createQuery(criteriaQuery)
			.getResultList();
	}

	@Transactional(readOnly = true)
	@Override
	public List<Singer> findAllWithAlbum() {
		return entityManager.createNamedQuery("Singer.findAllWithAlbum", Singer.class)
			.getResultList();
	}

	@Transactional(readOnly = true)
	@Override
	public Singer findById(Long id) {
		return entityManager.createNamedQuery("Singer.findById", Singer.class)
			.setParameter("id", id)
			.getSingleResult();
	}

	@Override
	public Singer save(Singer singer) {
		if (singer.getId() == null) {
			entityManager.persist(singer);
		} else {
			entityManager.merge(singer);
		}

		return singer;
	}

	@Override
	public void delete(Singer singer) {
		Singer merged = entityManager.merge(singer);
		entityManager.remove(merged);
	}
}
```

# Spring Data REST
Spring Data REST is one member of the Spring Data family that automatically creates REST APIs for repositories created by Spring Data. 也就是说，如果你使用Spring Data，那么就可以通过Spring Data REST来自动创建REST APIs. Spring Data REST is great at creating endpoints for performing CRUD operations against Spring Data repositories. The REST endpoints that Spring Data REST creates are at least as good as the one you created yourself, and possibly even better. 

## 如何使用Spring Data REST?
1. 添加Spring Data REST dependency
```properties
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-data-rest</artifactId>
</dependency>
```
添加了这个dependency后，the application gets auto-configuration that enables automatic creation of a REST API for any repositories that were created by Spring Data.

2. 设置一个base path，这样就不会和你自己创建的endpoints冲突了
```properties
spring.data.rest.base-path=/api
```

3. 如何知道Spring Data  REST都创建了哪些endpoints呢？
请求你配置的base path就可以，比如，localhost:8080/api

4. Spring Data REST创建endpoints时是如何确定path的呢？
通过将entity class的名字复数化，比如，Ingredient对应的path是ingredients，Order对应的path是orders. 这种复数化的方式某些情况下可能有问题，比如，Taco复数化的path是tacoes，而你可能期待的是tacos. 为了避免这种出入，你可以通过@RestResource来修饰的对应的entity，明确定义你期望的path名称，比如，@RestResource(rel="tacos", path="tacos")

5. paging and sorting
如果你使用的repository是PagingAndSortingRepository，那么Spring Data REST创建的endpoints也会支持paging和sorting. 比如，localhost:8080/api/tacos?sort=createdAt,desc&size=5