---
title: "Local unit test 與 JSONObject not mocked"
date: 2017-06-16 16:18:49
tags: [android, unit-testing]
---

為了縮短測試時間，在實務上會儘可能把 Android framework 的相依性抽離，讓大部份的 production code 可以用 [local unit test][local-unit-test] 在主機的 JVM 上進行測試，少數跟 Android framework 有相依性的 code 才送到 device/emulator 上透過 instrumented test 測試。

> **If your unit test has NO dependencies or only has SIMPLE dependencies on Android, you should run your test on a local development machine.** This testing approach is efficient because it helps you avoid the overhead of loading the target app and unit test code onto a physical device or emulator every time your test is run. Consequently, the execution time for running your unit test is greatly reduced.
> <div style="text-align: right">-- [Building Local Unit Tests][local-unit-test]</div>

 [local-unit-test]: https://developer.android.com/training/testing/unit-testing/local-unit-tests.html
 [instrumented-test]: https://developer.android.com/training/testing/unit-testing/instrumented-unit-tests.html

若有少量的 Android framework 相依性無法抽離，還是可以搭配 mocking framework (例如 [Mockito][]) 安排 Android framework 該有的行為，讓測試得以進行，否則會有下面的錯誤：

    java.lang.RuntimeException: Method xxx in android.xxx.SomeClass not mocked.

> At runtime, tests will be executed against a **modified version of `android.jar` where all final modifiers have been stripped off.** This lets you use popular mocking libraries, like Mockito.
> <div style="text-align: right">[Unit testing support - Android Studio Project][unit-testing-support]</div>

但下面的錯誤，又該如何解釋？

```
java.lang.RuntimeException: Method put in org.json.JSONObject not mocked. See http://g.co/androidstudio/not-mocked for details.

    at org.json.JSONObject.put(JSONObject.java)
```

 [mockito]: http://site.mockito.org/
 [unit-testing-support]: http://tools.android.com/tech-docs/unit-testing-support

<!--more-->

莫非 `org.json.JSONObject` 也是 Android framework 的一部份？試著把 `mockable-android-<VERSION>.jar` 解壓縮：

```
$ tree -L 2 org/
org/
├── apache
│   └── http
├── json
│   ├── JSONArray.class
│   ├── JSONException.class
│   ├── JSONObject.class
│   ├── JSONStringer.class
│   └── JSONTokener.class
├── w3c
│   └── dom
├── xml
│   └── sax
└── xmlpull
    └── v1
```

原來 `org.json.JSONObject` 也在 mockable `android.jar` 裡面，難怪實作一起被拔除了。

網路上建議的解法大致分為三種：

 1. 為 `testComple` 這個 build configuration 明確加上 `org.json:json` 的相依性。
 2. 用 [Robolectric][] 來營造完整的 Android 執行環境。
 3. 在 build file 裡加上 `unitTests.returnDefaultValues = true` 的設定。

 [robolectric]: http://robolectric.org/

基本上第 3 種方法只是延緩錯誤的發生 - 讓 Adroid framework 的 method 回傳 `0`/`null`，而不在當下丟出 `RuntimeException`，所以接下來若有用到這個回傳值，很可能就會丟出 `NullPointerException`，或是導致無法預期的結果。由於丟出 exception 的位置離案發現場有一段距離，這麼做只是徒增除錯的難度，所以不建議這麼做。

第 2 種方法，雖然可以避開 not mocked 的錯誤，但這與我們使用 local unit test 而非 instrumented test 來進行測試的出發點是相違背的，這會使得受測的程式即便出現 Android framework 的相依性，也不會出現 not mocked 的錯誤，所以也不建議這麼做。

> The `android.jar` file that is used to run unit tests does not contain any actual code - that is provided by the Android system image on real devices. Instead, all methods throw exceptions (by default). **This is to MAKE SURE your unit tests only test your code and do not depend on any particular behaviour of the Android platform** (that you have not explicitly mocked e.g. using Mockito).
> <div style="text-align: right">[Unit testing support - Android Studio Project][unit-testing-support]</div>

剩下的第 1 種方法，是我比較推薦的 - 在 `testComple` build configuration 額外宣告 `org.json:json` 的相依性：

```
dependencies {
   ...
   testCompile 'org.json:json:20160810'
}
```

背後的原理是在 test execution 的 classpath 前面安插 JSON library (`org.json.*`)，這樣執行期就不會用到 mockable `android.jar` 的版本。

```
16:20:55.008 [DEBUG] [org.gradle.process.internal.worker.DefaultWorkerProcessBuilder] Creating Gradle Test Executor 4
16:20:55.008 [DEBUG] [org.gradle.process.internal.worker.DefaultWorkerProcessBuilder] Using application classpath \
[junit-4.12.jar, ..., json-20160810.jar, ..., mockable-android-25.jar]
```

不過在 [SO 的一則討論][so-discussion]裡，有人提到：

> * Note that the latest version of JSON is built for Java 8, so you'll need to grab 20140107
> * You should use THIS version : `org.json:json:20080701`, which matches the Android version.

這確實是個問題，到底哪個版本才會跟 Android framework 裡的一樣？看了 Android API Level 25 `JSONObject` 的原始碼，發現這行註解：

```java
// Note: this class was written without inspecting the non-free org.json sourcecode.`
```

似乎 Android framework 的實作跟 `org.json:json` 這個 module 是無關的，只是實作上相容而已？

因此第 3 種方法也並非那麼完美，是不是會衍生其他的問題還要再觀察。雖然無法理解 `org.json.*` 為什麼會是 Andriod framework 的一部份，但還好其中也只有 `org.json.*` 比較常在 production code 用到，否則這類問題還真有點惱人。

 [so-discussion]: https://stackoverflow.com/a/30759769/1691598

Happy Testing :)

