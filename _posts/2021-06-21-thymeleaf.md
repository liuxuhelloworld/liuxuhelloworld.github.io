View libraries such as Thymeleaf are designed to be decoupled from any particular web framework. They are unware of Spring's model abstraction and are unable to work with the data that the controller places in **Model**. But they can work with servlet request attributes. Therefore, before Spring hands the request over to a view, it copies the model data into request attributes that Thymeleaf and other view templating options have ready access to.

Thymeleaf templates are just HTML with some additional element attributes that guide a template in rendering request data

```html
<p th:text="${message}">placeholder message</p>
```

```html
<div th:each="ingredient : ${wrap}">
	<input name="ingredients" type="checkbox" th:value="${ingredient.id}" />
	<span th:text="${ingredient.name}">INGREDIENT</span><br/>
</div>
```