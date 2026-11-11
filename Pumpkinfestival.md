[TOC]

# 信息收集

```
arp-scan -l
nmap -p- --min-rate 10000 192.168.174.194 | awk -F ' ' '{print$1}' | awk -F '/' '{print$1}' | tr '\n' ','
```

得到端口号21,80,6880

分别进行TCP详细扫描，漏洞扫描和UDP扫描

## TCP详细扫描

![image-20261110092130943](Pumpkinfestival.assets/image-20261110092130943.png)

- 存在/wordpress
- 多个枚举出的文件
- 21端口ftp可匿名登录
- 6880是ssh端口

## 漏洞扫描

![image-20261110092201711](Pumpkinfestival.assets/image-20261110092201711.png)

- 枚举出/robots.txt

## UDP扫描

```
nmap -sU --min-rate 10000 192.168.174.194 -oA /nmapscan/udp
```

不过这个被限速了，扫的太慢，就不再展示了

# 端口分析

## ftp端口

既然可以匿名登录，先查看里面有什么信息

```
ftp 192.168.174.194
anonymous
```

在secret目录中得到第一个token.txt

![image-20261110113458848](Pumpkinfestival.assets/image-20261110113458848.png)

## 80端口

默认页面![image-20261110144910288](Pumpkinfestival.assets/image-20261110144910288.png)

- 得到了两个用户名，jack和harry
- pumpkintokens，上面我们得到了一个，这个靶机的目标应该也有获取这些token

### 枚举目录

```
gobuster dir -u "http://192.168.174.194" -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-big.txt -x php,zip,txt,html
```

根据扫描结果，对于扫出的蓝色目录进行更深的扫描，查看文件信息

**robots.txt**

![image-20261110150832805](Pumpkinfestival.assets/image-20261110150832805.png)

列举了一些已经扫出来的目录

**/store/track.txt**

![image-20261110150927972](Pumpkinfestival.assets/image-20261110150927972.png)

- 得到一个tracking code
- 一个类似邮箱的东西，实际上`.local` 是 **MDNS 本地内网域名**，公网 DNS 不会解析这个域名，需要编辑/etc/hosts文件，还得到一个用户名admin，稍后查看这个域名

**/tokens/token.txt**

![image-20261110171036189](Pumpkinfestival.assets/image-20261110171036189.png)

第二个pumpkintoken

- 其余的目录基本都没有权限访问

### pumpkins.local

```
whatweb http://pumpkins.local
```

![image-20261110181308688](Pumpkinfestival.assets/image-20261110181308688.png)

在这里发现了wordpress架构，这应该是一个新的服务域名

- 用wpscan扫描

```
wpscan --url http://192.168.174.194/wordpress --api-token ****************** -e vt,u,vp
```

![image-20261110202407428](Pumpkinfestival.assets/image-20261110202407428.png)

枚举了admin和morse两个用户，用wpscan工具爆破，爆不出来，后来才知道，默认页面的![image-20261110203742459](Pumpkinfestival.assets/image-20261110203742459.png)

就是密码，登录/wp-login.php

![image-20261110203844514](Pumpkinfestival.assets/image-20261110203844514.png)

得到第三个pumpkintoken，但是没有利用的点，主题不允许改动文件，基本没有利用的点

**wordpress没有走通，再通过目录枚举拿信息**

```
gobuster dir -u "http://pumpkins.local" -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-big.txt -x php,zip,txt,html
```

![image-20261110205526462](Pumpkinfestival.assets/image-20261110205526462.png)

**/license.txt**

![image-20261110205616548](Pumpkinfestival.assets/image-20261110205616548.png)

第四个pumpkintoken

**/readme.html**

![image-20261110205727515](Pumpkinfestival.assets/image-20261110205727515.png)

- 发现一串加密符号，应该是base62，只有 `0-9 A-Z a-z`，一共 62 个，**没有任何特殊符号，没有填充符 =**（没有+和=）

![image-20261110210832212](Pumpkinfestival.assets/image-20261110210832212.png)

解码后发现是morse和jack的密码Ug0t!TrIpyJ（这个morse账号以及对应密码也可以用于登录wordpress）

