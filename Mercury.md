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