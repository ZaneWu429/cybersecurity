# upload-labs



## Pass01-前端js校验

![image-20251002213341538](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251002213341538.png)

这里是要求上传一个webshell，那就用locate找到现有的php脚本

![image-20251002213645022](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251002213645022.png)

查看本机的IP地址，这里不能用docker内部的地址，好像是因为docker，所以这个服务对外的端口是搭建docker主机的端口

![image-20251002213829314](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251002213829314.png)

开始监听

![image-20251002213902884](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251002213902884.png)

修改shell.php文件

![image-20251002214043261](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251002214043261.png)

尝试上传

![image-20251002214113488](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251002214113488.png)

![image-20251002214318439](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251002214318439.png)

发现这个弹窗，因为这个弹窗结合burpsuite的抓包，这里猜测可能是前端js验证，看一下源码

![image-20251002214417007](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251002214417007.png)

发现确实是，这里有两个思路，第一个是绕过前端的验证，但是在上传的时候上传shell.php，第二个是直接禁止js的动态实现

第一种：

先上传一个正常的文件，抓到正常的包，修改包实际上传的文件

![image-20251002215545898](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251002215545898.png)

![image-20251002215559117](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251002215559117.png)

![image-20251002215627916](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251002215627916.png)

第二种：
修改burpsuite的配置

![image-20251002215902951](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251002215902951.png)

![image-20251002215942063](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251002215942063.png)

## Pass02

这里我上传一个php文件

![image-20251002220232179](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251002220232179.png)

提示文件类型错误，那就有可能是Content-Type类型错误

<u>`Content-Type` 是 HTTP 请求和响应头中的一个字段，用于指示资源的 **MIME 类型（Multipurpose Internet Mail Extensions）**，即数据的格式或类型。它告诉客户端或服务器如何解析接收到的数据</u>

![image-20251003140754432](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003140754432.png)

这里修改一下Content-Type

![image-20251003141713607](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003141713607.png)

发现这里上传上去了，网页访问一下

![image-20251003141807847](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003141807847.png)

## Pass03

![image-20251003143709498](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003143709498.png)

在前端找不到对应的验证代码，这里猜测是后端进行文件校验，具体校验猜测是对最后面的文件后缀进行检测

![image-20251003143642072](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003143642072.png)

看源码也是这么说的

![image-20251003144113623](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003144113623.png)

那就把shell.php -> shell.php.jpg

![image-20251003144402026](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003144402026.png)

没想到这里还改名字了

这里还遇到一个问题，就是我上传文件后，执行不了，后面查询才知道是服务器配置文件的问题

```
sed -i '/<\/IfModule>/i\
    AddType application\/x-httpd-php .php .phtml .phps .php5 .pht .phar .php3
' /etc/apache2/mods-enabled/mime.conf

grep -A2 -B2 "AddType.*x-httpd-php" /etc/apache2/mods-enabled/mime.conf #验证
```

![image-20251003162349597](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003162349597.png)

![image-20251003162500609](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003162500609.png)

## Pass04

上传shell.php

![image-20251003162722825](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003162722825.png)

发现是这个

![image-20251003163309965](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003163309965.png)

![image-20251003163250012](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003163250012.png)

就上传了shell.phar可以成功，但是这道题实际上是.htaccess文件的上传，这里可以成应该是前面我设置.conf的时候修改的，这里还是尝试修改.htaccess文件，修改成另外一个可以上传的文件后缀

上传 `.htaccess` 内容为：`AddType application/x-httpd-php .l33t`

![image-20251003163756374](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003163756374.png)

上传成功

![image-20251003164025325](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003164025325.png)

```
cp shell.php shell.l33t
```

上传shell.l33t

![image-20251003164152615](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003164152615.png)

## Pass05

![image-20251003164403374](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003164403374.png)

抓包，修改content-type为 `image/jpeg`

![image-20251003164631018](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003164631018.png)

发现还是不行，尝试文件双后缀

![image-20251003164915631](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003164915631.png)

发现重命名了，命名为 `.jpg` 文件，那尝试上传 `.htaccess` 文件

