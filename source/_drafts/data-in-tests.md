---
title: Arranging data in tests
date: 2024-10-08
tags:
    - tests
categories:
    - EN
---

How to deal with data in test? 
Most manuals and examples do not cover this topic in depth, despite it's maybe the one of the most complicated things to make the right way in tests.

This article is about integration and functional tests. Because in the unit test we mostly do not rely on any external data. 

The main rule of testing says that tests should be isolated from each other. It means that tests should not depend on each other in any way. Isolation makes tests' support much easier and also allows to parallel tests' execution. It's pretty easy to know if tests are isolated properly - just run them in random order: if tests interact with each other (pass some values around or change global state) they will fail.


, or rely on data, provided or changed (or deleted) by other tests
in order to isolate tests we need some cleanup stage after each tests, on stage all the data changed by test should be reverted to the initial state so that next test could start from scratch and also required stage is data preparation when we see database with some test data required by tests here we are going to discuss different porches to manage that preparation and clean up in functional and integration tests.


basically, that preparation could be done in two ways the first one is to prepare all the data before running tests. It's so cold Charlotte fixture.
The other way is to allow each test to prepare the data it requires . The first way is simpler. But it introduces implicit dependencies between tests and fixtures and this can lead to hard to reason about tests when you look at the test and you don't know where is the data requires is defined. The best tests should be read as stories and shared fixture hampers this a lot the second way is to allow each test to prepare its data. This approach is harder to implement but it simplifies the supporting of test because in every single test you can see what happens from start to finish and what data is required for this concrete case from the other side. It makes test longer and more with irrelevant details so avoid this throwbacks. It is needed to write a lot of supporting record such as factories builders and so on.

cleanup stage can be implemented in several ways too. The simplest one is two reload shred fixture before each test, but this is sub optimal. The second way is to put the cleanup stage in each test so that each test knows how to revert changes it did. this approach is good for simple applications but if the logic is complex and test that's a lot of changes it may be difficult to revert this changes only so short approach is to wrap each test in a transaction which is rolled back after execution. this may cause problems when you do already use transactions in your application code so they will became nested and not all databases support transactions nesting. another sink to consider when implementing cleanup stage is the fact that it is easier to the back failed test when you can see data it modified so it is better to do cleanup before tests not after in order to be able to inspect the database state after the test. actually, if you prepare data directly in the test, you may not need the cleanup stage because test relies only on data eat prepared. This can simplify sink a lot.