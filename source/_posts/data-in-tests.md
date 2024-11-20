---
title: Data in integration tests
date: 2024-11-20
tags:
    - tests
categories:
    - EN
---

How to write tests that rely on some data in certain state?
<!-- more --> 

Managing test data is maybe one of the most complicated things to make right in tests.

This article is about integration and functional tests. Because in the {% post_link unit-test-definition-en 'unit test' %} we mostly do not rely on any external data.

### Tests isolation

The main rule of auto-testing is "**tests should be isolated from each other**", i.e. they should not depend on each other in any way. Isolation makes support much easier and also allows to parallel tests' execution. 

It's pretty easy to know if tests are isolated properly - just run them in random order: if tests interact with each other (pass some values around or change/read global state) they will fail.

_By the way, here is how to run tests in random order in PHPUnit: https://docs.phpunit.de/en/11.4/textui.html#test-order_ 

Most of the integration and functional tests rely on the test database and operate on some data. If you do not create new database for each test (you probably don't), this data can be considered as global state. So to provide tests isolation we need to ensure that no test rely on data that may be changed by another test. How to achieve this? It depends on what way we choose to prepare the test data.

### Test data preparation

In general, there are two main directions, each with its pros and cons:

- **"Shared Fixture":**

  All the data is prepared once before running tests. This approach is simple to implement, but it introduces implicit dependencies between tests and fixtures.

  By simply looking at the test, you can't see where the data comes from and in what state it is. You should keep in mind that there is a fixture defined somewhere else, and look at fixture first in order to completely understand the test. The best tests should be read as stories and "Shared Fixture" hampers this a lot, because your "story" is split.

  Here you can see concrete patterns adhering to the "Shared Fixture" idea: http://xunitpatterns.com/Shared%20Fixture%20Construction.html.
  
  _In PHP world, good example of this approach is `doctrine/data-fixtures` package._

- **"Fresh Fixture":**

  Each test prepares its own required data. It simplifies understanding of test because in every single test you can see what happens from start to finish and what data in what state is required for this concrete case. Tests really look like "stories" and serve as a good documentation, actually. 

  But it is harder to implement, comparing to "Shared Fixture": to avoid cluttering tests with irrelevant details (creation of objects, persistence code, etc.) you will probably need to write a lot of supporting code, such as factories, builders and so on. 

  Here you can see concrete patterns helping to implement "Fresh Fixture": http://xunitpatterns.com/Fresh%20Fixture%20Setup.html

  _In PHP world, I do not know a decent library helping to implementing "Fresh Fixture" pattern. Do you?_

### So what about isolation?

To eliminate tests' interdependencies through shared data, we should avoid sharing data between tests. 

With **"Fresh Fixture"** partial isolation is achieved by default in most cases: if each test creates, changes and asserts its own data, there is no sharing between tests. And a good outcome is that tests may run in parallel easily. 

But still this is not full isolation, and problems may happen when your tests do assertions against count of some data rows (rows created by other tests may interfere), or when database has unique constraints (rows created by other tests may block inserting new ones). Both are solvable problems, though: use unique values for keys when inserting data and filter by these values when counting. Creating globally unique values can be implemented using global sequence provider. For example, see [codeception/module-sequence](https://codeception.com/docs/modules/Sequence.html).

With **"Shared Fixture"** things are much worse, because all tests work with the same data, thus, by default, are not isolated from each other at all.

In both cases, in order to achieve full isolation we need to return data in its initial state between tests. This cleanup can be made before or after each test. Doing it before test helps with debugging, because data stays in place after the test, and you can inspect it manually.

Cleanup may be performed in the following ways:
- **Purging (and re-populating, if any) all the data:**

  The most straightforward approach, but the least performant. In large database it may take some time to purge all the data, or recreate the schema from scratch. Since this should be done on each single test, this approach may cause tests to run longer.
    
- **Rolling back uncommitted changes:**
    
  This approach implies wrapping each test in a transaction, which is rolled back after the test. Of course, it requires underlying storage to support transactions. 

  Note also that if tested code already uses transactions, nested transactions will appear, which are not always supported by storage or ORM. 

  _BTW, in PHP world, if you're using Doctrine, you can use [dmaicher/doctrine-test-bundle](https://github.com/dmaicher/doctrine-test-bundle). Thankfully to Doctrine's [transactions' nesting support](https://www.doctrine-project.org/projects/doctrine-dbal/en/4.2/reference/transactions.html#transaction-nesting), it solves the problem of nested transactions even if underlying storage does not allow them._

- **Tracking all the changes made during the test and applying reverse changes:**

  In each test, track all the changes performed on data, and revert them after the test. The problem is that it is easy to revert insertions and deletions made by test itself, but with updates it becomes much more complicated. Also, frequently the test cannot easily track changes made by tested code, not the test itself.
  
Obviously, only with _purging_ strategy it is possible to do cleanup **before** tests. _Rolling back transactions_ and _applying reverse changes_ are applicable only **after** the test. 

### To sum up

Comparison of approaches to test data preparation:

| Name             | Simplicity | Clarity | Cleanup                         | Useful libs (PHP)                |
|------------------|:----------:|:-------:|---------------------------------|----------------------------------|
| "Shared Fixture" |     +      |    -    | **Required in all cases**       | doctrine/data-fixtures           |
| "Fresh Fixture"  |     -      |    +    | Possible to avoid in most cases | codeception/module-sequence, ??? |

Comparison of approaches to test data cleanup:

| Name                 | Simplicity | Performance |               Allows parallelism                | Allows manual data inspection | Caveats                                                                                                                   |
|----------------------|:----------:|:-----------:|:-----------------------------------------------:|:-----------------------------:|---------------------------------------------------------------------------------------------------------------------------|
| Purging              |     +      |      -      |                        -                        | + (if done **before** tests)  |                                                                                                                           |
| Transaction rollback |    +/-     |      +      | + (but depends on transactions isolation level) |               -               | Nested transactions may happen, ensure that your storage or ORM support them                                              |
| Reverse changes      |     -      |     +/-     |                        -                        |               -               | Tracking and reverting of all the changes, made by test and tested code, is hardly possible in more or less complex cases |

### Conclusion

What way of data population and cleanup to choose? In general, I'd suggest to follow such guideline:
  - If project is relatively simple, i.e. contains several entities, and provides mostly CRUD-like operations, use "Shared Fixture". For cleanup, use what is simpler: for example, in PHP world, if you are using Doctrine, it's quite simple to set up automatic rollback of changes made during tests - just use aforementioned extension. If you are not a Doctrine user, maybe "purging and repopulating" will be simpler to implement and support.
  - Instead, if project contains lots of business logic, it is preferable to invest time in implementing "Fresh Fixture". This, aside of other benefits, will probably eliminate the need to do a cleanup at all. But if you still need full isolation, choose between "purging and repopulating" and "rolling back transactions" approaches, depending on what is simpler to implement. "Tracking and reverting" approach should be avoided because potentially it may become very complex shortly.
