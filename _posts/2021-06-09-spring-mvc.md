# What is Spring MVC?
Spring MVC是一个强大的web framework.

Spring MVC can also be used to create REST APIs.

## controller
A controller's primary job is to handle HTTP requests and either hand a request off to a view to render HTML (browser-displayed) or write data directly to the body of a response (RESTful).

## viewer
A viewer's job is to render the data into HTML that will be displayed in the browser.

Spring支持多种定义view的方式:
- JSP (JavaServer Pages): the servlet container itself (Tomcat by default) implements the JSP specification. Java servlet containers usually look for JSPs somewhere under /WEB-INF. JSP is only an option if you're building your application as a WAR file and deploying it in a traditional servlet container
- Thymeleaf: spring-boot-starter-thymeleaf
- FreeMarker: spring-boot-starter-freemarker
- Mustache: spring-boot-starter-mustache
- Groovy-based templates: spring-boot-starter-groovy-templates

Spring Boot will detect your chosen template library and automatically configure the components required for it to serve views for your Spring MVC controllers.

## model
A model's job is to describe the the application's domain.

# org.springframework.ui.Model
**Model** is an object that ferries data between a controller and whatever view is charged with rendering that data. Ultimately, data that's placed in **Model** attributes is copied into the serlvet response attributes, where the view can find them.

# Validation
在各个需要校验的地方写一堆if-else？it is cumbersome and difficult to read and debug.

Spring支持Java's Bean Validation API (也称为JSR-303)，所以我们只需要declare validation rules.

使用Spring Boot，the Validation API and the Hibernate implementation of the Validation API are automatically added to the project as transient dependencies of Spring Boot's web starter. The Validation API offers annotations that can be placed on properties of domain objects to declare validation rules. Hibernate's implementation of the Validation API adds even more validation annotations.

简单来说，你需要:
- 在domain classes中declare validation rules，@NotNull, @NotBlank, @Size, ...
- 在controller classes中明确需要执行validation，@Valid
- 修改form views展示validation errors

# WebMvcConfigurer
**WebMvcConfigurer** defines several methods for configuring Spring MVC.

# Spring MVC test
```java
@RunWith(SpringRunner.class)
@WebMvcTest(HomeController.class)
public class HomeControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void testHomePage() throws Exception {
        mockMvc.perform(get("/"))
            .andExpect(status().isOk())
            .andExpect(view().name("home"))
            .andExpect(content().string(containsString("Welcome to ...")));
    }
}
```
