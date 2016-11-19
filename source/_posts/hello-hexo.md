---
title: Hello, Hexo!
date: 2016-11-19 07:29:11
tags: [hexo, testcorner, markdown, github-pages, travis-ci]
---

大概快兩個月前，開始萌生要為 [Test Corner][tc] 架設一個專屬部落格的念頭，當時的想法是採用 [GitHub Pages][gh-pages] 並以 `testcorner.github.io` 做為入口。

用 `.github.io` 感覺就是潮潮的 (其實是為了跟 [Test Corner @ GitHub][tc-github] 接軌)，當然有 `testcorner.io` 更好，只是買 domain 又是一筆費用；目前 Test Corner 完全沒有編列任何預算，事實上是一毛錢都沒有，所有的活動都是靠大家熱情贊助，只好想辦法繼續承襲這個優良的傳統 >_<

直到上個星期，由於 12/01 [Test Corner #7][tc7] 聚會的時間快到了，希望開始為活動留下一點記錄，才開始逼自己要採取一些行動；過去的活動都只有影片、照片分別放在 [YouTube][] 跟 [Flickr][]，沒有再花時間整理，實在有點可惜。

就這樣，在路上遇見了 [Hexo][] ...

 [tc]: https://www.facebook.com/groups/test.corner/
 [gh-pages]: https://pages.github.com/
 [tc-github]: https://github.com/testcorner
 [tc7]: http://testcorner.kktix.cc/events/testcorner7
 [youtube]: https://www.youtube.com/channel/UCv3jyotTJNKhkDyvB6_fvIA
 [flickr]: https://www.flickr.com/photos/testcorner/
 [hexo]: https://hexo.io/

<!-- more -->

## 挑選平台

首先就是挑選平台問題，主要有幾點考量：

 * 可以自訂或選擇樣板。
 * 可以多人編輯，不需要共用一個帳號。
 * 可以用 Markdown 撰寫、離線編輯 (加分)。
 * 可以分析流量，例如 [Google Analytics (GA)][ga]。
 * 可以評論，例如 [Disqus](https://disqus.com/)
 * 可以分享，例如 [AddThis][]

由於之前都是用 [WordPress][wp]，上述 GA、Disqus、AddThis 也都支援，在[科科和測試][kksqa-wp]上多人編輯的體驗也還算不錯，再加上[內建支援 Markdown][wp-markdown]，自然就成為我個人的首選。不料在開始建立站台時，才發現 `testcorner.wordpress.com` 已經被他人佔用，嘔的是[上面只有一篇貼文，而且時間還是 2007-07](https://testcorner.wordpress.com/) ~~(一種佔著茅坑不拉屎的概念)~~。

 [ga]: https://analytics.google.com/
 [disqus]: https://disqus.com/
 [addthis]: http://www.addthis.com/
 [wp]: https://wordpress.com/
 [wp-markdown]: https://en.support.wordpress.com/markdown/
 [kksqa-wp]: https://kkboxsqa.wordpress.com/

沒辦法，就是無法接受諸如 `testcornerblog.wordpress.com`、`testcorner2.wordpress.com` 這類的建議。若要自訂 domain，除了購買 `testcorner.io`，WordPress 也要昇級到 Personal Plan 以上，這些都不是一次性的費用... 所以開始尋求其他的可能性。

11/15 在 FB 社團[蒐集大家的想法](https://www.facebook.com/groups/test.corner/permalink/1439552302729341/)：

> 想幫 Test Corner 建個 blog，可以記錄聚會的點滴，或許會發展出一些專欄也說不定 ...
>
> 結果發現 testcorner.wordpress.com 已經被佔用 (怎麼搶過來?)，不知道大家有什麼點子？ WordPress 自己推薦 testcornerblog.wordpress.com 覺得不太優？
>
> 補充一下，會選 WordPress 是因為它直接支援 Markdown 寫作，還是有更推薦的平台？ 

首先謝謝 [York Wu][york] 提議 [Medium][]，很久之前就想多瞭解一點 Medium，因為有越來越多的人透過它來*分享自己的想法*。

不過在看了一些 Medium 跟 WordPress 的比較之後，才知道兩者的定位不太一樣，並不適合拿來比較，前者重在讓所有的 writer 形成一個 community，讓 writer 可以找到其他 writer (discoverability)，因此幾乎不提供客制化 (customization) ，不能更換樣板就是最顯而易見的例子，而後者更適合用來經營品牌 (branding)，由於目前的想法比較偏 site/blog，所以 Medium 並不適合我們的需求。

> Neither WordPress or Medium are accurate replacements for the other. WordPress is better for a custom site/blog and the latter better for content discovery amongst other writers.
>
> [CMS Comparison: WordPress Vs. Medium [Infographic] WP Engine Blog](https://wpengine.com/blog/wordpress-vs-medium-infographic/)

而 [Dca Hsu][dca] 則提議 GitHub Pages + Hexo，在這之前 GitHub Pages 我只跟 [Jekyll][] (有不愉快的架設經驗，效能好像也令人垢病？)、[HubPress][] (用 [AsciiDoc][] 撰寫) 搭配使用過。

 [york]: https://www.facebook.com/york.wu
 [dca]: https://www.facebook.com/dca.hsu
 [medium]: https://medium.com/
 [jekyll]: https://jekyllrb.com/
 [hubpress]: http://hubpress.io/
 [asciidoc]: http://www.methods.co.nz/asciidoc/

## 遇見 Hexo!

這是我第一次聽到 Hexo，看了一下 [StaticGen][] 的排行，Hexo 受歡迎的程度跟 [spf13](https://github.com/spf13) 開發的 [Hugo][] 不相上下。

![Top 4 - StaticGen](/images/hello-hexo/staticgen-raning.png)

一開始發現，可供選用的[主題還真不少][hexo-themes]，而且多數的範例都有中文，這著實引起了我的好奇心，這才知道這個專案其實是台灣人 [Tommy Chen][tommy] (陳嘉輝) 主導的，在 [GitHub][hexo-github] 上可以得到上萬個星星、網路上的討論度極高 (包括華人地區)，當然要來試試。

Hexo 安裝、使用都很簡單。玩了半天後，決定就是它了 (Hugo 之後有時間再來研究吧)，尤其是找到 [NexT][next] 這個超威的主題之後。

 [staticgen]: https://www.staticgen.com/
 [spf13]: https://github.com/spf13
 [hugo]: http://gohugo.io/
 [hexo-themes]: https://hexo.io/themes/
 [hexo-github]: https://github.com/hexojs/hexo
 [tommy]: https://zespia.tw/
 [tommy-fb]: https://www.facebook.com/tommy351
 [next]: http://theme-next.iissnan.com/

首先就把九月初才剛上線的個人部落格 `imsardine.github.io`，從 HubPress 換成 Hexo，而這就是第一篇貼文，之後再跟大家分享 Hexo 跟 NexT 的學習心得。

## 套用到 Test Corner 上可能的問題

試想幾個多人參與編輯的情況，可能會遇到幾個問題：

 * NexT 主題不支援顯示文章的作者。
 * 不見得每個人都熟悉 Hexo 的安裝跟使用，可以整合 [Travis CI][travis-ci]，將網站發佈的流程自動化。
 * 之後得建立一套流程，將 `master` branch 鎖定，所有的異動都要另開 feature/topic branch，然後走 pull request (PR)，另外 `gh-pages` branch 也只有少數人 (例如 Travis CI) 可以直接寫入。

 [travis-ci]: https://travis-ci.org/

