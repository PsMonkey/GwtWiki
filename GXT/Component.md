`Component` 基本上是 GXT widget 的共同祖先。

`Component` 有提供 `enable()`、`disable()` 也有 setter 形式的 `setEnable()`。
也就是說，有別於 GWT 只有某些 widget 才有 `setEnable()`，
所有的 GXT widget 都可以有 enable / disable 的狀態。
各式 container 的 `setEnable()` 會連小孩也一起呼叫 `setEnable()`。


Container 系
============

（謎之聲：為什麼 `TabPanel` 不是... ＝＝?）


Container
----------

所有的 widget 都可以設定 `HasMargins` 的 layout data，
無論是加在哪一個 container 上都會有 margin 的效果。
這是因為 `Container.onInser()` 會特別檢查 `child.getLayoutData()` 是否為 `HasMargins` 並作處理。


SimpleContainer
---------------

會自動讓小孩的大小跟自己的大小一樣


BorderLayoutContainer
---------------------

假設 `northLD` 的 size 設定為 100、split 設定為 true（北邊區域可以動態調整大小），
調整北邊區域的大小會改變 `northLD.getSize()` 值。


ContentPanel
------------

在 ui.xml 當中作 `contentPanel.addButton()` 的方法：

```XML
	<core:ContentPanel>
		<core:button>
			<foo:Foo />
		<core:button>
		<core:button>
			<foo:Foo />
		<core:button>		
	</core:ContentPanel>
```


原來一個 `<core:button>` 只能塞一個 widget，我之前都誤會它了 [炸]。


Dialog
------

要知道使用者對 `Dialog` 做了什麼事情，是掛 `addDialogHideHandler()` 這個 handler。
從 `DialogHideEvent.getHideButton()` 可以知道是按下哪個 button。
不知道該說巧妙還是該說 WTF...... Orz


### PredefinedButton 的 I18N ###

從 API 上來看，要改變 `PredefinedButton`（對應的 `TextButton`）的顯示字串，
就是透過 `setDialogMessages()` 設定 `DialogMessages`。
實際上，這個 API 根本有問題...... [怒]，

以 `ConfirmMessageBox` 為例，流程大概是

	constructor
		setPredefinedButtons()
			清空既有 button
			createButtons()
				getText()
					讀 getDialogMessages() 設定對應字串


`setDialogMessages()` 純粹是 setter，沒有重設顯示字串，
所以 new 完 instance 再 `setDialogMessages()` 也沒意義。

最直白的 workaround 是

	msgBox.getButton(PredefinedButton.OK).setText("OK 的啦");


但是被嫌棄這樣違背 API 設計 / OO 原則（阿就原本 API 設計爛阿幹 T__T），
所以找出第二種 workaround

	msgBox.setDialogMessages(myDialogMessages);
	msgBox.setPredefinedButton(PredefinedButton.YES, PredefinedButton.NO);


簡單地說，就是再次呼叫 `setPredefinedButtons()`、重新製造 button，
此時就會用 `myDialogMessages` 來設定顯示字串啦。

雖然這某種程度還是破壞了 `ConfirmMessageBox` 的封裝，
不過從程式碼當中可以清楚知道到底有哪些 button，
然後學理上 `myDialogMessages` 整個 project 只會有一份，好像也不錯啦。


AccordionLayoutContainer
------------------------

小孩只能是 `ContentPanel`，而且（應該說「所以」？）
`ContentPanel` 還能塞 `AccordionLayoutAppearance` 來改變 `ContentPanel` 的外觀。 


HBox / VBox
-----------

做了 `setPack()`，會發現小孩的 layoutData 不管怎麼設定 flex 值，
都不會有預其中的大小變化，而是以小孩原本的大小呈現。
目前這是實驗結果，還沒追 source code [死]。


HorizontalLayoutContainer / VerticalLayoutContainer
---------------------------------------------------

`HorizontalLayoutData` 的 width 數值邏輯為
（`VerticalLayoutData` 的 height 比照辦理）：

