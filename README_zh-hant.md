# iOS 及 Android UI自動化測試規範

[Here is English version](https://github.com/hayasilin/ios-android-ui-automation-tests-guide/blob/master/README.md)

這是總結個人在開發可靠及穩定的UI自動化測試所得之規範及最佳實踐，歡迎同好的[反饋](https://github.com/hayasilin/ios-android-ui-automation-tests-guide/issues) 和 [Pull requests](https://github.com/hayasilin/ios-android-ui-automation-tests-guide/pulls)。

## 介紹

本規範適合在iOS及Android的UI自動化測試領域已經有些許經驗的工程師，使用iOS及Android原生的測試框架進行開發 (XCUITest / Espresso)，且將UI自動化測試整合至團隊的CI/CD流程中。

關於Apple跟Google的原生測試框架，如果這裡沒提到，那就在以下的文件裡：
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

## 目錄
- [開始開發UI自動化測試之前](#開始開發UI自動化測試之前)
  - [UI自動化測試開始很簡單但維護很昂貴](#UI自動化測試開始很簡單但維護很昂貴)
  - [什麼是Flaky tests](#什麼是Flaky-tests)
  - [為何UI自動化測試會有Flaky tests](#為何UI自動化測試會有Flaky-tests)
- [讓UI自動化測試可靠及穩定的最佳實踐](#讓UI自動化測試可靠及穩定的最佳實踐)
  - [遵從測試三角形](#遵從測試三角形)
  - [開發前的準備](#開發前的準備)
  - [什麼樣的測試適合UI自動化測試](#什麼樣的測試適合UI自動化測試)
  - [避免Flaky tests的10大實用方法](#避免Flaky-tests的10大實用方法)
- [UI測試的程式碼規範](#UI測試的程式碼規範)
  - [iOS](#ios)
  - [Android](#android)
- [常見問答](#常見問答)
  - [穩定及不穩定的UI自動化測試結果](#穩定及不穩定的UI自動化測試結果)
- [Appendix](#appendix)
  - [iOS XCUITest issues](#ios-xcuitest-issues)
  - [Android Espresso issues](#android-Espresso-issues)

## 開始開發UI自動化測試之前

### UI自動化測試開始很簡單但維護很昂貴
- UI自動化測試看起來很神奇，它模擬使用者的行為並自動測試你的App，我們需要它但實際上很昂貴。
- 因為App的UI是不斷變動的，所以測試碼也需要跟著變動，別忘了這些維護成本。
- 早期UI自動化測試是開發者用來驗證更改程式碼後，並沒造成損壞的開發者工具。不過近年來也成為QA團隊的測試工具，希望能用來減少很花時間的手動測試。但很容易會認為我們能自動化所有原本的手動測試，甚至可以達成捨棄手動測試的目標，當你開始這樣做卻忘了UI自動化測試的限制及維護成本，你首先會遇到惡名昭彰的**Flaky tests**。
- Flaky tests會導致不可靠也不穩定的UI自動化測試，造成UI自動化測試無法幫助QA測試：
  - 不穩定的結果只會讓QA及團隊對UI自動化測試失去信心，也不會相信測試後的報告，結果是你的團隊仍然會花時間去執行手動測試，這既花時間也讓你的團隊變慢。
  - 最後沒有人會繼續維護UI自動化測試，因為團隊再也沒人在意。

### 什麼是Flaky tests
- 當執行自動化測試時，有些測試項目這次會成功但下次會失敗，成功及失敗會反覆出現，即便你並沒有更動程式碼，而用人力親自去確認App其實沒問題。我們無法預測該測項下次會成功還是失敗。
- 下面是Flaky tests造成的例子。測項A跟測項C是Flaky tests因為它們成功及失敗的結果輪流出現，相對的測項B一直成功，是穩定的測項。
- 一般而言我們會把UI自動化測試放入CI/CD流程之中，並根據所設定的情況去自動啟動它。最常見的方式是與Jenkins整合。如果我們測項中有Flaky tests，那麼以下例子顯示每次Jenkins測試完畢後結果都很有可能是紅燈，因為哪怕只要是1個測項失敗，整個Jenkins Job結果就會顯示錯誤，結果是團隊必須要派人力去檢查為何測試失敗。
- 以下例子只有3個測項，如果你的團隊已經埋頭苦幹先寫好了100多個測項，想像一下如果測試沒寫好裡面有Flaky tests，你的團隊人員要花多少時間去確認是否真的是App的問題，還是Flaky tests造成的。

| Test Runs   | 1st     | 2nd      | 3rd      | 4th      | 5th     | 6th      | 7th      | 8th      | 9th      | 10th     | Next run?              |
| ----------- | ------- |--------- | ---------| -------- | ------- | -------- | -------- | -------- | -------- | -------- | ---------------------- |
| Test Case A | Success | **Fail** | Success  | Success  | Success | **Fail** | Success  | Success  | **Fail** | **Fail** | **?**                  | 
| Test Case B | Success | Success  | Success  | Success  | Success | Success  | Success  | Success  | Success  | Success  | **?**                  | 
| Test Case C | Success | **Fail** | **Fail** | **Fail** | Success | **Fail** | **Fail** | **Fail** | Success  | **Fail** | **?**                  | 
| ...                                                                                                                                              |
| Jenkins     | Success | **Fail** | **Fail** | **Fail** | Success | **Fail** | **Fail** | **Fail** | **Fail** | **Fail** | **Highly likely Fail** |

- 為何Flaky tests很糟糕:
  - 團隊會對UI自動化測試失去信心。
  - 如果你的CI/CD設計當UI自動化測試成功後將繼續其他動作，Flaky tests將停滯住CI/CD流程。
  - 無法達成App快速上線，更別提快速提供反饋讓團隊維持或提升App品質。

- Google處理Flaky tests的經驗
  - 2015 [Just Say No to More End-to-End Tests](https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html)
  - 2016 [Flaky Tests at Google and How We Mitigate Them](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html)
  - 2017 [Where do our flaky tests come from](https://testing.googleblog.com/2017/04/where-do-our-flaky-tests-come-from.html)
  
### 為何UI自動化測試會有Flaky tests

- UI自動化測試是測試整個App在end-to-end的串接是否運作正常，範圍包含了UI顯示，API資料解析及顯示，跨App行為操作，以及使用者的各種狀態改變等等。以上動作包含著無數種變化可能，只要有1個部分失敗，整個測試會失敗，此種特性也讓UI自動化測試本質上較為脆弱。
- 開發測試時，最重要的觀念是**測試孤立化**，如果沒做好則測試結果往往就會受到各種依賴的影響。
- 測試框架並不是萬能，它有其限制及本身的問題，在某些狀況下會造成測試失敗，而原因並不是你寫的程式碼。

**團隊內部溝通造成的Flaky因子:**
- UI及流程在每次改版都可能會改變。
- 不穩定的測試環境及測試資料。
- 元件的產生時間會根據框架版本及API回傳處理時間而變動，有時很快就產生但有時會花很長時間，不要使用等待(Sleep)，請使用等待元件顯示後才對元件執行動作的方法(Wait until)。
- 動態變化的UI元件: UI會根據伺服器端的設定而更改UI。
- 寫得不好的測試程式碼。

**外部因素造成的Flaky因子:**
- 原生測試框架已知的問題(XCUITests / Espresso)。
  - [Appendix](#appendix)。
  - 更新測試框架後，測試框架新的功能或判斷方式有時也會造成現有測項不可預期的問題。
- 各種實機裝置及模擬器行為的不同（如不同螢幕大小，不同OS版本，不同網頁瀏覽器等）。
- 跨App測試的不穩定。
- 平行化測試的不穩定。
- 透過IDE驗證測試及透過Command line工具執行測試，兩者有時會有些許不同的行為。
- 以及 **更多**。

## 讓UI自動化測試可靠及穩定的最佳實踐

### 遵從測試三角形

根據Google的[Just Say No to More End-to-End Tests](https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html)這篇文章，測試三角形是想像有個三角形，底部面積最大的部分是你的單元測試，接著往上你的測試會變得越來越大，但同時測試的數量會變得越來越少。

Google建議70/20/10的比例：70%單元測試，20%整合測試，以及10%的UI測試。實際的比例會根據每個團隊而有所不同，但整體來說，比例應該如同三角形的形狀一樣。另外請避免反模式，比如團隊主要依賴UI測試，並很少使用整合測試，且幾乎沒有單元測試。

如同正常的三角形在現實世界是最穩定的結構，測試三角形也是最穩定的測試策略。

如果你想知道更多有關iOS的單元測試，可以參考我另外一篇[iOS單元測試規範](https://github.com/hayasilin/unit-tests-ios-guide/blob/master/README_zh-Hant.md)。

### 開發前的準備
- 了解到維護成本，不同裝置依賴問題，測試環境的依賴問題，原生測試框架的限制等造成UI自動化測試的脆弱性。
- 因此，UI自動化測試應只是執行**煙霧測試(Smoke Testing)**，確認最常用的使用者情境即可，此外不需要做更多。
- 如果發現有某隻測項容易產生Flaky結果但無法快速解決，請移除或不執行該測項，Flaky tests不該被留存，直到被修復。
- 團隊沒有時間一直檢查Flaky tests，只有可靠且穩定的UI自動化測試可以幫助團隊：
  - 快速交付App。
  - 快速反饋，以即時維護及強化App品質。

### 什麼樣的測試適合UI自動化測試

**推薦**
- 成熟的功能：產品最重要的功能且短期內不會輕易被改變。
- 最常見的使用者流程。
- 很簡單就能被轉換成UI自動化測試的功能。

**不推薦**
- 仍在發展中的功能：該功能可能依照商業計劃仍會不斷變動。
- 因原生測試框架的限制，所以很難被變成自動化測試的測項。
- 需要花費很多成本才能做的測項，比如像App的推播通知。

**UI自動化測試無法將所有的測項都變成自動化，請選擇適合的測項並讓不適合的測項仍然以手動測試為主。**

### 避免Flaky tests的10大實用方法
1. 測試孤立：測試應該要獨立於其他測試之外。如果有測項改變狀態或調整資料庫，請在測項的setup或teardown清空該狀態。
2. 確認所有UI會產生的狀態。寫測項時當下所看到的UI並不代表所有狀態下都是該UI，當使用者的狀態改變，UI可能也會隨之變化，將所有可能的UI情境都思考過並讓測試能充分對應各種情境才能避免Flaky。
3. 每次完成測試程式碼後，請先將所有測項一起執行，並重複約10次的測試，確保新加入的程式碼不會造成Flaky tests。
4. 完成測試程式碼後，試著先在你定期執行UI自動化測試的裝置上執行，確保沒有裝置依賴的問題。
5. 將測項所需的畫面跳轉減至最少。最實用的的方法是使用原生測試框架，讓測試開始時從所需測試的畫面開始即可，並將狀態及資料事前設定好，既可加快測試速度，也可避免Flaky tests，XCUITest跟Espresso皆有提供該功能。
6. 選擇App最成熟的功能開始寫測試程式。
7. 不要過度設計你的測試程式，使用設計模式如Page object pattern讓每個測試程式既簡單又很容易維護。
8. 穩定度重要性大於完成度，不要想要寫越多測試越好，只需要測試最常用的功能，別忘了UI自動化測試只是煙霧測試(Smoke Testing)。不穩定的UI自動化測試對於團隊毫無幫助，因為需要浪費時間去調查Flaky tests而你的App實際上功能並無問題。
9. 每個測試項目只挑選畫面上1個最重要的UI元件驗證(Assert)即可。UI測試這個名稱常常讓人誤以為要測試整個畫面的所有UI都正確，但這只會增加Flaky tests的機會，實務上請把UI測試想成測試使用者的行為或情境，而不是用來測試UI位置或佈局。
10. 不需使用太多實機裝置執行平行化測試，數支手機的測試結果應該就提供團隊足夠的信心程度來決定App可否上線。

## UI測試的程式碼規範

### iOS

**Page object class**

- 類別名稱:
	- 以**Page**結尾

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

- 類別名稱:
	- 以**Tests**結尾
- 方法名稱:
	- 以**test**開頭
	- 描述測試行為

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

**點擊按鈕**

**建議**
```swift
tapWithExpect(element: mainPage?.searchButton)
```

**不建議**
```swift
searchPage?.searchButton.tap()

// ...Should use XCTWaiter to wait UI button exist before perfom action.
```

---

**驗證元件存在**

**建議**
```swift
XCTAssert(app.staticTexts["identifier"].waitForExistence(timeout: timeout))
```

**不建議**
```swift
XCTAssert(app.staticTexts["identifier"])

// ...Should use wait for existence with timeout.
```

---

**移動至特定畫面進行測試**

**建議**
```swift
app.launchArguments += ["setNavigateToThirdPage"]

app.launch()
```

**不建議**
```swift
mainPage?.navigateToFirstPageButton.tap()

firstPage?.navigateToSecondPageButton.tap()

secondPage?.navigateToThirdPageButton.tap()

// ...Too many page navigations will create flaky.
```

---

**測試刪除資料行為**

**建議**
```swift
app.launchArguments += ["setData"]

app.launch()

// Perfom delete action then assert.
```

**不建議**
```swift
thePage?.addDataButton.tap()

// ...Don't use UI to perform series of actions for preparing data or state.
```

---

**測試空資料頁面顯示**

**建議**
```swift
app.launchArguments += ["setClearDatabase"]

app.launch()

// Assert empty data page display
```

**不建議**
```swift
thePage?.deleteAllDataButton.tap()

// ...Don't use UI to perform series of actions for preparing data or state.
```

---

**驗證結果**

**建議**
```swift
XCTAssert(app.staticTexts["identifier"].waitForExistence(timeout: timeout))

// Just pick one UI element to assert in one test case, that will be enough.
```

**不建議**
```swift
XCTAssert(app.staticTexts["identifier"].waitForExistence(timeout: timeout))
XCTAssert(app.buttons["identifier"].waitForExistence(timeout: timeout))
XCTAssert(app.textFields["identifier"].waitForExistence(timeout: timeout))

// ...Don't assert too many UI elements in one test case.
```

### Android

**Page object class**

- 類別名稱:
	- 以**Page**結尾

```kotlin
class MainPage {
    val searchButton: ViewInteraction = onView(withId(R.id.identifier))
}
```

**Test suite class**

- 類別名稱:
	- 以**Tests**結尾
- 方法名稱:
	- 以**test**開頭
	- 描述測試行為

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

**確認元件出現**

```kotlin
fun withIdDisplayed(id: Int): Matcher<View>? = allOf(withId(id), isDisplayed())
```

**等待元件出現**

```kotlin
fun waitElement(resourceId: Int) {
    var device: UiDevice = UiDevice.getInstance(InstrumentationRegistry.getInstrumentation())
    var context: Context = ApplicationProvider.getApplicationContext()
    device.wait(Until.hasObject(By.res(context.resources.getResourceName(resourceId))), AndroidTestConfig.TIMEOUT)
}
```

**等待文字出現**

```kotlin
fun waitTextElement(resourceId: Int) {
    var device: UiDevice = UiDevice.getInstance(InstrumentationRegistry.getInstrumentation())
    var context: Context = ApplicationProvider.getApplicationContext()
    device.wait(Until.hasObject(By.textContains(context.getString(resourceId))), AndroidTestConfig.TIMEOUT)
}
```

**結合等待元件出現後執行點擊動作**

```kotlin
fun clickElement(resourceId: Int) {
    waitElement(resourceId)
    onView(withIdDisplayed(resourceId)).perform(click())
}
```

**結合等待元件出現後驗證**

```kotlin
fun checkElementIsDisplayed(id: Int) {
    waitElement(id)
    onView(withIdDisplayed(id)).check(matches(isDisplayed()))
}
```

**結合等待文字出現後驗證**

```kotlin
fun checkTextIsDisplayed(text: Int) {
    waitTextElement(text)
    onView(withText(text)).check(matches(isDisplayed()))
}
```

### Code conventions

**點擊按鈕**

**建議**
```kotlin
clickElement(mainPage.searchButton)
```

**不建議**
```kotlin
mainPage.searchButton.perform(click())

// ...Should use wait until function to wait button exist before perfom action.
```

---

**驗證元件存在**

**建議**
```kotlin
waitElement(R.id.identifier)
onView(withIdDisplayed(R.id.identifier)).check(matches(isDisplayed()))

// You can make a function to include code above
```

**不建議**
```kotlin
onView(withId(R.id.identifier)).check(matches(isDisplayed()))
```

---

**移動至特定畫面進行測試**

**建議**
```kotlin
private val onboardingPage: OnboardingPage = OnboardingPage()

@get:Rule
val activity = ActivityTestRule<OnboardingActivity>(OnboardingActivity::class.java)

@Test
fun testOnboardingPageSwipeLeftAndRight() {
  // Perform testing in onboarding page
}
```

**不建議**
```kotlin
mainPage.navigateToFirstPageButton.perform(click())

firstPage.navigateToSecondPageButton.perform(click())

secondPage.navigateToOnboardingPageButton.perform(click())

// ...Too many page navigations will create flaky.
```

---

**測試刪除資料行為**

**建議**
```kotlin
val database = Room.databaseBuilder(ApplicationProvider.getApplicationContext(), Database::class.java, "database")
                   .allowMainThreadQueries()
                   .build()
database.dataDao().insert(data)

// Use in memory databasee to prepare data before test start
```

**不建議**
```kotlin
thePage.addDataButton.perform(click())

// ...Don't use UI to perform series of actions for preparing data or state.
```

---

**測試空資料頁面顯示**

**建議**
```kotlin
val database = Room.databaseBuilder(ApplicationProvider.getApplicationContext(), Database::class.java, "database")
                   .allowMainThreadQueries()
                   .build()
database.dataDao().removeAll()

// Use in memory databasee to clean data before test start
```

**不建議**
```kotlin
thePage.deleteDataButton.perform(click())

// ...Don't use UI to perform series of actions for preparing data or state.
```

---

**驗證結果**

**建議**
```kotlin
checkElementIsDisplayed(R.id.identifier)

// Just pick one UI element to assert in one test case, that will be enough.
```

**不建議**
```kotlin
checkElementIsDisplayed(R.id.identifier)
checkElementIsDisplayed(R.id.loginButton)
checkElementIsDisplayed(R.id.listView)

// ...Don't assert too many UI elements in one test case.
```

## 常見問答
1. 在CI/CD流程中，多久需執行一次UI自動化測試確認App正常運作？
  - **建議1天1次即可**，在凌晨或半夜。
    - 因為團隊裡不論是客戶端還是伺服器端的工程師，往往需在工作時間時更新程式碼或更新環境，此時執行UI自動化測試只是讓Flaky tests發生的機會大增。除非你的團隊能確保一個超級穩定的測試環境及測試資料（但又是另1種成本），不然其實只需要在1天中選1個或2個最不會被影響的時間執行UI自動化測試確認App沒問題即可，而結果也足以提供團隊即時的反饋。
    - 理論中，會希望將UI自動化測試啟動時間設定如同單元測試一樣，希望能在每次工程師發Pull Request上傳程式碼時，不僅要執行單元測試，也連同UI測試一起執行，來確認所有測試都能通過後，才能將程式碼Merge進分支。但實務上很難做到，主要原因是當你的UI測試項目越來越多，執行所需的時間也越來越長，如同大家所知，單元測試時間短，但UI測試時間長，如果把UI測試也排進去會拉長工程師及該程式碼等待的時間，造成效率降低的問題。此外，如同前面所述，UI自動化測試在本質上是脆弱的，常會因各種不同的因素而造成失敗的結果，如果UI測試不斷有問題，也會造成阻斷CI/CD後續流程的問題，所以實務上建議不需要將UI自動化測試與Pull request的流程綁在一起。

2. 使用哪種工具比較好？原生測試框架(XCUITest / Espresso)？還是第三方跨平台的測試工具？
  - 我建議使用**原生測試框架**。
    - 因為原生測試框架可以更好的操控App，並且可以修改App裡的database去創造你想要的測試環境及資料，大幅減少Flaky的產生，這是第三方跨平台工具做不到的。另外原生測試框架也更穩定，在最新版本的iOS及Android的支援也更好。

3. 需要多少實機裝置去跑平行化測試才夠？
  - 我建議只需要**2-3**台iOS裝置，以及**2-3**台Android裝置即可。
    - 因為不同的實機裝置及OS版本會有不同的行為，越多的裝置就造成你的程式碼需要為不同的行為做應對。另外，負擔太重的平行化測試也會造成Flaky的問題，因此只要同時能夠讓**2-3**實機測試能通過，應該足夠帶給團隊對App的信心。當然，如果你的團隊認為可以測試越多台且無Flaky的問題，亦無不可。

4. 我們可以達到多穩定的UI自動化測試？
  - 這根據你的測項數量多寡。
    - Flaky tests往往發生在測項項目高於30以上，且你用數個手機執行平行化測試的時候。當我根據此規範開發UI自動化測試後，即使每天新增測試程式碼及測試項目，可以持續達到Jenkins連續跑15次執行UI自動化測試中，Jenkins一定會有14次是完全成功結果（iOS 及 Android App）。在15次的Jenkins中總共執行的測試項目總數計算Flaky rate約0.7%左右(根據Google的[Flaky Tests at Google and How We Mitigate Them](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html)，Google的Flaky rate為1.5%左右)。對我來說雖不是完美，但UI自動化測試的結果已經提供足夠的反饋資訊給團隊決定該版本是否能上線。此外，我的團隊不需要一直派人力去處理Flaky tests，如果Jenkins出現了失敗結果，絕大多數是App真正發生了問題而非Flaky tests。
    - 相對穩定的結果也讓我們能縮短QA手動測試的時間，最快可以達到僅執行1天的QA就可上線，因為每天UI自動化測試就會給工程師即時的反饋，讓團隊即時修正。
  - 以下是一個經典的穩定及不穩定的UI自動化測試在Jenkins的結果：

### 穩定及不穩定的UI自動化測試結果

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
- When running xcodebuild's parallel automation testing, UI transition may be laggy and test cases become flaky easily
- Sometimes iOS device can't input text into textfield, it's just freeze.
  - Shut down and restart the iOS device
- [Encountered a problem with the test runner after launch.(Underlying error: Lost connection to DTServiceHub)](https://stackoverflow.com/questions/39635861/unable-to-contact-local-dtservicehub-to-bless-simulator-connection)

### Android Espresso issues
- [Click action change to long click action](https://stackoverflow.com/questions/32330671/android-espresso-performs-longclick-instead-of-click/35254071)
- [Start to idling and could not end the test nor perform next test cases](https://stackoverflow.com/questions/21406246/google-espresso-java-lang-runtimeexception-could-not-launch-intent-intent-act)
  - Press system back button can solve the issue.
- During UI change with default animation, automation click too fast during animation so even the element is visible, but it's not clickable status.
  - Use wait until element shows or wait longer time.
- [androidx.test.espresso.PerformException: Error performing 'fast swipe' on view](https://stackoverflow.com/questions/44005338/android-espresso-performexception)
  - Even turn off all animation effect on that device the flaky still happen.
- Different Android enmulators when open webview, their element description can be different (Nexus 7.0 v.s. Pixel 2 8.0).
- [qemu2 cpu0 thread no response for 15009 ms](https://stackoverflow.com/questions/47956847/android-studio-emulator-error-detected-a-hanging-thread-qemu2-main-loop)
  - Go to AVD Manager and select "Quick boot" if it's set "Cold boot", or vice versa.
- Sometimes the app would crash because some instances is null when running automation happen in devices. It won't happen when doing manual testing.
  - No solution for now. Just rerun it then it won't happen hopefully.
- Espresso perform(swipeUp()) will become click action on profile page's myStart button on Samsung S9 (8.0) & Samsung J2 Prime (6.0.1) devices.
  - Use other element to swipe up or skip the action for the problem devices
