---
layout:     post
title:     Nmap和Hydra使用
subtitle:  扫描与爆破
date:       2020-5-5
author:     One-Punch-Man
header-img: "img/man-6.jpg"
catalog: true
tags: 
     - 网络安全
     - 基础知识
---

# 常见端口

| 服务        | 端口  |
| ----------- | ----- |
| FTP         | 20\21 |
| SSH         | 22    |
| Telnet      | 23    |
| SMTP        | 25    |
| HTTP        | 80    |
| HTTPS       | 443   |
| POP3        | 110   |
| IMAP        | 143   |
| SQL  Server | 1433  |
| MySQL       | 3306  |
| Oracle      | 1521  |
| RDP         | 3389  |

# Nmap

`nmap [扫描类型] [选项] {目标规范}`

### 端口状态介绍

- `open`:端口开放。
- `closed`:关闭的端口对于Nmap也是可访问的， 它接收Nmap探测报文并作出响应。但没有应用程序在其上监听。
- `filtered`:由于包过滤阻止探测报文到达端口，Nmap**无法确定该端口是否开放**。过滤可能来自专业的防火墙设备、路由规则或者主机上的软件防火墙。
- `unfiltered`:未被过滤状态意味着端口**可访问**，但是Nmap**无法确定它是开放还是关闭**。 只有用于映射防火墙规则集的 ACK 扫描才会把端口分类到这个状态。
- `open|filtered`:**无法确定端口是开放还是被过滤**， 开放的端口不响应就是一个例子。没有响应也可能意味着报文过滤器丢弃了探测报文或者它引发的任何反应。**UDP，IP协议,FIN, Null 等扫描会引起**。
- `closed|filtered`:**无法确定端口是关闭的还是被过滤的**。

### 目标规范

可以传递主机名、IP地址、网络等。例如`one-pnch-man.top`,`microsoft.com/24`,`192.68.0.1`,`10.0.0-254.1-254`。

- `-iL < input file name>`:从文件中读取待检测的目标，文件中的表示方法支持机名，ip，网段 。
- `-iR <num hosts>`:随机选取,进行扫描.如果`-iR`指定为**0**,则是无休止的扫描。
- ` --exclude <host1[,host2][,host3],...>`: 排除主机/网络进行扫描。
- ` --excludefile <exclude_file>`：排除所指定的文件中所包含的IP地址进行扫描。

### 主机发现

- ` -sL`:仅列出要扫描的目标数目，不进行扫描。
- `-sn`:Ping扫描，不进行端口扫描。
- `-Pn`:将所有主机都默认为在线，跳过主机发现，不检测主机存活。
- `-PS/PA/PU/PY[port list]`:对给定端口的TCP SYN / ACK，UDP或SCTP 进行Ping扫描。
- `-PE/PP/PM`:使用ICMP echo, timestamp and netmask 请求包扫描主机。
- `-PO[protocol list]`:使用IP协议包探测对方主机是否开启。
- ` -n/-R`:从不解析DNS /始终解析[默认值：有时]。
- ` --dns-servers <serv1[,serv2],...>`:指定自定义DNS服务器。
- ` --system-dns`:使用操作系统的DNS解析器。
- `  --traceroute`:显示跟踪到每个主机的跃点路径。

### 扫描技术

- `-sS / sT / sA / sW / sM`:TCP扫描
  - S是SYN扫描，半连接扫描，nmap只发送SYN报文，通过服务器是否响应SYN+ACK来判断对应端口是否开放
  - T是全连接扫描，会和服务器建立完整的三次握手，效率低
  - A发送ACK报文，通过服务器响应来判断是否开放，有的服务器不开会回复ICMP端口不可达，当回复RST时表示可能被拦截或者端口开放，不是一个准确的判断条件
  - W 是窗口扫描，发出的报文和ACK一样，利用的是在某些系统中如果端口开放，收到ACK包后会响应一个窗口非0的RST包
  - M是Maimon扫描，使用发现者的名字命名。其原理是向目标服务器发送FIN/ACK 报文，在某些系统中如果端口开放则会丢弃该报文不做响应，如果端口关闭则回复RST或者ICMP，Nmap可借此判断服务器端口的开放情况。
