# microservices-testing

how to test microservices based applications?

> There has been a shift in service based architectures over the last few years towards smaller, more focussed "micro" services. There are many benefits with this approach such as the ability to independently deploy, scale and maintain each component and parallelize development across multiple teams. However, once these additional network partitions have been introduced, the testing strategies that applied for monolithic in process applications need to be reconsidered.

![Testing Pyramid 1](img/testing-pyramid.png)

Unit tests are fast to execute, give the right level of feedback about what is broken. As you go up the pyramid, the tests are slower and it becomes harder to point out root cause of failures because the surface area is larger.

Its essential point is that you should have many more low-level unit tests than high level end-to-end tests running through a GUI.

![Test Pyramid](img/test-pyramid.png)

A common problem is that teams conflate the concepts of end-to-end tests, UI tests, and customer facing tests. These are all orthogonal characteristics. For example a rich javascript UI should have most of its UI behavior tested with javascript unit tests using something like Jasmine. A complex set of business rules could have tests captured in a customer-facing form, but run just on the relevant module much as unit tests are.

> I always argue that high-level tests are there as a second line of test defense. If you get a failure in a high level test, not just do you have a bug in your functional code, you also have a missing or incorrect unit test. Thus I advise that before fixing a bug exposed by a high level test, you should replicate the bug with a unit test. Then the unit test ensures the bug stays dead.

# Microservices can usually be split into similar kinds of modules

![Architecture](img/architecture.png)

* **Resources** act as mappers between the application protocol exposed by the service and messages to objects representing the domain. Typically, they are thin, with responsibility for sanity checking the request and providing a protocol specific response according to the outcome of the business transaction.

* Almost all of the service logic resides in a **domain model** representing the business domain. Of these objects, **services** coordinate across multiple domain activities, whilst **repositories** act on collections of domain entities and are often persistence backed.

* If one service has another service as a collaborator, some logic is needed to communicate with the external service. A gateway encapsulates message passing with a remote service, marshalling requests and responses from and to domain objects. It will likely use a client that understands the underlying protocol to handle the request-response cycle.

* Except in the most trivial cases or when a service acts as an aggregator across resources owned by other services, a micro-service will need to be able to persist objects from the domain between requests. Usually this is achieved using object relation mapping or more lightweight data mappers depending on the complexity of the persistence requirements. Often, this logic is encapsulated in a set of dedicated objects utilised by repositories from the domain.

* A resource receives a request and once validated, calls into the domain to begin handling of the request.

* If many modules must be coordinated to complete the business transaction, the resource delegates to a service. Otherwise, it communicates directly with the relevant module.

* Connections out to external services require special attention since they cross network boundaries. The system should be resilient to outages of remote components. Gateways contain logic to handle such error cases. Typically, communications with external services are more coarse grained than the equivalent in process communications to prevent API chattiness and latency.
  
* Similarly, communications with external datastores have different design considerations. Whilst a service is often more logically coupled to its datastore than to an external service, the datastore still exists over a network boundary incurring latency and risk of outage.
  
* The presence of network partitions affects the style of testing employed. Tests of these modules can have longer execution times and may fail for reasons outside of the team's control.

### Internal Resources 

are useful for more than just testing...

Though it may seem strange, exposing internal controls as resources can prove useful in a number of cases besides testing such as monitoring, maintenance and debugging. The uniformity of a RESTful API means that many tools already exist for interacting with such resources which can help reduce overall operational complexity.

The kinds of internal resources that are typically exposed include logs, feature flags, database commands and system metrics. Many microservices also include health check resources which provide information about the health of the service and its dependencies, timings for key transactions and details of configuration parameters. A simple ping resource can also be useful to aid in load balancing.

Since these resources are more privileged in terms of the control they have or the information they expose, they often require their own authentication or to be locked down at the network level. By namespacing those parts of the API that form the internal controls using URL naming conventions or by exposing those resources on a different network port, access can be restricted at the firewall level.

# Testing Strategies for Microservices

![Testing Strategies for Microservices](img/test-types.png)

## Unit Tests

Unit tests : exercise the smallest pieces of testable software in the application to determine whether they behave as expected.

## Integration Tests

Integration tests : verify the communication paths and interactions between components to detect interface defects.

### Gateway Integration Tests

### Persistence Integration Tests

## Component tests

limit the scope of the exercised software to a portion of the system under test, manipulating the system through internal code interfaces and using test doubles to isolate the code under test from other components.

### In Process

### Out Process

## Contract tests

Verify interactions at the boundary of an external service asserting that it meets the contract expected by a consuming service.

### Consumer Driven Contracts
  
