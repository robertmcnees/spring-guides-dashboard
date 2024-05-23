This guide walks you through the steps for scheduling tasks with Spring.

## What You Will Build

You will build an application that prints out the current time every five seconds by using
Spring Framework’s [`@Scheduled`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/annotation/Scheduled.html) annotation.

The source code for this guide can be found in [this repository](https://github.com/spring-guides/gs-scheduling-tasks).
If you would like a local copy of the code you can download it there.

## Enable Scheduling

Although scheduled tasks can be embedded in web applications, the simpler approach (shown in this guide) creates a standalone application. To do so, package everything in a single, executable JAR file, driven by a Java main() method. The following snippet (from `src/main/java/com/example/schedulingtasks/SchedulingTasksApplication.java`) shows the application class:

```java
@SpringBootApplication
@EnableScheduling
public class SchedulingTasksApplication {
```

Spring Initializr adds the `@SpringBootApplication` annotation to our main class. `@SpringBootApplication` is a convenience annotation that adds all of the following:

* `@Configuration`: Tags the class as a source of bean definitions for the application
context.
* `@EnableAutoConfiguration`: Spring Boot attempts to automatically configure your Spring application based on the dependencies that you have added.
* `@ComponentScan`: Tells Spring to look for other components, configurations, and
services. If specific packages are not defined, recursive scanning begins with the package of the class that declares the annotation.

Additionally, add the `@EnableScheduling` annotation. This annotation enables Spring’s scheduled task execution capability.

## Create a Scheduled Task

Create a new class `src/main/java/com/example/schedulingtasks/ScheduledTasks.java` called:

```java
@Component
public class ScheduledTasks {

	private static final Logger log = LoggerFactory.getLogger(ScheduledTasks.class);

	private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");

	@Scheduled(fixedRate = 5000)
	public void reportCurrentTime() {
		log.info("The time is now {}", dateFormat.format(new Date()));
	}
}
```

The [`Scheduled` annotation](https://docs.spring.io/spring-framework/reference/integration/scheduling.html#scheduling-annotation-support-scheduled) defines when a particular method runs.

**📌 NOTE**\
This example uses [`fixedRate()`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/annotation/Scheduled.html#fixedRate()), which specifies the interval between method
invocations, measured from the start time of each invocation. Other options are [`cron()`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/annotation/Scheduled.html#cron()) and [`fixedDelay()`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/annotation/Scheduled.html#fixedDelay()). For periodic tasks, exactly one of these three options must be specified, and optionally, [`initialDelay()`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/annotation/Scheduled.html#initialDelay()). For a one-time task, it is sufficient to just specify an initialDelay()

## Running the Application

You should now be able to run the application by executing the main method in `SchedulingTasksApplication`. You can run the program from your IDE, or by executing the following Gradle command in the project root directory:

```
./gradlew bootRun
```

Doing so starts the application, and the method annotated with @Scheduled runs. You should see log messages similar to:

```
20yy-mm-ddT07:23:01.665-04:00  INFO 19633 --- [   scheduling-1] c.e.schedulingtasks.ScheduledTasks       : The time is now 07:23:01
20yy-mm-ddT07:23:06.663-04:00  INFO 19633 --- [   scheduling-1] c.e.schedulingtasks.ScheduledTasks       : The time is now 07:23:06
20yy-mm-ddT07:23:11.663-04:00  INFO 19633 --- [   scheduling-1] c.e.schedulingtasks.ScheduledTasks       : The time is now 07:23:11
```

**📌 NOTE**\
This example uses [`fixedRate()`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/annotation/Scheduled.html#fixedRate()) scheduling, so the application runs indefinitely until you interrupt it manually.

## Testing with the awaitility Dependency

To properly test your application, you can use the [`awaitility` library](https://github.com/awaitility/awaitility). Since Spring Boot 3.2, this is a dependency that Boot manages. You can create a new test or view the existing test at `src/test/java/com/example/schedulingtasks/ScheduledTasksTest.java`:

```java
@SpringBootTest
public class ScheduledTasksTest {

	@SpyBean
	ScheduledTasks tasks;

	@Test
	public void reportCurrentTime() {
		await().atMost(Durations.TEN_SECONDS).untilAsserted(() -> {
			verify(tasks, atLeast(2)).reportCurrentTime();
		});
	}
}
```

This test automatically runs when you run the `./gradlew clean build` task.

## Building the Application

In this section, we will describe four different ways to run this guide:

1. [Building and executing a JAR file](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems)
2. [Building and executing a Docker container, using Cloud Native Buildpacks](https://docs.spring.io/spring-boot/docs/current/reference/html/container-images.html#container-images.buildpacks)
3. [Building and executing a native image](https://docs.spring.io/spring-boot/docs/current/reference/html/native-image.html#native-image.developing-your-first-application.native-build-tools)
4. [Building and executing a native image container, using Cloud Native Buildpacks](https://docs.spring.io/spring-boot/docs/current/reference/html/native-image.html#native-image.developing-your-first-application.buildpacks)

Regardless of how you choose to execute the application, the output should be the same.

To run the application, you can package the application as an executable jar.
The command `./gradlew clean build` will compile the application to an executable jar.
You can then run the jar with the command `java -jar build/libs/gs-scheduling-tasks-0.0.1-SNAPSHOT.jar`

Alternatively, if you have a Docker environment available, you could create a Docker image directly from your Maven or Gradle plugin, using buildpacks.
With [Cloud Native Buildpacks](https://docs.spring.io/spring-boot/docs/current/reference/html/container-images.html#container-images.buildpacks), you can create Docker compatible images that you can run anywhere.
Spring Boot includes buildpack support directly for both Maven and Gradle.
This means you can type a single command and quickly get a sensible image into a locally running Docker daemon.
To create a Docker image using Cloud Native Buildpacks, run the command `./gradlew bootBuildImage`.
With a Docker environment enabled, you can execute the application with the command `docker run docker.io/library/gs-scheduling-tasks:0.0.1-SNAPSHOT`

### Native Image Support

Spring Boot also supports [compilation to a native image](https://docs.spring.io/spring-boot/docs/current/reference/html/native-image.html#native-image.introducing-graalvm-native-images), provided you have a GraalVM distribution on your machine.
To create a [native image with Gradle](https://docs.spring.io/spring-boot/docs/current/reference/html/native-image.html#native-image.developing-your-first-application.native-build-tools.gradle) using Native Build Tools, first make sure that your Gradle build contains a `plugins` block that includes `org.graalvm.buildtools.native`.
```
plugins {
	id 'org.graalvm.buildtools.native' version '0.9.28'
...
```

You can run the command `./gradlew nativeCompile` to generate a native image. When the build completes, you will be able to run the code with a near instantaneous start up time by executing the command `build/native/nativeCompile/gs-scheduling-tasks`.

You can also create a [Native Image using Buildpacks](https://docs.spring.io/spring-boot/docs/current/reference/html/native-image.html#native-image.developing-your-first-application.buildpacks). You can generate a native image by running the command `./gradlew bootBuildImage`. Once the build completes, you can start your application with the command `docker run docker.io/library/gs-scheduling-tasks:0.0.1-SNAPSHOT`

## Summary

Congratulations! You created an application with a scheduled task.

## See Also

The following guides may also be helpful:

* [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/)
* [Creating a Batch Service](https://spring.io/guides/gs/batch-processing/)

Want to write a new guide or contribute to an existing one? Check out our [contribution guidelines](https://github.com/spring-guides/getting-started-guides/wiki).

**❗ IMPORTANT**\
All guides are released with an ASLv2 license for the code, and an [Attribution, NoDerivatives creative commons license](http://creativecommons.org/licenses/by-nd/3.0/) for the writing.
