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

## Deploy via AWS Console

### Create a function in AWS

- Author from scratch
- Basic information:
  - function name: `micronaut-lambda-simple`
  - runtime: Java 11 (Corretto)
  - architecture: x86_64
  - permissions:
    - execution role: Create a new role with basic Lambda permissions
    - creates an execution role: `micronaut-lambda-simple-role-<random>`
      - e.g., `micronaut-lambda-simple-role-b2fxbj9z`
      - contains a policy: `AWSLambdaBasicExecutionRole-<random>`
        - e.g., `AWSLambdaBasicExecutionRole-e19b426c-4e36-4440-a8fc-bfa263b41e09`
        - allows upload logs to CloudWatch Logs
        - CloudWatch policy statements can be replaced by attaching the AWS-managed `AWSLambdaBasicExecutionRole` policy

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "logs:CreateLogGroup",
            "Resource": "arn:aws:logs:eu-west-2:612248661359:*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": [
                "arn:aws:logs:eu-west-2:612248661359:log-group:/aws/lambda/micronaut-lambda-simple:*"
            ]
        }
    ]
}
```

### Upload code

- Create an executable jar including all dependencies
  - `gradlew shadowJar`
  - fat/uber jar created in `build/libs/`

```console
$ ./gradlew shadowJar
Starting a Gradle Daemon, 1 incompatible and 1 stopped Daemons could not be reused, use --status for details

BUILD SUCCESSFUL in 7s
3 actionable tasks: 1 executed, 2 up-to-date

$ ls build/libs/
micronaut-lambda-0.1-all.jar
```

- Upload the jar file to the function
  - code source
    - upload from: .zip or .jar file
- Set the handler for the function
  - runtime settings
    - handler: `com.jashburn.micronaut.lambda.FunctionRequestHandler`

### Test the function

- Test event
  - not necessary to provide an event name and/or use a template if you don't intend to save it
  - Event JSON:

```json
{
  "path": "/",
  "httpMethod": "GET",
  "headers": {
    "Accept": "application/json"
  }
}
```

- Output:

```json
{
  "statusCode": 200,
  "body": "{\"message\":\"Hello World\"}"
}
```
- Logs in CloudWatch log group: `/aws/lambda/micronaut-lambda-simple`

### Clean up

- Delete function: `micronaut-lambda-simple`
- Delete role: `micronaut-lambda-simple-role-<random>`
- Delete policy: `AWSLambdaBasicExecutionRole-<random>`
- Delete log group `/aws/lambda/micronaut-lambda-simple`

## Sources

- "Deploy a Serverless Micronaut Function to AWS Lambda Java 11 Runtime." _Micronaut.io_, 2022, [guides.micronaut.io/latest/mn-serverless-function-aws-lambda-gradle-java.html](https://guides.micronaut.io/latest/mn-serverless-function-aws-lambda-gradle-java.html). Accessed 20 November 2022.
