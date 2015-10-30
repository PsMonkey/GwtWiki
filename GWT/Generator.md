> # GWT Generator #
> 環境：GWT 2.6、Maven、SDM


前置準備
========
`pom.xml` 要多加上（一般的 GWT 開發不需要這個 dependency）：

	<dependency>
		<groupId>com.google.gwt</groupId>
		<artifactId>gwt-dev</artifactId>
		<scope>provided</scope>
	</dependency>
	
假設 interface 叫做 `FooBinder`（GWT 官方似乎喜歡用 binder 結尾），
則需要一個 extends `Generator` 的 class，假設叫做 `FooBinderGenerator`，
然後 `gwt.xml` 要加上：

	<generate-with class="PKG_NAME.FooBinderGenerator">
		<when-type-assignable class="PKG_NAME.FooBinder"/>
	</generate-with>
	

撰寫 tip
========
* 修改 generator 必須重新啟動 code server 才會執行新的版本。
* 可以使用 `System.out.println()`，也可以用 `generate()` 傳入的 `TreeLogger` 參數，
	都會出現在 console 中。


Reference
=========
* http://www.gwtproject.org/doc/latest/DevGuideCodingBasicsDeferred.html
* http://christiangoudreau.wordpress.com/2013/05/06/how-to-efficiently-write-gwt-generators/