![image-20251003165238749](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003165238749.png)

发现还是不行，不对，这里不应该尝试上传 `.htaccess` 文件的，因为上传后重新命名的

![image-20251003165804049](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003165804049.png)

查看提示和源码后发现，很多文件类型都被限制了

但是这里限制地不是很全面，漏了大写，所以这里使用大写绕过

![image-20251003172213420](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003172213420.png)



## Pass06

上传shell.php

![image-20251003172422322](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003172422322.png)

提示文件不允许上传，上传一个shell.jpg看看

![image-20251003172723690](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003172723690.png)

![image-20251003172804585](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003172804585.png)

![image-20251003172834351](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003172834351.png)

发现上传jpg文件就可以，但是改掉文件名了，这里猜测是大规模的黑名单

![image-20251003172957723](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003172957723.png)

确实是大规模的黑名单，还防范了大写绕过，查找了wp，发现是空格绕过

![image-20251003202733274](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003202733274.png)

bp抓包后，在上传文件的末尾加上空格，发现可以上传

但是不知道怎么回事利用不了，应该是linux系统的关系，windows系统会把空格强制去掉，保留完整的文件后缀名，但是linux则保留所有的

![image-20251003212738795](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003212738795.png)

$表示换行的意思，所以文件名还保留着空格

这里就可以上传 `shell.php. `

![image-20251003223028293](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003223028293.png)

## Pass07

上传shell.php文件试试看反馈

![image-20251003213204181](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003213204181.png)

![image-20251003213213629](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003213213629.png)

看样子这下是用不了一点，看看源码

![image-20251003213349769](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003213349769.png)

- `strrchr($str, $char)`：**从字符串中最后一个 `$char` 的位置开始，返回到字符串末尾的所有内容**。

- 所以对于文件名，它会返回 

  最后一个点（`.`）之后的所有字符

  ，包括：

  - 正常扩展名（如 `.php`）
  - **空格、特殊字符、甚至换行符**

这里是截取倒数最后一个dot后面的东西去和黑名单比较，那么尝试上传`shell.php.`，那么比较的就是 `.` ，很明显可以通过校验

![image-20251003214423884](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003214423884.png)



## Pass08

上传 `shell.php`

![image-20251003214555909](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003214555909.png)

![image-20251003214713002](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003214713002.png)

![image-20251003214746457](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003214746457.png)

这里是文件名后端校验，重命名，但是没校验实际文件类型，可以通过上传jpg的方式上传，但是没用php的方式解析上传的文件，所以上传后也解析不了，`.htaccess`文件也不能上传，因为会改名字

![image-20251003215213385](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003215213385.png)

源码已经去除了大小绕过，空格绕过，dot绕过，双写后缀名

源码中未对 `::$DATA` 过滤

> 在window的时候如果文件名+"::$DATA"会把::$DATA之后的数据当成文件流处理,不会检测后缀名，且保持::$DATA之前的文件名，他的目的就是不检查后缀名
> 例如:"webshell.php::$DATA"Windows会自动去掉末尾的::$DATA变成"webshell.php"

上传 `shell.php::$DATA`

![image-20251003215749629](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003215749629.png)

![image-20251003215832204](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003215832204.png)

因为是linux系统的问题，`::DATA` 还保留着，就不能进行利用了



## Pass09

尝试上传shell.php

![image-20251003221559913](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003221559913.png)

![image-20251003221758096](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003221758096.png)

这里可以看到应该是单纯的后端校验，还保留原来的文件名，但是还是前面的问题，这里不是以php解析的，所以不能执行php文件

![image-20251003222117796](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003222117796.png)

![image-20251003222128633](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003222128633.png)

本来还以为是白名单，这里把前面的利用都防住了

使用 `deldot()` 删除文件名末尾的点

> deldot() 函数从末尾向前检测，检测到第一个点后，会继续向前检测，但遇到空格会停下来

可以构造文件名： `shell.php. .` 绕过检测

![image-20251003222531170](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003222531170.png)

![image-20251003222545942](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003222545942.png)

![image-20251003222558744](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003222558744.png)

