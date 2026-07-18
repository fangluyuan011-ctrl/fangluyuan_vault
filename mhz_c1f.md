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