## 拿到shell

尝试进行ssh连接或者ftp账号登录，结果都没有成功，目前只剩下harry用户不知道密码，尝试爆破登录ftp或者ssh

![image-20261110211354774](Pumpkinfestival.assets/image-20261110211354774.png)

登录ftp后发现第五个pumpkintoken![image-20261110211504406](Pumpkinfestival.assets/image-20261110211504406.png)

切换到NOOOO目录时发现第六个pumpkintoken![image-20261110211630083](Pumpkinfestival.assets/image-20261110211630083.png)

最终得到一个data.txt

```
file data.txt
```

![image-20261110211722723](Pumpkinfestival.assets/image-20261110211722723.png)

发现是tar压缩的

```
tar -xvf data.txt
```

得到一个data文件

![image-20261110211940993](Pumpkinfestival.assets/image-20261110211940993.png)

发现还是压缩包，再解压

```
tar -xvf data
```

![image-20261110212015791](Pumpkinfestival.assets/image-20261110212015791.png)

还是压缩包，再解压

![image-20261110212041246](Pumpkinfestival.assets/image-20261110212041246.png)

得到了一个jack文件，这次格式是ASCII

![image-20261110212119969](Pumpkinfestival.assets/image-20261110212119969.png)

发现是16进制，转一下格式

```
xxd -r -p jack 111
```

- -r，反向：十六进制文本还原成二进制
- -p，-ps，plain 纯十六进制，**无偏移、无 ascii 列、无空格**

![image-20261110213108824](Pumpkinfestival.assets/image-20261110213108824.png)

发现是一个私钥文件

```
chmod 600 111
ssh-keygen -y -f 111(从私钥导出对应的公钥)
ssh-keygen -l -f 111（拿到公钥指纹）
```

直接

```
ssh -i 111 jack@192.168.174.194
```

报错![image-20261110213718169](Pumpkinfestival.assets/image-20261110213718169.png)

原因是公钥校验不通过，这个靶机比较老，只认PEM格式，现在新版OpenSSH默认是OPENSSH格式

- 正确老 PEM：`-----BEGIN RSA PRIVATE KEY-----`

- 新版格式：`-----BEGIN OPENSSH PRIVATE KEY-----`

部分老旧靶机不支持 OPENSSH 格式，会直接拒绝

**这个靶机就是个例子**

- 转换为PEM

```
ssh-keygen -p -m PEM -f 111
```

再次尝试

![image-20261110214001941](Pumpkinfestival.assets/image-20261110214001941.png)

又报错，但信息不一样，是**加密签名算法不兼容**，新版 OpenSSH 默认禁用了 rsa-sha1（老 RSA 签名算法），靶机 ssh 只支持老 RSA 签名，两边协商不到共同签名算法。

```
ssh -o PubkeyAcceptedAlgorithms=+ssh-rsa -i 111 jack@192.168.174.194 -p6880
```

![image-20261110214158755](Pumpkinfestival.assets/image-20261110214158755.png)

成功连接

# 提权

![image-20261110214358845](Pumpkinfestival.assets/image-20261110214358845.png)

进去账号file文件，发现第七个token

- 由于前面我的得到了jack的密码

```
sudo -l
```

![image-20261110214516448](Pumpkinfestival.assets/image-20261110214516448.png)

发现/home/jack/pumpkins/alohomora*文件可以直接以管理员身份运行，且我们可以编写，没有这个文件，那我们就直接创建，并写入**/bin/bash**

```
chmod 777 /home/jack/pumpkins/alohomora*
```

![image-20261110214702359](Pumpkinfestival.assets/image-20261110214702359.png)

成功提权

# 补充

有3个token没有找的

- 第八个

192.168.174.194的源代码有一个

![image-20261110214819918](Pumpkinfestival.assets/image-20261110214819918.png)

- 第九个

http://pumpkins.local

![image-20261110214916866](Pumpkinfestival.assets/image-20261110214916866.png)

默认网页

- 第十个![image-20261110215313239](Pumpkinfestival.assets/image-20261110215313239.png)

在profile下发现第四个token
