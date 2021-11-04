**java.beans.PropertyEditor** is an interface that converts a property's value to and from its native type representation into a **String**. Because **PropertyEditor**s are inherently lightweight classes, they have found uses in many settings, including Spring.


```java
public class DateTimePropertyEditor extends PropertyEditorSupport {
	private DateTimeFormatter dateTimeFormatter;

	public DateTimePropertyEditor(DateTimeFormatter dateTimeFormatter) {
		this.dateTimeFormatter = dateTimeFormatter;
	}

	@Override
	public void setAsText(String text) throws IllegalArgumentException {
		setValue(DateTime.parse(text, dateTimeFormatter));
	}
}

public class DateTimePropertyEditorRegistrar implements PropertyEditorRegistrar {
  private DateTimeFormatter dateTimeFormatter;

  public DateTimePropertyEditorRegistrar(String dateFormatPattern) {
    dateTimeFormatter = DateTimeFormat.forPattern(dateFormatPattern);
  }

  @Override
  public void registerCustomEditors(PropertyEditorRegistry registry) {
    registry.registerCustomEditor(DateTime.class, new DateTimePropertyEditor(dateTimeFormatter));
  }
}
```

```java
public class FullNamePropertyEditor extends PropertyEditorSupport {
    @Override
    public void setAsText(String text) throws IllegalArgumentException {
        String[] name = text.split("\\s");

        setValue(new FullName(name[0], name[1]));
    }
}

public class FullNamePropertyEditorRegistrar implements PropertyEditorRegistrar {
  @Override
  public void registerCustomEditors(PropertyEditorRegistry registry) {
    registry.registerCustomEditor(FullName.class, new FullNamePropertyEditor());
  }
}
```

```xml
    <util:list id="propertyEditorRegistrarsList">
        <bean class="registrar.FullNamePropertyEditorRegistrar" />
        <bean class="registrar.DateTimePropertyEditorRegistrar">
            <constructor-arg value="${date.format.pattern}" />
        </bean>
    </util:list>

    <bean id="customEditorConfigurer" class="org.springframework.beans.factory.config.CustomEditorConfigurer">
        <property name="propertyEditorRegistrars" ref="propertyEditorRegistrarsList" />
    </bean>
```