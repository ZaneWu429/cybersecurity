## Drippingblues

### 一. ==使用nmap探测目标机==

####  ① ==首先查看自己本机的ip地址==

`ip -br a`  简洁的查看ip地址

![image-20250124231120953](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250124231120953.png)

 `ip a` 查看全部的信息

![image-20250124231520321](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250124231520321.png)

#### ②==使用nmap扫描C段的所有IP地址，找到目标机==

`sudo nmap -sn 192.168.10.1/24`

![image-20250124232206942](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250124232206942.png)

可以看到ip地址在192.168.10.193

下一步nmap四扫

#### ③==第一扫==：端口探测

`sudo nmap --min-rate 10000 -p- 192.168.10.193 -oA nmapscan/ports`

![image-20250124232945189](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250124232945189.png)

##### 对信息进行批量化处理

``

#### ④==第二扫==：详细信息探测

`sudo nmap -sT -sV -sC -O -p`

![image-20250125161459426](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250125161459426.png)

##### 可以看到这里21端口开放，ftp支持匿名登陆，80端口有文件信息泄露，robots.txt，dripisreal.txt

#### ⑤==第三扫==：udp探测

`sudo nmap -sU --top-ports 20 192.168.10.193 -oA nmapscan/udp`

![](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250125162041539.png)

##### udp没有什么有价值的信息

#### ⑥==第四扫==：漏洞脚本探测

`sudo nmap --script=vuln 192.168.10.193 -oA nmapscan/vuln`

![image-20250125162059705](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250125162059705.png)

##### 可以看到没有检测出来什么好用的漏洞利用



### 二.==收集信息==

#### ①==21端口==

#### 我们先查看21端口，21端口运行着ftp服务，ftp是文件传输协议（File Transfer Protocol, FTP)是一种在网络中进行文件传输的广泛使用的标准协议。作为网络通信中的基础工具，FTP允许用户通过客户端软件域服务器进行交互，实现文件的上传、下载和其他文件操作。

![image-20250125163204871](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250125163204871.png)

#### 可以看到这里有个zip格式的压缩包，下载下来，记得开binary模式

![image-20250125163332706](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250125163332706.png)

#### 下载完成，退出看看文件

<img src="C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250125163923433.png" alt="image-20250125163923433" style="zoom:150%;" />

#### unzip

![image-20250125172625060](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250125172625060.png)

![image-20250125172704841](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250125172704841.png)

![image-20250125172755034](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250125172755034.png)

##### 需要密码，没有，尝试一下暴力破解

##### fcrackzip

`sudo fcrackzip -D -p /usr/share/wordlists/rockyou.txt -u respectmydrip.zip`

![image-20250125173958530](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250125173958530.png)

![image-20250125174428318](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250125174428318.png)

#### 压缩包里套压缩包，继续尝试暴力破解

![image-20250125174643082](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250125174643082.png)

#### 只需要注意drip，是什么意思？这个关键词要记住，虽然不知道是干嘛的，但是有一个合理的猜测，有可能是文件包含漏洞的参数

![image-20250125175238731](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250125175238731.png)

#### 应该是破解不出来，或者找个更大的字典

#### 先看看80端口吧

#### ②==80端口==

![image-20250125174843188](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250125174843188.png)

#### 有两个用户名，前面有泄露过`robots.txt`，先看看

![image-20250125175047513](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250125175047513.png)

#### 感觉像目录遍历漏洞，先进行目录爆破吧，看看还有。用gobuster

![image-20250125175529228](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250125175529228.png)

#### 连`robots.txt` 都没有爆破出来，指定下文件扩展名

`sudo gobuster dir -u http://192.168.10.193/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,js,txt,zip`

![image-20250125180753482](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250125180753482.png)

#### 还有个index.php，也看看

![image-20250125181217140](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250125181217140.png)

#### 没有什么有价值的信息，是一开始进来的界面，就只有两个用户名

#### 还是看回`robots.txt`

![image-20250125181344196](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250125181344196.png)

#### 有两个文件，尝试一下

#### 访问`dripisreal.txt`

![image-20250125181734701](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250125181734701.png)

你好，亲爱的黑客爱好者，

选择以下歌词：

https://www.azlyrics.com/lyrics/youngthug/constantlyhating.html

数一数n个单词，并排放置，然后用md5求和

即，你好你好>>md5sum你好你好你好你好

这是ssh的密码



### 三.==文件上传漏洞==

#### 看不是很懂啊，看看另外一个文件，这个文件这个应该是文件包含漏洞，可以尝试读取/etc/passwd，回想到前面的drip，这里可以尝试一下

`192.168.10.193/index.php?drip=/etc/passwd`

![image-20250210213030846](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250210213030846.png)

#### **这里确实是可以读出来的，这里的格式不太对**

![image-20250210213251448](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250210213251448.png)

#### **这样就可以了，可以看到thugger是有bash权限的**

#### **那下一步就尝试一下`/etc/dripispowerful.txt`**

![image-20250210213552567](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250210213552567.png)

password is:
imdrippinbiatch

密码出来了，但是不知道密码，但是我们用户名不多，但是这里我想尝试一下用自动化的方式找出用户名



### ③==22端口==

![image-20250210214939817](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250210214939817.png)

![image-20250210214919107](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250210214919107.png)

#### **找出ssh的账号密码了**

`sudo ssh thugger@192.168.10.193`

![image-20250210215222522](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250210215222522.png)

找一下userflag

![image-20250210215336501](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250210215336501.png)



### 四.==提权==

#### `sudo -l`

![image-20250210215808202](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250210215808202.png)