* x > 1：小孩的寬度就是 x
* x = -1：小孩自己決定自己的寬度（容易錯，使用注意）
* x < -1：小孩的寬度是「爸爸的寬度加上 x」
	換句話說，就是只留 -x 的寬度給兄弟姊妹
* 0 < x <= 1：小孩的寬度是「爸爸**剩下**的寬度乘上 x」。

舉實際的例子，假設一個 `HorizontalLayoutContainer` 的寬度是 1000，
底下有四個小孩、layoutData 的 width 設定值與實際計算完的寬度分別為：

* 100→100
* 0.5→(1000 - 100 - 200 - 600) * 0.5 = 50
* -1（實際上是 200）→200
* -400→600

> ###### 備註 ######
> GXT API 文件上針對 `0 < x <= 1` 這段沒有說明仔細，導致跟實際結果有出入。
> 以上是追 source code 的結果，詳情可以參閱 `HorizontalLayoutContainer.doLayout()`，
> 因為 `0 < x <=1` 的需求，所以要兩次迴圈才有辦法決定（如果有小孩設定 -1 可能不只）。
> 第一次迴圈決定 `pw`，然後第二次迴圈才依照 `pw` 實際指定小孩的寬度。


輸入系
======

ComboBox
--------

`setTriggerAction(TriggerAction.ALL)` 可以在使用者再次要求下拉選單時顯示所有值。
像 `TimeField` 就非常需要... Orz

要注意 ComboBox 的顯示值（或著說 `LabelProvider.getLabel()` 的回傳值）不能是 `\n` 結尾。
顯示上是不會有問題、也會觸發 `SelectionEvent`，
但是在 onblur 的時候不會觸發 `ValueChangeEvent`、ComboBox 的值會消失。

如果 item 的值是 null，顯示時不會觸發 `LabelProvider.getLabel()`，
在 expand 時選單會多一個高度很詭異的空白行。

`setEditable(false)` 好像直接包含 `setSelectOnFocus()` 與 `setForceSelection()` 的功能，
此乃實驗結果，有待 trace code 證明 [死]。
另外，如果搭配 `FieldLabel` 使用，居然點 `FieldLabel` 的 label 也會觸發 focus 效果，
這什麼黑魔法阿阿阿阿... Orz

如果 extend `ComboBox`，然後想要在裡頭用自己定義的 enum `PKG_NAME.Direction`，
在 Eclipse 當中會預設使用 `com.google.gwt.i18n.client.HasDirection.Direction`
（還不用 import... WTF  ＝＝"）。
目前唯一合理的解釋是 `ComboBox` 的祖先 `ValueBaseField` 有實作 
`HasDirectionEstimator` 跟 `AutoDirectionHandler.Target`，
裡頭有 import `com.google.gwt.i18n.client.HasDirection.Direction`，
可是這還是不科學阿阿阿...... Orz

详细说明 `setTriggerAction()`、`setForceSelection()`、`setEditable()`、
 `setSelectOnFocus()` 的作用和相互影响。

* setTriggerAction()是否显示全部下拉框中全部内容。
如果是 TriggerAction.QUERY，只能显示根据 ComboBox 内容筛选过后的内容。 
如果是 TriggerAction.ALL，在输入的时候会筛选对应内容，
但是在按下下拉框按钮的时候会显示全部内容。
默认值：TriggerAction.QUERY

* setForceSelect()是否允许 ComboBox 中的值不在下拉框中。
如果是 true，ComboBox 输入的值不在下拉框中，lose focus 后会清空。
（如果使用setValue()设定一个不在下拉框的输入值不会清空）
默认值：false

* setEditable()是否允许 ComboBox 中直接输入值。
如果是 false，无法在 ComboBox 中直接输入值。
默认值：true

* setSelectOnFocus()是否在 focus 的时候出现反白的效果。
如果是 true，会在 focus 的时候选中 ComboBox 中的值，出现反白的效果
(只在鼠标点击的时候出现效果，松开后效果消失)，使用 Tab 键进入 ComboBox 不会出现这个效果。
默认值：false

