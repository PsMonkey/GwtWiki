清除暫存檔
==========

Windows
-------

開發階段會產生大量的暫存檔，而且不會自己消失，
所以固定執行下面這個 bat 檔，以刪除（已知）與 GWT 有關的暫存檔

	cd %LocalAppData%\temp
	del ImageResourceGenerator*.*
	del gwt*.*
	del uiBinder_com.*
	for /d %%p in ("gwt*") do rmdir /s /q %%p
	for /d %%p in ("ResourceProvider*") do rmdir /s /q %%p

