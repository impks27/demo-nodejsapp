To set the context path for an app.jar deployed on Tomcat without renaming the jar, you can use one of the following methods:

Method 1: Using the `context.xml` file
1. Create a `context.xml` file in the `META-INF` directory of your jar file with the following content:

```
<Context path="/myapp" />
```

2. Redeploy the jar file.

Method 2: Using the Tomcat `context.xml` file
1. Locate the Tomcat `context.xml` file (usually in `conf/Catalina/localhost/`).
2. Add the following XML element:

```
<Context path="/myapp" docBase="app.jar" />
```

3. Restart Tomcat.

Method 3: Using the Tomcat Manager Web Application
1. Access the Tomcat Manager Web Application (e.g., `http://localhost:8080/manager/`).
2. Click on "Deploy" and select the "WAR file to deploy" option.
3. Enter the path to your jar file and set the "Context Path" to `/myapp`.
4. Click "Deploy".

Regardless of the method you choose, your application will be accessible at `http://localhost:8080/myapp`.

-----------
Gradle:

To set the context path for a Gradle project deployed on Tomcat, you can use the following methods:

Method 1: Using the `build.gradle` file
Add the following configuration to your `build.gradle` file:

```
war {
    contextPath = '/myapp'
}
```

Method 2: Using the `gradle.properties` file
Add the following property to your `gradle.properties` file:

```
properties
contextPath=/myapp
```

Then, in your `build.gradle` file, add:

```
war {
    contextPath = project.properties['contextPath']
}
```

Method 3: Using the `gradle.tomcat` plugin
Apply the `gradle.tomcat` plugin in your `build.gradle` file:

```
plugins {
    id 'com.bmuschko.tomcat' version '2.5'
}
```

Then, configure the Tomcat plugin:

```
tomcat {
    httpPort = 8080
    contextPath = '/myapp'
}
```

Regardless of the method you choose, your application will be accessible at `http://localhost:8080/myapp`.

---------------
Maven:

To set the context path for a Maven project deployed on Tomcat, you can use the following methods:

Method 1: Using the `pom.xml` file
Add the following configuration to your `pom.xml` file:

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-war-plugin</artifactId>
            <version>3.3.2</version>
            <configuration>
                <warName>myapp</warName>
                <contextPath>/myapp</contextPath>
            </configuration>
        </plugin>
    </plugins>
</build>
```

Method 2: Using the `maven-tomcat-plugin`
Add the following configuration to your `pom.xml` file:

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.tomcat.maven</groupId>
            <artifactId>tomcat7-maven-plugin</artifactId>
            <version>2.2</version>
            <configuration>
                <path>/myapp</path>
            </configuration>
        </plugin>
    </plugins>
</build>
```

Regardless of the method you choose, your application will be accessible at `http://localhost:8080/myapp`.