* 打V表示符合描述。打X表示不符合描述。
（包括使用 setValue() 的情况）

| setTriggerAction()|setForceSelect()|setEditable()|setSelectOnFocus()|下拉框只显示 ComboBox 筛选过内容|允许 ComboBox 中的值不在下拉框中|允许 ComboBox 中输入值|在 focus 的时候不出现反白的效果|
|:-----------------:|:--------------:|:-----------:|:----------------:|:------------------------------:|:-----------------------------:|:--------------------:|:-----------------------------:|
|TriggerAction.QUERY|true |true |true |V|X|V|X|
|TriggerAction.QUERY|true |true |false|V|X|V|V|
|TriggerAction.QUERY|  X  |false|  X  |V|V|X|V|
|TriggerAction.QUERY|false|true |true |V|V|V|X|
|TriggerAction.QUERY|false|true |false|V|V|V|V|
| TriggerAction.ALL |true |true |true |X|X|V|X|
| TriggerAction.ALL |true |true |false|X|X|V|V|
| TriggerAction.ALL |  X  |false|  X  |X|V|X|V|
| TriggerAction.ALL |false|true |true |X|V|V|X|
| TriggerAction.ALL |false|true |false|X|V|V|V|


DateField
---------

### 觸發 setValue() 的來源 ###

#### DatePicker ####

* 当 new 一个 `DateField` 时，就 new 了一个 `DateCell` 和一个 `DateTimePropertyEditor`；
* 当使用者点击画面上的 `DateCell` 时，会通过 `DateCell.onTriggerClick()` 呼叫 `DateCell.expand()`，在这里 new 了一个 `DatePicker`；
* 因为在 `DatePicker` 的 `Constructor` 中实现了 `sinkEvents()`，而且在 `onBrowserEvent()` 中实现了 `ONCLICK|ONMOUSEOVER` 等的具体操作，所以当画面上发生 ONCLICK Event 则会呼叫 `DatePicker.onClick(Event)`；

##### 按下 today #####

* 使用者在出现的 date picker 上点击【today】时，作 `addSelectHandler()` 让 select event 发生时去呼叫 `selectToday()`，结果呼叫 `DatePicker.setValue(Date, boolean)`，设置 `value` 的值；

##### 點擊日曆上的日期 #####

* 当使用者触发 ONCLICK Event 时，呼叫 `DatePicker.onClick(Event)`，若触发为当月的某一天则呼叫 `DatePicker.onDayClick(XElement)`；然后通过 `DatePicker.handleDateClick(XElement, String)` 呼叫 `DatePicker.setValue(Date, boolean)`，设置 `value` 的值；

#### 直接輸入字串 ####

* 当 new 一个 `DateField` 时，就 new 了一个 `DateTimePropertyEditor` 和一个 `DateCell`；
* 当使用者在 text 中输入数据时，呼叫 `ValueBaseField.setValue(T)`，实际上是呼叫了 `CellComponent.setValue(C, boolean, boolean)`，在这里设置 `value` 的值；
* (确实没有找到回到 `DatePicker.setValue()` 的地方，但确实这里应该会做 `resetTime()`，我也没有找到在什么时候做 `resetTime()` 的，/(ㄒoㄒ)/~~)


#### setValue() 的處理 ####

* 在 `DatePicker.setValue(date, true)` 時會作 `this.value = new DateWrapper(date).resetTime()`，
	`resetTime()` 的 source code 與結果有點對不起來。
	就結果而言，會將指定日期的時間部份設定為零點零分零秒。


其他
====

Grid
----

`setView(GridView)` 可以設定一些... 還不確定可以幹麼的東西 [喂喂]。
其中 `GridView.setForceFit()`，如果設定 `true` 則 Grid 不會出現 scroll bar；
若有多個 column、縮小其中一個 column，則其他 column 會變大補滿（不確定演算法）。

`GridView` 還可以設定 `GridViewConfig`，藉此設定 row / column 的 style──
正確的說是加掛 style name。

