> # 瀏覽器差異 #


ChangeEvent
===========

* GWT 2.6
* GXT 3.1

`CheckBox` 的 `ChangeEvent`（GWT）發生時，`CheckBox.getValue()` 會有差異：

* Chrome（35）：變更之前的值
* FireFox（30）：變更之後的值，個人認為這個比較合理  [遠目]

搭配 `ValueChangeEvent`，例如

```Java
	@UiHandler("fooCheckBox")
	void handleChange(ChangeEvent ce) {
		Window.alert("change event");
	}
	
	@UiHandler("fooCheckBox")
	void handleValueChange(ValueChangeEvent<Boolean> vce) {
		Window.alert("value change event");
	}
```


* Chrome（35）：先出現 change event、後出現 value change event
* FireFox（30）：順序顛倒

最後，以開發上來說，應該只要管 `ValueChangeEvent` 就好，
`ValueChangeEvent.getValue()` 的回傳值，至少目前測起來 Chrome 跟 FireFox 是一致的，
所以這個瀏覽器差異不知道什麼時候才會被炸到  [遠目]。