- `-sU`:UDP扫描，某些系统如果UDP端口不开放会回复ICMP差错报文（这也是Linux系统中traceroute的实现原理）。Nmap UDP端口扫描的强大之处在于它会针对知名端口构造初始交互报文，比如会针对UDP 500构造一个主模式协商的IKE报文。
- `-sN/sF/sX`:特定TCP标志位的扫描，N是空标志位；F是FIN置位；X是Xmas扫描将FIN、PSH、URG同时置位。收到RST说明端口关闭，无响应说明被过滤或者端口开放，不准。
- ` --scanflags <flags>`:自定义TCP扫描标志位进行扫描。
- `  -sI <zombie host[:probeport]>`:空闲扫描。Idle扫描需要一台没有流量的僵尸主机，这种扫描的实现原理是在一定的时间里，同一台主机发出的IP数据报文其ip头中的identification字段是累加的。探测分为3步：1、Nmap主机向僵尸机发包，通过僵尸机的响应包探测其ID；2、Nmap主机伪造僵尸机源地址向服务器的特定端口发送SYN包；3、Nmap主机再次探测僵尸机的ip.id。如果目标服务器端口开放，则必然会向僵尸机发送SYN/ACK，由于莫名其妙收到一个SYN/ACK 报文，僵尸机会向目标服务器发送RST报文，该报文的ip.id 是第一步+1，则第三步Nmap主机探测到的ip.id应该是第一步+2，说明目标主机端口开放。反之，如果目标主机端口未开放，则收到第二步的报文后会向僵尸机回复RST或者直接丢弃该报文不响应，无论哪种情况，都不会触发僵尸机发包，进而僵尸机的ip.id不会变化，第三步Nmap探测到的id应该是第一步+1。
- `-sY / sZ`:SCTP协议INIT或cookie-echo扫描。
- `-sO`:基于IP协议的扫描，通过变换IP报文头中的Protocol值来对服务器进行探测。
- ` -b <FTP relay host>`:FTP反弹扫描，借助FTP特性，通过FTP服务器连接想要扫描的主机实现隐身的目的。
- `  -p <port ranges>`:仅扫描指定的端口。
- ` --exclude-ports <port ranges>`:从扫描中排除指定端口。
- ` -F`:快速模式，扫描比默认端口数量更少的端口（100）。
- `  -r`:指定这个选项后，程序将从按照从小到大的顺序扫描端口,默认是随机扫描。
- ` --top-ports <number>`: 扫描开放概率最高的number个端口,出现的概率需要参考nmap-services文件，nmap默认扫描前1000个。
- `--port-ratio <ratio>`:按比例扫描知名端口，值在0-1之间，越小扫的越多。

### 服务/版本检测

- `-sV`:探测开放的端口的系统/服务信息
- `--version-intensity <level>`:设置版本扫描强度,强度水平说明了应该使用哪些探测报文。数值越高，服务越有可能被正确识别。默认是7。
- `--version-light`:  使用轻量级模式(--version-intensity 2)。
- `--version-all`:尝试所有探测（--version-intensity 9）。
- `--version-trace`: 显示出详细的版本侦测过程信息。

### 脚本扫描

- `-sC`:根据端口识别的服务,调用默认脚本(相当于--script = default)。
- `--script=<Lua scripts>`:调用指定的脚本。
- `--script-args=<n1=v1,[n2=v2,...]>`:给调用的脚本指定参数。
- `  --script-args-file=filename`:指定提供脚本参数的文件。
- `--script-trace`:显示所有发送和接收的数据。
- `--script-updatedb`:更新脚本数据库。
- `--script-help=<Lua scripts>`:查看调用的脚本帮助信息。

### 操作系统检测

- `-O`:启用操作系统检测。
- `--osscan-limit`:只对开放端口的有效主机进行系统探测。
- `--osscan-guess`:推测操作系统检测结果,当Nmap**无法确定**所检测的操作系统时，会尽可能地提供最相近的匹配，Nmap**默认**进行这种匹配。

### 时间与性能

