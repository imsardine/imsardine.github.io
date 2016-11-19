---
title: 測試人員的角色 - SWE、SET、TE
date: 2014-03-31 08:14:00
tags: [testing, career, test-automation, testability]
---

大家都知道 Google 在測試這個領域劃分了三個角色 - SWE、SET 跟 TE，不過似乎多數人並沒有沒弄清楚這些角色所被賦予的責任，以及不同角色之間的合作關係。

<!-- more -->

## RD 與 QA

先來看 SWE 跟 TE 這兩個角色：

 *  Software Engineer (SWE) - 一般認知的 RD

    除了開發產品功能與修正 bug 之外，RD/SWE 還得撰寫 unit test，尤其關注 _code quality_。

    如果 QA/TE 開始進行整合測試後，仍能發現一些顯而易見的錯誤 (尤其是那些無關系統整合的錯誤)，就代表著 RD/SWE 在測試這一塊做得不夠確實。

 *  Test Engineer (TE) - 一般認知的 QA

    * 產品專家 (product experts)，尤其關注 _product quality_。
    * 以黑箱測試 (black-box testing)跟 end-to-end testing 為主。
    * 從使用者的角度出發，著重在使用情節 (user scenarios)、效能、安全性等更高層級的測試上。
    * 撰寫自動化測試 (test automation)，讓測試工作更有效率。

## 合作關係

測試並非全是 QA/TE 的責任，對那些 unit test 就可以找出的錯誤，QA/TE 的角色只是幫忙 double check 而已。如果 RD/SWE 可以確實利用 unit test 顧好 code quality，QA/TE 的測試工作就比較不會被一些很基本的錯誤給絆住，減少遭遇 "修這裡、壞那裡" 的挫折，也才能夠把心力放在 user scenarios、效能、安全性等更高層級的測試上。 

> TEs act as a double-check on the diligence of the developers. **Any obvious bugs are an indication that early cycle developer testing was inadequate or sloppy.** When such bugs are rare, TEs can turn to their primary task of ensuring that the software runs common user scenarios, is performant and secure, is internationalized and so forth.
>
> [Google Testing Blog: How Google Tests Software - Part Two](http://googletesting.blogspot.tw/2011/02/how-google-tests-software-part-two.html)

## RD/SWE 與 QA/TE 之間

在 RD/SWE 與 QA/TE 之間，Google 提出了另一個 Software Engineer in Test (SET) 的角色。不過這個角色似乎經常遭到誤解，多數人對它的認知就是 "寫 automation 的 QA/TE"，你或許發現 test automation 的責任已經被歸予 QA/TE，那麼拿掉 test automation 的 SET 還剩什麼？

> The SET or Software Engineer in Test is also a developer role except their focus is on _testability_. They review designs and **look closely at code quality and risk**. They **refactor code** to make it more **testable**. SETs write unit testing frameworks and automation. They are **a partner in the SWE code base** but are more concerned with increasing quality and **test coverage** than adding new features or increasing performance. 
>
> SETs are developers that **provide testing features**. A framework that can isolate newly developed code by simulating its dependencies with **stubs, mocks and fakes** and submit queues for managing code check-ins. In other words, **SETs write code that allows SWEs to test their features**. Much of the actual testing is performed by the SWEs, SETs are there to **ensure that features are testable** and that the SWEs are actively involved in writing test cases.
>
> [Google Testing Blog: How Google Tests Software - Part Two](http://googletesting.blogspot.tw/2011/02/how-google-tests-software-part-two.html)

---

> *So what exactly do SETs do?*
>
> SETs are SWEs who are really into testing. We **help SWEs design and refactor their code so that it is more testable**. We **work with test engineers (TEs) to figure out how to automate difficult test cases**. We also **write harnesses, frameworks and tools to make test automation possible**. SETs tend to have the best understanding of **how everything plays together** (production code, manual tests, automated tests, tools, etc.) and we have to make that information _accessible_ to everyone on the team.
>
> [Google Testing Blog: Covering all your codebases: A conversation with a Software Engineer in Test](http://googletesting.blogspot.tw/2012/08/covering-all-your-codebases.html)

---

> The TE or Test Engineer is the exact reverse of the SET. It is a a role that puts testing first and development second. Many Google TEs spend a good deal of their time writing code in the form of **automation scripts** and code that drives usage scenarios and even mimics a user.
>
> [Google Testing Blog: How Google Tests Software - Part Two](http://googletesting.blogspot.tw/2011/02/how-google-tests-software-part-two.html)

 * 尤其關注 _testability_ 跟 _test coverage_。
 * 建構 test automation framework/infrastructure，並協助 QA/TE 解決 test automation 上遇到的難題、開發相對應的工具。
 * 跟 RD/SWE 合作設計出可測試 (testable)的程式，並協助開發 RD/SWE 撰寫 unit test 時會用到的 [mocks、fakes、stubs 跟 dummies 等](http://xunitpatterns.com/Mocks,%20Fakes,%20Stubs%20and%20Dummies.html)。
 * 跟 RD/SWE 在 code base 上有密切的合作，可以幫忙注意 _code quality_，也可以從白箱測試 (white-box testing)的角度切入，給 QA/TE 提供測試上的建議。

> <i class="fa fa-lightbulb-o fa-3x"></i>
> 在 [Google Job](https://www.google.com/about/careers/search/#t=sq&q=j&j=software+engineer+in+test) 有許多 Software Engineer in Test 的職缺可供參考。

## SET 的定位

事實上，SET 並沒有在 test automation 這件事上缺席，而是居於 RD/SWE 與 QA/TE 之間，必須同時協助 RD/SWE 的 unit test 與 QA/TE 的 test automation，最重要的任務就是確保 _testable_/_testability_：

 * 對 unit testing 而言，如果一開始沒有意識到 testability 的重要性，等程式開始出現 "改這裡、壞那裡" 的狀況時才開始想補 unit test，通常會發現程式本身並非 testable，必須對程式做幅度不小的調整，才能開始進行 unit testing，但少了 unit test 這張安全網，這種近似於 refactoring 的翻修，心臟得要夠強才行，在實務上能獲得多少支持也是一個問題...

 * 對 test automation 而言，最理想的方式當然是從 UI 進行操作 (將手動測試自動化)，但有時基於技術上的限制或是成本的考量，取捨後得要對程式進行一些修改 (或額外提供測試上會用到的功能)，好讓 automation script 遶開實作上比較困難的部份。

## 國內的狀況

仔細對照國內相關工作機會的內容，會發現 Software Engineer in Test 的工作多半是 SET + QA/TE，或者根本就是 QA/TE 的工作硬是冠上 SET 的頭銜而已，顯然 SET 在台灣有點被誤用或濫用了？徵才時常有意或無意地將 QA/TE 寫成 SET，畢竟 Software Engineer in Test 這個詞帶有 "Software Engineer"，相對於 QA/TE 比較容易吸引到有 programming 背景的工程師。

對這幾個角色有基本的認識之後，希望有助於在 RD/SWE、QA/TE、SET 不同的工作機會中做出正確的判斷。

