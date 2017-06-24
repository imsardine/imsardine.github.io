---
title: 如何用 Espresso 操作或檢查 RecyclerView 第 n 個項目的內容？
date: 2017-06-23 06:51:23
tags: [espresso, android, ui-testing, test-automation]
---

[`RecyclerView`][recyclerview] 能有效處理大量 item 的顯示，其中一項特性是會將捲出螢幕外的 view 回收再利用，螢幕外的 item 並不會出現在 view hierarchy 裡，因此螢幕外的 item 不能用 [`onView()`][onview] 來找，再加上 `RecyclerView` 並非 [`AdapterView`][adapterview]，所以 [`onData()`][ondata] 也不適用。

那要怎麼操作或驗證 `RecyclerView` 某個 item 的內容？而且那個 item 可能在螢幕之外...

![示意圖](/images/espresso-interact-with-verify-item-child-views/concept-structure.png)

 [recyclerview]: https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html
 [adapterview]: https://developer.android.com/reference/android/widget/AdapterView.html
 [onview]: https://developer.android.com/reference/android/support/test/espresso/Espresso.html#onView(org.hamcrest.Matcher%3candroid.view.View%3e)
 [ondata]: https://developer.android.com/reference/android/support/test/espresso/Espresso.html#onData(org.hamcrest.Matcher%3c?%20extends%20java.lang.Object%3e)

<!--more-->

## 要解決的問題

假設畫面的狀態如上面的示意圖，試想下面兩個問題：

 1. 如何取消 Item 011 的核取方塊？
 2. 如何檢查 Item 012 的核取方塊沒有勾選？

首先你會遇到如何把螢幕外的 item 捲進畫面的問題。所幸 [`RecyclerViewActions`][recyclerviewactions] 提供的 action 都會自動處理掉捲動的問題，包括單純把某個 item 捲進畫面的 `scrollTo*()`，以及對某個 item 進行操作的 `actionOn*()` (操作之前，會自動將 item 捲進畫面)，所以上面兩個問題看似都不難？

```java
// 取消 Item 011 的核取方塊？
onView(withId(R.id.recycler)).perform(
    actionOnItemAtPosition(10, click()));

// 檢查 Item 012 的核取方塊沒有勾選？
onView(withId(R.id.recycler)).perform(scrollToPosition(11))
    .check(matches(not(isChecked())));
```

很不幸地，其中：

 * 取消 Item 011 的核取方塊 - `actionOnItemAtPosition(10, click())` 的寫法會按到整個 item (`R.id.item`) 的中心點，而非核取方塊 (`R.id.item_checkbox`)。
 * 檢查 Item 012 的核取方塊沒有勾選 - 雖然 Item 12 會因為 `scrollToPosition(11)` 而捲進畫面，但 `check(matches(isChecked()))` 檢查的是 `onView(withId(R.id.recycler))` 所表示的整個 `RecyclerView`，而非 Item 12 下的核取方塊。

問題在於，目前 `RecyclerViewActions` 只能操作 item 本身，不能操作 item 內部的組成，而在檢查 item 這一塊則完全沒有著墨，更不用提如何檢查 item 的內部組成了。

## 解決方案

由於 `RecyclerViewActions` 將 item 捲進畫面的部份處理得很好，加上開發人員也習慣用它來處理 `RecyclerView`，所以我傾向朝 **擴充現有 API** 的方向來解決問題，而非創造一組全新的 API，這樣可以減少學習及後續溝通的成本。

將 API 的可讀性也一併考量進來，理想上 test code 會像是：

```java
// 取消 Item 011 的核取方塊
onView(withId(R.id.recycler)).perform(
    actionOnItemAtPosition(10, onChildView(withId(R.id.item_checkbox), click())));

// 檢查 Item 012 的核取方塊沒有勾選
onView(withId(R.id.recycler)).perform(scrollToPosition(11))
    .check(itemAtPosition(11).onChildView(withId(R.id.item_checkbox)).matches(not(isChecked())));
```

其中：

 * `onChildView(Matcher<View> childMatcher, ViewAction viewAction)` 實作 [`ViewAction`][viewaction]，擴充了現有 `RecyclerViewActions.actionOn*()` 的用法，將原先作用在整個 item 的 `viewAction`，轉移到符合 `childMatcher` 的 child view。
 * `itemAtPosition(int position).onChildView(Matcher<View> childMatcher).matches(Matcher<View> viewMatcher)` 實作 [`ViewAssertion`][viewassertion]，擴充了現有的 `ViewInteraction.check()`，將原先作用在整個 `RecyclerView` 的檢查 `viewMatcher`，轉移到特定 item 下的符合 `childMatcher` 的 child view；當然，若只寫 `itemAtPosition(position)` 則會作用在特定的 item。

