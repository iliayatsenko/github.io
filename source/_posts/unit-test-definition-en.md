---
title: Unit test definition
date: 2022-06-06
tags:
    - unit-tests
categories:
    - EN
---

Historically, there have been two views on what a unit test should be:

- **"London school"** (or "mockist" approach),
- **"Detroit school"** (or "classical" approach).

<!-- more -->

Both approaches agree that "units" should be isolated. The fundamental difference is what exactly is meant by a “unit” and what exactly to isolate from.

“Mockists” consider **a unit of code** (most often one specific class) as a unit, and “classics” consider it **a unit of behavior, or a “feature”** (a module of one or more classes).  ****“Mockists” believe that each unit should be tested in isolation from other units, and “classics” - that each feature should be tested in isolation from inter-process interactions (that is, tests should not rely on something “outside” the running process - system time, database, file system, network, etc.).

For example, we have a feature “Car”. The car has a starting price and engine power reserve. The car can drive until the number of kilometers traveled exceeds 300 000, while losing 0.01% in price per kilometer. Let's say we wrote the code like this:

```php
class Car
{
    private Engine $engine;
    private float $currentPrice;
    public function __construct(Engine $engine, float $startPrice)
    {
        $this->engine = $engine;
        $this->currentPrice = $startPrice;
    }
    public function run(int $kilometers)
    {
        $this->engine->run($kilometers);
        $this->currentPrice -= ($kilometers * $this->currentPrice / 10000);
    }
    public function getCurrentPrice()
    {
        return $this->currentPrice;
    }
}
class Engine
{
    const MAX_KILOMETERS = 300_000;
    private int $counter = 0;
    public function run(int $kilometers)
    {
        if ($this->counter > self::MAX_KILOMETERS) {
            throw new MaintenanceRequiredException();
        }
        $this->counter += $kilometers;
        echo 'Drrruuummmm!!!';
    }
}
```

Mokists treat both `Car` and `Engine`as two separate units and aim for a “one class = one test” match. All dependencies of the currently tested class are replaced with mocks:

```php
// mockists' unit-test
// CarTest.php
function test_car_becomes_cheaper()
{
    // Mock Engine, because now only the Car is under test
    $engineMock = Mock::create(Engine::class);
    $startPrice = 10_000;
    $car = new Car($engineMock, $startPrice);
    $car->run(1000);
    assertEquals(9990, $car->getCurrentPrice());
        // also need to check that Car had interacted with Engine
        $engineMock->assertCalledOnce('run', [1000]);
}
// EngineTest.php
function test_engine_cannot_run_more_than_300_000()
{
    // Now testing only Engine
    $engine = new Engine();
    expectException(MaintenanceRequiredException::class);
    $engine->run(300_001);
}
```

And for the “classics” there will be only one unit here - `Car`, since it implements the behavior we are interested in. And the `Engine` itself is only an implementation detail of this behavior, a separate test is not written for it:

```php
// CarTest.php
function test_car_becomes_cheeper()
{
    $startPrice = 10_000;
        // Testing Car as a whole, as a behavioral unit
    $car = new Car(new Engine(), $startPrice);
    $car->run(1000);
    assertEquals(9990, $car->getCurrentPrice());
}
function test_car_cannot_run_more_than_300_000()
{
        // Don't check interactions between classes,
        // treat the feature as one thing
    $car = new Car(new Engine(), 10_000);
    expectException(MaintenanceRequiredException::class);
    $car->run(300_001);
}
```

With this approach, there will be no correspondence “one class = one test”. And mocks are needed only to replace classes that provide inter-process interactions (for example, repositories, or network clients), i.e. [infrastructure code](https://www.notion.so/f679b40f4fe846439679865e6dc35dac?pvs=21) . About [the types of mocks](https://www.notion.so/11466bd83e20420b8bbb1906927cb014?pvs=21) - another time.

Both approaches have their pros and cons.

**"London School"**

+++ It's easier to determine the cause of test failures: just look at the class whose test failed.

+++ It's easy to instantiate the classes under test by simply mocking all the dependencies.

--- Due to the abundance of mocks and checks of inter-class interactions, tests become more “brittle”, that is, tied to 
the internal structure of the code. With the slightest refactoring, you will have to edit a lot of tests.

--- Tests describe expectations in a very granular way. While reading them it is difficult to understand the 
requirements for the system as a whole, there is a risk of “not seeing the forest behind the trees”.

--- Requires auxiliary libraries for creating mocks.

**"Detroit School"**

+++ Since we are testing only the behavior of the feature that is observable from the outside, the best ratio of “protection from bugs / labor costs for supporting tests” is achieved. Inside the feature, you can refactor in any way - for example, move the kilometer counter to a separate class if the calculation logic becomes more complicated, etc. - you don’t have to edit the tests.

+++ Tests are at the same time a kind of documentation, as they describe the expectations from the system as a whole, using formulations that are closer to the real world.

--- When tests fail, it is not immediately clear where exactly the reason is, since several real classes are involved 
in the test at once.

--- In tests, you need to create full-fledged instances with all dependent objects, which can be a daunting task and 
  require helper factory methods.

Personally, in this regard, I share the preferences of V. Khorikov - the author of a book about unit testing - and I write tests in the classical style, with a minimum number of mocks. More information on the topic can be found in [his book](https://www.amazon.com/gp/product/1617296279).