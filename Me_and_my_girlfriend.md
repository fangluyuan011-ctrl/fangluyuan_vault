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

首先我们先访问一下这个网页

![image-20260613143922683](Me_and_my_girlfriend.assets/image-20260613143922683.png)

意思是只有