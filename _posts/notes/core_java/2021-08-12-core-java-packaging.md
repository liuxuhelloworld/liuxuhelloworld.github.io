# WAR vs JAR
Given the options of deploying a WAR file or a JAR file, how do you choose? In generally, the choice comes down to whether you plan to deploy your application to a traditional Java application server or to a cloud platform:
- if you must deploy your application to Tomcat, WebSphere, WebLogic, or any other traditional Java application servers, you really have no choice but to build your application as a WAR file
- if you are planning to deploy your application to the cloud, then an executable JAR file is the best choice