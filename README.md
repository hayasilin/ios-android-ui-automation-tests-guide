# iOS and Android UI Automation Tests Guide

[Here is Traditional Chinese version 繁體中文版本](https://github.com/hayasilin/ios-android-ui-automation-tests-guide/blob/master/README_zh-hant.md).

This is a development guide of how to develop reliable and stable UI automation tests for iOS and Android apps that I conclude from my experience. I welcome your feedback in [issues](https://github.com/hayasilin/ios-android-ui-automation-tests-guide/issues) and [pull requests](https://github.com/hayasilin/ios-android-ui-automation-tests-guide/pulls).

## Introduction

Before you start reading this guide. I assume you have basic knowledge of UI automation tests for iOS or Android apps and you are using iOS and Android native test frameworks (XCUITest / Espresso). In addition, you have experience in implementing UI automation tests into your CI/CD automation pipeline.

Here are some of the documents from Apple and Google's native test frameworks. If something isn't mentioned here, it's probably covered in one of these:
- iOS
  - [About Testing with Xcode](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/testing_with_xcode/chapters/01-introduction.html)
    - [Automating the Test Process](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/testing_with_xcode/chapters/08-automation.html)
  - [User Interface Tests](https://developer.apple.com/documentation/xctest/user_interface_tests)
  - [Recording UI Tests](https://developer.apple.com/library/archive/documentation/ToolsLanguages/Conceptual/Xcode_Overview/RecordingUITests.html#//apple_ref/doc/uid/TP40010215-CH75-SW1)
  - **WWDC**
    - 2019
      - [Testing in Xcode](https://developer.apple.com/videos/play/wwdc2019/413/)
    - 2018
      - [What's New in Testing](https://developer.apple.com/videos/play/wwdc2018/403)
    - 2017
      - [What's New in Testing](https://developer.apple.com/videos/play/wwdc2017/409)
    - 2016
      - [Advanced Testing and Continuous Integration](https://developer.apple.com/videos/play/wwdc2016/409)
    - 2015
      - [UI Testing in Xcode](https://developer.apple.com/videos/play/wwdc2015/406)
      - [Continuous Integration and Code Coverage in Xcode](https://developer.apple.com/videos/play/wwdc2015/410/)

- Android
  - [Automate user interface tests](https://developer.android.com/training/testing/ui-testing)
  - Espresso
    - [Test UI for a single app](https://developer.android.com/training/testing/ui-testing/espresso-testing)
    - [Espresso](https://developer.android.com/training/testing/espresso)
  - UI Automator
    - [Test UI for multiple apps](https://developer.android.com/training/testing/ui-testing/uiautomator-testing.html)
    - [UI Automator](https://developer.android.com/training/testing/ui-automator)
  - [Create UI tests with Espresso Test Recorder](https://developer.android.com/studio/test/espresso-test-recorder)
  - [Android testing example](https://github.com/android/testing-samples)
  - [Test from the command line](https://developer.android.com/studio/test/command-line)
  - [UI/Application Exerciser Monkey](https://developer.android.com/studio/test/monkey)
  - Google Test Automation Conference
    - 2014 [Move Fast & Don't Break Things](https://docs.google.com/presentation/d/15gNk21rjer3xo-b1ZqyQVGebOp_aPvHU3YH7YnOMxtE/edit#slide=id.g3f5c82004_8611)

## Table of Contents
- [Before developing UI automation tests](#before-developing-ui-automation-tests)
  - [UI automation tests is easy to develop but expensive to maintain](#ui-automation-tests-is-easy-to-develop-but-expensive-to-maintain)
  - [What are flaky tests](#what-are-flaky-tests)
  - [Why UI automation tests make flaky tests](#why-ui-automation-tests-make-flaky-tests)
- [Best practice to make UI automation tests reliable and stable](#best-practice-to-make-ui-automation-tests-reliable-and-stable)
  - [Follow testing pyramid](#follow-testing-pyramid)
  - [The mind set of developing UI automation tests](#the-mind-set-of-developing-UI-automation-tests)
  - [Choose tests cases for UI automation tests](#choose-tests-cases-for-ui-automation-tests)
  - [Top 10 practical ways to avoid flaky tests](#top-10-practical-ways-to-avoid-flaky-tests)
- [UI tests code convention](#ui-tests-code-convention)
  - [iOS](#ios)
  - [Android](#android)
- [Common questions](#common-questions)
  - [Stable and Unstable UI automation tests results](#stable-and-unstable-ui-automation-tests-results)
- [Appendix](#appendix)
  - [iOS XCUITest issues](#ios-xcuitest-issues)
  - [Android Espresso issues](#android-espresso-issues)

## Before developing UI automation tests

### UI automation tests are easy to develop but expensive to maintain
- UI automation tests may seem to be amazing because it simulates actual user scenario and test our apps automatically, however, although it's definitely needed by team but it also comes with some costs.
- UI automation tests are expense to maintain because our apps will keep changing in agile development world and our test code need to be changed constantly as well. Don't forgot the maintenance cost.
- Originally UI automaiton testing are developer's tool to check that they don't break anything after they change code. Nowadays UI automation testing is widely used by QA testing as well to speed up testing by reducing time consuming manual testing. However, It's easily to think that we can automate everything with UI automation tests and get rid of all your manual testing without knowing its limitation and cost. The first and foremost thing you are going to deal with is the notorious **flaky tests**.
- Flaky tests will make unreliable and unstable UI automation tests, which is meaningless for QA testing.
  - Because the uncertainty of the automation test result makes QA and the team have low confidence on UI automation tests, in other words, they won't trust the result. Eventually your team still spend times to perform manual regression testing, which is time consuming and slow your team down.
  - As a result, no one wants to write or maintain UI automation code due to lack of attention of the team members.

### What are flaky tests
- Flaky tests are when you run your automation tests, some tests passes this time but fail next time even you didn't change any code. What's worse when you check the app by yourself, it works fine. It's unpredictable to know if the tests will pass or not.
- Flaky tests example is below, the Test A and Test C are flaky tests because pass and fail results take turns to show. On the contrary, Test B is not a flaky test because it keep passing. 
- In general, we will put UI automation tests into our CI/CD pipeline and trigger it automatically according to our setting. One of the most common tool we use is Jenkins. If your team have flaky tests, you can see the example below that your Jenkins result would be highly likely failed because Jenkins will mark the job failed even only 1 test case failed. Then, your team need to send one of the team member to check why it failed.
- Below example only has 3 tests cases. If your team has 100 more test cases and there are flaky tests. Imagine how many times your team need to check if it's really app's issue or flaky tests.

| Test Runs   | 1st     | 2nd      | 3rd      | 4th      | 5th     | 6th      | 7th      | 8th      | 9th      | 10th     | Next run?              |
| ----------- | ------- |--------- | ---------| -------- | ------- | -------- | -------- | -------- | -------- | -------- | ---------------------- |
| Test Case A | Success | **Fail** | Success  | Success  | Success | **Fail** | Success  | Success  | **Fail** | **Fail** | **?**                  | 
| Test Case B | Success | Success  | Success  | Success  | Success | Success  | Success  | Success  | Success  | Success  | **?**                  | 
| Test Case C | Success | **Fail** | **Fail** | **Fail** | Success | **Fail** | **Fail** | **Fail** | Success  | **Fail** | **?**                  | 
| ...                                                                                                                                              |
| Jenkins     | Success | **Fail** | **Fail** | **Fail** | Success | **Fail** | **Fail** | **Fail** | **Fail** | **Fail** | **Highly likely Fail** |


- Why flaky tests are bad:
  - Make the team has no confidence on UI automation tests.
  - It's a blocker for making CI/CD pipeline if you design a series of actions depend on the result of UI auotmation tests.
  - Can't achieve the goal of fast app delivery, let alone maintain or improve app's quality by providing quick feedback during development

- Google's experience on handling flaky tests
  - 2015 [Just Say No to More End-to-End Tests](https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html)
  - 2016 [Flaky Tests at Google and How We Mitigate Them](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html)
  - 2017 [Where do our flaky tests come from](https://testing.googleblog.com/2017/04/where-do-our-flaky-tests-come-from.html)
  
### Why UI automation tests make flaky tests

- Since UI automation testing is end-to-end testing that integrates the features of UI Display, API handling, cross-app behavior, and user status change, if one of the part fails then the whole test fails too. There are too many variables when running end-to-end testing and it makes UI automation has vulnerability nature and easily cause flaky tests.
- Didn't do well on **Test Isolation**.
- Native test frameworks have its limitation, sometimes your tests fail becasue of its limitation instead of your test code.

**Internal flaky factors:**
- UI or user scenario may change in every release.
- Team don't have a stable test environment or test data.
- Element's creation has strong dependency on different factors, such as framework version or API response. The creation time can be short or can be long. Don't use just time to wait (or use sleep), you should use wait until element exists before perfoming action to the element.
- Dynamic UI element design: app's UI will change constantly according to server's config.
- Poorly written test code.

**External flaky facotors:**
- Known issues of native test frameworks (XCUITests / Espresso).
  - [Appendix](#appendix).
  - After Updating new version of native test frameworks, it may cause some issues to your current tests due to its new behaviors.
- Devices, Emulators, and Simulators dependency (Different screen sizes, different OS versions, differnt browser behavior, ect.).
- Cross Apps testing instability.
- Prallel testing instability.
- IDE run and command line run may have different behaviors.
- and **more...**.

## Best practice to make UI automation tests reliable and stable

### Follow testing pyramid

According to Google's [Just Say No to More End-to-End Tests](https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html), testing pyramid is try to imagine a pyramid, the bulk of your tests are unit tests at the bottom of the pyramid. As you move up the pyramid, your tests gets larger, but at the same time the number of tests (the width of your pyramid) gets smaller.

Google often suggests a 70/20/10 split: 70% unit tests, 20% integration tests, and 10% end-to-end tests. The exact mix will be different for each team, but in general, it should retain that pyramid shape. Try to avoid anti-pattern like the team relies primarily on end-to-end tests, using few integration tests and even fewer unit tests. 

Just like a regular pyramid tends to be the most stable structure in real life, the testing pyramid also tends to be the most stable testing strategy.

If you want to know more about iOS unit testing, you can check my [iOS Unit Tests Guide](https://github.com/hayasilin/unit-tests-ios-guide).

### The mind set of developing UI automation tests
- Aware of the maintenance cost, device dependency, test environment and data instability, and limitation of UI automation tests before start developing it.
- The testing level for UI automation testing is a **Smoke Testing**, which it to go over the happy path of your app. UI automation should not do more.
- If there are automation tests is flaky, remove it right away with no mercy.
- Your team won't have time to keep checking or fixing flaky tests. Only a stale UI automation can actually help your team to achieve goals below:
  - Fasten app delivery.
  - Provide quick feedback to maintain and improve app's quality.

## Choose tests cases for UI automation tests

**Pros**
- Mature functionalities, which is stable and won't be changed in the short term.
- Most common user scenarios / Happy path.
- Suitable to automate.

**Cons**
- Features are keep changing because of business plan.
- Hard to automate or beyond test framework's limitation.
- It's costly to make it automation, for example, app's push notification.

- UI automation tests can not make all test cases automation, choose the right test cases and leave others remain in manual testing.
  
### Top 10 practical ways to avoid flaky tests
1. Test isolation: test should not depend on other tests. If the test case changes status or manipulate database, please clean it up before or after the test case.
2. The UI you see now may not to be the only UI. UI may change according to user's status. Consider every factor that could cause flaky and make your tests robust to deal with all known UI changes.
3. Need to run all the tests around 10 times after complete new test code to see it won't become flaky tests.
4. After develop test code, run your new tests on the devices that your team used for UI automation tests to check there are no device dependency issues.
5. Less navigation of screen is better. The automation test flow should be as simple as possible. One of the best ways is to set up the test and prepare suitable data and environment before the test start. Both XCUITest and Espresso provide similiar to do it.
6. Choose the mature functionalities to develop UI automation carefully.
7. Don't over design your test code or make it too complex, keep it simple and use design pattern like page object pattern to make it easy to maintain.
8. Stability is more important than completion. Don't try to cover as many as tests you can, just automate the tests that is easy to automate, remember UI automation testing is just a smoke testing. Unstable and unreliable UI automation tests are meaningless to your team. Because it's waste of time to investigate flaky tests.
9. Pick just one UI element to assert in every test case. The name of UI testing easily confuse us and make we think that we need to test the every UI element on the screen. However, it will only create flaky tests. In practical, please think UI testing as a tool to test user behavior or user scenario instead of testing UI element's position or layout.
10. Don't use too many iOS or Android devices to run parallel testing. Test result from several iOS or Android devices should provide your team enough confidence to decide release your app or not.

## UI tests code convention

### iOS

**Page object class**

- Class name:
  - **Page** suffix

```swift
class MainPage {
    var app: XCUIApplication

    init(app: XCUIApplication) {
        self.app = app
    }

    lazy var searchButton: XCUIElement = {
        app.buttons["identifier"]
    }()
}
```

**Test suite class**

- Class name:
  - **Tests** suffix
- Function name:
  - **test** prefix
  - test action description

```swift
class SearchTests: XCTestCase {
    let app = XCUIApplication()

    var mainPage: MainPage?
    var searchPage: SearchPage?

    override func setUp() {
        super.setUp()
        continueAfterFailure = false

        mainPage = MainPage(app: app)
        searchPage = SearchPage(app: app)

        app.launch()
    }

    func testSearchResultsDisplay() {
        // Perfom testing with elements in searchPage and mainPage
    }
```

**XCTWaiter**

```swift
extension XCTestCase {
    enum UIStatus: String {
        case exist = "exists == true"
        case notExist = "exists == false"
        case selected = "selected == true"
        case notSelected = "selected == false"
        case hittable = "hittable == true"
        case unhittable = "hittable == false"
    }

    func tapWithExpect(element: XCUIElement?, status: UIStatus = .hittable, timeout: TimeInterval = UITestConfig.timeout) -> Bool {
        let expectation = XCTNSPredicateExpectation(predicate: NSPredicate(format: status.rawValue), object: element)
        let result = XCTWaiter.wait(for: [expectation], timeout: timeout)
        if result == .completed {
            element?.tap()
            return true
        } else {
            return false
        }
    }

    func expect(element: XCUIElement?, status: UIStatus = .exist, timeout: TimeInterval = UITestConfig.timeout) -> Bool {
        let expectation = XCTNSPredicateExpectation(predicate: NSPredicate(format: status.rawValue), object: element)
        let result = XCTWaiter.wait(for: [expectation], timeout: timeout)
        if result == .timedOut {
            return false
        } else {
            return true
        }
    }
```

### Code conventions

**Tap a UI button**

**Do**
```swift
tapWithExpect(element: mainPage?.searchButton)
```

**Don't**
```swift
searchPage?.searchButton.tap()

// ...Should use XCTWaiter to wait UI button exist before perfom action.
```

---

**Assert element exist**

**Do**
```swift
XCTAssert(app.staticTexts["identifier"].waitForExistence(timeout: timeout))
```

**Don't**
```swift
XCTAssert(app.staticTexts["identifier"])

// ...Should use wait for existence with timeout.
```

---

**Go to certain page and perform testing**

**Do**
```swift
app.launchArguments += ["setNavigateToThirdPage"]

app.launch()
```

**Don't**
```swift
mainPage?.navigateToFirstPageButton.tap()

firstPage?.navigateToSecondPageButton.tap()

secondPage?.navigateToThirdPageButton.tap()

// ...Too many page navigations will create flaky.
```

---

**Test delete data action**

**Do**
```swift
app.launchArguments += ["setData"]

app.launch()

// Perfom delete action then assert.
```

**Don't**
```swift
thePage?.addDataButton.tap()

// ...Don't use UI to perform series of actions for preparing data or state.
```

---

**Test empty data page display**

**Do**
```swift
app.launchArguments += ["setClearDatabase"]

app.launch()

// Assert empty data page display
```

**Don't**
```swift
thePage?.deleteAllDataButton.tap()

// ...Don't use UI to perform series of actions for preparing data or state.
```

---

**Assertion**

**Do**
```swift
XCTAssert(app.staticTexts["identifier"].waitForExistence(timeout: timeout))

// Just pick one UI element to assert in one test case, that will be enough.
```

**Don't**
```swift
XCTAssert(app.staticTexts["identifier"].waitForExistence(timeout: timeout))
XCTAssert(app.buttons["identifier"].waitForExistence(timeout: timeout))
XCTAssert(app.textFields["identifier"].waitForExistence(timeout: timeout))

// ...Don't assert too many UI elements in one test case.
```

### Android

**Page object class**

- Class name:
  - **Page** suffix

```kotlin
class MainPage {
    val searchButton: ViewInteraction = onView(withId(R.id.identifier))
}
```

**Test suite class**

- Class name:
  - **Tests** suffix
- Function name:
  - **test** prefix
  - test action description

```kotlin
@RunWith(AndroidJUnit4::class)
@MediumTest
class SearchTests {
    private val searchPage: SearchPage = SearchPage()
    private val mainPage: MainPage = MainPage()

    @get:Rule
    val activityRule = MainActivityTestRule(MainActivity::class.java)

    @Test
    fun testSearchResultsDisplay() {
        // Perfom testing with elements in searchPage and mainPage
    }
}
```

**Wait until element is displayed**

```kotlin
fun withIdDisplayed(id: Int): Matcher<View>? = allOf(withId(id), isDisplayed())
```

**Wait until element exists**

```kotlin
fun waitElement(resourceId: Int) {
    var device: UiDevice = UiDevice.getInstance(InstrumentationRegistry.getInstrumentation())
    var context: Context = ApplicationProvider.getApplicationContext()
    device.wait(Until.hasObject(By.res(context.resources.getResourceName(resourceId))), AndroidTestConfig.TIMEOUT)
}
```

**Wait until text exists**

```kotlin
fun waitTextElement(resourceId: Int) {
    var device: UiDevice = UiDevice.getInstance(InstrumentationRegistry.getInstrumentation())
    var context: Context = ApplicationProvider.getApplicationContext()
    device.wait(Until.hasObject(By.textContains(context.getString(resourceId))), AndroidTestConfig.TIMEOUT)
}
```

**Combination of wait until element exists then perform click action**

```kotlin
fun clickElement(resourceId: Int) {
    waitElement(resourceId)
    onView(withIdDisplayed(resourceId)).perform(click())
}
```

**Combination of wait until element exists then assert**

```kotlin
fun checkElementIsDisplayed(id: Int) {
    waitElement(id)
    onView(withIdDisplayed(id)).check(matches(isDisplayed()))
}
```

**Combination of wait until text exists then assert**

```kotlin
fun checkTextIsDisplayed(text: Int) {
    waitTextElement(text)
    onView(withText(text)).check(matches(isDisplayed()))
}
```

### Code conventions

**Tap a UI button**

**Do**
```kotlin
clickElement(mainPage.searchButton)
```

**Don't**
```kotlin
mainPage.searchButton.perform(click())

// ...Should use wait until function to wait button exist before perfom action.
```

---

**Assert element exist**

**Do**
```kotlin
waitElement(R.id.identifier)
onView(withIdDisplayed(R.id.identifier)).check(matches(isDisplayed()))

// You can make a function to include code above
```

**Don't**
```kotlin
onView(withId(identifier)).check(matches(isDisplayed()))
```

---

**Go to certain page and perform testing**

**Do**
```kotlin
private val onboardingPage: OnboardingPage = OnboardingPage()

@get:Rule
val activity = ActivityTestRule<OnboardingActivity>(OnboardingActivity::class.java)

@Test
fun testOnboardingPageSwipeLeftAndRight() {
  // Perform testing in onboarding page
}
```

**Don't**
```kotlin
mainPage.navigateToFirstPageButton.perform(click())

firstPage.navigateToSecondPageButton.perform(click())

secondPage.navigateToOnboardingPageButton.perform(click())

// ...Too many page navigations will create flaky.
```

---

**Test delete data action**

**Do**
```kotlin
val database = Room.databaseBuilder(ApplicationProvider.getApplicationContext(), Database::class.java, "database")
                   .allowMainThreadQueries()
                   .build()
database.dataDao().insert(data)

// Use in memory databasee to prepare data before test start
```

**Don't**
```kotlin
thePage.addDataButton.perform(click())

// ...Don't use UI to perform series of actions for preparing data or state.
```

---

**Test empty data page display**

**Do**
```kotlin
val database = Room.databaseBuilder(ApplicationProvider.getApplicationContext(), Database::class.java, "database")
                   .allowMainThreadQueries()
                   .build()
database.dataDao().removeAll()

// ...Don't use UI to perform series of actions for preparing data or state.
```

**Don't**
```kotlin
thePage.deleteDataButton.perform(click())

// ...Use UI to perform series of actions to clean data will cause falky easily.
```

---

**Assertion**

**Do**
```kotlin
checkElementIsDisplayed(R.id.identifier)

// Just pick one UI element to assert in one test case, that will be enough.
```

**Don't**
```kotlin
checkElementIsDisplayed(R.id.identifier)
checkElementIsDisplayed(R.id.loginButton)
checkElementIsDisplayed(R.id.listView)

// ...Don't assert too many UI elements in one test case.
```

## Common questions
1. What frequency should we trigger our UI automation tests in CI/CD pipeline to test our app is functioning properly?
  - My recommendation is **once a day** at early morning or late night.
    - Because usually your team, including app developers or server side developers, are modifying code or environemnt's config for new features during working hours. So it's better to avoid running UI automation tests during working hour to avoid flaky factors.
    - In theory, we may imagine that we can run UI tests when developers send their pull request, just like unit tests. After all tests are passed, the developer can merge the code into branch. However, in reality it's infeasible. Because when you have more UI test cases, you need more time to run. As you already know, unit tests run fast but UI tests take longer time to run, it will make developers and code need to wait long time before they can merge their code, which cause efficiency issue. Furthermore, as I mentioned above, UI tests have fragile nature, it would fail for various reasons. If your UI tests keep failing, it will block your CI/CD pipeline. Hence practically I suggest we don't need to make UI tests into pull request process.

2. What tools should we use? native test frameworks (XCUITest / Espresso) or third-party cross platform testing tools?
  - I recommend to use **native test frameworks**.
    - Because most of the time your test cases need to test different situations. By using native test frameworks, you can modify app's status or database to set up the test data or test environment you need, which reduce the flaky factors.

3. How many devices used for running parallel testing?
  - I recommend only need to use **2-3** iOS devices for iOS parallel testing and use **2-3** Android devices for Android parallel testing.
    - Because different devices or OS version may have differnet behaviors, more devices mean higher flaky possibility. So the test result from 2-3 devices should be able to give your team enough information to decide whether to release your app or not.

4. How reliable and stable can be achieved for UI automation tests
  - It depends on your test cases number. 
    - Flaky tests happen easily when you have more than 30 test cases while running parallel testing. After following this guide, I can achieve around 14 Jenkins runs successfully in a row in 15 Jenkins runs even we keep adding new test code everyday (Both iOS and Android apps). The flaky rate in total test cases run in the 15 Jenkins is around 0.7% (According to Google's [Flaky Tests at Google and How We Mitigate Them](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html), Google has 1.5% Flaky rate). To me it's not perfect but it can provide enough information for my team to decide whether the build is release-ready or not. In addition, my team don't need someone to keep dealing with flaky tests. If Jenkins result is failure, something must goes wrong with our app instead of flaky tests.
    - A relative reliable and stable test result can shorten QA manual testing time. Our QA team only needs 1 day to do manual testing. Becase UI automation tests can provide quick feedback if developers break something.
  - A common stable and unstable UI automation tests results in Jenkins are listed below.

### Stable and Unstable UI automation tests results

| Results | Stable | Unstable |
| ------- | -------|----------|
| Jenkins | <img width=300 src="https://github.com/hayasilin/ios-android-ui-automation-tests-guide/blob/master/resources/jenkins_result_stable.png"> | <img width=300 src="https://github.com/hayasilin/ios-android-ui-automation-tests-guide/blob/master/resources/jenkins_result_unstable.png"> |

## Appendix

### iOS XCUITest issues
- [Error getting main window kAXErrorServerNotFound](https://stackoverflow.com/questions/44677342/failed-to-get-main-window-after-30-retries-kaxerrorservernotfound)
  - One way to make it more stable is to delete app / restart iOS devices, which can help to reduce its frequency.
- [Failed to scroll to visible (by AX action) Button](https://stackoverflow.com/questions/33422681/xcode-ui-test-ui-testing-failure-failed-to-scroll-to-visible-by-ax-action)
  - Even the element is shown on the screen, due to it may have hidden before, sometimes can't find the element properly.
  - Don't use XCUIElement.ishittable property to check the element can be tap or not on multi-layer views.
- Even there is no cell showing on UITableView in screen, but when using XCUIApplication().tables.firstMatch.cells.count, the return number is not 0 or bigger than 0, which cause flaky if we want to check the UI has no items.
  - Don't check right away when entering the page, wait a while.
  - Use several checks to handle the error, but sometimes it still failed.
- When running xcodebuild's parallel automation testing, UI transition may be slow and test cases become flaky easily.
- Sometimes iOS device can't input text into textfield, it's just freezes.
  - Shut down and restart the iOS device.
- [Encountered a problem with the test runner after launch.(Underlying error: Lost connection to DTServiceHub)](https://stackoverflow.com/questions/39635861/unable-to-contact-local-dtservicehub-to-bless-simulator-connection)

### Android Espresso issues
- [Click action change to long click action](https://stackoverflow.com/questions/32330671/android-espresso-performs-longclick-instead-of-click/35254071)
- [Start to idling and could not end the test nor perform next test cases](https://stackoverflow.com/questions/21406246/google-espresso-java-lang-runtimeexception-could-not-launch-intent-intent-act)
  - Press system back button can solve the issue.
- During UI change with default animation, automation click too fast during animation so even the element is visible, but it's not clickable status.
  - Use wait until element shows or wait longer time.
- [androidx.test.espresso.PerformException: Error performing 'fast swipe' on view](https://stackoverflow.com/questions/44005338/android-espresso-performexception)
  - Even turn off all animation effect on that device the flaky still happen.
- Different Android emulators when open webview, their element description can be different (Nexus 7.0 vs. Pixel 2 8.0).
- [qemu2 cpu0 thread no response for 15009 ms](https://stackoverflow.com/questions/47956847/android-studio-emulator-error-detected-a-hanging-thread-qemu2-main-loop)
  - Go to AVD Manager and select "Quick boot" if it's set "Cold boot", or vice versa.
- Sometimes the app would crash because some instances are null when running automation happen in devices. It won't happen when doing manual testing.
  - No solution for now. Just rerun it then it won't happen hopefully.
- Espresso perform(swipeUp()) will become click action on recycle view on Samsung S9 (8.0) & Samsung J2 Prime (6.0.1) devices.
  - Use other element to swipe up or skip the action for the problem devices.
