# Spring JDBC packages
Spring对JDBC的支持由以下几个包通力实现：
- org.springframework.jdbc.core
This contains the foundations of JDBC classes in Spring. It includes the core JDBC class, JdbcTemplate, which simplies programming database operations with JDBC.

- org.springframework.jdbc.datasource
This contains helper classes and DataSource implementations that you can use to run JDBC code outside a JEE container.

- org.springframework.jdbc.object
This contains classes that help convert the data returned from the database into Java objects.

- org.springframework.jdbc.support
- org.springframework.jdbc.config
This contains classes that support JDBC configuration within Spring's ApplicationContext.

# DataSource
Spring可以帮你管理database connection，你只需要实现javax.sql.DataSource就可以了。换句话说，DataSource负责提供并管理Connections.

## DriverManagerDataSource
最简单的DataSource实现，不支持connection pooling.

### XML配置示例
```xml
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="${jdbc.driverClassName}" />
        <property name="url" value="${jdbc.url}" />
        <property name="username" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />
    </bean>

    <context:property-placeholder location="jdbc.properties" />
```

### Java Configuration示例
```java
@Configuration
@PropertySource("jdbc2.properties")
public class DbConfig {
	@Value("${driverClassName}")
	private String driverClassName;

	@Value("${url}")
	private String url;

	@Value("${username}")
	private String username;

	@Value("${password}")
	private String password;

	@Bean
	@Lazy
	public DataSource dataSource() {
		try {
			DriverManagerDataSource dataSource = new DriverManagerDataSource();
			dataSource.setDriverClassName(driverClassName);
			dataSource.setUrl(url);
			dataSource.setUsername(username);
			dataSource.setPassword(password);

			return dataSource;
		} catch (Exception ex) {
			return null;
		}
	}
}
```

## dbcp2.BasicDataSource
```xml
    <bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driverClassName}" />
        <property name="url" value="${jdbc.url}" />
        <property name="username" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />
    </bean>

    <context:property-placeholder location="jdbc.properties" />
```

## HikariDataSource
```xml
    <bean id="dataSource" class="com.zaxxer.hikari.HikariDataSource">
        <property name="jdbcUrl" value="${jdbcUrl}" />
        <property name="username" value="${username}" />
        <property name="password" value="${password}" />
        <property name="dataSourceProperties">
            <props>
                <prop key="cachePrepStmts">${dataSource.cachePrepStmts}</prop>
                <prop key="prepStmtCacheSize">${dataSource.prepStmtCacheSize}</prop>
                <prop key="prepStmtCacheSqlLimit">${dataSource.prepStmtCacheSqlLimit}</prop>
                <prop key="useServerPrepStmts">${dataSource.useServerPrepStmts}</prop>
                <prop key="useLocalSessionState">${dataSource.useLocalSessionState}</prop>
                <prop key="rewriteBatchedStatements">${dataSource.rewriteBatchedStatements}</prop>
                <prop key="cacheResultSetMetadata">${dataSource.cacheResultSetMetadata}</prop>
                <prop key="cacheServerConfiguration">${dataSource.cacheServerConfiguration}</prop>
                <prop key="elideSetAutoCommits">${dataSource.elideSetAutoCommits}</prop>
                <prop key="maintainTimeStats">${dataSource.maintainTimeStats}</prop>
            </props>
        </property>
    </bean>

    <context:property-placeholder location="jdbc-hikari.properties" />
```

## Spring embedded database
从Spring 3.0开始，Spring开始支持embedded database.

```xml
    <jdbc:embedded-database id="dataSource" type="H2">
        <jdbc:script location="classpath:db/schema.sql" />
        <jdbc:script location="classpath:db/test-data.sql" />
    </jdbc:embedded-database>
```

# JdbcTemplate
JdbcTemplate是Spring JDBC的核心，JdbcTemplate provides a means by which developers can perform SQL operations against a relational database without all the ceremony and boilerplate typically required when working with JDBC.

## NamedParameterJdbcTemplate
相比JdbcTemplate以placeholder的方式指定入参，NamedParameterJdbcTemplate支持以参数名的形式指定入参。

## RowMapper

## ResultSetExtractor

# MappingSqlQuery
MappingSqlQuery提供了一种更object-oriented的方式来做sql查询。

## 代码示例
```java
public class SelectAllSingers extends MappingSqlQuery<Singer> {
	private static final String SQL_SELECT_ALL_SINGERS = "select id, first_name, last_name, birth_date from singer";

	public SelectAllSingers(DataSource dataSource) {
		super(dataSource, SQL_SELECT_ALL_SINGERS);
	}

	@Override
	protected Singer mapRow(ResultSet rs, int rowNum) throws SQLException {
		Singer singer = new Singer();
		singer.setId(rs.getLong("id"));
		singer.setFirstName(rs.getString("first_name"));
		singer.setLastName(rs.getString("last_name"));
		singer.setBirthDate(rs.getDate("birth_date"));

		return singer;
	}
}
```

# SqlUpdate
SqlUpdate提供了一种更object-oriented的方式来做sql更新。

## 代码示例
```java
public class UpdateSinger extends SqlUpdate {
	private static final String SQL_UPDATE_SINGER = "update singer set first_name = :first_name, " +
		"last_name = :last_name, birth_date = :birth_date where id = :id";

	public UpdateSinger(DataSource dataSource) {
		super(dataSource, SQL_UPDATE_SINGER);
		super.declareParameter(new SqlParameter("first_name", Types.VARCHAR));
		super.declareParameter(new SqlParameter("last_name", Types.VARCHAR));
		super.declareParameter(new SqlParameter("birth_date", Types.DATE));
		super.declareParameter(new SqlParameter("id", Types.INTEGER));
	}
}
```

# KeyHolder
```java
	public void insert(Singer singer) {
		Map<String, Object> paramMap = new HashMap<>();
		paramMap.put("first_name", singer.getFirstName());
		paramMap.put("last_name", singer.getLastName());
		paramMap.put("birth_date", singer.getBirthDate());

		KeyHolder keyHolder = new GeneratedKeyHolder();
		insertSinger.updateByNamedParam(paramMap, keyHolder);
		singer.setId(keyHolder.getKey().longValue());
	}
```

# BatchSqlUpdate
暂略

# SqlFunction
暂略
