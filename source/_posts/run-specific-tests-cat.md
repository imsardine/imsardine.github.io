---
title: 只執行某些測試 (connectedAndroidTest)
date: 2016-12-07 08:53:00
tags: [android, test-automation]
---

`connectedAndoidTest` 預設會將所有 test 執行在所有連接的 device/emulator 上，若只想執行某些 test 要怎麼做？

`connectedAndroidTest` 背後其實也是透過 [`am instrument`][am-instrument] 搭配 [`AndroidJUnitRuner`][android-junit-runner] 在 device/emulator 上執行測試：

<!-- more -->

    am instrument -w [<FLAGS>] <TEST_PACKAGE>/<RUNNER_CLASS>

其中 `RUNNER_CLASS` 通常是 `android.support.test.runner.AndroidJUnitRunner`。

`AndroidJUnitRunner` 本身支援許多跟測試篩選 (test filtering) 有關的 flag (`-e <KEY> <VALUE>`)，例如要執行特定一個 test 可以用 `-e class com.example.FooTest#testFoo`。

早期 `connectedAndroidTest` 不支援從 command line 傳入 instrumentation runner argument (也就是上面的 flag)，只能先將 test app 安裝上去，再用 `am instrument` 執行測試。例如：

```
$ ./gradlew installDebugAndroidTest
$ adb shell am instrument -w -e class com.example.FooTest#testFoo \
     com.example.test/android.support.test.runner.AndroidJUnitRunner
```

不過 Android Plugin for Gradle 從 1.3.0 版開始支援這項功能：

> **Android plugin for Gradle, revision 1.3.0 (July 2015)**
>
> Added support for specifying instrumentation test-runner arguments from the command line. For example:
>
> ```
> ./gradlew connectedCheck \
>    -Pandroid.testInstrumentationRunnerArguments.size=medium \
>    -Pandroid.testInstrumentationRunnerArguments.class=TestA,TestB
> ```
> -- [Android Plugin for Gradle Release Notes][plugin-release-notes]

也就是以下面的形式從 command line 提供 runner argument：

    -Pandroid.testInstrumentationRunnerArguments.<KEY>=<VALUE>

所上面的例子可以改用：

```
$ ./gradlew connectedAndroidTest \
     -Pandroid.testInstrumentationRunnerArguments.class=com.example.FooTest#testFoo
```

`AndroidJUnitRunner` 還支援其他 case filtering 的條件，包括：

 * `-e class com.example.FooTest#testFoo,com.example.BarTest#testBar` - 只執行一或多個 test。
 * `-e class com.example.FooTest,com.example.BarTest` - 只執行一或多個 class 裡所有的 test。
 * `-e package com.example.foo` - 只執行某個 (Java) package 裡所有的 test。
 * `-e annotation com.example.MyAnnotation` - 只執行有標示某個 annotation 的 test。
 * `-e notAnnotation com.example.MyAnnotation,com.example.AnotherAnnotation` - 排除有標示任一 annotation 的 test。
 * `-e size small | medium | large` - 只執行標示有 [`@SmallTest`][small-test]、[`@MediumTest`][medium-test] 或 [`@LargeTest`][large-test] 的 test。

更多用法可以參考[`AndroidJUnitRunner` 的說明][android-junit-runner]。

至於如何讓 test 只執行在某些 device/emulator 上會在[另一篇](/2016/12/07/run-on-specific-devices-cat/)說明，但這兩者是可以並用的，例如：

```
$ ANDROID_SERIAL=emulator-5554,ABC123DEF ./gradlew connectedAndroidTest \
     -Pandroid.testInstrumentationRunnerArguments.size=small
```

只在 `emulator-5554` (emulator) 與 `ABC123DEF` (device) 上執行標示有 `@SmallTest` 的 test。

 [am-instrument]: https://developer.android.com/studio/test/command-line.html#AMSyntax
 [android-junit-runner]: https://developer.android.com/reference/android/support/test/runner/AndroidJUnitRunner.html
 [plugin-release-notes]: https://developer.android.com/studio/releases/gradle-plugin.html
 [small-test]: https://developer.android.com/reference/android/support/test/filters/SmallTest.html
 [medium-test]: https://developer.android.com/reference/android/support/test/filters/MediumTest.html
 [large-test]: https://developer.android.com/reference/android/support/test/filters/LargeTest.html

