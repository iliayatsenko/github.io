---
title: Data in integration tests
date: 2024-10-08
tags:
    - tests
categories:
    - EN
---

### Intro
How to write tests that rely on some data in certain state?
Managing test data is maybe one of the most complicated things to make right in tests.

This article is about integration and functional tests. Because in the unit test we mostly do not rely on any external data.

### Tests isolation
The main rule of auto-testing is "**tests should be isolated from each other**", i.e. they should not depend on each other in any way. Isolation makes tests' support much easier and also allows to parallel their execution. It's pretty easy to know if tests are isolated properly - just run them in random order: if tests interact with each other (pass some values around or change and read global state) they will fail.

_By the way, here is how to run tests in random order in PHPUnit: https://docs.phpunit.de/en/11.4/textui.html#test-order (or https://docs.phpunit.de/en/11.4/configuration.html#the-executionorder-attribute)_

Most of the integration and functional tests rely on test database and operate on some data. This data can be considered as global state. So to provide tests isolation we need to ensure that no tests rely on data that may be changed by another tests. How to achieve this? It depends on what way we choose to prepare the test data.

### Test data preparation
In general, there are two main directions, each with its pros and cons:

- "Shared Fixture": all the data is prepared once before running tests.
- "Fresh Fixture": each test prepares its own required data.

**"Shared Fixture"** is simpler to implement. But it introduces implicit dependencies between tests and fixtures. By simply looking at the test, you can't see where the data comes from and in what state it is. You should keep in mind that there is a fixture defined somewhere else, and look at fixture first in order to completely understand the test. The best tests should be read as stories and "Shared Fixture" hampers this a lot, because your "story" is split. 

Here you can see concrete patterns following the "Shared Fixture" idea: http://xunitpatterns.com/Shared%20Fixture%20Construction.html. 

_In PHP world, good example of this approach is `doctrine/data-fixtures` package._

On the other hand, **"Fresh Fixture"** simplifies understanding of test because in every single test you can see what happens from start to finish and what data in what state is required for this concrete case. Tests really look like "stories" and serve as a good documentation, actually. But it is harder to implement, comparing to "Shared Fixture": to avoid cluttering tests with irrelevant details (creation of objects, persistence code, etc.) you will probably need to write a lot of supporting code, such as factories, builders and so on. Here are the patterns helping to implement "Fresh Fixture": http://xunitpatterns.com/Fresh%20Fixture%20Setup.html


### So what about isolation?
To eliminate tests' interdependencies through shared data, we should avoid sharing data between tests. With "Fresh Fixture" this is achieved by default in most cases: if each test creates, changes and asserts its own data, there is no sharing between tests. Except rare cases when you need to  





, or rely on data, provided or changed (or deleted) by other tests
in order to isolate tests we need some cleanup stage after each tests, on stage all the data changed by test should be reverted to the initial state so that next test could start from scratch and also required stage is data preparation when we see database with some test data required by tests here we are going to discuss different porches to manage that preparation and clean up in functional and integration tests.



cleanup stage can be implemented in several ways too. The simplest one is two reload shred fixture before each test, but this is sub optimal. The second way is to put the cleanup stage in each test so that each test knows how to revert changes it did. this approach is good for simple applications but if the logic is complex and test that's a lot of changes it may be difficult to revert this changes only so short approach is to wrap each test in a transaction which is rolled back after execution. this may cause problems when you do already use transactions in your application code so they will became nested and not all databases support transactions nesting. another sink to consider when implementing cleanup stage is the fact that it is easier to the back failed test when you can see data it modified so it is better to do cleanup before tests not after in order to be able to inspect the database state after the test. actually, if you prepare data directly in the test, you may not need the cleanup stage because test relies only on data eat prepared. This can simplify sink a lot.