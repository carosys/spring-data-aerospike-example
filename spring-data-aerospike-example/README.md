# Spring Data Aerospike Example Application

## Preparation

### Install Aerospike
Follow the instructions at [Aerospike](http://www.aerospike.com/docs/operations/install/)

## Quick Start

### Maven configuration

Add the Maven dependency:

```xml
<dependency>
<groupId>org.springframework.data</groupId>
	<artifactId>spring-data-aerospike</artifactId>
	<version>1.0.0.BUILD-SNAPSHOT</version>
</dependency>
```

If you'd rather like the latest snapshots of the upcoming major version, use our Maven snapshot repository and declare the appropriate dependency version.

```xml
	<dependencies>

		<dependency>
			<groupId>org.springframework.data</groupId>
			<artifactId>spring-data-commons</artifactId>
			<version>${springdata.commons}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.data</groupId>
			<artifactId>spring-data-aerospike</artifactId>
			<version>1.0.0.BUILD-SNAPSHOT</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.data</groupId>
			<artifactId>spring-data-keyvalue</artifactId>
			<version>${springdata.keyvalue}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<version>4.1.7.RELEASE</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-tx</artifactId>
			<version>4.1.7.RELEASE</version>
		</dependency>

		<dependency>
			<groupId>com.aerospike</groupId>
			<artifactId>aerospike-client</artifactId>
			<version>${aerospike}</version>
		</dependency>

		<!-- Logging -->
		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>1.2.17</version>
		</dependency>
		<dependency>
			<groupId>joda-time</groupId>
			<artifactId>joda-time</artifactId>
			<version>2.7</version>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<pluginManagement>
			<plugins>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-assembly-plugin</artifactId>
				</plugin>
				<plugin>
					<groupId>org.codehaus.mojo</groupId>
					<artifactId>wagon-maven-plugin</artifactId>
				</plugin>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-compiler-plugin</artifactId>
					<version>3.1</version>
					<configuration>
						<source>1.6</source>
						<target>1.6</target>
					</configuration>
				</plugin>

			</plugins>

		</pluginManagement>
		<resources>
			<resource>
				<directory>${project.basedir}/src/main/lua</directory>
				<includes>
					<include>**/*.lua</include>
				</includes>
			</resource>
		</resources>
	</build>

	<repositories>
		<repository>
			<id>spring-libs-snapshot</id>
			<url>https://repo.spring.io/libs-snapshot</url>
		</repository>
	</repositories>

	<pluginRepositories>
		<pluginRepository>
			<id>spring-plugins-release</id>
			<url>https://repo.spring.io/plugins-release</url>
		</pluginRepository>
		<pluginRepository>
			<id>jcenter</id>
			<url>https://dl.bintray.com/asciidoctor/maven</url>
		</pluginRepository>
	</pluginRepositories>

```

### AerospikeTemplate

AerospikeTemplate is the central support class for Aerospike database operations. It provides:

* Basic POJO mapping support to and from Bins
* Convenience methods to interact with the store (insert object, update objects) and Aerospike specific ones.
* Connection affinity callback
* Exception translation into Spring's [technology agnostic DAO exception hierarchy](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/dao.html#dao-exceptions).

### Spring Data repositories

To simplify the creation of data repositories Spring Data Aerospike provides a generic repository programming model. It will automatically create a repository proxy for you that adds implementations of finder methods you specify on an interface.  

For example, given a `Person` class with first and last name properties, a `PersonRepository` interface that can query for `Person` by last name and when the first name matches a like expression is shown below:

```java
public interface PersonRepository extends CrudRepository<Person, Long> {

  List<Person> findByLastname(String lastname);

  List<Person> findByFirstnameLike(String firstname);
}
```

The queries issued on execution will be derived from the method name. Extending `CrudRepository` causes CRUD methods being pulled into the interface so that you can easily save and find single entities and collections of them.

You can have Spring automatically create a proxy for the interface by using the following JavaConfig:

```java
@Configuration
@EnableAerospikeRepositories(basePackageClasses = ContactRepository.class)
class ApplicationConfig extends AbstractAerospikeConfiguration {
	public @Bean(destroyMethod = "close") AerospikeClient aerospikeClient() {

		ClientPolicy policy = new ClientPolicy();
		policy.failIfNotConnected = true;

		return new AerospikeClient(policy, "localhost", 3000);
	}

	public @Bean AerospikeTemplate aerospikeTemplate() {
		return new AerospikeTemplate(aerospikeClient(), "test");
	}
}
```

This sets up a connection to a local Aerospike instance and enables the detection of Spring Data repositories (through `@EnableAerospikeRepositories`).

This will find the repository interface and register a proxy object in the container. You can use it as shown below:

```java
@Service
public class MyService {

  private final PersonRepository repository;

  @Autowired
  public MyService(PersonRepository repository) {
    this.repository = repository;
  }

  public void doWork() {

     repository.deleteAll();

     Person person = new Person();
     person.setFirstname("Oliver");
     person.setLastname("Gierke");
     person = repository.save(person);

     List<Person> lastNameResults = repository.findByLastname("Gierke");
     List<Person> firstNameResults = repository.findByFirstnameLike("Oli*");
 }
}
```

## Contributing to Spring Data

Here are some ways for you to get involved in the community:

* Get involved with the Spring community on Stackoverflow and help out on the [spring-data-Aerospike](http://stackoverflow.com/questions/tagged/spring-data-Aerospike) tag by responding to questions and joining the debate.
* Watch for upcoming articles on Spring by [subscribing](http://spring.io/blog) to spring.io.

## A more robust example is available on GitHub 
A Spring Boot Web Application example that uses Spring boot, Thymeleaf, Spring MVC, Tomcat and Aerospike is available on GitHub [spring-boot-web-aerospike-application](https://github.com/carosys/spring-boot-web-aerospike-application)