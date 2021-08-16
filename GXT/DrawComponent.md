DrawComponent
=============

可以視為 GXT 的 canvas，更正確地說是 canvas container / wrapper。
實際上的 canvas 是 `Surface`，會用 deferred binding 的方法抽換實做，
目前大抵上預設是 `SVG`，遇到 IE6～8 會替換成 `VML`，
另外還有隱藏實驗版（JavaDoc 找不到）的 `Canvas2d` 版。
因為 wrapper 過，所以基本上可以不用理會底層實做，GXT 會提供一個一致的 API。

目前建議都作 DrawComponent.setBackground(Color.NONE)，
真的要 background 再自己指定一個 `RectangleSprite`。
不然（似乎是在） resize 的時候，background 的 sprite 會莫名其妙跑到前面去，
讓某些 sprite 被蓋掉、即使那些 sprite 給了很大的 z-index。

`ImageSprite` 沒設定過座標，在視覺上會等同於在 (0, 0)。
不過這會影響 `getBBox()` 回傳的座標仍然是 (Double.NaN, Double.NaN)，
導致 `DrawComponent` 的 sprite handler 會無法觸發。
其他 sprite 應該大同小異，所以簡單地說就是：「`Sprite` 都要設定座標」。


判斷游標離開 DrawComponent
--------------------------

雖然游標仍然在 `DrawComponent` 上，不過如果上頭有多個 sprite，
游標在離開 sprite 就會呼叫 `DrawComponent.onMouseOut()`。
所以，要真正判斷游標是否離開 DrawCompont 的方法是

```Java
@Override
public void onMouseOut(Event event) {
	super.onMouseOut(event);

	if (getElement().getBounds().contains(event.getClientX(), event.getClientY())) {
		//表示還在 DrawComponent 內
	}
}
```

Chart
=====

`chart.redrawChart()` 之類的繪圖時刻如果炸 `java.lang.NegativeArraySizeException` 之類狀況，
先檢查一下 chart 的大小是否合理。如果 chart 沒有大小（1 * 1），那麼炸 exception 好像也很合理。

`NumericAxis.clear()`（實際是 `CartesianAxis.clear()`）不會清 fields，
但是不清 fields 好像也不會怎樣？細節不明 Orz

如果做了 `NumericAxis.setMaximum()` 等動作，
則相關的 series 必須要設定正確的 `setYAxisPosition()`，
否則會出現 series 跟 axis 對不上的各種靈異狀況。
反之如果 axis 沒做、series 不用設定也不會出問題。


TimerAxis
---------

如果用 `TimeAxis` 作 X 軸（不確定作 Y 軸會怎樣 XD），
直接設定 `setStartDate()` 跟 `setEndDate()` 就會幫你過濾掉不在時間範圍內的 data，
不用自己整理 store。

`TimerAxis` 的時間區間過濾機制會影響 chart 的 `getCurrentStore()`（`substore`）的值。
這在「清空 / 重塞 store、又變更其他 axis」的前提下，`chart.redrawChart()` 會炸。
因為 render 實際處理的是 `getCurrentStore()`，如果舊的 store 格式跟新的 axis 格式對不上，就會出錯。
解法是在變更其他 axis 之後作 `TimerAxis.drawAxis(false)`（傳入 true / false 好像沒差），
在 `drawAxis()` 裡頭的 `applyData()`（`TimeAxis` 有 override）會重新計算 / 設定 `currentStore`。
當然這有點浪費，因為在 `chart.render()` 裡頭其實會對各個 axis 作 `drawAxis(false)`，
只能說理論上沒辦法控制 `TimerAxis` 一定要先作，`applyData()` 是 protected 所以沒辦法直接呼叫
前提觸發的條件又不是很 general，只好覆蓋新鮮的肝臟，結束這一回合 T__T。


LineSeries
----------

預設情況下，若第 n 個點的值為 `Double.NaN`，
則會直接連起 n - 1 跟 n + 1。
設定 `setGapless(false)` 就會斷掉形成 gap。


Legend
------

`Legend` 的資料來源是從 `Series.setYField()` 傳入的 `ValueProvider` 的 `getPath()`。
