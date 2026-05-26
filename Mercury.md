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

```

