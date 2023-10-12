# PineUnit by Guardian667

A comprehensive testing framework for Pine Script on TradingView. Built with well-known testing paradigms like Assertions, Units and Suites. It offers the ability to log test results in TradingView's built-in Pine Protocol view, as well as displaying them in a compact table directly on your chart, ensuring your scripts are both robust and reliable.

Unit testing Pine Script indicators, libraries, and strategies becomes seamless, ensuring the precision and dependability of your TradingView scripts. Beyond standard function testing based on predefined input values, PineUnit supports series value testing. This means a test can run on every bar, taking into account its specific values. Moreover, you can specify the exact conditions under which a test should execute, allowing for series-based testing only on bars fitting a designated scenario.



- [Quick Start](#quick-start)
- [Usage Guide](#usage-guide)
  - [Create and configure Your TestSession](#create-session)
  - [Create Tests and organise them](#create-test)
  - [Test series-based functions on each bar](#series-test)
  - [Starting series-based Tests after a defined bar threshold](#since-bar)
  - [Multiple Assertions within one single Test](#multi-assert)
  - [Test if series-based Functions don't deliver a result to early](#until-bar)
  - [Execute a series-based Test only on a specific bar](#on-bar)
  - [Define "When" conditions for Tests](#when)
  - [Ignore Tests](#ignore)
  - [Difference between ignored and unexecuted tests](#ignore-unexecuted)
  - [Show Test Results](#test-report)
- [Performance](#performance)
- [Contribution](#contribution)
- [License](#license)
- [A Personal Note](#note)

<a id="quick-start"></a>
## Quick Start
To get started swiftly with PineUnit, follow this minimalistic example.
```
import Guardian667/PineUnit/1 as PineUnit

var testSession = PineUnit.createTestSession()

var trueTest = testSession.createSimpleTest("True is always True")
trueTest.assertTrue(true)

testSession.report()
```
After running your script, you'll notice a table on your chart displaying the test results. For a detailed log output, you can also utilize the Pine Protocol view in TradingView.
```
--------------------------------------------------------------
T E S T S
--------------------------------------------------------------
Running Default Unit
Tests run: 1, Failures: 0, Not executed: 0, Skipped: 0
```
To further illustrate, let's introduce a test that's destined to fail:
```
var bullTest = testSession.createSeriesTest("It's allways Bull Market")
bullTest.assertTrue(close > open, "Uhoh... it's not always bullish")
```
After executing, the test results will reflect this intentional discrepancy:
```
--------------------------------------------------------------
T E S T S
--------------------------------------------------------------
Running Default Unit
Tests run: 2, Failures: 1, Not executed: 0, Skipped: 0 <<< FAILURE! - in

It's allways Bull Market
Uhoh... it's not always bullish ==> expected: <true>, but was <false>
```
This shows how PineUnit efficiently captures and reports discrepancies in test expectations.

It's important to recognise the difference between `createSimpleTest()` and `createSeriesTest()`. In contrast to a simple test, a series-based test is executed on each bar, making assertions on series values.

<a id="usage-guide"></a>
## Usage Guide
You find the following examples within the demo section of the file `PineUnit.pine` and within the library `PineUnit` on TradingView

<a id="create-session"></a>
### Create and configure Your TestSession
First create a TestSession. Remember to use the `var` keyword to ensure the TestSession is initialized only once at the script's first execution and maintain its state. 
```
var testSession = PineUnit.createTestSession()
testSession.setMaxFailingBars(3)
var displaySettings = testSession.displaySettings
displaySettings.onlyFailures()
```
Here is a complete list of the method on type `TestSession` which can be called fluently.
- `setActive(true|false)` Toggles is the TestSession's activity. Convenience: `deactivate()`.
- `setMaxFailingBars(NUMBER)` For tests evaluating series values on each bar, you can tell PineUnit to stop test executions, once test failed on a minimum amount of bars. Default: 1
- `setTimezone(string)` By default the timezone of the `symbol` is used to display test results. Unfortunately it is not possible to detect the configured timezone within a chart. So set this manually to your timezone by something like `"Europe/Berlin"`
- `setLogBarOnSeriesTests(true|false)` The bar_index and time of a failure is not logged by default, since each failure is logged exclusively on the correspnding bar before the report is created, for navigation by the cross-hair icon. By calling this method, the bar and time information can also be included within he report. 


### Configure your Result Display
The result display, that is shown on your chart, can be adjusted. First make a reference by `var displaySettings.testSession.displaySettings` and call the methods you want fluently.

Here is a complete list of the method on type `DisplaySettings`
- `setShow(bool)` Hides the result display. Convenience: `hide()`
- `setPosition(string)` Sets the position ofthe result display. Use Pine Script's constants like `position.bottom_center`.
- `setTextSize(string)` Stes the text size within the result display.  Use Pine Script's constants like `size.normal`
- `setShowSuccess(bool)` Convenience: `hideSuccess()`
- `setShowFailure(bool)` Convenience: `showOnlyFailures()`
- `setShowIgnored(bool)` Convenience: `hideIgnored()`
- `setShowUnexecuted(bool)` Convenience: `hideUnexecuted()`

**Tipp**: If the chart covers the result display, right-click on the result dsiplay > Visual order > Bring to front 

<a id="create-test"></a>
### Create Tests and organise them
#### Test without a Unit or Suite
Without the need to organise tests, e.g. in a short script, irectly create a test without wrapping it inside a unit or suite. Here we executed both a successful and a failing test:
```
var myTest = testSession.createSimpleTest("True is always True")
myTest.assertTrue(true)

var falseTest = testSession.createSimpleTest("False is never True")
falseTest.assertTrue(false)
```
This method is best for quick and one-off tests where you don't need the organizational benefits of units or suites. The test log contains such result within the `Default Unit`or the `Default Suite` and look like this
```
--------------------------------------------------------------
T E S T S
--------------------------------------------------------------
Running Default Unit
Tests run: 2, Failures: 1, Not executed: 0, Skipped: 0 <<< FAILURE! - in

False is never True
expected: <true>, but was <false>

Results :
Tests run: 2, Failures: 1, Not executed: 0, Skipped: 0
``` 

#### Test within a Unit
For a slightly more organized approach, you can create a unit and then define your tests within that unit:
```
var universeUnit = testSession.createUnit("Universe Unit")
var universeTest = universeUnit.createSimpleTest("Is 42 actually 42?")
universeTest.assertEquals(42, 42, "Uhoh... the universe is borken!")
```
Units are useful for grouping related tests, especially when you have multiple tests validating different functionalities of a single concept or function.

This Unit was created on the TestSession directly, so the test log shows it under the Default Suite.
```
--------------------------------------------------------------
T E S T S
--------------------------------------------------------------
Running Default Unit
Tests run: 2, Failures: 1, Not executed: 0, Skipped: 0 <<< FAILURE! - in

False is never True
expected: <true>, but was <false>

Running Universe Unit
Tests run: 1, Failures: 0, Not executed: 0, Skipped: 0

Results :
Tests run: 3, Failures: 1, Not executed: 0, Skipped: 0
```

#### Test within a Suite
For the most structured and organized approach, first create a suite, then create units within that suite, and finally add your tests to the units:
```
var piSuite = testSession.createSuite("Pi (We love ya!)")
var piUnit = piSuite.createUnit("Pi Unit")
var piTest = piUnit.createSimpleTest("Pi should begin correctly")
piTest.assertTrue(str.startswith(str.tostring(math.pi), "3"), "The first digit of Pi is not three!")
```
This approach is best when you have a large number of tests and want to categorize them at two levels - first at the suite level (e.g., by feature or module) and then within each suite, further categorizing by units (e.g., by function or method).

Now the test log contains an additional section for that Suite
```
--------------------------------------------------------------
T E S T S Pi (We love ya!)
--------------------------------------------------------------
Running Pi Unit
Tests run: 1, Failures: 0, Not executed: 0, Skipped: 0

Results :
Tests run: 1, Failures: 0, Not executed: 0, Skipped: 0
```

<a id="series-test"></a>
### Test series-based functions on each bar
For functions that compute values consecutively on every bar, you can utilize series tests to validate them across all bars by calling `createSeriesTest("Name")` and a Unit, Suite or TestSession.

In the following example, we're trying to calculate the highest value of the last n bars.
```
highest(int length) =>
    highest = high
    for i = 1 to length-1 by 1
        highest := (high[i] > highest) ? high[i] : highest
    highest
    
var highestUnit = testSession.createUnit("Highest")
var highestOfLastThreeTest = highestUnit.createSeriesTest("Should calculate highest of last three bars")
highestOfLastThreeTest.assertEquals(math.max(high, high[1], high[2]), highest(3), "That is not the highest price of the last three bars")
```
Unfortunately this test fails already on the very first bar, since the verification method we use `math.max` returns `na` if at least one of the input values is `na`. We keep this in mind for the next axample. More important is that you now see this extra line in the log output, before the overall test results are display:
```
[2023-09-18T00:00:00.000+02:00]: Highest
That is not the highest price of the last three bars ==> expected: <NaN>, but was <34638.5>
```
When you hover over that message you find a crosshair icon, that you can click to navigate to the bar where the error occured and investigate what was going on there.

Within the test results you find now a report of this failed test
```
Running Highest
Tests run: 1, Failures: 1, Not executed: 0, Skipped: 0 <<< FAILURE! - in

Should calculate highest of last three bars
That is not the highest price of the last three bars ==> expected: na, but was <34638.5>
```

<a id="until-bar"></a>
### Test if series-based Functions don't deliver a result to early
Many functions require a certain number of bars to produce a meaningful result. For example, a 21-period moving average shouldn't provide any value before the 21st bar.

Reexamining the highest function from our earlier discussion, we've opted to adjust its operation. We now want this function to only return a meaningful result when an adequate number of bars are available for its computation; if not, it should revert to na. To affirm that it behaves as expected, we'll introduce a test deploying the `untilBar()` method. The number supplied to this method marks a clear limit, designating the bar up to which the test is conducted, where the limit is exclusive. Specifically for our test, it will be applied on the initial two bars but excluded from the third bar and any following ones.
```
var highestNaOnFirstBarsTest = highestUnit.createSeriesTest("Should return na on first bars").untilBar(3)
highestNaOnFirstBarsTest.assertNa(highest(3))
```
When we inspect our test feedback, it highlights that our highest function starts yielding results prematurely.
```
Running Highest
Tests run: 2, Failures: 2, Not executed: 0, Skipped: 0 <<< FAILURE! - in

Should calculate highest of last three bars
That is not the highest price of the last three bars ==> expected: na, but was <34638.5>

Should return na on first bars
expected: na, but was <34638.5>
```
We aim for it to provide a valid 'highest' output only when the defined lookback period is genuinely available. Of course this isn't happening currently, but as soon as we are going to fix that behavior, the test will tell us if we were successfull.

<a id="since-bar"></a>
### Starting series-based Tests after a defined bar threshold
Our test still reports the failure `Should calculate highest of last three bars`. This arises because we're employing `math.max()` to compute the expected value. This function behaves precisely how we ultimately want our custom function to act. Since we have introduced the Test that verifies if the function return `na` on the first bars, we now can restrict the verification for computed results on the after. 

To address this, we need to instruct our test to initiate only after a certain number of bars have passed by calling the `sinceBars()` method. Specifically, we tell the test to start assertions on the third bar. That's when we can determine a meaningful maximum across a 3-bar span.
```
var highestOfLastThreeTest = highestUnit.createSeriesTest("Should calculate highest of last three bars").sinceBar(3)
```
After this modification, this failing test no longer surfaces in our report.
```
Running Highest
Tests run: 2, Failures: 1, Not executed: 0, Skipped: 0 <<< FAILURE! - in

Should return na on first bars
expected: na, but was <34638.5>
```
This result indicates that our function starts computing correctly after the configured bar threshold. Only the bug that it returns an value on too early bar sohuld be fixed.


<a id="on-bar"></a>
### Execute a series-based Test only on a specific bar
Both `sinceBar()` and `untilBar()` can be applied to a single Test. If the same bar number is passed to both, the series-based Test will execute only on that specific bar. This is useful when you want to validate a Unit's behavior under a specific scenario, particularly if the series values are being manually configured from bar to bar. For such scenarios, you can use the `onBar()` method for simplicity.

Another approach is to trigger the assert function specifically when `bar_index` matches a designated value, for instance, `bar_index == 3`. This can be essential, depending on the function being tested. It's worth noting that conditions within assertions are always evaluated, regardless of any `when()` conditions or `sinceBar()` and `untilBar()` configurations. It is not possible to control if a test condition is computed or not and all assertion conditions are assessed on every bar. This method can provide stability in certain scenarios, though such advanced test setups are beyond the scope of this guide.

<a id="multi-assert"></a>
### Multiple Assertions within one single Test
Frequently, functions possess multiple characteristics that need to be confirmed. With PineUnit, you can incorporate several assert functions into a single test, facilitating comprehensive validation. For the test to succeed, all the included assertions must individually pass.

Consider a function that calculates the average price of a bar, as well as the deviation of this acerage price from its opening price:
```
averagePrice(int maLength) =>
    avg = (open + high + low) / 4
    avgDiff = avg - close
    [avg, avgDiff]
```
For this function, we need to validate both the computed average price and its difference from the opening price. This is achieved by setting up two assertions on a single Test:
```
[actualAverage, actualDiff] = averagePrice(21)
var averageUnit = testSession.createUnit("Average Price")
var averagePriceTest = averageUnit.createSeriesTest("Should calculate")
averagePriceTest.assertEquals(math.avg(open, high, low, close), actualAverage, "Wrong Average")
averagePriceTest.assertEquals(math.avg(open, high, low, close) - close, actualDiff, "Wrong Diff")
```
If any assertion fails, the test will report a failure, listing all failing assertions. In this example both assertions have failed and therefore both failures are reported:
```
Running Average Price
Tests run: 1, Failures: 1, Not executed: 0, Skipped: 0 <<< FAILURE! - in

Should calculate
Wrong Average ==> expected: <34631.25>, but was <25971.625>
Wrong Diff ==> expected: <-7.25>, but was <-8666.875>
```

<a id="when"></a>
### Define "When" Conditions for Tests
Consider a function momentum() that calculates the direction and distance of the current candle:
```
momentum() =>
    direction = close > open ? "Up" : close < open ? "Down" : "Side"
    diff = close - open
    [direction, diff]
```
First, we set up this function as a Unit and pre-compute its result:
```
var momentumUnit = testSession.createUnit("Momentum")
[direction, diff] = momentum()
```
Next, we'll set up a couple of series based tests to validate the behavior of this unit across different price movement scenarios: rising, falling, or stagnant prices. We tell the Test in which scenario it should be executed by calling the `when()` function. The key feature here is that each test's assertions only activate when a specific scenario (like `close > open` for rising candles) occurs:
```
var momentumUpTest = momentumUnit.createSeriesTest("Up")
momentumUpTest.when(close > open)
momentumUpTest.assertEquals("Up", direction, "Wrong direction")
momentumUpTest.assertEquals(close - open, diff, "Wrong distance")

var momentumDownTest = momentumUnit.createSeriesTest("Down")
momentumDownTest.when(close < open)
momentumDownTest.assertEquals("Down", direction, "Wrong direction")
momentumDownTest.assertEquals(open - close, diff, "Wrong distance")

var momentumSideTest = momentumUnit.createSeriesTest("Side")
momentumSideTest.when(close == open)
momentumSideTest.assertEquals("Side", direction, "Wrong direction")
momentumSideTest.assertEquals(0, diff, "Wrong distance")
```
Make sure to not call it fluently, as we've did e.g. with the `sinceBar()` function. The Test must decide on each bar if the scenario is met and this would not happen if the call is appended whithin a call on a `var` declaration. 

`momentumDownTest.assertEquals(open - close, diff, "Wrong distance")` of the `Down` Test fails because the function does not compute the absolute distance.

`momentumUpTest.assertEquals(close - open, diff, "Wrong distance")` of the `Up` Test however, the same distance calculation does not fail when prices rise, as its specific condition (close > open) isn't met on such a rising bar. The error within the partially correct calculation `close - open` does not lead a failure in that case.
```
Running Momentum
Tests run: 3, Failures: 1, Not executed: 0, Skipped: 0 <<< FAILURE! - in

Down
Wrong distance ==> expected: <4.5>, but was <-4.5>
```

<a id="ignore"></a>
### Ignore Tests
You can choose to ignore specific tests using the `setIgnored(bool)` or `ignore()` methods. This ensures that the outcomes of these tests don't influence the overall test results. Both methods can be applied to Test, Unit, and Suite instances. Here are use cases in which this can be helpful:
- **Test Driven Development**: When tests are written first (following the TDD approach), a developer may only want to focus on those tests that are directly related to the code they're working on. Ignoring other tests can streamline this process.
- **Temporary Ignorance**: Sometimes, during refactoring or other development stages, a test might not be immediately relevant. However, instead of deleting it, marking it as ignored ensures it's retained for future use.

Here are examples how a Test, a Unit or a whole Suite can be ignored:
```
var ignoredTest = testSession.createSimpleTest("Ignored Test").ignore()
ignoredTest.assertTrue(true)

var ignoredUnit = testSession.createUnit("Ignored Unit").ignore()
var unignoredTestInIgnoredUnit = ignoredUnit.createSimpleTest("Unignored")
unignoredTestInIgnoredUnit.assertTrue(true)

var ignoredSuite = testSession.createSuite("Ignored Suite").ignore()
var unignoredUnitInIgnoredSuite = ignoredSuite.createUnit("Unignored Unit")
var unignoredTestInIgnoredSuite = unignoredUnitInIgnoredSuite.createSimpleTest("Unignored")
unignoredTestInIgnoredSuite.assertTrue(true)
```
Any tests which are ignored will be labeled as Skipped in the test results.
```
Running Ignored Unit
Tests run: 0, Failures: 0, Not executed: 0, Skipped: 1
```
As we've seen, Units or Suites can be ignored completeley. Beside that a unignored Units can contain ignored Tests and unignored Suites can contain ignored Units or Tests. A Unit or Suite is deemed successful if it comprises solely of successful tests or ignored tests. But, if every test within a Unit or Suite is ignored, the entire Unit or Suite is designated as "Skipped."

<a id="ignore-unexecuted"</a>
#### Difference between ignored and unexecuted tests
When a test is defined with a `When` condition, there may be instances where it's never executed. While this isn't necessarily a test failure, it shouldn't be interpreted as a test success either. Tests that haven't run due to their specified condition not being met will be explicitly indicated in the test results:
```
Tests run: 0, TestFailures: 0, Not executed: 1, Skipped: 0
My Unexecuted Test
Not executed
```
It's crucial to note that an ignored test isn't labeled as unexecuted. To illustrate, consider two tests: one is set to be ignored, and the other is set with a When condition that never materializes:
```
var ignoreAndUnexecuteUnit = testSession.createUnit("IgnoreAndUnexecuteUnit")

var myIgnoredTest = ignoreAndUnexecuteUnit.createSeriesTest("My Ignored Test")
myIgnoredTest.ignore()
myIgnoredTest.assertTrue(false)

var myUnexecutedTest = ignoreAndUnexecuteUnit.createSeriesTest("My Unexecuted Test")
myUnexecutedTest.when(false)
myUnexecutedTest.assertTrue(false)
```
The resulting output would be:
```
Running IgnoreAndUnexecuteUnit
Tests run: 0, Failures: 0, Not executed: 1, Skipped: 1

My Unexecuted Test < NOT EXECUTED!
```
For Units or Suites, the labeling depends on the composition of their tests:

- If they only contain unexecuted tests, alongside either successful or ignored tests, they are labeled as "unexecuted".
- If they only have successful and ignored tests, they are labeled as "successful".

This highlights that the presence of solely unexecuted tests gives a different outcome from a mix of successful and ignored tests.

<a id="test-report"</a>
### Show Test Results
At the end of your script create the test report.
```
testSession.report()
```
Here is the test report of the examples:
```
--------------------------------------------------------------
T E S T S
--------------------------------------------------------------
Running Default Unit
Tests run: 2, Failures: 1, Not executed: 0, Skipped: 1 <<< FAILURE! - in

It's allways Bull Market
Uhoh... it's not always bullish ==> expected: <true>, but was <false>

Running Average Price
Tests run: 1, Failures: 1, Not executed: 0, Skipped: 0 <<< FAILURE! - in

Should calculate
Wrong Average ==> expected: <34631.25>, but was <25971.625>
Wrong Diff ==> expected: <-7.25>, but was <-8666.875>

Running Highest
Tests run: 2, Failures: 1, Not executed: 0, Skipped: 0 <<< FAILURE! - in

Should return na on first bars
expected: na, but was <34638.5>

Running IgnoreAndUnexecuteUnit
Tests run: 0, Failures: 0, Not executed: 1, Skipped: 1

My Unexecuted Test < NOT EXECUTED!

Running Ignored Unit
Tests run: 0, Failures: 0, Not executed: 0, Skipped: 1
Running Momentum
Tests run: 3, Failures: 1, Not executed: 0, Skipped: 0 <<< FAILURE! - in

Down
Wrong distance ==> expected: <4.5>, but was <-4.5>

Running Universe Unit
Tests run: 1, Failures: 0, Not executed: 0, Skipped: 0

Results :
Tests run: 9, Failures: 4, Not executed: 1, Skipped: 3

--------------------------------------------------------------
T E S T S Ignored Suite
--------------------------------------------------------------
Running Unignored Unit
Tests run: 0, Failures: 0, Not executed: 0, Skipped: 1

Results :
Tests run: 0, Failures: 0, Not executed: 0, Skipped: 1

--------------------------------------------------------------
T E S T S Pi (We love ya!)
--------------------------------------------------------------
Running Pi Unit
Tests run: 1, Failures: 0, Not executed: 0, Skipped: 0

Results :
Tests run: 1, Failures: 0, Not executed: 0, Skipped: 0

==============================================================
O V E R A L L
==============================================================
Tests run: 10, Failures: 4, Not executed: 1, Skipped: 4
```

<a id="performance"></a>
## Performance
Frankly, PineUnit can drive the PineScript compiler and runtime a lil bit crazy, depending on the amount of tests. If you experience performance issues you can improve the overall performance a little bit, yeah... just a bit 😀

Execute your session setup and the calls on **simple** Tests only on the first bar. Keep in mind that the `var` declarations must not be in such a conditional section.

```
if barstate.isfirst
    testSession.setMaxFailingBars(1)
    displaySettings.setShow(false)

// Test setup ...
var addTest = testSession.createSimpleTest("add()")
// ...

if barstate.isfirst
    addTest.assertEquals(myAdd(2,3), 5)
```
It can also be helpful to restrict series-based Tests to be actually executed only on a given amount of bars:
```
// Test setup ...
var calculateOnBarTest = testSession.createSeriesTest("cob()")
// ...

if bar_index < 100
    calculateOnBarTest.assertEquals(cob(), <expected>)
```
Deeper discussion: We cannot provide lambdas in Pine Script, so the test values are calculated always, even if the test is "executed" or not, e.g. the results of simple Tests are only verified on the first bar, but unfrotunately our input is calculated always on each bar.

<a id="contribution"></a>
## Contribution
We welcome contributions to the PineUnit project! Whether it's reporting bugs, proposing enhancements, or contributing code, your input is invaluable.


1. **Fork & Clone**:Fork the PineUnit repository, clone it to your local machine and set up the upstream remote:
```
git clone https://github.com/YOUR-USERNAME/PineUnit.git
cd PineUnit
git remote add upstream https://github.com/Guardian667/PineUnit.git
```
2. **Make Changes**: Create a new branch based on master, make your changes, commit and push them to your fork.
3. **Pull Request**: Submit a pull request from your fork's branch to the develop branch of the PineUnit repository. Make sure to give a detailed explanation of your changes.
4. **Review & Merge**: Your changes will be reviewed. Once they are approved, they will be merged into the project.

<a id="license"></a>
## License
This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0

@ Guardian667

<a id="note"></a>
## A Personal Note
As a software developer experienced in OO-based languages, diving into Pine Script is a unique journey. While many aspects of it are smooth and efficient, there are also notable gaps, particularly in the realm of testing. We've all been there: using `plotchar()` for debugging, trying to pinpoint those elusive issues in our scripts. I've come to appreciate the value of writing tests, which often obviates the need for such debugging. My hope is that this Testing Framework serves you well and saves you a significant amount of time, more that I invested into developing this "Baby."