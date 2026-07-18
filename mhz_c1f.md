在开始打靶机之前就先遇到了一个问题，我发现很多靶机都这样，这里同一总结一下关于靶机找不到ip，端口扫不出的问题

### 网卡配置

靶机开机后按住shift，出现界面如图，按e键进入安全模式：

![image-20260718223814906](mhz_c1f.assets/image-20260718223814906.png)

**找到ro，删除该行后边内容，并将ro 。。。修改为：** `rw signie init=/bin/bash`![image-20260718223834395](mhz_c1f.assets/image-20260718223834395.png)

替换后

![image-20260718223852017](mhz_c1f.assets/image-20260718223852017.png)

按`ctrl+x`进入bash

输入指令`ip a`查看当前网卡，除lo外的，记住名字![image-20260718223907904](mhz_c1f.assets/image-20260718223907904.png)

首先查看/etc/network/文件目录下是否有interfacers文件

如果没有interfaces文件或者文件内容为：

![image-20260718223926138](mhz_c1f.assets/image-20260718223926138.png)

退出编译，查看/etc/netplan的内容：

![image-20260718223943552](mhz_c1f.assets/image-20260718223943552.png)

看到`.yaml`文件，对其进行编辑，将框内内容改为`ip a`的网卡名称

![image-20260718224012091](mhz_c1f.assets/image-20260718224012091.png)

![image-20260718224028345](mhz_c1f.assets/image-20260718224028345.png)

![image-20260718224042364](mhz_c1f.assets/image-20260718224042364.png)

保存之后直接重启系统，大功告成

- 这个流程只根据这个靶机，其他另外总结，流程是这样

### 信息收集

```
nmap -p- --min-rate 10000 192.168.174.152
```

有两个端口，分别为22，80

```
nmap -sT -sC -sV -O -p22,80 192.168.174.152 -oA /nmapscan/tcp
```

```
nmap --script=vuln -p22,80 192.168.174.152 -oA /nmapscan/vuln
```

![image-20260718224351825](mhz_c1f.assets/image-20260718224351825.png)

都没有发现什么重要信息

#### 22端口

先不考虑

#### 80端口

![image-20260718224439311](mhz_c1f.assets/image-20260718224439311.png)

只是一个apache服务页面，没有什么特别信息

- 进行fuzz测试，没有结果

- ```
  gobuster dir -u "http://192.168.174.152" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
  ```

  

![image-20260718224641288](mhz_c1f.assets/image-20260718224641288.png)

- 查看notes.txt

![image-20260718224707900](mhz_c1f.assets/image-20260718224707900.png)

- 查看remb.txt和remb2.txt

#### remb.txt

![image-20260718224815794](mhz_c1f.assets/image-20260718224815794.png)

#### remb2.txt

显示404没有

## ssh登录

用从remb.txt中得到的账户密码登录

![image-20260718224931020](mhz_c1f.assets/image-20260718224931020.png)

想得到完整终端功能

一般步骤

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
stty raw -echo; fg
```

(这个靶机只用第一条命令就可以正常使用了)

##### 先收集信息

![image-20260718225351947](mhz_c1f.assets/image-20260718225351947.png)

- 得到第一个flag，起初我以为这个最后一行注释是什么密码，但到最后也没用

- 在first_stage用户下的文件依次查看，均没有重要信息

随后按以下步骤依次查看

- sudo -l
- ![image-20260718225833225](mhz_c1f.assets/image-20260718225833225.png)

- id
- ![image-20260718225624751](mhz_c1f.assets/image-20260718225624751.png)

- uname -a
- ![image-20260718225640782](mhz_c1f.assets/image-20260718225640782.png)

- find / -writable -type f 2>/dev/null | grep -v sys | grep -v proc
- ![image-20260718225718979](mhz_c1f.assets/image-20260718225718979.png)
- find / -perm -u=s -type f 2>/dev/null
- ![image-20260718225801508](mhz_c1f.assets/image-20260718225801508.png)
- getcap -r / 2>/dev/null
- ![image-20260718225927846](mhz_c1f.assets/image-20260718225927846.png)
- cat /etc/crontab
- ![image-20260718230008340](mhz_c1f.assets/image-20260718230008340.png)

经过以上很多尝试均没有结果

接下来尝试一下直接看其他用户的信息

```
cd /home
ls -la
```

![image-20260718230156926](mhz_c1f.assets/image-20260718230156926.png)

发现可疑文件夹Paintings

![image-20260718230220445](mhz_c1f.assets/image-20260718230220445.png)

发现里面有四张图片，而且我们有读的权限

```
scp first_stage@192.168.174.152:/home/mhz_c1f/Paintings /exam
```

(这是ssh中下载连接机器文件的命令，前提是我们要有对应文件的读权限)

- 尝试了file，strings，binwalk，exiftool均没有关键信息，接下来尝试是否有隐写

```
steghide info <filename>（有空格的话需要加\来表示转义）
```

![image-20260718232446907](mhz_c1f.assets/image-20260718232446907.png)

因为我们没有密码，四个都尝试一下后发现spinning the wool.jpeg可以直接探测，并且存在remb2.txt

```
steghide extract -sf spinning\ the\ wool.jpeg
```

![image-20260718232701589](mhz_c1f.assets/image-20260718232701589.png)

得到remb2.txt的内容

```
su mhz_c1f
```

```
sudo -l
```

![image-20260718232932131](mhz_c1f.assets/image-20260718232932131.png)

1. 第一个 ALL： (ALL : xxx)  → 括号左半，可切换目标用户

- 含义：执行 sudo -u 用户名 时，能切换到哪个系统用户
- 取值示例：
- (ALL) ：可以切换系统里任何用户（root、www-data、mysql等）
- (root) ：仅能切换root，不能切其他普通用户
- (www-data) ：只能切换网站服务用户，拿不到root

2. 第二个 ALL： (xxx : ALL)  → 括号右半，可切换目标用户组

- 含义：执行 sudo -g 用户组 时，能切换到哪个用户组
- 取值示例：
- (:ALL) ：可以切换系统任意用户组（root组、docker组等）
- (:root) ：仅能切换root组
- (:docker) ：仅能切换docker组

3. 末尾的 ALL：命令段，允许执行的程序

- 含义：当前用户能sudo运行哪些命令
- 取值示例：
- ALL ：无限制，系统所有二进制程序都能sudo执行
- /usr/bin/cat ：仅允许sudo cat，其他命令无权
- !/usr/bin/su, ALL ：允许所有命令，但禁止sudo su切换root

所以我们可以直接切换到root用户