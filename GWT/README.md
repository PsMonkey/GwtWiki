compatible 規範
===============

符合下列條件的 class 才能在 GWT 內使用：

* primitive data type
* array（陣列元素也必須 compatible）
* enum
* JRE emulation class（包含可用 method）
* 繼承 compatible class
* 所有 field 都符合 compatible 原則

另外，`synchronized` 與 `finalize()` 加了不會出問題，但是 GWT 會直接忽略。

*（沒有考慮撰寫 emulate class 的招數）*

reference：

* JRE compatibility：http://www.gwtproject.org/doc/latest/DevGuideCodingBasicsCompatibility.html
* JRE emulation class： http://www.gwtproject.org/doc/latest/RefJreEmulation.html


RPC（serializable）規範
-----------------------

* 滿足 compatible 規範
* implement `IsSerializable` 或是 `Serializable`
* 有 default（沒有參數）constructor，任何 access modifier 均可（private 也無所謂）。
	或是根本沒有任何 constructor。
* 所有 field 都符合 RPC 原則。
	* 例外：final field（不會在 GWT RPC 中傳遞）
	* 例外：transient field（接收端會得到 null）

*（沒有考慮 custom serialization 的招數）*

備註：enum 只傳遞名稱。例如 `FooEnum.FOO`，就只會傳 `FOO`，裡頭的 field 值一概忽略。

reference：

* RPC： http://www.gwtproject.org/doc/latest/DevGuideServerCommunication.html#DevGuideSerializableTypes


雜項
----

