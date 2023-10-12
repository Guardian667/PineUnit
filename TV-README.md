[b]PineUnit by Guardian667[/b]

A comprehensive testing framework for Pine Script on TradingView. Built with well-known testing paradigms like Assertions, Units and Suites. It offers the ability to log test results in TradingView's built-in Pine Protocol view, as well as displaying them in a compact table directly on your chart, ensuring your scripts are both robust and reliable.

Unit testing Pine Script indicators, libraries, and strategies becomes seamless, ensuring the precision and dependability of your TradingView scripts. Beyond standard function testing based on predefined input values, PineUnit supports series value testing. This means a test can run on every bar, taking into account its specific values. Moreover, you can specify the exact conditions under which a test should execute, allowing for series-based testing only on bars fitting a designated scenario.

[url=https://github.com/Guardian667/PineUnit]Detailed Guide & Source Code[/url]

[b]Quick Start[/b]
To get started swiftly with PineUnit, follow this minimalistic example.
[pine]
import Guardian667/PineUnit/1 as PineUnit

var testSession = PineUnit.createTestSession()
var trueTest = testSession.createSimpleTest("True is always True")
trueTest.assertTrue(true)

testSession.report()
[/pine]
After running your script, you'll notice a table on your chart displaying the test results. For a detailed log output, you can also utilize the Pine Protocol view in TradingView.
[pine]
--------------------------------------------------------------
T E S T S
--------------------------------------------------------------
Running Default Unit
Tests run: 1, Failures: 0, Not executed: 0, Skipped: 0
[/pine]
To further illustrate, let's introduce a test that's destined to fail:
[pine]
var bullTest = testSession.createSeriesTest("It's allways Bull Market")
bullTest.assertTrue(close > open, "Uhoh... it's not always bullish")
[/pine]
After executing, the test results will reflect this intentional discrepancy:
[pine]
--------------------------------------------------------------
T E S T S
--------------------------------------------------------------
Running Default Unit
Tests run: 2, Failures: 1, Not executed: 0, Skipped: 0 <<< FAILURE! - in

It's allways Bull Market
Uhoh... it's not always bullish ==> expected: <true>, but was <false>
[/pine]
This shows how PineUnit efficiently captures and reports discrepancies in test expectations.

It's important to recognise the difference between `createSimpleTest()` and `createSeriesTest()`. In contrast to a simple test, a series-based test is executed on each bar, making assertions on series values.

[b]License[/b]
This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0

@ Guardian667

[b]A Personal Note[/b]
As a software developer experienced in OO-based languages, diving into Pine Script is a unique journey. While many aspects of it are smooth and efficient, there are also notable gaps, particularly in the realm of testing. We've all been there: using `plotchar()` for debugging, trying to pinpoint those elusive issues in our scripts. I've come to appreciate the value of writing tests, which often obviates the need for such debugging. My hope is that this Testing Framework serves you well and saves you a significant amount of time, more that I invested into developing this "baby."