看起来linux可以正确解析文件名最后是dot的，也可以解析 `shell.php. .` 、 `shell.php. ` ，就是不能解析 `shell.php `



## Pass10

上传shell.php

![image-20251003224741328](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003224741328.png)

发现是顺利上传了，但是后缀名被截断了，使用bp看看

![image-20251003224944825](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003224944825.png)

确实是，那输入 `shell.php.php` 看看是单次还是多次循环

![image-20251003225128046](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003225128046.png)

看起来像是替换，那就用双写

![image-20251003225225235](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003225225235.png)

确实上传上去了，那下一步就是访问

![image-20251003225333147](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003225333147.png)

确实可以



## Pass11

![image-20251003225459118](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003225459118.png)

看起来是白名单的样子

![image-20251003225551271](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003225551271.png)

还真是白名单，只允许上传 `jpg` 、`png` 、`gif` 的文件，那上传一下 `shell.php.jpg` 试试看

![image-20251003225959605](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251003225959605.png)

还是不行，看了一下wp发现还可以用 `00` 绕过，

条件： php版本 < 5.3.4 ; magic_quotes_gpc=Off

> strrpos(string,find,start) 函数查找字符串在另一字符串中最后一次出现的位置（区分大小写）。
> substr(string,start,length) 函数返回字符串的一部分*(从start开始 ，长度为 length)*
>
> 这里定位最后出现的 . ，并截断，再和后面的时间拼接

```text
$ext_arr = array('jpg','png','gif');
    $file_ext = substr($_FILES['upload_file']['name'],strrpos($_FILES['upload_file']['name'],".")+1);
    if(in_array($file_ext,$ext_arr)){
        $temp_file = $_FILES['upload_file']['tmp_name'];
        $img_path = $_GET['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;
```

源码中对后缀进行白名单检测，只允许 jpg ，png，gif

但上传的路径可控，这里可以使用 %00截断

1. 上传`shell.jpg`的一句话木马
2. `save_path=../upload/shell.php%00`
3. 成功上传后，%00后的不会被识别

这里的 `save_path` 参数还是自己控制的，但是我的php是5.5.38的，显然不满足利用条件

![image-20251004111622710](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004111622710.png)



## Pass12

![image-20251004111844373](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004111844373.png)

![image-20251004112043982](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004112043982.png)

![image-20251004112033634](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004112033634.png)

这题和上一题差不多，只是这题变成了POST传参

![image-20251004112847338](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004112847338.png)

通关步骤：

1. 准备shell.jpg，上传并拦截请求；
2. 在BP中找到POST参数（如save_path=upload/），改为save_path=upload/shell.php+（+用于定位）；
3. 右键选择「Convert Selection to Hex」，找到+对应的Hex值2B，改为00；
4. 确保filename="shell.jpg"（符合白名单）；
5. 放包后，路径被0x00截断为upload/shell.php，文件保存为shell.php；

![image-20251004112928631](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004112928631.png)

![image-20251004112957111](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004112957111.png)

还是一样的问题，php版本太高了



## Pass13

![image-20251004143930758](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004143930758.png)

![image-20251004143941181](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004143941181.png)

```
function getReailFileType($filename){
    $file = fopen($filename, "rb");
    $bin = fread($file, 2); //只读2字节
    fclose($file);
    $strInfo = @unpack("C2chars", $bin);    
    $typeCode = intval($strInfo['chars1'].$strInfo['chars2']);    
    $fileType = '';    
    switch($typeCode){      
        case 255216:            
            $fileType = 'jpg';
            break;
        case 13780:            
            $fileType = 'png';
            break;        
        case 7173:            
            $fileType = 'gif';
            break;
        default:            
            $fileType = 'unknown';
        }    
        return $fileType;
}

$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $file_type = getReailFileType($temp_file);

    if($file_type == 'unknown'){
        $msg = "文件未知，上传失败！";
    }else{
        $img_path = UPLOAD_PATH."/".rand(10, 99).date("YmdHis").".".$file_type;
        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        } else {
            $msg = "上传出错！";
        }
    }
}
```

结合提示，这里的利用是上传图片马+文件包含调用图片马

