---
title: “Can you hear me?” - Surviving Audio Quality Testing
tags: [gtac, test-automation, audio]
categories:
  - Google Test Automation Conference (GTAC)
  - 2016
date: 2016-11-22 21:54:43
---

這個場次由 [Dan Hislop][dan] 與 Alexander Brauckmann 跟大家分享在 [Citrix][] 是如何進行音質測試 (audio quality testing)。

 [dan]: https://www.linkedin.com/in/dan-hislop-075b6315
 [citrix]: https://www.citrix.com/

Citrix 有許多產品跟即時通訊 (real-time communication) 有關，音訊的品質對這類產品至關重要，所以在公司內部提出了一個通用方案，以解決不同產品在音質測試方面共通的問題。

一開始，Alex 問了一句 Can you hear me? 真是絕妙的開場 :)

<!-- more -->

## 音訊測試的挑戰

Citrix GoTo 系列產品每個月要處理的音訊長達 14 億分鐘 (audio minutes)，在音訊處理量不斷成長的同時，也努力從不同地方著手讓音訊相關的客訴 (customer escalation) 減少。

![問題的規模與品質提昇](/images/gtac-2016-can-you-hear-me/scale-and-quality.png)
(圖片來源：[Google Test Automation Conference][slide])

 [slide]: https://www.youtube.com/watch?v=2yN53k9jz3U&t=4h54m8s

沒有聲音對這類產品是個大忌 (deal breaker)，可以想見音訊測試的重要性。雖然 Citrix 內部有一些音訊的專家，但這些人也都只有兩隻耳朵，所以要想辦法克隆 (clone) 他們的專業，並**整合進自動化測試，讓許多非專業人士也可以利用**。

手動測試音質除了耗時之外，**最大的問題是對音質的判斷是_主觀 (subjective)_ 的，透過自動化，可以讓這件事變得客觀 (objective) 且可重複執行 (repeatable)**。

![音訊測試的挑戰](/images/gtac-2016-can-you-hear-me/challenges.png)
(圖片來源：[Google Test Automation Conference][slide])

## 測試金字塔

關於音訊的測試，分為 3 個層次，一樣可以用金字塔來表達－越上層越難自動化，越需要音訊方面的專業。

![音質測試金字塔](/images/gtac-2016-can-you-hear-me/pyramid.png)
(圖片來源：[Google Test Automation Conference][slide])

Dan 以水管工程 (plumbing) 類比，由下往上逐層說明：

### Audio Functionality

指_基本操作_的測試。例如按下靜音鍵 (mute) 會變紅燈，取消靜音 (unmute) 則會變綠燈，但聲音是不是就真的不會傳到另一端，不是這個層級要關心的。

以水管工程類比，假設要從房子外面接水管進到屋內的水龍頭，這裡只是檢查管子是否有接好，並沒有真的打開水閥。

### Audio Presence

指_有沒有音訊_的測試。延續上面的例子，按下靜音鍵時，另一端聽不到聲音，反之取消靜音時，另一端就要能聽得到聲音，但聲音是否清楚，不是這個層級要關心的。

以水管工程類比，這次真的打開水閥，檢查水是否能順利地從屋內的水龍頭流出來，不管水是否可以飲用。

### Audio Quality

指_音訊好壞_的測試，能否聽得清楚對方在講什麼。

以水管工程類比，則是檢查水龍頭流出的水，是否可以飲用。

## 音質測試

音質測試屬於 End-to-End (E2E) 層級，主要的流程如下：

![音質測試四個元素](/images/gtac-2016-can-you-hear-me/audio-quality-testing.png)
(圖片來源：[Google Test Automation Conference][slide])

 1. 準備數個話質清析的音訊檔，以確保不同語音 (voice) 都有不錯的效果。
 2. 為發話端 (client 1) 安排一個假的麥克風 (mock mic)，直接注入 (inject) 音訊檔，就像有人透過麥克風講話一樣。
 3. 為受話端 (client 2) 安排一個假的揚聲器 (mock speaker)，直接收錄 (capture) 另一端傳來的音訊，就像有人戴著耳機在聽一樣，並將它存成另一個音訊檔。
 4. 將步驟 2 輸入的音訊檔，及步驟 3 輸出的音訊檔，一起丟給後端的 scoring server 評分比較，就能知道輸出音訊的品質。

## AQA Service

上述的 scoring server，指的就是內部的 Audio Quality Analysis (AQA) Service，提供不同的 API，方便現有的自動化測試直接整合－上傳音訊檔，然後取得評分，以決定測試成功或失敗。

