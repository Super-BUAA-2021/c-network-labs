## 网络层实验

### TODO

- [ ] 重做题目2和题目3

> 本实验默认你已进行过入门实验，了解一些基本操作，尽管『jbw-入门教程』十分差劲

开始实验前，打开超级终端，运行`reset save`、`reboot`恢复初始设置，前者直接y，后者先n后y。

### ARP报文分析

打开连线组网软件，组网如下，完成后提交。

![image-20211115114734865](img/image-20211115114734865.png)

接着配置PCA和PCB的IP地址。PCA：

![image-20211115211240013](img/image-20211115211240013.png)

PCB：

![image-20211115211314203](img/image-20211115211314203.png)

后，依次执行：打开PCA、PCB的cmd，输入`arp -a`。

> 题1表格第一部分，即2.6.1中步骤2中的执行结果
>
> | 输入命令      | 结果                                                        |
> | ------------- | ----------------------------------------------------------- |
> | PCA中`arp -a` | ![image-20211115165132075](img/image-20211115165132075.png) |
> | PCB中`arp -a` | ![image-20211115165224172](img/image-20211115165224172.png) |

若上述步骤中，结果不是`No ARP`，则需要输入`arp -d`清空。

然后打开PCA和PCB的Wireshark，如下图点击配置，注意IP要选择刚刚设置的IP。

![image-20211115115518103](img/image-20211115115518103.png)

![image-20211115115558550](img/image-20211115115558550.png)

PCA和PCB都如上图一样配置，然后打开PCA的cmd窗口，输入`ping 192.168.1.21`，ping PCB。

![image-20211115164454394](img/image-20211115164454394.png)

后停止PCA和PCB的Wireshark对报文的截获，总体截图(包含ARP+ICMP)用于题2，在PCA中点击ARP，展开链路层和网络层截图，用于题3。

> 题2：
>
> 答：报文截获结果如下：
>
> [截图×2]
>
> 
>
> 有2个ARP报文，8个ICMP报文，”Opcode“字段值为1代表request，2代表reply

> 题3，两张截图，将第一条ARP请求报文和第一条ARP应答报文填入表中
>
> | 字段项                   | ARP请求数据报文 | ARP应答数据报文 |
> | ------------------------ | --------------- | --------------- |
> | 链路层Destination项      |                 |                 |
> | 链路层Source项           |                 |                 |
> | 网络层Sender MAC Address |                 |                 |
> | 网络层Sender IP Address  |                 |                 |
> | 网络层Target MAC Address |                 |                 |
> | 网络层Target IP Address  |                 |                 |

点击Ctrl+S保存报文信息，保存PCA和PCB的报文，命名为`ping1-学号`，方便后续分析。

接下来，打开PCA、PCB的cmd，输入`arp -a`查看。

> 题1第二部分，即2.6.1中步骤4中的执行结果，填入截图（**如果此时arp -a没能出现列表，就再ping一次即可，好像还有时间要求，几次ping不要隔太久**）
>
> | 输入命令      | 结果                                                        |
> | ------------- | ----------------------------------------------------------- |
> | PCA中`arp -a` | ![image-20211115165815937](img/image-20211115165815937.png) |
> | PCB中`arp -a` | ![image-20211115165851609](img/image-20211115165851609.png) |

之后重新运行PCA和PCB的wireShark，在PCA中ping 192.168.1.21，后停止wireShark并保存报文为`ping2-学号`。打开ping1和ping2对比，将ping2报文截图，在ping2中filter`arp`再截图一张。

> 题4(1)：
>
> 答：少了ARP报文。原因是ARP缓存已经保存下了上一次ARP报文的内容，也就是PCB的MAC地址。所以在这一次的ping中，PCA不再需要用ARP询问PCB的MAC地址。ARP缓存可以保存之前收到的ARP回复，减少ARP请求的次数。
>
> [此处要有截图两张]
>
> ![image-20211115212812307](img/image-20211115212812307.png)
>
> ![image-20211115212828436](img/image-20211115212828436.png)

重新组网连线，组网如下，连接后提交。

![image-20211115213223741](img/image-20211115213223741.png)

设置PCA和PCB的默认网关。PCA：

![image-20211115213406298](img/image-20211115213406298.png)

PCB：

