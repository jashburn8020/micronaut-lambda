# Using Micronaut to implement an AWS Lambda function

## Setup

- Create an application of the serverless function type using the Micronaut Command Line Interface
  - `mn create-function-app <package>.<project_name> --features=aws-lambda --build=gradle --lang=java`
    - `<package>`: the Java package for the Micronaut code
    - `<project_name>`: the name of the project root directory
  - e.g., the following will create the application in the `micronaut-lambda` directory with code in the `com.jashburn.micronaut.lambda` package:

```console
$ mn create-function-app com.jashburn.micronaut.lambda.micronaut-lambda --features=aws-lambda --build=gradle --lang=java
| Application created at /home/jashburn/devel/sandbox/java/micronaut-lambda

$ cat settings.gradle 
rootProject.name="micronaut-lambda"

$ ls src/main/java/com/jashburn/micronaut/lambda/
FunctionRequestHandler.java
```

- If using IntelliJ IDEA, launch it and open the project root directory as a project
  - downloads various libraries
  - enable annotation processing
- Function handler code: [`src/main/java/com/jashburn/micronaut/lambda/FunctionRequestHandler.java`](src/main/java/com/jashburn/micronaut/lambda/FunctionRequestHandler.java)
- Function test code: [`src/test/java/com/jashburn/micronaut/lambda/FunctionRequestHandlerTest.java`](src/test/java/com/jashburn/micronaut/lambda/FunctionRequestHandlerTest.java)
  - test: `./gradlew test`
  - test report in `build/reports/tests/test/index.html`

```console
$ ./gradlew test
Starting a Gradle Daemon (subsequent builds will be faster)

> Task :compileJava
Note: Creating bean classes for 1 type elements
warning: unknown enum constant When.MAYBE
  reason: class file for javax.annotation.meta.When not found
1 warning

BUILD SUCCESSFUL in 7s
4 actionable tasks: 4 executed
```