![音質測試四個元素(細節)](/images/gtac-2016-can-you-hear-me/audio-quality-testing2.png)
(圖片來源：[Google Test Automation Conference][slide])

其中 [POLQA][] (唸做 [po-ca] 波卡) 是第三方元件，用來做客觀的話質量測 (voice quality measurement)，但必須先取得授權 (license) 才行 (可以在這份[清單][polqa-license]裡找到 Citrix Online Germany GmbH)，由於 POLQA 授權會綁定某些機器，所以 AQA Service 也能一併處理掉授權上的限制。

> Consequently, any commercial entity that wishes to integrate a technology like POLQA within a product, and/or intends to use POLQA in their commercial enterprise must obtain a licensed right to use the essential patents held by the POLQA coalition. Except for educational use (e.g. non-commercial projects at universities) and contributions that yield at the development of new ITU-T standards, or amendments and revisions of P.863, **any use of POLQA implies a license agreement**.
>
> [Frequently Asked Questions - POLQA][polqa-faq]

 [polqa]: https://en.wikipedia.org/wiki/POLQA
 [polqa-faq]: http://www.polqa.info/information/faq.html
 [polqa-license]: http://www.polqa.info/licensees.html

SQA Service 只含括各平台通用的部份，所以每個平台上還是得各自實作 mock mic/speaker，可惜這部份沒有提到細節。

### 架構

AQA 的架構如下。以 Node.js 整合原生的 POLQA 演算法、不同的編解碼器 (codec)，包括 [Opus][]、[LAME][] 等。

![AQA Service 架構](/images/gtac-2016-can-you-hear-me/aqa-architecture.png)
(圖片來源：[Google Test Automation Conference][slide])

AQA 對外則提供 [REST][] 與 [GRPC][] API 兩種介面，供不同的程式語言調用。API 的操作方式大致如下：

 1. 建立 job。
 2. 上傳兩個要做比較的音訊檔。(按照現場展示，上傳檔案發生在建立 job 之後)
 3. 啟動 job 開始比較兩個音訊檔。
 4. 取得評分結果。

 [opus]: http://opus-codec.org/
 [lame]: http://lame.sourceforge.net/
 [rest]: https://en.wikipedia.org/wiki/Representational_state_transfer
 [grpc]: http://www.grpc.io/

### POLQA

在音質測試這個部份，AQA 採用 POLQA 演算法評定話質 (voice quality)。POLQA 的全名是 Perceptual Objective Listening Quality Assessment，強調感知 (perception) 與客觀性 (objectivity)。

![AQA Service 架構](/images/gtac-2016-can-you-hear-me/polqa.png)
(圖片來源：[Google Test Automation Conference][slide])

提供兩組訊號 - 輸入 (Reference/Played-out Signal) 與輸出 (Degraded/Recorded Signal)，利用 POLQA 演算法就能求得一個介於 5 (好) 跟 1 (差) 之間的 [MOS (Mean Opinion Score)][mos] 分數，代表一般人所感知到的話質。

 [mos]: https://en.wikipedia.org/wiki/Mean_opinion_score

### Audio Presence Testing

除了音質測試，AQA 似乎也支援 Audio Presence 這層測試，雖然現場展示沒有提到。

![Audio Presence Testing](/images/gtac-2016-can-you-hear-me/audio-presence-testing.png)
(圖片來源：[Google Test Automation Conference][slide])

可以細分為：

 * Frequency Analysis - 能識別出同時間在講話的不同人 (頻率不同)，可以驗證某個人的聲音是否有被播放出來。
 * Speech Presence - 能識別出哪些聲音是語音 (speech)。
 * Amplitude Analysis - 可以檢查某個的人聲音不會太大或太小。

### 現場展示

![Audio Presence Testing](/images/gtac-2016-can-you-hear-me/live-demo.png)
(圖片來源：[Google Test Automation Conference][slide])

下面是測試在 5% 掉封包 (packet loss) 狀況下的音質，並驗證 MOS 分數要在 4.0 以上：

```python
import os
import AQAServiceClient.aqaservice as aqa

url = "http://10.39.199.14:8080"
serviceClient = aqa.AQAServiceClient(url)

# ...

def test_aqaserviceclient_polqa_5loss():
    # create job
    job = serviceClient.createPolqaJob()

    # add files
    job.setReferenceFile("English_Female1_1.wav")
    job.setDegradedFile("English_Female1_1-0-from-Win-7-Reference-to-Win-10-Laptop_5loss.wav")

    # add tags
    job.addTag("gtacDemo")
    job.addTag("original")

    # start job
    evaluation = job.start({})

    # evaluation state. Sync
    evaluationState = evaluation.getState()
    assert evaluationState.getMfMOSLQO() > 4.0
```

