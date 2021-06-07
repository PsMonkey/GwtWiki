自己寫的 composite 要能自動調整大小，
要注意是 extend GXT 的 `com.sencha.gxt.widget.core.client.Composite`，
而不是 GWT 的 `com.google.gwt.user.client.ui.Composite`

在 ui.xml 下，要知道某個 widget 下可以卡哪些特殊 tag，就去挖 source code，看看有沒有 `@UiChild`。
例如 `ContentPanel` 有 `addButton()` 跟 `addTool()` 掛 `@UiChild`，所以就可以這樣寫

```XML
	<c:ContentPanel>
		<c:button><f:Foo /></c:button>
		<c:tool><f:Foo /></c:tool>
	</c:ContentPanel>
```


因為 GXT 的 component 都是這樣處理的（GWT 反而不是... WTF），
所以當然就可以自動建立清單啦... 參見 [UiChildList.md](UiChildList.md)。

`FooTabPanel` extends `TabPanel` 不明瞭為什麼這樣還是會去 call 到 `TabPanel.add()`：

	//in FooTabPanel
	@Override
	@UiChild(tagname="tab")
	public void add(IsWidget widget, TabItemConfig config) {
		Window.alert("不會出現的訊息");
	}

	//in ui.xml
	<foo:FooTabPanel>
		<foo:tab><b:TextButton text="FOO" /></foo:tab>
	</foo:FooTabPanel>


如果 method 名字改成不是 `add()` 就沒問題...... 


browser event handling
======================

+ `sinkEvents()` 傳入 component 打算處理的 event 代碼
	（參見 `com.google.gwt.user.client.Event`）
+ override `onBrowserEvent()`（最好還是做 `super.onBrowserEvent()`

如果要處理的是鍵盤輸入行為，照著做但是沒反應，
試試看 `getElement().setTabIndex(0)`，
參考自 GXT 的 `Menu`，原理不明... [遮臉]

（上面這段其實跟 GXT 沒啥關係，都是 GWT 的哏。
只是已經 N 年沒有用過 GWT 的 component 了，所以寫到這來 XD）

另外 GXT 有一個 `KeyNav` 可以處理鍵盤輸入。
但也不按照 `sinkEvents()` 的流程走，寫法也是各種 WTF，
不要浪費時間用它 XD


store & value provider
======================

store 就是資料來源，value provider 就是決定顯示 vo 資料的方法

ui.xml 裡頭一定只能

```XML
	<ui:with type="com.sencha.gxt.data.shared.TreeStore" field="store" />
	<ui:with type="com.sencha.gxt.core.client.ValueProvider" field="valueProvider" />
```


不能

```XML
	<ui:with type="foo.MyTreeStore" field="store" />
	<ui:with type="foo.MyValueProvider" field="valueProvider" />
```


甚至宣告時 

```Java
	//正常
	@UiField(provider=true)	TreeStore<Foo> store = new MyTreeStore();
	
	//不行
	@UiField(provider=true)	MyTreeStore store = new MyTreeStore();
```


幹這什麼黑魔法。


XTemplate 相關
==============

`XTemplates` 的 method 回傳內容一定是 `SafeHtml`，
`@XTemplate` 的靜態內容基本上不會作 HTML convert，
但是動態內容（大括號內的東西）就一定會作轉換。


自訂 property formatter
-----------------------

`@XTemplate` 中的語法是 `{wtf:foo}`，`wtf` 是某個變數 or 某個變數的某個 property，
`foo` 是 formatter 的名稱，後頭還可以帶參數（參數能不能抓參數的值還不確定）。
要作到這個功能，首先得要弄出一個 formatter 像這樣：

```Java
	public WtfFormatter implements Formatter<Wtf> {
		@Override
		public String format(Wtf data) {
			return "WTF?";	//看要作什麼處理......
		}
	}
```


然後還得要有一個提供 static method 的 class 像這樣：

```Java
	public FormatterUtil {
		//這邊如果叫 getFormat() 等一下會很省事，可是我不喜歡一個 formatter 就要開一個 class
		public static WtfFormatter wtfName(){
			return new WtfFormatter();
		}
	}
```


然後 `XTemplates` 要掛上 `@FormatterFactories` 像這樣：

```Java
	@FormatterFactories(
		@FormatterFactory(
			factory=FormatterUtil.class, 
			method=@FormatterFactoryMethod(name="foo", method="wtfName")
		)
	)
```

	
必須要透過 `@FormatterFactory` 來告訴 generator 要去哪裡找 formatter 的 method，
所以要給 class 跟 method。
method 的部份再透過 `@FormatterFactoryMethod` 來指定是哪個 method（`method`）、
以及在 `XTemplates` 當中叫啥名字（`name`）。