可以透過 `GridView.getScroller()` 控制 `Grid` 的 scroll 狀態。
當 `refresh()` 結束之後不想強制 scroll 回左上角，就必須得靠這招。


### AbstractGridEditing ###

`setEditableGrid()` 如果傳入 null 值是表示移除 edit 的效果，
因為一開始會作 `groupRegistration.removeHandler()`，然後在傳入值不為 null 才會掛相關 handler。
這在初始化設定（尤其搭配 ui.xml）時要特別注意 [淚目]

使用者的編輯行為會透過 `GridEditing` 影響 `Grid` 的 `Store`，
實際作為是增加 `Change` 跟 `Record`。
目前看起來，`Store` 只有提供兩個對應的 method：`commitChanges()` 跟 `rejectChanges()`
來清空 `modifiedRecords`→才會讓畫面上的 dirty 樣子消失（應該是 `StoreUpdateEvent` 的結果）。
`commitChanges()` 跟 `rejectChanges()` 都會觸發 `StoreUpdateEvent`，
實驗結果：`rejectChanges()` 如果影響 n 個 item，就會炸 n 個 `StoreUpdateEvent`。

從 `Store.getModifiedRecords()` 可以得到有改變（dirty）的 item：

```Java
	for (Store<Foo>.Record r : store.getModifiedRecords()) {
		Foo fooInstance = r.getModel();
		//此時 fooInstance 的內容為初始值，可以趁機作一些事情（？）
	}
```


要讓 `fooInstance` 的內容變成現在的值，有幾種作法：

* `r.commit(boolean)`：單筆
* `store.commitChanges()`：整個 store
* 根據 `Record` / `Change` 來修改 `fooInstance` [暈]

至於要發 RPC、萬一 RPC 失敗要能 rollback... 目前還沒有想法 [死]


LabelToolItem
-------------

`LabelToolItem` 轉換成 DOM 就只是一個 `div` 包一個字串，當然有掛一些 css（沒有特別的內容），
除了 package name 很奇怪之外，基本上應該可以視為 GXT 版的 label。
（謎之聲：之前測試不知道是測試到哪裡去了... (艸  ）


TabPanel
--------

在 ui.xml 當中只能這樣寫

```XML
	<ui:with field="tabPanelConfig" type="com.sencha.gxt.widget.core.client.TabItemConfig">
		<ui:attributes text="tab 名字" />
	</ui:with>
	<gxt:TabPanel>
		<gxt:child config="tabPanelConfig">
			<foo:Foo />
		</gxt:child>
	</gxt:TabPanel>
```


有點討厭，不過就算自己設計好像也沒辦法改變什麼 Orz


Toggle
------

如果只是把 `HasValue` 加到 `Toggle` 當中、畫面也還沒進行任何操作，
畫面顯示是會正常的，但是 `toggle.getValue()` 會是 null。
所以 `toggle.add()` 完還是 `toggle.setValue()` 一下比較保險。


DrawComponent
-------------

獨立出去紀錄在 [DrawComponent](DrawComponent.md)。


DnD
===

`new DragSource(dSource)` 會讓 `dSource` 變成可以 drag 的 widget，
實際上要看實作，像 `TreeDragSource` 就是讓 tree 的 item 變成可以 DnD。
`new DropTarget(dTarget)` 會讓 `dTarget` 變成可 drop 的對象。
在沒有特別的設定下（也就是 default group），任何的 `DropTarget` 都會接收任何 `DragSource` 的 DnD 行為。
反過來說，如果 `dSource.setGroup("5566")`，那麼 `dTarget` 必須也要是 5566 這個 group，
才有辦法接收源自於 `dSource` 的 DnD 行為。

如果 `parent`、`child` 這兩個 widget 都是 drop target（不考慮 group），
`parent` 跟 `child` 同時在畫面上、且 `parent` 包含 `child`，
則如果在 `child` 上作 drop 的動作，只會觸發 `child` 的 `DndDropHandler`、
而不會觸發 `parent` 的 `DndDropHandler`。