- `-T<0-5>`:设置时序模板（越高越快）。
- ` --min-hostgroup/max-hostgroup <size>`:并行主机扫描组大小。
- ` --min-parallelism/max-parallelism <numprobes>`:探针并行化。
- ` --min-rtt-timeout/max-rtt-timeout/initial-rtt-timeout <time>`:指定
        探测往返时间。
- ` --max-retries <tries>`:限制端口扫描探针重传的次数。
- `  --host-timeout <time>`:在很长一段时间后放弃目标。
- `--scan-delay/--max-scan-delay <time>`:调整探针之间的延迟。
- ` --min-rate <number>`:每秒发送数据包的速度不低于number。
- `--max-rate <number>`:每秒发送数据包的速度不超过number。

### 防火墙/IDS躲避和哄骗

- `  -F; --mtu <val>`:
  - `-F`:这个选项可避免对方识别出我们探测的数据包。指定这个选项之后， Nmap将使用8字节甚至更小数据体的数据包。
  - `--mtu <val>`:这个选项用来调整探测数据包的包大小。MTU（Maximum Transmission Unit，最大传输 单元）必须是8的整数倍，否则Nmap将报错。
- `  -D <decoy1，decoy2 [，ME]，...>`:这个选项应指定假IP，即诱饵的IP。启用这个选项之后，Nmap在发送侦测数据包的时候会掺杂一些源地址是假IP（诱饵）的数据包。这种功能意在以藏木于林的方法掩盖本机的真实IP。也就是说，对方的log还会记录下本机的真实IP。您可使用RND生成随机的假IP地址，或者用RND：number的参数生成number个假IP地址。您所指定的诱饵主机 应当在线，否则很容易击溃目标主机。另外，使用了过多的诱饵可能造成网络拥堵。尤其是在扫描客户的网络的时候，您应当极力避免上述情况。
- `  -S <IP_Address>`:欺骗源地址。
-   `-e <iface>`:使用指定的接口。
- `-g/--source-port <portnum>`:使用给定的端口号。**如果防火墙只允许某些源端口的入站流 量，这个选项就非常有用。**
- ` --proxies <url1，[url2]，...>`:通过HTTP / SOCKS4代理连接。
- ` --data <hex string>`:将自定义有效负载附加到发送的数据包。
- `--data-string <string>`:将自定义ASCII字符串附加到发送的数据包。
- `--data-length <num>`:这个选项用于改变Nmap发送数据包的默认数据长度，以避免被识别出来是Nmap的扫描数据。
- ` --ip-options <options>`:使用指定的IP选项来发送数据包。
- ` --ttl <val>`:设置IP生存时间字段。
- `  --spoof-mac <mac address/prefix/vendor name>`:MAC地址伪装。
- `--badsum`: 发送带有虚假TCP / UDP / SCTP校验和的数据包。

### 输出

- `-oN/-oX/-oS/-oG <file>`:以普通/XML/大写 /便于通过bash或者perl处理非xml的格式输出到给定的文件中。
- ` -oA <basename>`:将扫描结果以标准格式、XML格式和Grep格式一次性输出。
- ` -v`: 提高输出信息的详细度（使用-vv效果更好）。
- ` -d`:提高调试级别，最高是9（使用-dd以获得更大的效果）。
- `--reason`:显示端口处于确定状态的原因。
- ` --open`:仅显示打开（或可能打开）的端口。
- `  --packet-trace:`:显示所有发送和接收的数据包
- `--iflist`:显示主机接口和路由信息（用于调试）。
- `--append-output`:把输出追加到而不是破坏指定的文件。
- ` --resume <filename>`:恢复中止的扫描。
- ` --stylesheet <path/URL>`:XSL样式表，可将XML输出转换为HTML。
- `--webxml`:从namp.org得到XML的样式。
- `--no-stylesheet`: 忽略XML声明的XSL样式表。

### MISC

- `-6`:启用IPv6扫描
- `-A`:启用操作系统检测，版本检测，脚本扫描和跟踪路由。
- `--datadir <dirname>`:说明用户Nmap数据文件位置。
- ` --send-eth/--send-ip`:使用原始以太网帧/IP数据包发送。
- `--privileged`:假设用户具有完全特权。
- `--unprivileged`:假设用户缺乏原始套接字特权
- `-V`:打印Nmap版本号并退出。
- `-h`:打印一个短的帮助屏幕，列出大部分常用的 命令选项，这个功能与不带参数运行Nmap是相同的。

