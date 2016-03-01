> # 瀏覽器差異 #


Video
=====

* GWT 2.7（理論上跟 GWT 版本無關）

在 PC（Chrome 48）上，沒設定 `autoplay` 會顯示第 1 個 frame的畫面，設定 `autoplay` 會自動開始播放。

在 mobile device（Chrome 48）上，沒設定 `autoplay` 會顯示一片黑，設定 `autoplay` 也不會自動開始播放。
在 `CanPlayThroughHandler` 當中也沒辦法呼叫 `Video.play()`，必須在其他外部 event handler 才能呼叫。
