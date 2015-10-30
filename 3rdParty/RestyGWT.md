> # RestyGWT #


* 官網：http://resty-gwt.github.io/
* repo：https://github.com/resty-gwt/resty-gwt


JSON decode
===========

VO 的 field 名稱必須跟對應的 JSON key 值相同，
且 modifier、或是其 setter 的 modifier 不能是 private，
如此才可以自動 decode。

JSON 的 key 值對應不到 VO 的 field 不會出錯，就是忽略不理；
VO 的 field 對應不到 JSON 的 key 值也不會出錯，就是 default 值。
如果 field 的 data type 無法對應到 JSON，
例如 JSON 對應的是個 array、但是 field 的 data type 是 `String`，
在 `MethodCallback` 就會進到 `onFailure()`、得到 `org.fusesource.restygwt.client.ResponseFormatException`。
