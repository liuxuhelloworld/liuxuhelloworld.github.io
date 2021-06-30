Spring Boot applications tend to bring everything they need with them and don't need to be deployed to some application server.

# Spring Boot DevTools
## automatic application restart
DevTools会监测代码或配置文件的变动，并自动重启应用。准确地说，当你使用DevTools时，the application is loaded into two separate class loaders in the Java virtual machine (JVM). One class loader is loaded with your Java code, property files, and pretty much anything that is in the src/main/ path of the project. The other class loader is loaded with dependency libraries, which aren't likely to change as often. When a change is detected, DevTools reloads only the class loader containing your project code and restarts the Spring application context, but leaves the other class loader and the JVM intact.

## automatic browser refresh  and template cache disable
Template cache是默认的，这样templates don't need to be reparsed with every request they serve. DevTools帮你自动关闭template cache，这样方便调试前端内容

## built-in H2 console
http://localhost:8080/h2-console
jdbc:h2:mem:testdb





