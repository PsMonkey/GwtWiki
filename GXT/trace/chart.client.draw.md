DrawComponent
=============

基本上可以視為 `Surface` 的 wrapper，也就是把 Surface 給 component 化。
維護 `Sprite[]` 的責任在 `Surface` 身上。


運作原理
--------

假設實際繪圖動作只有 `Surface.draw()`，
由此從 call hierarchy 反推 `DrawComponent` 的行為，
則繪圖動作進入點為 `render()`←`redrawSurface()` / `redrawSurfaceForce()`。
（`redrawSurfaceForce()` 暫時不理他，因為只有 `SVG.getHiddenSVG()` 確定有用到，不是 cross-surface 所以追下去沒啥意義）。
`redrawSurface()` 的觸發點只有一個：`onResize()`。

什麼時候會觸發 `onResize()` 呢？（好像不該在這裡追？）
基本上 `Component.setPixelSize()` 跟 `Component.setSize()` 都會。
在某些情況下（細節略 [逃]），`setSize()` 會導向 `setPixelSize()`。
再後頭就先不追了...... Orz


method
------

### constructor ###

預設大小是 500 * 500，底色是白色

`createSurface()`（就是 `Surface.create()`）會直接對 `surface.width` 給值（跟 drawComponent 一樣大）
然後在 `setPixelSize()`→`setSurfaceSize()`
又會改成 drawComponent 扣掉 border、padding 的大小，去呼叫 `surface.setWidth()`？
這有點 WTF 阿？


### render() ###

毫無反應，呼叫 `surface.draw()`


### onResize() ###

* super.onResize()（基本上只是調 mask 大小）
* redrawSurface()


### redrawSurface() / redrawSurfaceForce() ###

都是呼叫 render，只是 redrawSurface() 用 scheduleDeferred() 的方式呼叫。


______________________________________________________________________


Surface
=======

> 暫時只討論 child class 是 `SVG` 的狀況
> `VML` 是肯定不會去管他的 [茶]，`Canvas2D` 等變成正式版再說。


method
-------

### draw() ###

基本上只有處理 background color，其他 sprite 還是要看實做 `SVG.draw()`（最後會再繞回 `SVG.renderSprite()`）。
在這裡可以發現，background color 也是由一個撐滿大小的 `RectangleSprite` 來做出效果。


### renderAll() ###

* 對每一個 Sprite 作 `SVG.renderSprite()`


SVG
===

`DomSurface` 先跳過...


method
------

### draw() ###

* 呼叫 `Surface.draw()`
* 確保 `surfaceElement` 初始化。
	這邊會發現 SVG 層也有一個 bgRect ＝＝"
* `Surface.renderAll()`


### renderSprite() ###

* `surfaceElement` 如果是 null 就 return。
	問題是為什麼 `surfaceElement` 會是 null @_@?
* 如果 sprite 的 `XElement` 不存在，則 `createSprite(sprite)`
* 如果 sprite 都沒有改變，就 return。
	* 反之，一定會作 `applyAttributes()` 然後 clear dirty flag，
	至於會不會作 `applyZIndex()` 跟 `transform()` 則是看對應 dirty flag。

	
### createSprite() ###

從這裡可以看得出來，`SVG` 基本上只能 supprot：

* `CircleSprite`
* `EllipseSprite`
* `ImageSprite`
* `PathSprite`
* `RectangleSprite`
* `TextSprite`

呼叫 `applyZIndex()`


### applyZIndex ###

基本上 SVG 並沒有 z-index 的屬性，純粹就是先掛在 DOM 上的就在上面，後掛在 DOM 的就在下面。
於是乎 `applyZIndex()` 的內容（看起來）就純粹是以 `Sprite` 的 `zIndex` 來排出上下的關係。


----------------------------------------------------------------------


Sprite
======

預設 `zIndex` 是 10

又有 `Sprite(Sprite)` 又有 `copy()`，實在令人困惑。

乍看之下 `Sprite` 裡頭沒有定義座標 `(X, Y)` 的 field 有點不合理，
但是看到 `CircleSprite`，因為是圓心所以是 `(centerX, centerY)` 就釋懷了，
等到看到 `PathSprite` 這神奇的 sprite 就完全崩潰了...... [死]