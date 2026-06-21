## 端口扫描

```
nmap -p- --min-rate 10000 192.168.174.148
```

```
nmap -p21,22,80,111,445,2049,2121,20048 -sT -sC -sV -O 192.168.174.148 -oA /nmapscan/tcp
```

![image-20260617212510633](My_file_server1.assets/image-20260617212510633.png)

![image-20260617212534441](My_file_server1.assets/image-20260617212534441.png)

```
nmap -p21,22,80,111,445,2049,2121,20048 --script=vuln 192.168.174.148 -oA /nmapscan/detail
```

![image-20260617212814941](My_file_server1.assets/image-20260617212814941.png)

#### 21端口

```
ftp 192.168.174.148
```

先尝试anyonmous匿名登录，登录失败

#### 22端口

ssh服务，最次考虑

#### 80端口

```
gobuster dir -u "http://192.168.174.148" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,php,zip
```

结合nmap扫描结果

有/icons和/readme.txt两个目录文件

先进入网页

![image-20260617213317563](My_file_server1.assets/image-20260617213317563.png)

从源代码看出这指向一个连接，但进去后是个网址https://www.armourinfosec.com/，感觉像宣传的，没意思

##### /icons/

- ![image-20260617213627243](My_file_server1.assets/image-20260617213627243.png)

这些图片都是一些注释，没找到有用的

##### /readme.txt

- ![image-20260617213714663](My_file_server1.assets/image-20260617213714663.png)

拿到了一个密码，虽然暂时不知道用户是谁，反正试过了不是root的

#### 111，2049端口

- 说明可以查看挂载

```
showmount -e 192.168.174.148
```

![image-20260618184452658](My_file_server1.assets/image-20260618184452658.png)

网段要在192.168.56.0/24才能访问，感觉有点麻烦先不管

2121端口

服务是ProFTPD1.3.5，通过搜索存在mod_cpoy漏洞

![image-20260618205912090](My_file_server1.assets/image-20260618205912090.png)

445端口

- samba服务，版本是smbd4.9.1 ，可以先考虑从这里入手

```
smbmap -H 192.168.174.148
```

![image-20260618200320608](My_file_server1.assets/image-20260618200320608.png)

存在一个共享目录，smbdata，权限是读和写

```
enum4linux -a 192.168.174.148
```

![image-20260618201735584](My_file_server1.assets/image-20260618201735584.png)

有一个名为smbuser的用户，此时我们就有了一组登录凭证

![image-20260618201912054](My_file_server1.assets/image-20260618201912054.png)

再次确认smbdata这个目录

```
smbclient //192.168.174.148/smbdata
```

通过smbclient访问这个共享目录

![image-20260618202056866](My_file_server1.assets/image-20260618202056866.png)

发现两个文件值得注意。secure和sshd_config

```
cat secure
```

看起来像日志

![image-20260618202416438](My_file_server1.assets/image-20260618202416438.png)

看到用户及组都是smbuser，还有一个密码，但后续我没用上

```
cat sshd_config
```

![image-20260618203136376](My_file_server1.assets/image-20260618203136376.png)

- 翻译一下， Authentication:身份验证，Authorized:授权的

- PermitRootLogin yes:允许root通过ssh登录
- AuthorizedKeysFile      .ssh/authorized_keys：被授权的密钥文件
- PasswordAuthentication no:不允许密码登录，那就只能尝试使用