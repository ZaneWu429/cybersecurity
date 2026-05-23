# **==windows==**

递归搜索

`gci -r -file c:\users`

打印

`gc C:\users\tony\Desktop\user.txt`

通过特定文件寻找目录

`gci c:\ ricoh.png -recurse -EA SilentlyContinue`



提权阶段：winpeas

设置执行策略：`set-executionpolicy unrestricted -scope currentuser`

在evil-WinRM下载，一般在 `c:\programdata\apps`下载

`upload winPEASx64_ofs.exe`

验证大小：`gci winPEASx64_ofs.exe`

运行winpeas :`.\winPEASx64_ofs.exe --help`