### 常用指令

```powershell
nmap -Pn -sV www.moonsec.com
nmap -Pn -sV 103.97.177.22
nmap -Pn -sV 103.97.177.22 –O
nmap -Pn -sV -A 103.97.177.22 
nmap -Pn -sV 103.97.177.22 --open -oN moonsec.com.txt
nmap -p0-65535 192.168.0.107
nmap -v -A -F -iL target.com.txt -oX target_f.xml
nmap -v -A -F -iL 1.txt -oN target_f.txt
nmap -v -A -p1-65535 -iL target.com.txt -oX target_all.xml
```

# Hydra

### 语法

`hydra [[[-l LOGIN|-L FILE] [-p PASS|-P FILE]] | [-C FILE]] [-e nsr] [-o FILE] [-t TASKS] [-M FILE [-T TASKS]] [-w TIME] [-W TIME] [-f] [-s PORT] [-x MIN:MAX:CHARSET] [-SuvVd46] [service://server[:PORT][/OPT]]`

- `server`:目标ip地址。
- `service`: 指定服务名，支持的服务和协议：`telnet ftp pop3[-ntlm] imap[-ntlm] smb smbnt http-{head|get} http-{get|post}-form http-proxy cisco cisco-enable vnc ldap2 ldap3 mssql mysql oracle-listener postgres nntp socks5 rexec rlogin pcnfs snmp rsh cvs svn icq sapr3 ssh smtp-auth[-ntlm] pcanywhere teamspeak sip vmauthd firebird ncp afp等等`。
- `OPT`:可选项。

### 参数

- `-R`:继续从上一次进度接着破解。
- `-S `:采用SSL链接。
- `-s PORT `:可通过这个参数指定非默认端口。
- `-l LOGIN`:指定破解的用户，对特定用户破解。
- `-L FILE `:指定用户名字典。
- `-p PASS`:指定密码破解，少用，一般是采用密码字典。
- `-P FILE`:指定密码字典。
- `-e ns`:可选选项，n：空密码试探，s：使用指定用户和密码试探。
- `-C FILE`:使用冒号分割格式，例如“登录名:密码”来代替-L/-P参数。
- `-M FILE `:指定目标列表文件一行一条。
- `-o FILE`:指定结果输出文件。
- `-f `:在使用-M参数以后，找到第一对登录名或者密码的时候中止破解。
- `-t TASKS`: 同时运行的线程数，默认为16。
- `-w TIME`:设置最大超时的时间，单位秒，默认是30s。
- `-v / -V `: 显示详细过程。

### 常用指令

```powershell
ssh
hydra -L /root/user -P /root/passwd ssh://192.168.1.0 -f -o /root/crack.txt -V
ftp
hydra -L /root/user -P /root/passwd ftp://192.168.1.0 -f -o /root/crack.txt -V
rdp
hydra -L /root/user -P /root/passwd rdp://192.168.1.0 -f -o /root/crack.txt -V
mssql
hydra -L /root/user -P /root/passwd mssql://192.168.0.129 -f -o /root/crack.txt -v
mysql
hydra -L /root/user -P /root/passwd mysql://192.168.0.129 -f -o /root/crack.txt –v -s 3306
oracle
hydra -P /root/passwd oracle://192.168.0.129 -f -o /root/crack.txt –v
redis
hydra -P /root/passlist.txt -e nsr -t 16 192.168.0.101 redis
postgresql 弱口令检测
hydra -P /root/passlist.txt -e nsr -t 16 192.168.0.101 postgresql
指定多个ip进行穷举
hydra -L /root/user -P /root/passlist -M /root/ip.txt  -V -o /root/crack mysql -t 16
hydra -L /root/user -P /root/passlist ssh://192.168.0.112 -vV -f
hydra -L /root/user -P /root/passlist ssh://192.168.0.112 -vV -f -o /root/crack.txt
hydra -L /root/user -P /root/passlist ftp://192.168.0.106 -vV -f -o /root/crack.txt
hydra -l sa -P /root/passlist mssql://192.168.0.103 –vV
```

