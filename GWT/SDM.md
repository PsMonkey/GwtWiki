> # Super Dev Mode #

修改 gwt.xml，目前測試結果是一定得要重開 SDM。（DevMode 好像可以不用）

`Dev Mode On` bookmark 內容主要是把現在的頁面卡一段 `<script src="http://localhost:9876/dev_mode_on.js" />`，
所以 code server 的 IP / port 號改變的話 bookmark 就得重拉一次。


RPC 突發異常的解法
------------------

如果時常發生 GWT RPC 無法運作，JSP container 炸出來的錯誤訊息大意是：

1. 找不到 `UNIQUE_ID.gwt.rpc` 這個檔案
1. `FOO_POJO` 沒有繼承 `IsSerializable`，沒辦法 serialize

那麼在 JSP container 啟動的參數加上 `-Dgwt.codeserver.port=PORT_NUMBER`（預設值是 9876），
基本上就可以正常運作。

背後的原理在於 `RemoteServiceServlet` 有針對 SDM 作特製，
會試圖嘗試從 code server（`http://localhost:PORT_NUMBER/policies/UNIQUE_ID.gwt.prc`）取得 RPC policy 相關檔案。

Reference：

* 官方解釋：https://code.google.com/p/google-web-toolkit/issues/detail?id=7522#c12
* RemoteServiceServlet commit：https://gwt.googlesource.com/gwt/+/9faa03e06112984dd4ac5ff100a2515c2d3f74f8/user/src/com/google/gwt/user/server/rpc/RemoteServiceServlet.java