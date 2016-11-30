---
title: tee 指令的用法
date: 2014-03-19 06:58:56
tags: [linux, command]
---

## 簡介

`tee` 的行為很特別，一方面會從 stdin 讀取資料，另一方面又會將資料同時寫到 stdout 跟 `FILE`，就像是 _一分二的轉接頭_。

    tee [-a] <FILE>

<!-- more -->

> <i class="fa fa-lightbulb-o fa-3x"></i>
> 實際上 `tee` 的名稱就是這麼來的，tee 指的就是 "T 字管" 或 "三通管"。
>
> ![T 字管](/images/tee.png)
> (圖片來源：[tee (command) - Wikipedia, the free encyclopedia](http://en.wikipedia.org/wiki/Tee_(command))

## 常見的用法

### 記錄螢幕輸出的訊息

    <COMMAND> [2>&1] | tee [-a] <FILE>

讓 `COMMAND` 持續寫出到 stdout 的內容，除了即時顯示在螢幕之外，也一併記錄檔案 `FILE`。其中 `2>&1` 可以將 stderr 的內容轉到 stdin，讓 `tee` 一併做處理。

以 `adb logcat` 為例，讓持續寫出的 logs 除了顯示在螢幕之外，也一併寫到 `/tmp/logcat`。

```
$ adb logcat -v threadtime -b main | tee /tmp/logcat
03-19 06:58:36.470  5220  5222 D dalvikvm: GC_CONCURRENT freed 970K, 11% free 15396K/17287K, paused 4ms+4ms, total 74ms
03-19 06:58:38.685  1900  1923 E DataRouter: After the usb select 
03-19 06:58:38.685  1900  1923 E DataRouter: read usb data[len:1]
...

$ cat /tmp/logcat
03-19 06:58:36.470  5220  5222 D dalvikvm: GC_CONCURRENT freed 970K, 11% free 15396K/17287K, paused 4ms+4ms, total 74ms
03-19 06:58:38.685  1900  1923 E DataRouter: After the usb select 
03-19 06:58:38.685  1900  1923 E DataRouter: read usb data[len:1]
...
```

## 常用的選項

 *  `-a, --append`
    附加到 `FILE` 後面，而非覆寫。

