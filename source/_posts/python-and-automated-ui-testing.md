---
title: Python 與 UI 自動化測試
date: 2017-06-10 10:15:00
tags: [python, ui-testing, test-automation, testing-pyramid, e2e-testing,
       selenium, webdriver, appium, page-objects]
---

因緣際會下，今年有機會在 [PyCon 2017][pycon-2017] 進行短講，雖然近期的工作跟 Android 單元測試 (unit testing) 比較相關，但在 Python 的場子分享 Android 似乎不太好 :P

恰巧幾天前從 Google I/O 2017 的一場演說 [Test-Driven Development on Android with the Android Testing Support Library][android-tdd-with-atsl] 得到不少啟發，尤其整個 TDD workflow 是由 UI testing 驅動的做法，想說就從 Python 的角度切入，看看不同平台的 UI testing 是否有 Python 可以發揮的空間。

先從 UI testing 與 E2E testing 的關係開始說起...

 [android-tdd-with-atsl]: /2017/06/04/io17-tdd-android-with-atsl/

[pycon-2017]: https://tw.pycon.org/2017/

<!--more-->

## UI 自動化測試與 E2E Testing

廣義地來說，UI 自動化測試 (automated UI testing) 是指：

> 在某個已知的狀況下 (known state)，用程式模擬使用者透過 UI 操作受測的應用程式 (application under test, AUT)，並驗證結果是否符合預期。

而這裡是專指用來實作 end-to-end (E2E) testing 的 UI testing，也就是測試金字塔 (testing pyramid) 的最頂層，由於測試速度較慢、維護成本較高，所以實務上 E2E test 的數量會比金字塔底層的單元測試 (unit test) 少很多。

![測試金字塔](/images/testing-pyramid.png)

[Google 的專家建議][google-recommendation] unit test、integration test、E2E test 的數量可以維持 70:20:10 的比例。雖然各專案的狀況不同，但至少要維持金字塔的形狀，避免過度偏重 E2E testing 而變成倒金字塔或甜筒，或是用 E2E testing 來取代 integration testing 而變成沙漏的形狀。

自動化測試是敏捷開發不可缺少的一部份，當然 E2E testing 也不例外，以確保整個系統運作起來是沒問題的，否則需要靠手動測試的部份越來越多，若沒有足夠的時間做回歸測試 (regression testing)，就無法確保舊有的部份沒有被改壞。

 [google-recommendation]: https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html

## Web

講到 web 自動化測試，相信許多人會直接想到 [Selenium][]。沒錯，它已經是 web 自動化測試的業界標準 (de facto standard)。

![Selenium 架構](/images/python-and-automated-ui-testing/selenium-arch.png)

它成功的關鍵在於中間的 WebDriver protocol，目前已經是 [W3C 的候選推薦標準 (W3C Candidate Recommendation)][w3c-webdriver]。

根據這個公開的協定，client 端已發展出適用於不同程式語言的 WebDriver API (也就是 language bindings)，在另一端 browser 也各自發展出介接 WebDriver protocol 與內部引擎的 driver，結果就是可以透過任何程式語言操控任何支援 WebDriver 的 browser (包括所有主流的 browser)。

 [selenium]: http://www.seleniumhq.org/
 [w3c-webdriver]: https://w3c.github.io/webdriver/webdriver-spec.html
 [wikipedia]: https://en.wikipedia.org/

下面的例子是用 Selenium 搭配 Firefox，測試在 [Wikipedia][] 搜尋 "hello world" 可以找到標題為 "Hello, World!" program 文件：

```python
import unittest, time
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

class HelloWorldTest(unittest.TestCase):

    def setUp(self):
        self._driver = webdriver.Firefox()

    def test(self):
        browser = self._driver

        browser.get('http://en.wikipedia.org')
        browser.find_element_by_id('searchInput').send_keys('hello world' + Keys.ENTER)

        heading = WebDriverWait(browser, 5).until(
                EC.visibility_of_element_located((By.ID, 'firstHeading'))
            ).text
        self.assertEqual(heading, '"Hello, World!" program')

    def tearDown(self):
        self._driver.quit()

if __name__ == '__main__':
    unittest.main()
```

要執行這個測試，除了 Firebox，你還要安裝 [`selenium`][selenium-package] 套件，以及用來驅動 Firebox 的 [GeckoDriver][gecko-driver]。

若是 browser 不在同一台機器，可以利用 Selenium Server 當做跳板，像是這個樣子：

![Selenium 架構 (搭配 Selenium Server)](/images/python-and-automated-ui-testing/selenium-arch-server.png)

只要改用 Remote WebDriver，並將 driver 安裝在 Selenium Server 所在的機器即可：

```python
    def setUp(self):
        # self._driver = webdriver.Firefox()
        self._driver = webdriver.Remote(
                'http://your-selenium-server:4444/wd/hub',
                webdriver.DesiredCapabilities.FIREFOX.copy())
```

 [selenium-package]: https://pypi.python.org/pypi/selenium
 [gecko-driver]: https://github.com/mozilla/geckodriver/

## Mobile

在行動平台方面，近幾年隨著官方開始投注更多的心力，由第三方 UI 測試工具主導的局勢慢慢起了變化，第三方工具的開發雖然還是相當活躍，但能否即時追得上官方的腳步會是最大的考驗 (以支援最新的平台)，這樣的情況同時發生在 Android 跟 iOS 上...

