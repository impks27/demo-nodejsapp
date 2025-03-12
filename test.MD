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