AQA 也提供有儀表板 (dashboard) 可以追蹤歷次的測試結果：

![Audio Presence Testing](/images/gtac-2016-can-you-hear-me/aqa-dashboard.png)
(圖片來源：[Google Test Automation Conference][slide])

## 心得

雖然說，音質測試需要音訊方面的專家，但即便是 Ctrix 自己最後也是找第三方 POLQA 合作，顯見音質檢測的技術門檻很高。

### 話質測試

這次分享的內容，與其說是音質測試，更確切地說是話質測試 (voice quality testing)，在[官方 FAQ][polqa-faq] 裡是這麼定位 POLQA 的：

> POLQA®, or “Perceptual Objective Listening Quality Analysis” - is the **next-generation voice quality testing standard** for fixed, mobile and IP-based networks that was adopted in 2011 as ITU-T Recommendation P.863 and successor to P.862/PESQ.

事實上，在 POLQA 首頁跟 FAQ 裡完全沒有用到 _audio_ 這個字，但 _voice_ 跟 _speech_ 倒是被反覆被提到很多次。再加上這份[比較表](http://sevana.biz/pesq-polqa-aqua/)明確點出 POLQA 在 Music 這個項目缺席，POLQA 似乎真的不適合應用音樂的領域。

雖然上面那份比較表提到 [AQuA](http://sevana.biz/products/aqua/technology/) 有支援 Music 這個項目，但感覺就像 POLQA 一樣，不是一個開放的標準，細節不是一般人可以碰觸得到的，授權費或許也相當可觀？

> No, POLQA is not open source and also there is no public source code available for download from the ITU-T website.
>
> [Frequently Asked Questions - POLQA][polqa-faq]

### 音質測試 (音樂)

雖然這次的分享不適用於音樂的音質測試，但還是從 AQA Service 的設計得到了一些啟發，尤其是 mock mic 與 mock speaker 的部份，雖然沒有提到細節，但至少是可以研究的方向。

![音質測試四個元素](/images/gtac-2016-can-you-hear-me/audio-quality-testing.png)
(圖片來源：[Google Test Automation Conference][slide])

在 Q&A 階段有人問道「這樣的測試是否會受語言或方言影響？」

![語言/方言的影響](/images/gtac-2016-can-you-hear-me/languages-dialects.png)
(圖片來源：[Google Test Automation Conference][slide])

Dan 提到背後的原理是_比對波形_ (waveform)，所以就這個測試而言，跟採用的語言是無關的。或許音樂的音質測試也是如此，只要想辦法將輸出的音訊錄製下來，再跟原始音訊做波形比對即可？

就錄制音訊的部份：

 * 從[這個討論](http://stackoverflow.com/questions/17676142/record-android-audio-output)看來，要直接錄製 Android 系統的音訊輸出 (audio output) 是辦不到的。
 * 可以利用 `MediaRecorder` [直接從麥克風收錄 Android 播放出來的聲音](https://developer.android.com/guide/topics/media/audio-capture.html)，但容易受到環境的影響。
 * 可以考慮用音源線將訊號從耳機孔導到電腦的音源輸入 (audio input)，再用電腦將音訊錄制下來，這樣就不會受到環境音的干擾，而且也不受平台的限制。

就比較音訊的部份：

 * 要驗證是否有播放正確的歌曲，不管品質為何 (屬於金字塔 Audio Presence 的層級)，可以參考[聲學指紋][acoustic-fingerprint] (acoustic fingerprint) 與 [worldveil/dejavu][] (作者也說明了它[背後的原理][dejavu-how])。
 * [audiodiff][] 可以比對兩個音訊檔。
 * [PyMedia][] 可以解碼 MP3。

 [acoustic-fingerprint]: https://en.wikipedia.org/wiki/Acoustic_fingerprint
 [audio-fingerprinting]: http://willdrevo.com/fingerprinting-and-audio-recognition-with-python/
 [pymedia]: http://pymedia.org/
 [audiodiff]: http://audiodiff.readthedocs.io/
 [worldveil/dejavu]: https://github.com/worldveil/dejavu
 [dejavu-how]: http://willdrevo.com/fingerprinting-and-audio-recognition-with-python/

上面都是一些初步的想法／發現，等之後有時間再深入研究...