* 預設設定下，在 production mode 的 assert 是不會被觸發的。
	（[參考資料](http://www.gwtproject.org/doc/latest/FAQ_Client.html#How_do_I_enable_assertions?)）


Generator
=========

因為哏太多而且不是正常的開發流程，所以另外開 [GwtGenerator](GwtGenerator.md)。


Deferred Binding
================

`when-type-is` 的 class 沒有限定一定要 interface（官方文件好像也沒特別註明），
目前測試用 abstract class 也沒有問題。


JSNI
====

`$wnd` 是對應到 JS 的 `window`，也就是說 JSNI 的 `$wnd.alert()` 就等於 JS 的 `window.alert()`。
如果需要 JS 的全域變數，基本上就只能讓 `$wnd` 多加一個變數來處理，
實際上這等同在 host page 的 `<script>` 宣告一個變數：

	//host page script block
	var value = "foo";
	
	//in JSNI
	$wnd.newGlobalArray = []; //JS 可以亂加 field 不要意外，延伸出來的 JS 哏這裡略過不提
	$wnd.alert($wnd.value);	//alert 視窗的訊息會是「foo」


`$doc` 是對應到 JS 的 `document`。
注意，如果在 JSNI 裡頭直接操作 `document`，並不會是 host page 的 `document` instance，
而是 host page 裡頭某個 iframe 裡頭的 `document`。
這背後機制暫時不明... [死]


JSON 相關
---------

client side 的 GWT 將 JSON 字串轉回（偽）Java instance 的議題。

> ###### 注意 ######
> 請不要忘記這是 JS、這是 JS、這是 JS（很重要所以要講三次），
> 所以隨便 casting 是很合理的（謎之聲：教練我想寫 Java [淚目]）。


### Overlay Type ###

Overlay Type 的基礎是 `JavaScriptObject`。
繼承 `JavaScriptObject` 的 class 必須：

* `package` 等級的 class modifier
* 有 `protected` 等級的 default constructor

基本上不太需要宣告 field，主要是用 JSNI 的 getter：

	public final native String getId() /*-{ return this.id; }-*/;


其他轉換 tip：

* primitive type、enum（名稱一樣應該就 OK）可以直接轉換。
* `Date` 轉換：

	```Java
	public final Date getFooDate() {
		return DateTimeFormat.getFormat(PredefinedFormat.ISO_8601).parse(fooDate());
	}
	
	private final native String fooDate() /*-{ return this.fooDate; }-*/;
	```

對一個 JSON 字串作 `JsonUtils.safeEval()` 就會得到 `JavaScriptObject` 或其子孫，端看泛型怎麼指定。
後續呼叫 `JavaScriptObject.cast()` 也可以轉成想要的（繼承 JavaScriptObject）的 class。


#### 炸點 ####
假設收到這樣一個 JSON 字串

	["ID", "DATA"]

然後這段程式碼就會炸奇怪的問題：

	HashMap<String, JavaScriptObject> map = new HashMap<>();
	JsArrayMixed array = JsonUtils.safeEval(json);
	map.put(array.getString(0), array.getObject(1));	//這邊不會有事
	JavaScriptObject foo = map.get(array.getString(0));	//這行會炸 casting 問題
	
	WtfJSO wtf = new WtfJSO(array.getObject(1));	//constructor 參數的型態也是 JavaScriptObject
	//上面這樣就不會出問題... WTF?


### 相關工具 ###

* Online Prettify：http://json.parser.online.fr/
* Online Prettify：http://jsonviewer.stack.hu/


UI 相關
=======

UiBinder
--------

用 UiBinder 弄畫面，執行時一直炸找不到 fooWidget class 的錯誤（但是 Eclipse 沒有 compile error），
先單純用程式 new FooWidget() 出來，通常就會知道 FooWidget 錯在哪裡了。
（2.5.1 時常見不小心用了 generic 的 `<>` 就炸了，但是 Eclipse 不會炸錯誤 Orz）


### ui:import ###

似乎是 ui.xml 中可以使用 enum 的唯一方法
（[ui:import ref](http://stackoverflow.com/questions/9492658/can-i-use-enum-values-as-field-values-inside-uibinder-template)）。


I18N
----

### gwt.xml ###
在 `gwt.xml` 當中要加上有要實作的語系

```XML
	<extend-property name="locale" values="zh_CN"/>
```


至於設定 default locale 沒啥用

```XML
	<set-property-fallback name="locale" value="en"/>
```


個人覺得沒啥用，倒是指定了語系就得要有 `extend-property` 然後也找得到對應的 properties 檔。
而且 default 的 properties 檔案還是不能少...... ＝＝"

掛 `<collapse-all-properties />` 好像不受影響（還是我誤會了這個設定值的意義...... Orz）


### 實際操作 ###

內建原生機制無法徵測 browser 的語系，必須在 host page 給 `<meta>`

	<meta name="gwt:property" content="locale=zh_CN">
	
或是在 URL 掛上 query string

	http://127.0.0.1/hostPage.html?locale=zh_CN
	
會優先使用 query string 的設定值。

在 SDM 下，加上 query string 之後、甚至重新 compile 好像都不會順利切換語系。
另外開一個新的 tab 測試會比較實在。


Image
-----

用 `ImageResource` 的 `Image` 如果要調整大小，

```Java
	new Image(imgResource);	//size 太大會留白
	new Image(imgResource.getSafeUri());	//OK
```

	
______________________________________________________________________


GWT 活跳跳的證據
----------------

* 2011 年
	* https://plus.google.com/+RayCromwell/posts/ivVepvxCu3g  
		Ray Cromwell 列出使用 GWT 的 Google Product。
* 2013 年：
	* https://plus.google.com/+ThomasBroyer/posts/iCq9E2Wk3Jz
		新版的 Google Drive 的 Sheet 使用 GWT 撰寫
	* http://www.slideshare.net/cromwellian1/gwtcreate-keynote-san-francisco  
		Ray Cromwell 在 Gwt.Create 報告 Google Drive 使用 GWT 撰寫的投影片（p.6）
* 2014 年：
	* https://news.ycombinator.com/item?id=8552296#up_8554339
		Google Inbox、Google Calendar 是 GWT（70%）+ Closure（30%）

註：Ray Cromwell 是 Googler、現任 GWT 委員長。