# Burpsuite

[TOC]

·Repeater

·Intruder

## Repeater（Ctrl+R）

![Showing a sample request in burp suite repeater](https://tryhackme-images.s3.amazonaws.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/3a4a4ee2008a2ed6a1aa8e9af3601ab2.png)

### 1.Request List（请求列表）

### 2.Request Controls（请求控制）

### 3.Requset and Response View（请求与响应视图）

1.Pretty：默认选项

2.Raw：显示服务器直接收到的未修改的响应，无需格外格式化

3.Hex：以字节级表示方式分析响应，处理二进制文件时有用

4.Render（渲染）：看网页的样子

右侧的\n可以看到“显示不可打印的字符”，如\r\n

------

### 4.Layout Options（布局选项）

### 5.Inspector（检查器）

检查器位于右侧， 使我们能够比 使用原始编辑器更直观地分析和修改请求。

个人不喜欢用，相当于在用hackerbar一样

------

### 6.Target（目标）

### 注意：

类似/about/ID这样的路径，也可能存在sql注入，感觉像那种用户页面，比如id=1/2/3/4这种的，加个 ' 或者 " 都可以试试有没有sql注入

------

## Intruder

![image-20260407204855128](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20260407204855128.png)

### 1.Positions（位置）

基本功能，攻击方式

#### Attack Types：

·Sniper：一个一个试，只能用一个词表，并且只能爆破一个

·Battering ram：词表不限，同时选择的爆破对象的载荷一样

·Pitchfork：可选择1-20个词典，所有的payload按顺序对应逐步进行爆破，和机器检查一样

·Cluster bomb：流量最大，会将所有词典的payload能组合的全部试一遍

### 2.Payloads

选择默认词典和数量，顺序

### 3.Payload configuration（载荷列表设置）

词典

### 4.Payload processing（载荷预处理规则）