![image-20211115213520567](img/image-20211115213520567.png)

> 题4(2):按照图-4重新进行组网，并确保连线正确。修改计算机的IP地址，并将PC A的默认网关修改为192.168.1.10，PC B的默认网关修改为192.168.2.10。考虑如果不设置默认网关会有什么后果？
>
> 答：如果不设置默认网关，那么PCA ping PCB时发现PCB不在同一局域网后，由于没有默认网关，也就无法交给默认网关转发，最后只能放弃发送，ping不通。

在PCA的超级终端中，输入以下命令设置VLAN2和VLAN3：

```shell
<S1>sys
[S1]vlan 2
[S1-vlan2]port e 1/0/1
[S1-vlan2]inter vlan2
[S1-Vlan-interface2]ip add 192.168.1.10 255.255.255.0
[S1-Vlan-interface2]vlan 3
[S1-vlan3]port e 1/0/13
[S1-vlan3]inter vlan3
[S1-Vlan-interface3]ip add 192.168.2.10 255.255.255.0
```

然后类似上面的部分步骤，打开WireShark，选择本地设置的IP，运行，然后在PCA中ping 192.168.2.22。执行完后，停止PCA和PCB的截获，将报文信息保存为`ping3-学号`。并在PCA和PCB中分别点击第一条ARP展开链路层和网络层，查看详情并截图。

> 题5：
>
> 答：截获的ARP报文内容如下：PCA请求报文：
>
> ![image-20211115222255171](img/image-20211115222255171.png)
>
> PCB应答报文：
>
> ![image-20211115222319730](img/image-20211115222319730.png)
>
> 根据上图报文，填写表格：
>
> | 字段项                   | ARP请求数据报文 | ARP应答数据报文 |
> | ------------------------ | --------------- | --------------- |
> | 链路层Destination项      |                 |                 |
> | 链路层Source项           |                 |                 |
> | 网络层Sender MAC Address |                 |                 |
> | 网络层Sender IP Address  |                 |                 |
> | 网络层Target MAC Address |                 |                 |
> | 网络层Target IP Address  |                 |                 |
>
> 此时ARP请求的目标的IP地址为PCA默认网关的地址，而不是PCB的地址。
>
> 1. 区别：相同网段下，ARP会直接请求目标的MAC地址；不同网段下，ARP会请求他的默认网关的MAC地址。
>
> 2. 相同点：都各有一个ARP请求和应答报文，请求的发送者和应答的接收者都是同一台主机。

在执行完ping后，在PCA和PCB的cmd中输入`arp -a`，截图，填入第一题第三部分中。

> 题1第三部分，即2.6.2中步骤3中的执行结果，填入截图
>
> | 输入命令      | 结果                                                        |
> | ------------- | ----------------------------------------------------------- |
> | PCA中`arp -a` | ![image-20211115221405532](img/image-20211115221405532.png) |
> | PCB中`arp -a` | ![image-20211115221422002](img/image-20211115221422002.png) |

### ICMP报文分析

连线组网，提交：

![image-20211115223435810](img/image-20211115223435810.png)

修改PCA的IP：

![image-20211115223658011](img/image-20211115223658011.png)

修改PCB的IP：

![image-20211115223826360](img/image-20211115223826360.png)

打开PCA的终端，输入：

```shell
<S1>sys
[S1]vlan 2
[S1-vlan2]port e 1/0/1
[S1-vlan2]inter vlan2
[S1-Vlan-interface2]ip add 10.1.2.1 255.255.255.0
[S1-Vlan-interface2]vlan 3
[S1-vlan3]port e 1/0/23
[S1-vlan3]inter vlan3
[S1-Vlan-interface3]ip add 10.1.3.1 255.255.255.0
```

接着类似前面，打开wireshark运行，在PCA中ping 10.1.3.10，停止截获并保存，filter“icmp”并截图总体ICMP报文图，并在PCA中截图第一个ICMP发送和回应的详情，进行分析。

