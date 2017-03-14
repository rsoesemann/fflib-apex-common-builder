# fflib-apex-common-builder

#Testdata Builder Extension to fflib_apex-common

[![Build Status](https://travis-ci.org/financialforcedev/fflib-apex-common-builder.svg)](https://travis-ci.org/financialforcedev/fflib-apex-common-builder)

**Dependencies:** Must deploy [FinancialForce Apex Common](https://github.com/financialforcedev/fflib-apex-common) before deploying this library

There is awesome support in fflib_apex-common for Apex unit test. They should be fast and isolated. With [ApexMocks](https://github.com/financialforcedev/fflib-apex-mocks) [both is achieved](http://andyinthecloud.com/2015/03/22/unit-testing-with-apex-enterprise-patterns-and-apexmocks-part-1/). But complex code also needs integration tests where code is execute on more complex test data. In the world of Enterprise software outside of Salesforce.com there are experts that have created patterns for flexible and readable (fluent, concise) test data generation.

Among them the most notable is [Nat Pryce](http://www.natpryce.com/) who wrote a [great book](http://www.amazon.com/Growing-Object-Oriented-Software-Guided-Tests/dp/0321503627) about testing and somewhat invented the [TestDataBuilder pattern](http://www.natpryce.com/articles/000714.html). This extension ports this pattern to Salesforce.com and Apex 

- By incorporating **a single short** Builder class for each test-relevant Domain SObject we could centralize all the creation knowledge and eliminating redundancy. 
- By internally leveraging the 'fflib_SObjectUnitOfWork' for the DML all **test run dramatically faster**.
- The [Fluent API](https://en.wikipedia.org/wiki/Fluent_interface) style of the Builder pattern combined with having all the database wiring encapsulated in the Unit of work made each test much more understandable.

<a href="https://githubsfdeploy.herokuapp.com?owner=financialforcedev&repo=fflib-apex-common-builder">
  <img alt="Deploy to Salesforce"
       src="https://raw.githubusercontent.com/afawcett/githubsfdeploy/master/src/main/webapp/resources/img/deploy.png">
</a>

##Usage Examples


