知道靶机ip为192.168.174.135

```
nmap -sT -sV -sC -O -p- --min-rate 10000 192.168.174.135 -oA /nmapscan/tcp
```

```
nmap -sU --min-rate 10000 -p- 192.168.174.135 -oA /nmapscan/udp
```

分别进行tcp和udp扫描

![image-20260526210844767](Mercury.assets/image-20260526210844767.png)

从t'c