完整的實作可以在[這裡][gist]找到 (Gist)，這裡就不細談，而是把重點擺在為什麼 API 要以這種方式擴充，背後有什麼考量...

 [viewaction]: https://developer.android.com/reference/android/support/test/espresso/ViewAction.html
 [viewassertion]: https://developer.android.com/reference/android/support/test/espresso/ViewAssertion.html
 [gist]: https://gist.github.com/imsardine/c31dd61b3d97710f0d6828f1aedc9633

## 為什麼不那樣做？

當然在這之前，網路上也有其他人提出過不同的方法，有很大的機會會找到 Danny Roa 在 2015 年提出的構想 - [Espresso: Matching Views in RecyclerView][danny-idea]，用起來像是這個樣子：

```java
// 取消 Item 011 的核取方塊
onView(withId(R.id.recycler)).perform(
    actionOnItemViewAtPosition(10, R.id.item_checkbox, click()));

// 檢查 Item 012 的核取方塊沒有勾選
onView(withId(R.id.recycler)).perform(scrollToPosition(11));
onView(withRecyclerView(R.id.recycler_view).atPositionOnView(11, R.id.item_checkbox))
    .check(matches(not(isChecked())));
```

主要的差別在於：

 * 在定位 recycler view、item 或 child view 時都只接受 resource ID，少了一點彈性。
 * 操作時捨棄了原有的 `RecyclerViewActions.actionOn*()`，所以自創的 `actionOnItemViewAtPosition()` 為了實現自動捲動，仿 `RecyclerViewActions` 在內部[重新實作了 `ScrollToPositionViewAction`][danny-reimpl]。
 * 檢查時從 `onView()` 下手，直接定位到某個 item (或是底下的 child view)，所以無法把 `perform(scrollToPosition(11))` 寫在同一行，把 item 捲入畫面的動作要分開處理。

若你仔細研究過 [`RecyclerViewMatcher.matchesSafely(View view)`][danny-impl] 的實作，會發現若要支援以 view matcher 來找 `RecyclerView`，效率恐怕會是個問題？

在思考如何擴充現有 API 的過程中，也有人提出可以將 check/assertion 視為一種 action 的想法，也就是在 `ViewAction` 裡做檢查 (有可能拋出 `AssertionError`)，例如：

```java
// 檢查 Item 012 的核取方塊沒有勾選
onView(withId(R.id.recycler)).perform(actionOnItemAtPosition(11,
    checkItem(withId(R.id.item_checkbox), matches(not(isChecked())))));
```

當然，這在技術上是可行的，但...

> Just because you can doesn't mean you should.

不是對 `ViewAction` 的解釋太過狹隘，而是這明顯打破了 Espresso API - `onView(Matcher<View>).perform(ViewAction).check(ViewAssertion)` 對 `ViewAction` 所賦予的責任，更何況 `ViewAction` 的 API 文件也明確寫著：

> Do not throw `AssertionError`s. Assertions belong in `ViewAssertion` classes.

所以這個方向就不在考慮之列。

 [danny-idea]: http://dannyroa.com/2015/05/10/espresso-matching-views-in-recyclerview/
 [danny-impl]: https://github.com/dannyroa/espresso-samples/blob/master/RecyclerView/app/src/androidTest/java/com/dannyroa/espresso_samples/recyclerview/RecyclerViewMatcher.java#L45
 [danny-reimpl]: https://github.com/dannyroa/espresso-samples/blob/master/RecyclerView/app/src/androidTest/java/com/dannyroa/espresso_samples/recyclerview/TestUtils.java#L58

## 結論

未來希望讓 `itemAtPosition(int position)` 也能支援 `Matcher<VH> viewHolderMatcher` 與 `Matcher<View> itemViewMatcher`，使用上會更具彈性。

如果你也有相同的困擾，建議試試這裡提出的解法，有機會也比較一下 Danny Roa 的方法，希望能聽到你使用上的回饋。

## 參考資料

 * [Testing RecyclerViews at Specific Positions with Espresso](https://spin.atomicobject.com/2016/04/15/espresso-testing-recyclerviews/) (2016-04-15)
 * [Espresso: Matching Views in RecyclerView](http://dannyroa.com/2015/05/10/espresso-matching-views-in-recyclerview/) (2015-05-10)