It's nearly impossible for you to know all the ways consumers might use your services. With a [consumer-driven contract](http://martinfowler.com/articles/consumerDrivenContracts.html) model, it's the consumer's responsibility to provide a suite of tests that specify what types of interactions are needed and in which format. Your service would then agree to this contract and ensure that it's not broken. This gets rid of dependencies on other services. This approach also enables you to verify that the contract is being fulfilled at build time.

Tools like [Pact](https://github.com/realestate-com-au/pact) will give you a better understanding of how you can achieve this type of functionality for developing and testing microservices. Once you have a consumer-driven contract process in place, the next key step in testing microservices is to shift-right into the previously forbidden world of production.

## End-to-end tests

verify that a system meets external requirements and achieves its goals, testing the entire system, from end to end.

## Synthetic Transactions (Tests running production)

Unit and integration tests are used to test the individual parts of the microservice, and component tests are used to test the service as a whole. 

---

However, there are three kinds of tests we are interested in running from our continuous integration build: unit tests, component tests, and acceptance tests:

- **Unit tests** are written to test the behavior of small pieces of your application in isolation (say, a method, or a function, or the interactions between a small group of them). They can usually be run without starting the whole application. They do not hit the database (if your application has one), the filesystem, or the network. They don’t require your application to be running in a production-like environment. Unit tests should run very fast—your whole suite, even for a large application, should be able to run in under ten minutes.

- **Component tests** test the behavior of several components of your application. Like unit tests, they don’t always require starting the whole application. However, they may hit the database, the filesystem, or other systems (which may be stubbed out). Component tests typically take longer to run.

- **Acceptance tests** test that the application meets the acceptance criteria decided by the business, including both the functionality provided by the application and its characteristics such as capacity, availability, security, and so on. Acceptance tests are best written in such a way that they run against the whole application in a production-like environment. Acceptance tests can take a long time to run—it’s not unheard of for an acceptance test suite to take more than a day to run sequentially.

These three sets of tests, combined, should provide an extremely high level of confidence that any introduced change has not broken existing functionality.

---

- Single service testing: Tests carried out in isolation by a team that owns a particular microservice in a system.
- Staging environment: Tests that are run on objects in a staging environment. The microservices that form a particular application are deployed into a staging environment for testing.
- Production environment: Tests carried out on the live production system. 

Tests should be automated as part of the build, release, run (delivery) pipeline.

## Single Service Testing

This section defines single service testing as the tests that the owners of an individual service must create and run. These tests exercise the code that is contained inside the logical boundary in the diagram of the internal microservice architecture. Three types of tests apply here: Unit tests, component tests, and integration tests. Unit and integration tests are used to test the individual parts of the microservice, and component tests are used to test the service as a whole. 

### Testing domain or business function

The code in your microservice that performs business function should not make calls to any services external to the application. This code can be tested by using unit tests and a testing framework such as JUnit1. The unit tests should test for behavior and either use the actual objects (if no external calls are needed) or mock the objects involved in any operations.

When writing tests using the actual objects, a simple JUnit test suffices. For creating mocks of objects, you can either use the built-in capabilities of Java EE or use a mocking framework. The @Alternatives annotation2 in the Context and Dependency Injection (CDI) specification enables injection of mock objects instead of the actual beans. Plenty of mocking frameworks are available for Java. For example, JMockit3 is designed to work with JUnit to allow you to mock objects during testing. In the most basic test using JMockit, you can create mocked objects by using the @Mocked annotation and define behavior when this object is called by using the Expectations() function.

### Testing resources

Classes that expose JAX-RS endpoints or receive events should be tested by using two types of tests: Integration tests and contract tests.

#### Integration tests
Integration tests are used to verify the communication across network boundaries. They should test the basic success and failure paths in an exchange. Integration tests can either be run in the same way as unit tests, or by standing up the application on a running server. To run integration tests without starting the server, call the methods that carry JAX-RS annotations directly. During the tests, create mocks for the objects that the resource classes call when a request comes in.

To run integration tests on a running server, you can use one of the methods described in “Tests on a running server” on page 82. To drive the code under test, use the JAX-RS clientprovided by JAX-RS 2.0. to send requests.

Integration tests should validate the basic success and error paths of the application. Incorrect requests should return useful responses with the appropriate error code.

#### Consumer driven contract

A consumer of a particular service has a set of input and output attributes that it expects the service to adhere to. This set can include data structures, performance, and conversations. The contract is documented by using a tool like Swagger. Generally, have the consumers of a service drive the definition of the contract, which is the origin of the term consumer driven contract.

Consumer driven contract tests are a set of tests to determine whether the contract is being upheld. These tests should validate that the resources expect the input attributes defined in the contract, but also accepts unknown attributes (it should just ignore them). They should also validate that the resources return only those attributes that are defined in the documentation. To isolate the code under test, use mocks for the domain logic.

Maintaining consumer driven contract tests introduces some organizational complexity. If the tests do not accurately test the contract defined, they are useless. In addition, if the contract is out of date, then even the best tests will not result in a useful resource for the consumer. Therefore, it is vital that the consumer driven contract is kept up to date with current consumer needs and that the tests always accurately test the contract.

Contract tests require the actual API to be implemented. This technique requires the application be deployed onto a server.  Use tools such as the Swagger editor4 to create these tests. The Swagger editor can take the API documentation and produce implementations in many different languages. 

Another dimension to contract testing is the tests that are run by the consumer. These tests must be run in an environment where the consumer has access to a live version of the service, which is the staging environment. 

#### Tests on a running server
A few different methods are available for starting and stopping the application server as part of your automated tests. There are Maven and Gradle plug-ins for application servers such as WebSphere Application Server Liberty that allow you to add server startup into your build lifecycle. This method keeps complexity out of your application and contains it in the build code. For more information about these plug-ins, see the following websites:

- Maven: https://github.com/WASdev/ci.maven
- Gradle: https://github.com/WASdev/ci.gradle

Another solution is to use Arquillian. Arquillian can be used to manage the applications server during tests. It allows you to start and stop the server mid-test, or start multiple servers. Arquillian is also not affected by the container, so if you write Arquillian tests, they can be used on any application server. This feature is useful for contract testing because the consumers do not have to understand the application server or container that is used by the producer. For more information about Arquillian, see the following website:
http://arquillian.org

### Testing external service requests

Inevitably, your microservice must make calls to external services to complete a request, such as calls to other microservices in the application or services external to the application. The classes to do this construct clients that make the requests and handle any failures. The code can be tested by using two sets of integration tests: One at the single service level and one in the staging environment. Both sets test the basic success and error handling of the client. More information about the tests in the staging environment is available in 7.4.2, “Integration” on page 86.

The integration tests at the single service level do not require the service under test or the external services to be deployed. To perform the integration tests, mock the response from the external services. If you are using the JAX-RS 2.0 client to make the external requests, this process can be done easily by using the JMockit framework

### Testing data requests

In a microservice architecture, each microservice owns its own data. If you follow this guideline, the developers of a microservice are also responsible for any external data stores used. The code that makes requests to the external data store and performs data mapping and validation is contained in the repositories layer. When testing the domain logic, this layer should be mocked. Tests for data requests, data mapping, and validation are done by using integration tests with the microservice and a test data store deployed locally or on a private cloud. The tests check the basic success and error paths for data requests. If the data mapping and validation for your application requires extensive testing, consider separating out this code and testing it using a mocked database client class.

**Test data**

The local version of the data store must be populated with data for testing. Think carefully about what data you put in the data store. The data should be structured in the same way as production data but should not be unnecessarily complicated. It must serve the specific purpose of enabling data request tests.

###  Component testing

Component tests are designed to test an individual microservice as one piece. The component is everything inside the network boundary, so calls to external services are either mocked or are replaced with a “test-service.” There are advantages and disadvantages to both scenarios.

**Using mocks**

By mocking the calls to external services, you have fewer test objects to configure. You can easily define the behavior of the mocked system by using frameworks like JMockit, and no tests will fail due to network problems. The disadvantage of this approach is that it does not fully exercise the component because you are intercepting some of the calls, increasing the risk of bugs slipping through.

**Test services**

To fully exercise the communication boundaries of your microservice, you can create test services to mimic the external services that are called in production. These test services can also include a test database. The test services can also be used as a reference for consumers of your microservice. The disadvantage of this system is that it requires you to maintain your test services. This technique requires more processor cycles than maintaining a mocking system as you must fully test the test microservice and create a deployment pipeline.

After you are using a mocking framework for other levels of testing, it makes sense to reuse those skills. However, if you do take the mocking approach, you must make sure that the tests in your staging environment exercise inter-service communications effectively. 

### Security verification

Security is important in a distributed system. You can no longer put your application behind a firewall and assume that nothing will break through.

Testing the security of your microservice is slightly different depending on how you implement security. If the individual services are just doing token validation, then test at the individual service level. If you are using another service or a library to validate the tokens, then that service should be tested in isolation and the interactions should be tested in the staging environment.

The final type of tests to run is security scanners such as IBM Security AppScan® to highlight security holes. A


# References

* https://martinfowler.com/articles/microservice-testing/
* https://www.thoughtworks.com/insights/blog/architecting-continuous-delivery
* CD Book
