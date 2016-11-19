---
title: HubPress 初體驗
date: 2016-09-03 08:54:00
updated: 2016-11-19 17:02:00
tags: [asciidoc, github-pages]
---

[Markdown][] 或許是因為相對容易上手而被廣泛採用 (GitHub、Stack Overflow 等)，但一方面也因為語法過於簡單的關係，在撰寫技術文件時顯得有些侷限 (聽聽看 [Pro Git 的作者 Scott Chacon 怎麼說][scott-says])，所以我一直以來都偏好使用 [AsciiDoc][]，不過似乎國內的使用者並不多？

> <i class="fa fa-lightbulb-o fa-3x"></i>
> 網路上有許多技術文件、書籍，都是用 AsciiDoc 寫的，AsciiDoc 首頁 [Documents written using AsciiDoc][adoc-samples] 一節可以找到更多的應用實例。

 [markdown]: http://daringfireball.net/projects/markdown/
 [asciidoc]: http://www.methods.co.nz/asciidoc/
 [scott-says]: https://medium.com/@chacon/living-the-future-of-technical-writing-2f368bd0a272
 [adoc-samples]: http://www.methods.co.nz/asciidoc/index.html#X6

重新找了一下可以用 AsciiDoc 寫 blog 的方案，意外發現 2015/02 開始的 [HubPress][]。使用一天的心得：

 [hubpress]: http://hubpress.io/

<!-- more -->

## 容易上手

只要 fork [`HubPress/hubpress.io`][hp-github] 這個 GitHub repository 到自己的帳號底下，[簡單調整幾個設定][hp-start] 之後，就可以開始存取 `https://<USERNAME>.github.io`。

做為一個 web application (背後採用 [Asciidoctor.js][adoc-js])，在本地端不需安裝任何東西，相較之前安裝 [Jekyll][]、[Awestruct][] 的經驗，真的簡單很多。

 [hp-github]: https://github.com/HubPress/hubpress.io/
 [hp-start]: https://github.com/HubPress/hubpress.io#getting-started
 [adoc-js]: https://github.com/asciidoctor/asciidoctor.js
 [jekyll]: https://jekyllrb.com/
 [awestruct]: http://awestruct.org/

(fork 這種佈署方式好特別，也難怪 fork 數會高逹 2,715，但同時間星星數也有 2,592，顯見滿意度還頗高的)

## 整合 Disqus 與 Google Analytics

分別在 HubPress admin console 的 Settings 頁面填上 Disqus shortname 及 Google Analytics 的 tracking ID 即可。

## 內建佈景主題 (theme)

[有幾個佈景主題][hp-themes]可以選用，不過無法預覽，每次換主題到會產生新的 commit (會動到 `hubpress/config.json`)，有點惱人。(換來換去，似乎還是預設的 Casper 最順眼)

 [hp-themes]: https://github.com/HubPress/hubpress.io/tree/master/themes

## 對中文的支持還不完整

文件標題跟標籤 (`hp-tags`) 目前還不能用中文 ([#43][hp-issue-43], [#145][hp-issue-145])，因為會成為檔案或目錄名稱。

標籤我習慣用英文 (小寫、單字用 `-` 分開，例如 `xml-rpc`)，所以問題不大，至於中文標題則可以搭配 `hp-alt-title` (alternative title) 這個 document attribute 重新定義 (檔案) 名稱。例如：

```
= HubPress 初體驗
:hp-alt-title: hubpress-first-time
```

之後產生的 HTML 檔名就會是 `YYYY-MM-DD-hubpress-first-time.html`。

 [hp-issue-43]: https://github.com/HubPress/hubpress.io/issues/43
 [hp-issue-145]: https://github.com/HubPress/hubpress.io/issues/145

## 可以在本地端編輯

可以離線編輯 `_posts/*.adoc` (官方推薦 [Atom][] 跟 [AsciidocFX][])，但要發佈貼文還是得透過 admin console。

 [atom]: https://atom.io/
 [asciidocfx]: http://asciidocfx.com/

## 不支援圖示？

設定 `:icons: font` 沒有作用，單純用 `:icons:` 也沒有圖示。

這一點真的很可惜，尤其 [Asciidoctor] 的 [Font Awesome][font-awesome] 還真的滿有質感的。

 [asciidoctor]: http://asciidoctor.org/
 [font-awesome]: http://asciidoctor.org/docs/user-manual/#icons

## Syntax highlighting 在許多 theme 上仍有問題

[#407](https://github.com/HubPress/hubpress.io/issues/407)。

## 結論

雖然還是有許多問題，但整體上撰寫、發佈的流程都還算流暢，因此接下來貼文都會利用 HubPress 發佈在這裡，有什麼心得再跟大家分享。

[Update] 2016-11-19 在這之後用 HubPress 寫了幾篇文章後，感覺 HubPress 的管理介面在文章數多了之後，極有可能出現狀況。幾天前，因緣際會遇見了 Hexo，不到一天的時間就[決定改用 Hexo 取代原有的 HubPress](/2016/11/19/hello-hexo/)。