> 题6：共有 8 个ICMP报文
>
> 答：以下是PCA截获报文结果：
>
> ![image-20211115230051664](img/image-20211115230051664.png)
>
> 第一条ICMP回送请求报文：
>
> ![image-20211115230544811](img/image-20211115230544811.png)
>
> 第一条ICMP回送应答报文：
>
> ![image-20211115231011475](img/image-20211115231011475.png)
>
> 一共有8个ICMP报文，其中四个为回送请求报文，四个为回送应答报文。
>
> 种类字段名：Type，代码字段名：Code。
>
> 通过Sequence number (BE) 和Sequence number (LE) 字段可保证回送请求和回送应答报文一一对应。

运行PCA和PCB的wireshark，打开pingtest软件，如下配置，点击发送：

![image-20211115231506597](img/image-20211115231506597.png)

然后看一下报文截获情况，先不要停止，继续使用pingtest软件如下配置和发送：注意这次目标地址是10.1.3.10，和之前不一样。

![image-20211115232945159](img/image-20211115232945159.png)

接着就可以停止截获，保存报文信息，做以下分析：

> 题7：（第一次pingtest-两个ICMP）
>
> 答：PCA报文截获情况如下：
>
> 地址掩码请求报文：
>
> ![image-20211115231729655](img/image-20211115231729655.png)
>
> 地址掩码应答报文：
>
> ![image-20211115231825404](img/image-20211115231825404.png)
>
> | 地址掩码请求报文    |        | 地址掩码应答报文    |        |
> | ------------------- | ------ | ------------------- | ------ |
> | ICMP字段名          | 字段值 | ICMP字段名          | 字段值 |
> | Type                |        | Type                |        |
> | Code                |        | Code                |        |
> | Checksum            |        | Checksum            |        |
> | Identifier(BE)      |        | Identifier(BE)      |        |
> | Identifier(LE)      |        | Identifier(LE)      |        |
> | Sequence number(BE) |        | Sequence number(BE) |        |
> | Sequence number(LE) |        | Sequence number(LE) |        |
> | Address mask        |        | Address mask        |        |

> 题8：第二次pingtest（第二次pingtest）
>
> PCA时间戳请求报文：
>
> ![image-20211115233056767](img/image-20211115233056767.png)
>
> PCA时间戳应答报文：
>
> ![image-20211115233112756](img/image-20211115233112756.png)
>
> | 地址掩码请求报文    |        | 地址掩码应答报文    |        |
> | ------------------- | ------ | ------------------- | ------ |
> | ICMP字段名          | 字段值 | ICMP字段名          | 字段值 |
> | Type                |        | Type                |        |
> | Code                |        | Code                |        |
> | Checksum            |        | Checksum            |        |
> | Identifier(BE)      |        | Identifier(BE)      |        |
> | Identifier(LE)      |        | Identifier(LE)      |        |
> | Sequence number(BE) |        | Sequence number(BE) |        |
> | Sequence number(LE) |        | Sequence number(LE) |        |
> | Originate timestamp |        | Originate timestamp |        |
> | Receive timestamp   |        | Receive timestamp   |        |
> | Transmit timestamp  |        | Transmit timestamp  |        |
>
> ICMP询问报文的作用：ICMP询问报文可以主动与目标主机交换一些必要的信息（例如地址掩码、时间），以达到通信前协商的效果。

类似地，重新运行PCA和PCB的wireshark，然后在PCA中：

```shell
$ ping 10.1.3.20
$ ping 10.1.4.10
```

自然是ping不通的，但能截获到报文。同样的保存报文信息。

> 题9：
>
> (1) 请比较这两种情况有何不同？
>
> 答：ping 10.1.3.20的时候，PCA会将报文交给S1来转发，S1会根据网络号和子网号从E1/0/23口转发出去；而在ping 10.1.4.10时，虽然PCA依然会将报文交给S1来转发，但是S1找不到10.1.4.*对应的网络应该从哪一个端口转发出去，因此会放弃转发，并给PCA返回ICMP差错报告报文（终点不可达）。
>
> (2) 截获了哪种ICMP差错报文？其类型和代码字段值是什么？此报文的ICMP协议部分又分为了几部分？其作用是什么？
>
> 答：截获ICMP差错报文的类型为目标不可达。协议的内容有：
>
> - Type：ICMP报文的类型，这里为3，即为目标不可达；
> - Code：ICMP代码，这里为0，表示网络不可达；
> - Checksum：ICMP校验和，用于做校验；
> - 数据部分：为封装后的不可达的IP报文。
