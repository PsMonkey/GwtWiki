> # jqm4gwt #


* 官網：https://github.com/jqm4gwt/jqm4gwt
* repo：https://github.com/jqm4gwt/jqm4gwt


Project Setup
=============

官方文件有點殘缺，自己重寫一次...... ＝＝"

以下使用 standalone 的方式。

1. `pom.xml` 加入

	```XML
	<dependency>
		<groupId>com.sksamuel.jqm4gwt</groupId>
		<artifactId>jqm4gwt-standalone</artifactId>
		<version>1.4.6.Final</version>
		<scope>provided</scope>
	</dependency>
	<dependency>
		<groupId>com.sksamuel.jqm4gwt</groupId>
		<artifactId>jqm4gwt-library</artifactId>
		<version>1.4.6.Final</version>
	</dependency>
	```

	* 官方文件沒有特別註明要加 `jqm4gwt-library` 這個 dependency
	* `jqm4gwt-standalone` 一定要在 `jqm4gwt-library` 前面。
	
		> jqm4gwt-standalone must be the first in java build path order, before jqm4gwt-library
		
1. `foo.gwt.xml` 當中加入

	```XML
	<inherits name='com.sksamuel.Jqm4gwt' />
	```

1. （如果是後來才導入 jqm4gwt）清除 `/webapp/foo`
1. 跑一次 `mvn install`，這樣產生 `/foo` 目錄中才會出現 `css`、`js` 的目錄。
1. 把產生出來的 `/foo` 複製回 `/webapp/foo`