这里的文件校验头检查只检查了前面的两个字节，但是我还是用真实的图片进行拼接

```
xxd C02.jpg # 16进制打开
cat C02.jpg shell.php > 2.jpg # 制作图片马，将shell.php拼接到C02.jpg的后面
```

正常上传

![image-20251004144508074](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004144508074.png)

![image-20251004144607127](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004144607127.png)

![image-20251004144622281](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004144622281.png)



## Pass14

![image-20251004152427428](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004152427428.png)

```
function isImage($filename){
    $types = '.jpeg|.png|.gif';
    if(file_exists($filename)){
        $info = getimagesize($filename);
        $ext = image_type_to_extension($info[2]);
        if(stripos($types,$ext)){
            return $ext;
        }else{
            return false;
        }
    }else{
        return false;
    }
}

$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $res = isImage($temp_file);
    if(!$res){
        $msg = "文件未知，上传失败！";
    }else{
        $img_path = UPLOAD_PATH."/".rand(10, 99).date("YmdHis").$res;
        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        } else {
            $msg = "上传出错！";
        }
    }
}
```

这道题有一个新的函数`getimagesize()`

- **不能防止“图片马”**！
  如果一个文件**既是合法图片，又在末尾追加了 PHP 代码**（如 `cat image.jpg shell.php > evil.jpg`），`getimagesize()` 仍然会认为它是合法图片！
- 因为它只解析图片头部结构，**不检查文件末尾是否有额外数据**。

> 🔒 所以：
> **`getimagesize()` 只能验证“是不是图片”，不能验证“图片里有没有藏恶意代码”**。 

就是说需要一个真实的照片，而不是像上道题简单的前两个字节校验，但是我前面那道题也是拿真实的图片拼接的，所以照样可以用，上道题的2.jpg用不了，猜测是图片本身的问题，换张图片看看，后面才发现是jpeg的图片，jpeg也不行，没法了



## Pass15

![image-20251004154902784](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004154902784.png)

重要局限性（和 `getimagesize()` 一样）

> ❌ **`exif_imagetype()` 无法防御“图片马”！** 

原因：

- 它只检查文件**开头的 Magic Bytes**；
- 如果一个文件是**合法图片 + 末尾追加 PHP 代码**（如 `cat image.jpg shell.php > evil.jpg`），它的文件头仍然是合法的；
- 因此 `exif_imagetype()` 会返回 `IMAGETYPE_JPEG`，**认为它是正常图片**。

✅ 所以：

- 它能防止**纯文本伪装成图片**（如把 `<?php ... ?>` 保存为 `.jpg`）；
- 但**不能防止“合法图片藏后门”**。

![image-20251004155240164](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004155240164.png)

这样的话，就是前面的图片都可以用



## Pass16

第十七关主要是把二次渲染绕过
imagecreatefromjpeg（）函数
二次渲染是由Gif文件或 URL 创建一个新图象。成功则返回一图像标识符/图像资源，失败则返回false，导致图片马的数据丢失，上传图片马失败。
进行通关
按照原来的方法进行上传，我们可以发现还是可以上传的，但是配合包含漏洞却无法解析，这时我们把上传的图片复制下来用Notepad打开，发现我们原来写的php代码没有了，这就是二次渲染把我们里面的php代码删掉了。
我们
把原图和他修改过的图片进行比较，看看哪个部分没有被修改。将php代码放到没有被更改的部分，配合包含漏洞，就可以了。
使用HxD Hex Editor进行比较

![image-20251004161454514](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004161454514.png)

可以看到明显变小了，试试010 editor

编辑sh.gif文件，查看对应的十六进制码，为后面的替换做准备

```
<?php phpinfo();?>
00000000: 3c3f 7068 7020 7068 7069 6e66 6f28 293b  <?php phpinfo();                                                   │
00000010: 3f3e 0a                                  ?>.    
```

![image-20251004163308793](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004163308793.png)

![image-20251004163358420](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004163358420.png)

