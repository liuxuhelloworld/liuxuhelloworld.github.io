# 如何使用Spring Security?
<artifactId>spring-boot-starter-security</artifactId>

Spring Security的第一步就是把spring-boot-starter-security添加到dependency中，when the application starts, autoconfiguration will detect that Spring Security is in the classpath and will set up some basic security configuration.

这些basic security configuration提供了什么呢？
- there is only one user, the username is #user#
- there is no login page
- all HTTP request paths require authentication, authentication is prompted with HTTP basic authentication

这些basic security configuration是不能满足真实需求的:
- user store，支持多用户，支持用户注册
- 我们需要login page，而不是使用HTTP basic dialog box
- 对不同的request path使用不同的security rules，比如，the homepage and registration pages should not require authentication

# 如何配置user store？
Spring Security支持多种配置user store的方式
- an in-memory user store
- a JDBC-based user store
- an LDAP-backed user store
- a custom user details service

无论使用哪种方式，都只需要继承WebSecurityConfigurerAdapter并重载configure方法。

## in-memory user store
把相关用户信息写死到配置中，适合测试时使用。

```java
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth
          .inMemoryAuthentication()
            .withUser("user1")
              .password("{noop}user1")
              .authorities("ROLE_USER")
            .and()
            .withUser("user2")
              .password("{noop}user2")
              .authorities("ROLE_USER");
    }
```

## JDBC-based user store
```java
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth
          .jdbcAuthentication()
            .dataSource(dataSource)
            .usersByUsernameQuery(
                "select username, password, enabled from Taco_Users " +
                "where username=?")
            .authoritiesByUsernameQuery(
                "select username, authority from Taco_User_Authorities " +
                "where username=?")
            .passwordEncoder(new BCryptPasswordEncoder(10));
    }
```

## custom user details service
JDBC-based user store是直接从数据库里读取信息，把这个过程封装一层，就是custom user details service，这样做有什么好处呢？因为我们总要做用户管理的，即用户注册、登录、鉴权等，这些也是和数据库打交道，把JDBC-based user store封装一层，一样是从数据库里读取信息，不过我们可以灵活地增加很多其他处理。

```java
   @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth
          .userDetailsService(userDetailsService)
          .passwordEncoder(encoder());
    }
```

```java
@Service
public class TacoUserDetailsService implements UserDetailsService {

    private UserRepository userRepository;

    @Autowired
    public TacoUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username);
        if (user != null) {
            return user;
        } else {
            throw new UsernameNotFoundException("User '" + username + "' not found");
        }
    }
}
```

# 如何配置security rules呢？
你把custom user details service搞好以后，就会发现一个问题，我们请求注册页面，但是会要求登录，而还没有注册就没有办法登录，这里的问题就是不应当对注册页面做登录保护。

**WebSecurityConfigurerAdapter**还有一个**configure**方法，可以用来对不同的request paths配置不同的security rules.

```java
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http
          .authorizeRequests()
            .antMatchers("/*/design", "/*/orders/current", "/*/orders")
              .hasRole("USER")
            .antMatchers("/", "/**")
              .permitAll()
          .and()
          .formLogin()
            .loginPage("/login")
            .loginProcessingUrl("/login")
            .usernameParameter("username")
            .passwordParameter("password")
            .defaultSuccessUrl("/", true)
          .and()
          .logout()
            .logoutSuccessUrl("/");
    }
```

# CSRF(Cross-Site Request Forgery)
CSRF是一种常见的安全攻击，if you are using cookies (or other authentication methods that the browser can do automatically) then you need CSRF protection. It involves subjecting a user to code on a maliciously designed web page that automatically (and usually secretly) submits a form to another application on behalf of a user who is often the victim of the attack.

那么如何预防这种攻击呢？很简单，在服务端生成并存储一个CSRF token，展示form时以隐藏字段下发给客户端，当客户端提交form时，把CSRF token再带回来，the request is then intercepted by the server and compared with the token that was originally generated，如果匹配，说明是真实的请求，如果不匹配，说明是伪造的请求。

Spring Security支持CSRF protection，并且默认开启CSRF protection.

同样的一个路径，get请求正常，但是post请求报错403，那么可能就是CSRF protection在起作用，需要在form中添加类似<input type="hidden" name="_csrf" th:value="${_csrf.token}" />这样的东西即可解决。

# 如何获取当前用户信息？
## Principal
```java
		@PostMapping
    public String processOrder(@Valid Order order, Errors errors, SessionStatus sessionStatus, Principal principal) {
        if (errors.hasErrors()) {
            return "jpaOrderForm";
        }

        log.info("Order submitted: " + order);

        order.setUser(userRepository.findByUsername(principal.getName()));

        orderRepository.save(order);

        // close session
        sessionStatus.setComplete();

        return "redirect:/";
    }
```

## Authentication
```java
    @PostMapping
    public String processOrder(@Valid Order order, Errors errors, SessionStatus sessionStatus,
        Authentication authentication) {

        if (errors.hasErrors()) {
            return "jpaOrderForm";
        }

        log.info("Order submitted: " + order);

        order.setUser((User)authentication.getPrincipal());

        orderRepository.save(order);

        // close session
        sessionStatus.setComplete();

        return "redirect:/";
    }
```

## @AuthenticationPrincipal
```java
    @PostMapping
    public String processOrder(@Valid Order order, Errors errors, SessionStatus sessionStatus, 
                               @AuthenticationPrincipal User user) {
        if (errors.hasErrors()) {
            return "jpaOrderForm";
        }

        log.info("Order submitted: " + order);

        order.setUser(user);

        orderRepository.save(order);

        // close session
        sessionStatus.setComplete();

        return "redirect:/";
    }
```

## SecurityContextHolder
```java
    @PostMapping
    public String processOrder(@Valid Order order, Errors errors, SessionStatus sessionStatus) {
        if (errors.hasErrors()) {
            return "jpaOrderForm";
        }

        log.info("Order submitted: " + order);

        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        order.setUser((User)authentication.getPrincipal());

        orderRepository.save(order);

        // close session
        sessionStatus.setComplete();

        return "redirect:/";
    }
```