目前各平台官方提供的工具有：

 * iOS - [XCUITest][]，用 Swift 或 Objective-C 寫 test code，打包後在裝置上執行。
 * Android - [Espresso][] 與 [UI Automator][uiautomator]，用 Java 寫 test code，也是打包後在裝置上執行；前者只能用於單一 app 內的測試，後者則能夠跨多個 app。

 [xcuitest]: https://developer.apple.com/videos/play/wwdc2015/406/
 [espresso]: https://developer.android.com/training/testing/ui-testing/espresso-testing.html
 [uiautomator]: https://developer.android.com/training/testing/ui-testing/uiautomator-testing.html

是不是 Python 在行動平台的 UI 自動化測試上就沒有可以發揮的空間了？

其實不然，答案就在上面提到的 WebDriver，只要為 iOS 與 Android 實作不同的 driver，背後跟各平台專屬的測試工具介接，自然就能用 Python (或其他語言) 來操控 app。

![Appium 架構](/images/python-and-automated-ui-testing/appium-arch.png)

事實上這就是 [Appium][] 正在做的事，它將 XCUITest 與 UI Automator 重新包裝過，讓我們可以透過熟悉的語言與 WebDriver API 操控 iOS 與 Android app。

Andriod 的測試寫起來會像是這個樣子：

```python
import unittest
from selenium import webdriver

class AndroidTest(unittest.TestCase):

    def setUp(self):
        caps = {
            'platformName': 'Android',
            'automationName': 'UIAutomator2',
            'app': '/path/to/app-debug.apk',
            'deviceName': 'DEVICE_SERIAL_NO'
        }
        self._driver = webdriver.Remote('http://your-appium-server:4723/wd/hub', caps)

    def test(self):
        app = self._driver

        app.find_element_by_id('username').send_keys('somebody')
        app.find_element_by_id('password').send_keys('secret')
        app.find_element_by_id('login_button').click()
        # ...

    def tearDown(self):
        self._driver.quit()

if __name__ == '__main__':
    unittest.main()
```

基本上跟 web 搭配 Selenium Server 的寫法差不多，一樣採用 Remote WebDriver (`selenium.webdriver.Remote`)，只不過 server 是指向 Appium server。

除非要使用 Appium 在標準 Selenium 上擴充的功能，才需要額外安裝 [`Appium-Python-Client`][appium-python-client] 套件，並改採 `from appium import webdriver`。

 [appium]: http://appium.io/
 [appium-python-client]: https://github.com/appium/python-client

這或許沒有像直接用 Swift / Java 寫測試那麼酷炫，但 Python 做為一個具有生產力的語言，加上你已經熟悉 WebDriver API，透過 Appium 就能將 UI 自動化測試延伸到行動平台，這對於開發能量有限的測試團隊來說，是一個合理的選擇。

Appium 過去比較令人垢病的問題是不太穩定，由於 E2E testing 原本就因涉及的層面較廣而容易有時好時壞 (flaky) 的狀況，所以 Appium 的問題會讓狀況變得更糟。不過這一點在 Appium 1.5.0 開發團隊[將整個 Appium 重寫][appium-rewrite]之後已經獲得改善。

 [appium-rewrite]: https://www.slideshare.net/saucelabs/10-things-you-didnt-know-about-appium-whats-new-in-appium-15/13

## Page Object 與 Page Factory

由於 UI 很容易變動的關係，在實作 UI test 時會建議套用 [Page Object Pattern][page-objects]，從使用者的觀點針對畫面上 test code 會操作和驗證的部份進行塑模 (modeling)，把 UI 操作的細節都藏在 page object 裡，進而提高 test code 本身的可讀性 (readability) 與可維護性 (maintainability)。

也就是說，UI 的變動無可避免，但 UI 一旦有變動時，我們可以只要調整少量的 test code 即可。

Page Object Pattern 實現的方式會因語言、測試工具而異。以上面的 `HelloWorldTest` 為例，搭配 page object 改寫後的 test code 會像這個樣子：

```python
    def test(self):
        home = HomePage(self._driver).visit()
        article = home.search('hello world')
        self.assertEqual(article.title, '"Hello, World!" program')

        # browser.get('http://en.wikipedia.org')
        # browser.find_element_by_id('searchInput').send_keys('hello world' + Keys.ENTER)
        # 
        # heading = WebDriverWait(browser, 5).until(
        #         EC.visibility_of_element_located((By.ID, 'firstHeading'))
        #     ).text
        # self.assertEqual(heading, '"Hello, World!" program')
```

用 Python 實作 page object 的過程中，會發現 Python bindings 並未提供 [Page Factory][page-factory] 的支援 - 用宣告式 (declarative) 語法來找元件，所以之前試著用較 Pythonic 的方式實現 Page Factory 的概念，用起來像是這個樣子：

```python
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from pagefactory_support import callable_find_by as find_by, visible

class HomePage:

    _search_box = find_by(id_='searchInput')

    def __init__(self, driver):
        self._driver = driver

    def visit(self):
        self._driver.get('http://en.wikipedia.org')
        return self
    
    def search(self, keyword):
        self._search_box().send_keys(keyword + Keys.ENTER)
        return ArticlePage(self._driver)

class ArticlePage:

    _heading = find_by(id_='firstHeading')

    def __init__(self, driver):
        self._driver = driver

    @property
    def title(self):
        heading = WebDriverWait(self._driver, 5).until(visible(self._heading))
        return heading.text
```

更多細節可以參考 [Implementing Page Factory (or PageFactory) Pattern in Python][page-factory-python]。

 [page-objects]: https://github.com/SeleniumHQ/selenium/wiki/PageObjects
 [page-factory]: https://github.com/SeleniumHQ/selenium/wiki/PageFactory
 [page-factory-python]: https://jeremykao.wordpress.com/2015/06/10/pagefactory-pattern-in-python/

總的來說，Python + WebDriver 是一項很划算的投資。Happy testing :-)

