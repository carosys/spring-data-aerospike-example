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
<dependency>
  <groupId>org.springframework.data</groupId>
  <artifactId>spring-data-aerospike</artifactId>
  <version>1.6.0.BUILD-SNAPSHOT</version>
</dependency>

<repository>
  <id>spring-libs-snapshot</id>
  <name>Spring Snapshot Repository</name>
  <url>http://repo.spring.io/libs-snapshot</url>
</repository>
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
* Create [JIRA](https://jira.springframework.org/browse/DATADOC) tickets for bugs and new features and comment and vote on the ones that you are interested in.  
* Github is for social coding: if you want to write code, we encourage contributions through pull requests from [forks of this repository](http://help.github.com/forking/). If you want to contribute code this way, please reference a JIRA ticket as well covering the specific issue you are addressing.
* Watch for upcoming articles on Spring by [subscribing](http://spring.io/blog) to spring.io.

Before we accept a non-trivial patch or pull request we will need you to sign the [contributor's agreement](https://support.springsource.com/spring_committer_signup).  Signing the contributor's agreement does not grant anyone commit rights to the main repository, but it does mean that we can accept your contributions, and you will get an author credit if we do.  Active contributors might be asked to join the core team, and given the ability to merge pull requests.
