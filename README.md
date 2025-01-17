# Cloud Lab

Cloud Native Lab - Simple Workshop demonstrating Cloud-Native development with Spring Boot and Pivotal Cloud Foundry.

# Items covered:

## Spring Boot

* Initializr
* Web
* Actuator
* Profiles
* Cache
* Data
* Data Migration (Flyway)
* Scheduling
* Messages (AMQB)

## PCF

* Deployment
* Scaling / Auto-Scaling
* Self-Healing
* Metrics
* Logging
* Market Place
* MySQL
* Redis
* RabbitMQ


## PCF

## Prerequisites

### Modern Java JDK Installed (at-least Java8)

### Cloud Foundry Command Line Interface (cf CLI) installed

[https://docs.cloudfoundry.org/cf-cli/install-go-cli.html](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)

### Java IDE of choice ..

Intellij recommended .. The Community addition is free.

Using an non-Java IDE, can cause issues with test formatting as well as lack of "auto imports".

### Access to a Pivotal Cloud Foundry instance

Access will be provided during the workshop, or you can sign-up for a free access at : https://run.pivotal.io

## 0 - Initialization

Key points:
* Spring Boot Initializr
* Maven
* Maven Wrapper

### 0.1 - Generate a Spring Boot Template from https://start.spring.io
Stick to the default settings, however update:
- for dependencies add **Spring Web Starter**

<img src="img/init-screen.png">

Download it, and unzip it.

> NOTE - Make sure that the **Spring Web Starter** Dependency was added -- you should see *spring-boot-starter-web* in your pom.xml

Note the mvnw file:
This is the Wrapper components for Maven: ensuring build script version consistency, removing the dependency of having these build tools installed , and simplifying CI build agent dependencies (only a JDK will be required).

### 0.2 - Import the project into your IDE

Note make sure to do an *import* and not just *open* , to allow for your IDE to correctly pull down dependencies.

## 1 - WebApplication with SpringBoot

Key points:
* Spring Boot Web
* Creating a Simple Helloworld endpoint
* Running the App locally
* No XML!

### 1.1 - Implement a HelloWorld endpoint
This can be done by creating a *HelloWorldController* Java class file with:


```java

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloWorldController {

    @RequestMapping(method = RequestMethod.GET, path="/")
    public String helloWorld(){
        return "Hello world";
    }
}
```

### 1.2 - Run the application

> NOTE: Maven instructions are for Bash/Linux/MacOS ... for Windows , replace calls to *./mvnw* with *mvnw.cmd*

```sh
./mvnw spring-boot:run
```

On Windows machines:

```sh
mvnw.cmd spring-boot:run
```

> NOTE: if you have trouble running Maven Wrapper (mvnw) it is most likely due to Proxy issues,  try regular Maven (not wrapper)

---

To build the artifact:

Run:

```sh
./mvnw package
```

On Windows machines:

```sh
mvnw.cmd package
```

To run your built application: 

```sh
java -jar ./target/demo-0.0.1-SNAPSHOT.jar
```

Note Tomcat is embedded inside of the build artifact (you don't need an external application Server).

### 1.3 - Test the root endpoint
The address will be: localhost:8080

You can test via a browser or commandline:
```sh
curl localhost:8080
```

### 1.4 - BONUS - Add Unit Tests for HelloWorld endpoint
This can be done by creating a *HelloWorldControllerTests* Java class file in the test/java directory with:

```java

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.test.context.junit4.SpringRunner;

import static org.assertj.core.api.AssertionsForClassTypes.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class HelloWorldControllerTests {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    public void testHelloWorld(){
        String body = restTemplate.getForObject("/",String.class);

        assertThat(body).contains("Hello world");
    }
}
```

With Intellij, you can now run the Test by right clicking on it.

From the commandline you can run them with:

```sh
./mvnw test
```

On Windows machines:

```sh
mvnw.cmd test
```

**Fix the broken test -- hint the strings don't match**


Note: RestTemplate and TestRestTemplate are the conventional ways for invoking / and testing HTTP/Rest calls.


## 2 - WebApplication on PCF

Key points:
* Deploying to PCF
* Scaling in PCF

### 2.1 - Login into the PCF instance that you are using (if required)

```sh
cf login -a ENTER_API_URL_HERE
```

Enter your Username and Password.

### 2.2 - Deploy your application to PCF

```sh
cf push cloud-lab -p target/demo-0.0.1-SNAPSHOT.jar
```

Note with multiple lab participants, the default *route* generated based on application name will probably NOT be available, to solve this deploy with the random-route parameter.

```sh
cf push cloud-lab -p target/demo-0.0.1-SNAPSHOT.jar --random-route
```

This will automatically create a new application in your default PCF development space, with the specific jar artifact deployed.

Note that PCF will automatically detect that this is a Java application, and use the appropriate *BuildPack*.

### 2.3 - Login into the PCF portal to view your newly deployed / created application

If you are using Pivotal Web Services, the portal is at:

https://run.pivotal.io

Click through to your app by selecting the default space and org.

Your route to the application (URL) will be presented besides your application.

### 2.4 - Test the / endpoint at this route <ROUTE>/

### 2.5 - Scale the App

Either provision more instances or more space.

This can be done via the command line or via the GUI.

To scale up to 2 instances:

```sh
cf scale cloud-lab -i 2
```

Via the GUI observe additional instances being spun up.

### 2.6 - BONUS - Add a HomePage Controller to display PCF information using VCAP variables

Add the following dependency to your `pom.xml` file.

```java
<!-- https://mvnrepository.com/artifact/io.micrometer/micrometer-core -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-core</artifactId>
    <version>1.1.5</version>
</dependency>

```

Create the HomePageController.

```java

import io.micrometer.core.instrument.Metrics;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HomePageController {

    @Value("${vcap.application.name:localMachine}")
    private String applicationName;

    @Value("${vcap.application.space_name:localSpace}")
    private String spaceName;

    @Value("${vcap.application.instance_id:localInstanceId}")
    private String instanceId;

    @RequestMapping("/")
    public String index(){

        Metrics.counter("application.indexpage.request").increment();

        return "PCF Info: " + applicationName + "@" + spaceName + " " + instanceId;
    }

}
```

Build and redeploy to PCF. Hit the homepage URL and note the PCF information.

This includes Round Robin load balancing when you have multiple instances running.

## 3 - Operations with Spring Boot

Key points:
* Spring Boot Actuator - including walk-through of available endpoints
* Spring Boot Dependency Management (no need for individual artifact versioning)
* Exposing additional Actuator information
* Build / and GIT information

The Spring Actuator Dependency adds out-of-the-box endpoints for monitoring and interacting with your application.

### 3.1 - Add the Spring Boot Actuator dependency to your build script.

The full name of the dependency is : *org.springframework.boot:spring-boot-starter-actuator*


```xml
<dependencies>
    <!-- other dependencies -->

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>

     <!-- other dependencies -->
</dependencies>
```

Your pom.xml should now contain a section that looks like: 


```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
	
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-actuator</artifactId>
	</dependency>

	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
</dependencies>
```

Re-run the application

```sh
./mvnw spring-boot:run
```

### 3.2 - Check the localhost:8080/actuator/health endpoint

Verify that it works on your local running instance of the app:

```sh
curl localhost:8080/actuator/health
```

You can also use a browser

### 3.3 - Expose additional information on the Health Endpoint

To the application.properties file in resources add:

```properties
management.endpoint.health.show-details=ALWAYS
```

Later on in the Workshop this endpoint will also show external dependency status and information (including Data, Cache, and Messaging).

### 3.4 - Check for new information on the health endpoint

This will require running rebuilding the application:

```sh
./mvnw spring-boot:run
```

```sh
curl localhost:8080/actuator/health
```

### 3.5 - Enable ALL Actuator endpoints

Currently exposed actuator endpoints can be viewed at: http://localhost:8080/actuator

For security reasons, many of the these endpoints are turned off by default.

They can be ALL enabled by adding the following to your application.properties file:

```properties
management.endpoints.web.exposure.include=*
```

Rebuild, and check the http://localhost:8080/actuator endpoint for available ones.

### 3.6 - BONUS - Add build information the /info endpoint

We want to be able to easily view build information from running instances of our app.

You will need to generate a META-INF/build-info.properties in your class path .. this can be automated :

For Maven in the pom.xml .. update the build block to the following .. note the addition of the executions block:


```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
			<executions>
				<execution>
					<goals>
						<goal>build-info</goal>
					</goals>
				</execution>
			</executions>
		</plugin>
	</plugins>
</build>
```

Rebuild, and check the http://localhost:8080/actuator/info endpoint.

### 3.7 - BONUS - Add GIT Information to the /info endpoint

For Maven users, the spring-boot-starter-parent POM includes a pre-configured plugin to generate a git.properties file. To use it, add the following declaration to your POM:

```xml
<build>
	<plugins>
		<plugin>
			<groupId>pl.project13.maven</groupId>
			<artifactId>git-commit-id-plugin</artifactId>
		</plugin>
	</plugins>
</build>
```

**Note this requires that you have run a "git init" command on your working directory for this project**

### 3.8 - BONUS - Add Custom Application Metrics counter to your Hello Controller

```java
@RequestMapping("hello")
public String helloWorld(){

    Metrics.counter("application.helloworld.hit").increment();

    return "Hello world!!";
}
```

Restart your app, hit the endpoint .. and access the /actuator/metrics endpoint.

Note the addition of application.helloworld.hit


## 4 - Operations on PCF

Key points:
* Dealing with Crashes
* Metric and Logging
* Auto-Scaling
* Zero down-time deployments

### 4.1 - Add an endpoint to your App to simulate JVM crashes

Add a new KillController with a Kill endpoint/

```java
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class KillController {

  @RequestMapping("/kill")
  public void kill(){
      System.exit(1);
  }
}
```

Rebuild your app, and redeploy to PCF.

```sh
 ./mvnw package && cf push cloud-lab -p target/cloud-lab-0.0.1-SNAPSHOT.jar
```

### 4.2 - In separate terminal window, TAIL the PCF app logs

```sh
cf logs cloud-lab
```

### 4.3 - Call the /kill endpoint

Note that PCF will automatically bring up a new instance.

This can be monitored from the PCF Dev Portal.

You can also view what happened in the logging window from the previous step.

For additional information, you can also drill down into the *PCF Metrics" option in the Application page in PCF. This includes more in-depth logging, and crash analysis.

### 4.4 - BONUS - Create a PCF manifest to simplify deployments

https://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html

```sh
cf create-app-manifest cloud-lab
```

You can customize deployment settings, as well as default binary path.

### 4.5 - BONUS - Add Auto-Scaling to Your Application

https://docs.run.pivotal.io/appsman-services/autoscaler/using-autoscaler.html

### 4.6 - BONUS - Perform a Blue-Green Deployment to Reduce Downtime and Risk

Blue-green deployment is a technique that reduces downtime and risk by running two identical production environments called Blue and Green.

At any time, only one of the environments is live, with the live environment serving all production traffic. For this example, Blue is currently live and Green is idle.

#### 4.6.1 - BONUS - Deploy a new instance of our cloud-lab

```sh
cf push cloud-lab-2 -p build/libs/demo-0.0.1-SNAPSHOT.jar
```

Right now, we have 2 deploys apps running (they can be different version of the application).

#### 4.6.2 - Route all cloud-lab subdomain traffic to cloud-lab2 (in addition to our original cloud-lab instance).

For reference, current Routes can be viewed with:

```sh
cf routes
```

Note the domain used for your cloud-lab apps.

Route cloud-lab subdomain traffic to cloud-lab:

```sh
 cf map-route cloud-lab-2 ENTER_PCF_DOMAIN --hostname cloud-lab
```

**Replace ENTER_PCF_DOMAIN with domain from *cf routes* step.**

##### 4.6.3 - Unmap cloud-lab traffic to the cloud-lab subdomain

```
cf unmap-route cloud-lab ENTER_PCF_DOMAIN --hostname cloud-lab
```

Note, all cloud-lab subdomain traffic will now be mapped to our recent deploy.

**Replace ENTER_PCF_DOMAIN with domain from *cf routes* step.**

Cloud Foundry community members have written plugins to further automate the blue-green deployment process. These include:

Autopilot: Autopilot is a Cloud Foundry Go plugin that provides a subcommand, zero-downtime-push, for hands-off, zero-downtime application deploys.
BlueGreenDeploy: cf-blue-green-deploy is a plugin, written in Go, for the Cloud Foundry Command Line Interface (cf CLI) that automates a few steps involved in zero-downtime deploys.


### 4.7 - BONUS - Enable Custom Application Metrics Forwarding to PCF Metrics

#### 4.7.a (PCF 2.4 or greater )

##### 4.7.a.1 Add the Prometheus Micrometer dependency to your build script

Add the io.micrometer:micrometer-registry-prometheus dependency to your pom.xml:

Maven:
```xml
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-registry-prometheus</artifactId>
    </dependency>
```

##### 4.7.a.2 Ensure Metric Registrar CLI plugin is installed

```sh
    cf install-plugin -r CF-Community "metric-registrar"
```

##### 4.7.a.3 Build and Deploy your Application 

##### 4.7.a.4 Register your Metrics endpoint with PCF

```sh
    cf register-metrics-endpoint cloud-lab /actuator/prometheus
``` 

#### 4.7.b (PCF 2.3 or earlier )

Create and Bind the Forwarder Service -- 

This is required for Custom Application Metrics.

##### 4.7.b.1 Ensure *Metric Forwarder* service is available in the CF MarketPlace

```sh
cf marketplace
```

Contact your PCF Cloud Ops team if it is not.

##### 4.7.b.2 Create the Service

You can use a *plan* and *name* of your choice.

```sh
cf create-service metrics-forwarder unlimited myforwarder
```

##### 4.7.b.3 Bind the Service to your Application

```sh
cf bind-service cloud-lab myforwarder
```

##### 4.7.b.4 Restage your Application

```sh
cf restage metrics-demo
```


## 5 - Configuration with Spring Boot

Key points:
* Spring Boot Profiles
* Configuration precedence

Spring Boot lets you externalize your configuration so that you can work with the same application code in different environments.

https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html

### 5.1 - Allow injection of the helloWorld message via external sources
```java
@RestController
public class HelloWorldController {

    @Value("${helloworld.message:'Hola Mundo - default!'}")
    private String helloMessage;

    @RequestMapping("hello")
    public String helloWorld(){
        return helloMessage;
    }
}
```

Note #1 : the usage of the default value in case nothing is found (optional).

Note #2 : You may have to fix broken tests!!

### 5.2 - Add a configurable value for helloworld.message in the application.properties file.

```properties
helloworld.message="Hello World - default config file"
```

### 5.3 - Rebuild / Restart your app, and verify the /hello endpoint.

Default should be "Hello World - default config file"

## 6 - Configuration on PCF

Key points:
* The cloud profile
* Spring Cloud Config Server overview

### 6.1 - Add a custom HelloWorld message for cloud deployments

PCF deploys will automatically load *cloud* profile settings.

In the resources folder, add a *application-cloud.properties* file.

Add:

```properties
helloworld.message="Hello World - cloud only"
```

Build and Redeploy to PCF.

Verify the updated message at the /hello endpoint.

Note: You can also use the Spring Config Server (available as a Service in the MarketPlace) to inject properties from an external source such as a GitHub repo.

## 7 - Caching with Spring Boot

Key points:
* Spring Boot Cache
* The Cacheable annotation

### 7.1 - Add a slow , costly endpoint to the Application.

One example would be performing an uppercase operation on a String with a time delay.

```java
@RestController
public class CacheExampleController {

  @RequestMapping("/uppercase")
  public String uppercase(String input ){
      try {Thread.sleep(5000); } catch (InterruptedException e) {}

      return input.toUpperCase();
  }
}
```

Rebuild, rerun the app.

From the browser you can call : *http://localhost:8080/uppercase?input=test*

Or using curl:

```sh
curl http://localhost:8080/uppercase?input=test
```

Note how the /uppercase endpoint is always slow.

### 7.2 - Add the Spring Boot Cache dependency to your build script (pom.xml)

The full name of the dependency is : *org.springframework.boot:spring-boot-starter-cache*

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

### 7.3 - Enable Caching on the endpoint by using the *Cacheable* Annotation.

Updated CacheExampleController should look like:

```java
@Cacheable("uppercase")
@RequestMapping("/uppercase")
public String uppercase(String input ){
    try {Thread.sleep(5000); } catch (InterruptedException e) {}

    return input.toUpperCase();
}
```

You will also need to turn on Caching (app wide) by adding the EnableCaching annotation to your app.

This can be done in the CloudLabApplication class:

```java
@SpringBootApplication
@EnableCaching
public class CloudLabApplication {

	public static void main(String[] args) {
		SpringApplication.run(CloudLabApplication.class, args);
	}
}
```

Restart your app, and verify that subsequent calls to the endpoint return much quicker.

Note, the default cache implementation uses local in-memory caching, but this can be easily change to use 3rd party caching solutions such as Redis.

### 7.4 - BONUS - Add an eviction endpoint to the controller

Given a String, evict it from the Cache.

Hint: CacheEvict annotation

## 8 - Caching on PCF

Key points:
* Spring Boot Redis
* CF MarketPlace / Service Broker
* Binding Services to App Instances
* Creating a Redis Instance in PCF


### 8.1 - Add the Spring Boot Redis and commons-pool dependencies to your build script (pom.xml).

The dependencies are:
*org.springframework.boot:spring-boot-starter-data-redis*
*org.apache.commons:commons-pool2*

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
	<groupId>org.apache.commons</groupId>
	<artifactId>commons-pool2</artifactId>
</dependency>
```

Rebuild, and redeploy to PCF.

### 8.2 - Create a Redis Service Instance in PCF

You can view available self-self / on-demand provisioning services via the marketplace.

```sh
cf marketplace
```

To create a Redis Service run:

```sh
cf create-service p-redis shared-vm custom-redis
```

### 8.3 - Bind the Service to our application

```sh
cf bind-service cloud-lab custom-redis
```

Restage your app:
```sh
cf restage cloud-lab
```

Confirm connection to your Redis Server using the health endpoint: /actuator/health

Also confirm that cache is still working.

**TIP -- To get more readable "pretty" JSON output in browser, install a JSON Extension **

### 8.4 - BONUS - Default to Simple (non-Redis caching locally) and Redis Caching in the cloud

In application.properties add:

```properties
spring.cache.type=SIMPLE
```

In application-cloud.properties add:

```properties
spring.cache.type=REDIS
```

Verify that Caching works locally and in PCF.

## 9 - Data with Spring Boot

Key points:
* Spring Boot Data / Rest
* Auto-generated Rest Compliant endpoints for Entity Domain Objects
* Default Data using import.sql
* Project Lombok

### 9.1 - Add the Spring Boot Data JPA , Rest and H2Database dependencies to your build script.

The full name of the dependencies are :
*org.springframework.boot:spring-boot-starter-data-jpa*
*org.springframework.boot:spring-boot-starter-data-rest*
*com.h2database:h2*


### 9.2 - Add a Simple Domain Object

One example would be a Person object with First and Last names variables.

```java
@Entity
public class Person {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    private String firstName;
    private String lastName;

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }
}

```

### 9.3 - Add a Repository Interface

Also create a PersonRepository interface:

```java
public interface PersonRepository extends JpaRepository<Person, Long> {}

```

### 9.4  - Add Default Sample Data

Create a import.sql file in the resources/ directory.

```sql
INSERT INTO PERSON(id, first_name, last_name) VALUES (1, 'Tony', 'Stark');
INSERT INTO PERSON(id, first_name, last_name) VALUES (2, 'Steve', 'Rogers');

```



### 9.5 - Test the new persons endpoints

Restart your app, and view the localhost:8080/persons.

The default is configured to use the embedded H2 Database.

Note how the Interface / and Entity Object were sufficient for Spring Boot to generate Rest compliant endpoints with fully implemented database calls.

Note, the default data implementation uses a local in-memory storage (H2), but this can be easily changed to something else like MySQL.

### 9.6 - BONUS - Enable logging of all DML/DDL SQL statements

Add the following to the *applications.properties* file.

```properties
spring.jpa.show-sql=true
```

Restart your app, you should now see all SQL statement in the log output.

This can be useful in identifying what actual SQL calls Spring Data is making.

### 9.7 - BONUS - Simplify the Person Entity class by auto-generating Getters/Setters.

Hint - Add the Lombok dependency , and use the Data annotation.

Project Lombok is a java library that automatically plugs into your editor and build tools to can reduce Java Boiler Plate code in a number if ways including: Getter/Setter generation, Builder Pattern implementation, Checked Exception handling, and Resource Management.

Full list of features are available at:

https://projectlombok.org/features/all

## 10 - Data on PCF

Key points:
* MySQL Connector
* Creating a MySQL Instance in PCF

### 10.1 - Add the MySQL dependency to the build script.

The full name of the dependency is :
*mysql:mysql-connector-java*


### 10.2 - Create a MySql Service Instance in PCF

You can view available self-self / on-demand provisioning services via the marketplace.

```sh
cf marketplace
```

To create a MySQL Service run:

```sh
cf create-service p-mysql 100mb custom-mysql
```

### 10.3 - Bind the Service to our application

```sh
cf bind-service cloud-lab custom-mysql
```

Restage your app:
```sh
cf restage cloud-lab
```

Confirm connection to your MySQL Server using the health endpoint: /actuator/health

Try to hit the /persons endpoint.

It won't work as the database used does not have the required Persons tables created.

Where as with local H2 database usage (from the previous) step, Spring Boot auto-generates (from scratch) the necessary tables schemas each time the app is run. With a long-lived / externalized / and shared database (such as MySQL) , we need a better strategy for keeping database schemas up-to-date.




## 11 - Database Migrations

Key points:
* High-level Database Migration Versioning Tools

Spring Boot supports two higher-level migration tools: Flyway and Liquibase.

We will use Flyway for performing MySQL migrations.

### 11.1 - Add the FlyAwayDB dependency to your build script:

The full name of the dependency is :
*org.flywaydb:flyway-core*

### 11.2 - Add a Base Database Init Script:

In the resources folder , create a db/migration sub-folder.

In it create a V1__init.sql file (resources/db/migration/V1__init.sql).

Add the following to it:

```sql
CREATE TABLE PERSON (
	id int NOT NULL AUTO_INCREMENT,
	first_name varchar(255) not null,
	last_name varchar(255) not null,
	PRIMARY KEY (ID)
);

INSERT INTO PERSON (first_name, last_name) VALUES ('Peter', 'Parker');
```

### 11.3 - Rebuild and redeploy to PCF

You can look at http://localhost:8080/actuator/flyway to review the list of scripts.

FlyAway will only apply updates as needed, and keeps track of scripts run (in the flyway_schema_history table).

### 11.4 - BONUS - Add a middleName value to the Person Object , and create Database Migration scripts for this

```java
    private String middleName;

    public String getMiddleName() {
        return middleName;
    }

    public void setMiddleName(String middleName) {
        this.middleName = middleName;
    }
```

Create a V2__person_middle_name_addition.sql file (in db/migration).

Add the following to it:

```sql
ALTER TABLE person ADD middle_name varchar(255);
```

Build and redeploy to PCF.

Flyway will automatically upgrade the Database Schema version to v2.

### 11.5 - BONUS - Add caching to a database lookup call

Hint: Add a method definition with a Caching annotation to the PersonRepository interface.

### 11.6 - BONUS - Deploy the PivotalMySQLWeb App into PCF to view the flyway_schema_history schema / and row values.

App can be downloaded from:

https://github.com/pivotal-cf/PivotalMySQLWeb

It will also need to be bound to your MySQL instance.

## 12 - Scheduling with Spring Boot

### 12.1 - Enable Scheduling in your Spring Boot Application

You will also need to turn on Scheduling (app wide) by adding the EnableScheduling annotation to your app.

This can be done in the CloudLabApplication file:

```java
@SpringBootApplication
@EnableCaching
@EnableScheduling
public class CloudLabApplication {

    public static void main(String[] args) {
        SpringApplication.run(CloudLabApplication.class, args);
    }
}
```

EnableScheduling ensures that a background task executor is created. Without it, nothing gets scheduled.

### 12.2 - Create a Simple Task to be executed every 5 seconds

Our example will display the time:

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

Restart your app.


Note the additional time messages in the output.



### 12.3 - View currently scheduled tasks by hitting the scheduledtasks endpoint

http://localhost:8080/actuator/scheduledtasks

### 12.4 - BONUS - Add additional Scheduled tasks with different scheduling options

Other options for scheduling tasks and can be seen at :
https://docs.spring.io/spring/docs/current/spring-framework-reference/integration.html#scheduling

## 13 - Messaging with Spring Boot

Key points:
* Spring Boot AMQP
* Sending and Receiving Messages

### 13.1 - Add the Spring Boot AMQP dependency to your build script.

The full name of the dependency is : *org.springframework.boot:spring-boot-starter-amqp*


### 13.2 - Create a Controller for Sending and Receiving Queue Messages

In a new QueueController java file , add:

```java
@RestController
public class QueueController {

    @Autowired
    private AmqpTemplate amqpTemplate;

    @RequestMapping("/sendmessage")
    public void sendMessage(String input){
        amqpTemplate.convertAndSend("myQueue",input);
    }

    @RequestMapping("/getmessage")
    public String getMessage(){
        Message message = amqpTemplate.receive("myQueue");
        if(message == null)
            return "Queue empty";

        return message.toString();
    }
}

```

This will send messages to the default the "Default" exchange, with the "myQueue" routing key.

*For purpose of workship, unlike Caching / Data , we will go directly to testing in PCF*

If do have a local instance of RabbitMQ, you can point to it by updating your application properties with the correct connection settings:

application.properties
```properties
spring.rabbitmq.host = 127.0.0.1
spring.rabbitmq.port = 5672
spring.rabbitmq.username = guest
spring.rabbitmq.password = guest
```

Your RabbitMQ instance will need to have a queue create called **myQueue**. Messages sent to the Default exchange with the **myQueue** routingkey will be automatically routed to the **myQueue** queue.

## 14 - Messaging on PCF

Key points:
* Creating a RabbitMQ Instance in PCF

### 14.1 - Create a RabbitMQ Service Instance in PCF

You can view available self-self / on-demand provisioning services via the marketplace.

```sh
cf marketplace
```

To create a RabbitMQ Service run:

```sh
cf create-service p-rabbitmq standard custom-rabbitmq
```

### 14.2 - Bind the Service to your application

```sh
cf bind-service cloud-lab custom-rabbitmq
```

### 14.3 - Add the *myQueue* queue to your RabbitMQ instance

In the PCF Dev Portal:

In your DevSpace, select your RabbitMQ instance, and click Manage.

This will pull up the RabbitMQ control GUI.

<img src="img/queue-screen.png">

Under the Queues tab, select add a new queue.

Call it "myQueue".

The Default Exchange will route to specific queues based on specified routing key.

Confirm connection to your RabbitMQ Server using the health endpoint: /actuator/health

Rebuild and deploy your app to PCF , with the changes from the previous step.


### 14.4 - Test messaging using the /getmessage and /sendmessage endpoints.

To send message, curl or point your browser to /sendmessage?input=YOUR_MESSAGE_STRING

To receive messages, curl or point your browser to /getmessage


## 15 - BONUS - Advanced Observability

For reference see the Metrics Demo repository at https://github.com/vicsz/spring-boot-metrics-demo.

### 15.1 - Add Alerting on Error Level Log Messages (i.e. Slack)

This will involve creating your own custom Slack Webhook, and updating your Logback.xml with a Slack Appender.

Tip - to avoiding disclosing your Slack Webhook URL, instead of adding it to source control, inject is as an environment variable in PCF.