发现中间一大片是相同的，二次渲染很可能是渲染不到这个部分的，这里选择gif文件就是因为二次渲染的话，gif改动是很少的，但是在实际情况下，如果想要嵌入更大量的php，依赖人工是不太可能的，这里只是写了一个很小的shell，确定这个代码可以利用，上传后，利用文件包含漏洞就可以了

![image-20251004162938775](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004162938775.png)



## Pass17

![image-20251004165016360](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004165016360.png)

```
$is_upload = false;
$msg = null;

if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');
    $file_name = $_FILES['upload_file']['name'];
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $file_ext = substr($file_name,strrpos($file_name,".")+1);
    $upload_file = UPLOAD_PATH . '/' . $file_name;

    if(move_uploaded_file($temp_file, $upload_file)){
        if(in_array($file_ext,$ext_arr)){
             $img_path = UPLOAD_PATH . '/'. rand(10, 99).date("YmdHis").".".$file_ext;
             rename($upload_file, $img_path);
             $is_upload = true;
        }else{
            $msg = "只允许上传.jpg|.png|.gif类型文件！";
            unlink($upload_file);
        }
    }else{
        $msg = '上传出错！';
    }
}
```

**核心原理**

后端逻辑：先保存文件→校验后缀→合法则重命名，不合法则删除。可利用“保存后→删除前”的时间差，访问文件生成永久WebShell。

**最致命的问题：先上传，再检查！**

```
if(move_uploaded_file($temp_file, $upload_file)){
    if(in_array($file_ext,$ext_arr)){
         $img_path = UPLOAD_PATH . '/'. rand(10, 99).date("YmdHis").".".$file_ext;
         rename($upload_file, $img_path);
         $is_upload = true;
    }else{
        $msg = "只允许上传.jpg|.png|.gif类型文件！";
        unlink($upload_file);
    }
}else{
    $msg = '上传出错！';
}
```

风险：

- **在 `unlink()` 执行前，恶意文件已存在于 Web 可访问目录中！**
- 攻击者可以**在文件被删除前快速访问它**（Race Condition 竞态条件）；
- 即使只有几毫秒窗口，自动化脚本也能利用。

**通过步骤：**

1. 准备shell.php，内容为：

   ```
   <?php fputs(fopen('sh.php', 'w'), "<?php phpinfo();?>");?>
   ```

2. 上传shell.php，用burpsuite拦截请求，发送到 `Intruder`;

3. 在Intruder中设置"无限发包"（Payload类型选Null payloads，数量设置10000），开始攻击

4. 同时运行python脚本，不断访问shell.php:

   ```
   import requests
   url = "http://127.0.0.1/upload/shell.php"
   while True:
   	res = requests.get(url)
   	if res.status_get == 200:
   		print("successful!")
   		break
   ```

   

![image-20251004170747750](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004170747750.png)

先运行python脚本，再发包

![image-20251004180640323](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004180640323.png)

## Pass18



## Pass19

```
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array("php","php5","php4","php3","php2","html","htm","phtml","pht","jsp","jspa","jspx","jsw","jsv","jspf","jtml","asp","aspx","asa","asax","ascx","ashx","asmx","cer","swf","htaccess");

        $file_name = $_POST['save_name'];
        $file_ext = pathinfo($file_name,PATHINFO_EXTENSION);

        if(!in_array($file_ext,$deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH . '/' .$file_name;
            if (move_uploaded_file($temp_file, $img_path)) { 
                $is_upload = true;
            }else{
                $msg = '上传出错！';
            }
        }else{
            $msg = '禁止保存为该类型文件！';
        }

    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

**核心原理**
move_uploaded_file()会忽略文件名末尾的/.，后端仅校验用户输入的后缀，可通过cmd.php/.绕过。或者使用hex编码，将后缀名后一位改为00截断符

**通关步骤**

1. 准备shell.png（伪装图片），上传并拦截请求；
2. 将save_name改成 `upload-19.php/.`
3. 后端校验png合法，move_uploaded_file()忽略/.，实际保存为upload-19.php；

![image-20251004213203173](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004213203173.png)

![image-20251004213148679](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004213148679.png)

![image-20251004213410569](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004213410569.png)

![image-20251004213422815](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20251004213422815.png)



## Pass20

