[TOC]

## 信息收集

```
arp-scan -l
nmap -p- --min-rate 10000 192.168.174.163
```

开放了两个端口，分别是22，80

22端口我们先跳过，重点查看80端口

- 分别进行tcp详细扫描，漏洞扫描和udp扫描

![image-20260818154533147](DC_7.assets/image-20260818154533147.png)

![image-20260818161707095](DC_7.assets/image-20260818161707095.png)

![image-20260818161810382](DC_7.assets/image-20260818161810382.png)

我们可以得知靶机的80网页用的是Drupal的CMS框架，版本是8

## 80端口分析

先进行目录爆破

```
gobuster dir -u "http://192.168.174.163" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,zip
```

爆破出的目录大部分都是Drupal的目录框架，介绍一下Drupal

------

## Drupal

- **官网**：[drupal.org](https://drupal.org/)
- **语言**：PHP，数据库常用 MySQL/PostgreSQL
- **特点**：模块化、企业级、安全性相对较好（但历史漏洞多）
- **版本**：7.x、8.x、9.x、10.x（目前 7 已停止支持，但很多老站仍在使用）
- **后台入口**：`/user/login`（登录页面）；管理后台通常为 `/admin`（需登录且具有管理权限）
- **关键目录/文件**：
  - `sites/default/settings.php`：核心配置文件，含数据库凭据、盐值等
  - `sites/default/files/`：上传目录（常被用于写 Webshell）
  - `modules/`：核心及第三方模块
  - `themes/`：主题
  - `profiles/`：安装配置文件

**与 Joomla/WordPress 的区别**：

- 后台路径不同：Joomla `/administrator`，WordPress `/wp-admin`，Drupal `/user/login`
- Drupal 使用模块（Module）而非插件（Plugin），功能更底层，安全风险常出在自定义模块和核心 API。

### Drupal 常见攻击面与漏洞类型

| 攻击面                   | 漏洞类型                   | 说明                                                   |
| :----------------------- | :------------------------- | :----------------------------------------------------- |
| **核心漏洞**             | SQL注入、RCE、反序列化     | 最著名的是 Drupalgeddon 系列，影响极广                 |
| **第三方模块**           | RCE、XSS、SQL注入          | 模块质量参差不齐，如 `webform`, `views`, `ckeditor` 等 |
| **REST API**             | 未授权访问、信息泄露、RCE  | Drupal 8+ 默认启用 JSON:API/REST，配置不当导致数据泄露 |
| **后台登录**             | 弱口令、暴力破解、用户枚举 | 登录页无验证码，可爆破                                 |
| **配置文件**             | 信息泄露                   | `settings.php` 备份、`.git` 泄露等                     |
| **文件上传**             | 上传绕过、任意文件写入     | 结合模块漏洞实现 Webshell                              |
| **用户注册/枚举**        | 用户名枚举、权限配置错误   | 注册页面可能暴露有效用户名                             |
| **跨站请求伪造（CSRF）** | 配置错误                   | Drupal 自身有令牌，但某些模块可能缺失                  |

和wordpress，joomal一样，都有其专属的扫描器，Drupal用的是droopescan，但是我的配置导致我不能用，不过也可以用nmap，利用脚本nmap --script=http-drupal-enum --script-args http-drupal-enum.root="/" <target>,也可以使用，不会的话可以在nmap官网上查询

------

![image-20260818161916493](DC_7.assets/image-20260818161916493.png)接下来依次查看爆破出的目录，看看有没有信息

- 查看默认页面

![image-20260818162618862](DC_7.assets/image-20260818162618862.png)

告诉我们不要暴力破解，要跳出常规的思维模式来思考问题，其他的目录基本也没什么信息，经过很长时间的尝试，想找一下CMS版本，插件（模板），主题的漏洞，或者用专用工具扫描，都没有什么作用，没办法只能借鉴一下大佬的wp，然后才知道![image-20260818164910384](DC_7.assets/image-20260818164910384.png)

这个@DC7USER直接在搜索引擎搜索，可以直接搜索出github上的一个用户，然后进去发现包含许多源码的文件，这就是利用了社工信息收集，确实挺新颖的哈![image-20260818165156884](DC_7.assets/image-20260818165156884.png)

在config.php中发现了用于连接mysql的php内置函数，里面有用户名和密码，尝试一下能不能登录Drupal或者连接ssh

![image-20260818171122615](DC_7.assets/image-20260818171122615.png)

成功进入，进行了一系列的信息收集，没有可以利用的地方，然后开始查看我们当前用户有权限的文件，以及去/home等目录查找更多利于提权的信息

![image-20260818171305478](DC_7.assets/image-20260818171305478.png)

发现未知文件mbox和backups目录

- 查看mbox文件![image-20260818172554177](DC_7.assets/image-20260818172554177.png)

发现这是一个邮件，由于第一次遇到这个种靶机，我要区别一下怎么看出这是一个原始邮件（EML格式），而且他是定时的

------

## mail

### 1. 标准的“邮件头”格式（最核心标志）

邮件头是一系列以**“字段名: 值”**形式出现的关键信息，位于文本最顶部。截图里包含这些：

- **`From: root@dc-7 (Cron Daemon)`** → 明确说明发件人是谁。
- **`To: root@dc-7`** → 明确说明收件人是谁。
- **`Subject: Cron <root@dc-7> /opt/scripts/backups.sh`** → 这是邮件主题。
- **`Date: Thu, 29 Aug 2019 17:00:22 +1000`** → 发送时间。

普通日志（比如 `/var/log/syslog`）通常是以时间戳开头，不会有这种 `From:`、`To:`、`Subject:` 这种标准的邮件头。

### ✅ 2. 路由追踪信息（Received 字段）

文件中间有几行以 `Received: from` 开头的部分，这是记录邮件经过的服务器路径。例如：
`Received: from root by dc-7 with local (Exim 4.89)`
这表示邮件是由 `root` 用户通过本地的 Exim 邮件服务（本地投递）生成的。这是内部邮件投递的典型特征。

### ✅ 3. 邮件正文与头部的“空行分隔”

仔细观察，你会发现邮件头和正文之间有一个**空行**。
头部是：

text

```
Subject: Cron <root@dc-7> /opt/scripts/backups.sh
MIME-Version: 1.0
...
Message-Id: <E113EPu-0000CV-5C@dc-7>
Date: Thu, 29 Aug 2019 17:00:22 +1000
```

接着空一行，然后是正文：

text

```
Database dump saved to /home/dc7user/backups/website.sql    [success]
gpg: symmetric encryption ... failed: File exists
```

这个**空行**是标准的邮件协议（RFC 822）规定的，用于区分控制信息和内容。

- 然后我们也可以执行mail命令来查看邮件

  ------

  1. **扫一眼开头两行**：看有没有 `From` 或 `Return-path`。
  2. **找 `Subject` 行**：如果有，说明有人或系统定时任务（Cron）发了一封主题为执行备份脚本的邮件。
  3. **看收件人**：如果都是发给 `root@dc-7`，说明这是系统产生的通知，通常保存在收件箱里。

我们可以通过这三个步骤快速判断

- 仔细观察的话我们是可以从Subject这一行发现干了什么的![image-20260818173325562](DC_7.assets/image-20260818173325562.png)

定时执行了/opt/scripts/backups.sh脚本

- 因为这个邮件的发送人和接收人都是自己，所以我们也可以直接查看收件箱，直接执行命令mail![image-20260818173529967](DC_7.assets/image-20260818173529967.png)

（我们在/etc/crontab种没有看到这个任务，说明他有可能是root用户的定时任务）

- 查看/opt/scripts/backups.sh

![image-20260818174537209](DC_7.assets/image-20260818174537209.png)

这里面有两个新命令了解一下，gpg和drush，chown适用于改变文件或文件夹的文件所有者或文件所属用户组