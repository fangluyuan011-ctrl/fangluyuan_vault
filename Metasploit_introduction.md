# Metasploit

------

·Exploit（漏洞利用）

·Vulnerability（漏洞）

·Payload（有效载荷）

## msfconsole

1.设置参数

2.利用

- ### show options(设置变量)

- `show` 命令可以在任何上下文中使用，后跟模块类型（辅助、有效载荷、漏洞等 ）来列出可用模块

- 主命令行界面（）在终端上直接输入msfconsole

- 帮助命令用help
- 换模块 eg：use exploit/windows/smb/ms17_010_eternalblue

- ### info:可以获得关于任何模块的更多信息

- ##### search：可以 用CVE编号、漏洞名（如

  eternalblue、heartbleed等）或目标系统进行搜索。

  搭配上use numbers可以直接切换对应的模块

- set PARAMETER_NAME VALUE(设置参数,一般先用show options查看是个好习惯)

  #### “R”开头是目的，”L“是本地

  - unset all ，清除所有参数

  - setg命令，等于在所有模块同时设置值，unsetg用于相反效果

  - back，返回上一个模块

    ### Using modules

    - 设置好参数后，输入命令exploit，启动模块
    - exploit -z，会在运行漏洞后返回后台会话
    - ctrl+z用于后台会话
    - sessions命令用于访问上下文现有会话
    - 要与任何会话交互，用sessions -i 命令，后加上所需的会话编号

------



## Modules

可以用tree -L 1(lots) nops(modules)/ 来查看

eg：tree -L 1 nops/

| Auxiliary | 任何辅助模块，如scanners, crawlers(爬虫) and fuzzers，都可以在这里找到。 |
| --------- | ------------------------------------------------------------ |
| Encoders  | 编码器允许你编码漏洞利用和负载基于特征码的杀毒和安全解决方案拥有已知威胁数据库。它们通过将可疑文件与数据库进行比对来检测威胁，并在匹配时发出警报。因此，编码器的成功率有限，因为杀毒软件还能进行额外检查。 |
| Evasion   | 虽然编码器会编码载荷，但不应被视为直接规避杀毒软件的手段。另一方面，“规避”模块会尝试，效果或多或少。 |
| Exploits  | 漏洞利用，按目标系统整齐组织。                               |
| NOPs      | NOPs（无操作）真的什么都没做。它们以 Intel x86 CPU 家族的 0x90 表示，之后 CPU 在一个周期内将不做任何工作。它们常被用作缓冲器，以实现有效载荷大小的一致。 |
| Payloads  | 有效载荷是将在目标系统上运行的代码。                         |
| Post      | Post模块将在上述渗透测试流程的最后阶段——利用后阶段非常有用。 |

其中payloads下有

- Adapters:适配器(adapter)将单个有效载荷包裹起来，将其转换为不同格式。例如，一个普通的单一有效载荷可以被包裹在 Powershell 适配器中，适配器会发出一个 PowerShell 命令来执行该负载。

- Singles:自包含的有效载荷（添加用户、启动notepad.exe等），无需下载额外组件即可运行。

- Stagers:负责建立Metasploit与目标系统之间的连接通道。在处理分级有效载荷时非常有用。“分级有效载荷”会先在目标系统上传一个分段器，然后再下载其余的载荷（阶段）。这带来了一些优势，因为初始有效载荷的大小相较于一次性发送的全部有效载荷会相对较小。

- Stages:由stager下载。这样你就能使用更大尺寸的载荷。



- generic/shell_reverse_tcp

- windows/x64/shell/reverse_tcp

  两者都是反向Windows shells。前者是直联（或单）有效载荷，如“shell”和“reverse”之间的“_”所示。而后者则是分级有效载荷，正如“shell”和“reverse”之间的“/”所示。

------



## Tools