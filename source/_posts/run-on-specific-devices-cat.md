---
title: 只在某些裝置上執行測試 (connectedAndroidTest)
date: 2016-12-07 08:53:00
tags: [android, test-automation]
---

`connectedAndoidTest` 預設會將所有 test 執行在所有連接的 device/emulator 上，若只想執行在某些 device/emulator 上要怎麼做？

就像 [#66129][] 所描述的情節一樣，這樣的需求通常會發生在 build server 上，因為同時連接有許多不同用途的 device/emulator。但這件事有點弔詭，因為相關的討論並不多：

<!-- more -->

 * [Issue 66129 - android - Run tests only on specified devices (gradle instrumentTest) - Android Open Source Project - Issue Tracker - Google Project Hosting][#66129]
 * [Issue 180700 - android - Filter multiple devices for execution (aka extended ANDROID_SERIAL) - Android Open Source Project - Issue Tracker - Google Project Hosting](https://code.google.com/p/android/issues/detail?id=180700&thanks=180700&ts=1437571648)
 * [Change Ib522bbaf: Allow multiple serial numbers in ANDROID_SERIAL. | android-review.googlesource Code Review][#160929]

 [#66129]: https://code.google.com/p/android/issues/detail?id=66129
 [#160929]: https://android-review.googlesource.com/#/c/160929/

根據 [#160929][] 的說法，可以透過 `ANDROID_SERIAL` 環境變數來指定一或多個 device/emulator (用逗號隔開)，在 Android Plugin for Gradle 2.1.2 上試過是有作用的。例如：

```
$ ANDROID_SERIAL=emulator-5554,ABC123DEF ./gradlew connectedAndroidTest
```

只在 `emulator-5554` (emulator) 與 `ABC123DEF` (device) 上執行測試。

