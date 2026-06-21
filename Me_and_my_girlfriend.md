事先声明：这个靶机让我做的有点无语:(

## 信息收集

```
nmap -p- --min-rate 10000 192.168.174.147
```

扫出来22，80端口

```
nmap -sV -sT -sC -O -p22,80 192.168.174.147 
```

![image-20260611210731151](Me_and_my_girlfriend.assets/image-20260611210731151.png)

没有有用信息

```
nmap --script=vuln -p22,80 192.168.174.147
```

![image-20260611213730123](Me_and_my_girlfriend.assets/image-20260611213730123.png)

枚举出三个目录/robots.txt,/config,/misc

用gobuster工具扫描目录

![image-20260614230757651](Me_and_my_girlfriend.assets/image-20260614230757651.png)

首先我们先访问一下这个网页

![image-20260613143922683](Me_and_my_girlfriend.assets/image-20260613143922683.png)

查看源码![image-20260614162125371](Me_and_my_girlfriend.assets/image-20260614162125371.png)

意思是只有本地ip可以访问，也就是localhost或127.0.0.1

这里要么是burpsuite一直抓包，每一步都要加一个

```
X-Forwarded-For: 127.0.0.1
```

要么是使用一个小插件

![image-20260613144326058](Me_and_my_girlfriend.assets/image-20260613144326058.png)

![image-20260613144340305](Me_and_my_girlfriend.assets/image-20260613144340305.png)

这里要一直是以localhost的身份访问

查看每个网页的源代码都没有什么信息，就先注册一个用户

![image-20260614162328842](Me_and_my_girlfriend.assets/image-20260614162328842.png)

成功注册

![image-20260614222218948](Me_and_my_girlfriend.assets/image-20260614222218948.png)

在Profile中可以看到自己的个人信息

此时我注意到url上有一个明显的参数

```
index.php?page=profile&user_id=13
```

从这里我们可以尝试sql注入，fuzz等

尝试将id的参数改为1，发现页面有变化

![image-20260614222346647](Me_and_my_girlfriend.assets/image-20260614222346647.png)

- 注意：这里我有点蠢了，信息都摆在页面上了，竟然不知道可以直接查看源代码来得到隐藏的密码信息

一共1到5都有信息

逐一尝试ssh登录

![image-20260614222546064](Me_and_my_girlfriend.assets/image-20260614222546064.png)

发现只有这个是有效用户

![image-20260614222719642](Me_and_my_girlfriend.assets/image-20260614222719642.png)

- .bash_history里只有两个退出命令

- 在.my_secret目录中有两个文件，分别是flag1.txt和my_note.txt

![image-20260614223037927](Me_and_my_girlfriend.assets/image-20260614223037927.png)

得到另一个主人公bob，当时的第一想法是有没有可能让我从bob用户那里拿到什么

![image-20260614223147197](Me_and_my_girlfriend.assets/image-20260614223147197.png)

但是没有这个用户，接下来进行信息收集，尝试是否可以提权

![image-20260614223336193](Me_and_my_girlfriend.assets/image-20260614223336193.png)

通过

```
sudo -l
```

php命令是我们可以不需要密码，直接以root权限运行的

我们直接在gtfobins中查看关于php的提权方向![image-20260614231611867](Me_and_my_girlfriend.assets/image-20260614231611867.png)

```
sudo php -r 'system("/bin/sh -i");'
```

![image-20260614231746610](Me_and_my_girlfriend.assets/image-20260614231746610.png)

成功拿到权限和flag

![image-20260614232156279](Me_and_my_girlfriend.assets/image-20260614232156279.png)

- 再复习一下能得到完整终端权限的方式

------

另一种方法是我在实际操作中做的多余的东西，查看了url里的config和misc目录，里边分别有一个文件，但是下载下来什么也没有

当我拿到alice用户时

```
ls -la /var/www/html
```

![image-20260614231044821](Me_and_my_girlfriend.assets/image-20260614231044821.png)

可以发现还有其他信息

在shell中查看process.php没有什么重要的

![image-20260614231306378](Me_and_my_girlfriend.assets/image-20260614231306378.png)

在config目录下的config.php中

![image-20260614231347246](Me_and_my_girlfriend.assets/image-20260614231347246.png)

直接给我们mysql数据库中的（‘主机地址’，‘数据库用户名’，‘数据库密码’，‘数据库名’）那么我们也可以直接登录root账户了