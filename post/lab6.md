# 传输层实验

> 作者：周勤、黄泽桓

**对于路由器、交换机，实验前都需要先清空设置，下文将不再说明**

**本次实验大部分在Linux上完成，小部分在Windows上完成**

**本次实验主要分为四个任务：**

1. TCP协议基本分析
2. 滑动窗口机制和窗口侦查机制分析
3. 慢启动、拥塞避免及拥塞处理实验
4. UDP实验（Winxp）

## 实验须知

### Linux配置IP地址的方法

鼠标**右键**点击托盘区网络图标，并点击Edit Connections：

![image-20211121213256906](img/image-20211121213256906.png)

点击Add按钮，在IPv4设置中设置IP地址、网关等，然后依次点击Apply和OK：

![image-20211121213352418](img/image-20211121213352418.png)

需要使用对应的配置时，鼠标**左键**点击托盘区网络图标，再点击Disconnect断开原有连接，然后选择对应的配置进行连接：

![image-20211121213416427](img/image-20211121213416427.png)

### WireShark预设置

**注意**，本次实验需要获取到 sequence number 的绝对值，因此需要将 WireShark 默认获取相对值的设置取消掉，具体过程如下：

点击 WireShark 的 Edit，后点击打开preferences，左侧导航栏展开Protocols，找到TCP，若Relative sequence number被勾选了的话，要去掉勾选。然后点击apply，再OK。

### 虚拟机联网

联网主要是为了将保存的报文文件或其它信息文件传输到本机上，这样方便做完实验后再分析数据。如果希望边做实验边分析数据和截图的话，可以不需要了解本部分。

需要注意的是，建议做完**所有Linux实验**再联网将实验数据一起发回，这样不用重复连线组网，因为本次Linux上进行的实验组网都一样。

进入PCA的连线组网程序，点击主机联网：

![image-20211121231602269](img/image-20211121231602269.png)

接下来需要使用DHCP获取主机地址，在托盘区点击这个图标，选择Auto_DHCP即可（若没有这个配置，则可以自行配置DHCP）：

![image-20211121231619239](img/image-20211121231619239.png)

配置完之后，可以使用`ifconfig`命令来进行验证，还可使用`ping 10.111.1.29`命令来验证与FTP服务器的连通性（对于PCB这样的多网卡机器，可以使用`ifconfig eth1 down`命令来关闭网卡1，防止网卡1的干扰）：

![image-20211121231700507](img/image-20211121231700507.png)

若使用FTP传输文件，则这时就可以点击托盘区的gFTP图标，打开gFTP程序。

输入FTP服务器的地址，协议选择FTP，点击电脑图标进行连接后，就可以上传到自己的文件夹了：

![image-20211121231815188](img/image-20211121231815188.png)

另外，还可以通过**发送邮件**（我就是这样干的）等方式将数据发回本机，因为都连接外网了，所以可以乱搞，但是由于配置原因，所以这样可能稍微卡一些。联网记得访问 gw.buaa.edu.cn 登录，这样才能真的上网，另外默认是谷歌搜索引擎，所以建议直接搜网址，或者进百度主页搜索。

## 实验1：TCP协议基本分析

### 步骤1 配置路由器和交换机

连接到服务器后，登录Linux虚拟机，用户名root，密码network，在PCA的桌面找到并打开连线组网软件。

由于是Java写的程序，所以跨平台做的比较好，如下图连线组网并且提交即可（连线组网可能比较慢，请耐心等待，**如果遇到白屏和打不开等问题，大概率是后端炸了**）：

![image-20211121213957641](img/image-20211121213957641.png)

打开PCC配置R1，打开终端输入minicom命令，进入超级终端。

**需要注意的是，打开了之后，千万不要关闭monicom终端，不过可以最小化，其他程序也建议不要关**

根据如下的命令配置R1的IP地址即可（如果GigabitEthernet用不了，可以用Ethernet）：

```sh
sys
interface GigabitEthernet 0/0
port link-mode route
ip address 192.168.1.1 255.255.255.0
interface GigabitEthernet 0/1
port link-mode route
ip address 192.168.2.1 255.255.255.0
```

打开PCA配置S1，同样打开终端输入minicom命令，进入超级终端。

**需要注意的是，打开了之后，千万不要关闭monicom终端，不过可以最小化，其他程序也建议不要关**

进入超级终端后，需要配置IP地址和两个VLAN，配置命令如下：

```sh
sys
vlan 2 to 3
interface Ethernet 1/0/1
port access vlan 2
interface Ethernet 1/0/2
port access vlan 2
interface Ethernet 1/0/23
port access vlan 3
interface Ethernet 1/0/24
port access vlan 3
```

在虚拟机中，PCA和PCB的IP地址都已经配好了，只需要在**托盘区**点击网络图标进行选择并连接即可。PCA需要选择TCP_Sender，PCB需要选择TCP_Receiver：

![image-20211121233316779](img/image-20211121233316779.png)

如果机器没有预先配好，则需要按照**Pre**中的步骤进行配置，实验组网图如下：

![image-20211122010459819](img/image-20211122010459819.png)

另外，PCB是多网卡机器，**若自己配置PCB的IP地址**，则建议关闭PCB的网卡1，命令为：

```sh
ifconfig eth1 down
```

设置IP后若发现点击网络图标无法显示可连接的网络，则右键点击，勾选 Enable Networking。

都连接好之后可以使用ping命令验证一下连接性，还可以用`ifconfig`命令验证IP地址的配置。

### 步骤2 启动Wireshark和TcpTest

在PCB打开Wireshark，监听Eth0口，过滤器填写tcp，点击Apply：

![image-20211121222036888](img/image-20211121222036888.png)

在PCB的桌面打开TcpTest，点击作为接收端，如下图修改配置，然后点击接收：

![image-20211127155122830](img/image-20211127155122830.png)

在PCA中，同样打开并设置Wireshark，打开Wireshark之后，需要在桌面上先运行`tcplog_init`脚本：

![image-20211127155011383](img/image-20211127155011383.png)

### 步骤3 报文截图与分析

等待文件发送完毕后，停止PCA和PCB的抓包，保存或截图PCA和PCB的抓包记录。（关于FTP服务器的使用见下文）

保存或截图之后，可以对TCP三次握手，四次挥手，以及中间过程进行分析，并填写实验报告。

### 步骤4 分析Linux内核数据

在Wireshark停止抓包之后，在PCA的桌面上点击并运行`tcplog_read`脚本：

![image-20211121222732239](img/image-20211121222732239.png)

然后在PCA的`/root`目录下，进入`TCPLog`文件夹，保存或截图如下文件，该文件记录了Linux内核Tcp协议栈的参数。（关于FTP服务器的使用见下文）

保存或截图之后，进行分析并填写实验报告即可：

![image-20211121223045546](img/image-20211121223045546.png)

## 实验2：滑动窗口机制和窗口侦查机制分析

基本操作与[实验1](#实验1：TCP协议基本分析)步骤一致，区别在于TcpTest的设置，需要设置休眠时间，如下图所示。

![image-20211126223857831](img/image-20211126223857831.png)

![image-20211126205828206](img/image-20211126205828206.png)

这样获得的数据，与实验1的区别在于，实验2生成的tcpsndwnddata.txt文件中，rcv_wnd接收方窗口会降低，如下图所示。

![image-20211127155759295](img/image-20211127155759295.png)

## 实验3：慢启动、拥塞避免及拥塞处理实验

即10Mbps转发速率实验和100Kbps转发速率实验，转发速率实验组网和前面相同。

这部分内容，实验指导书讲得比较详细了，这里放几张实验指导书的截图：

继续在PCA和PCB上进行实验即可，**如果之前关闭了minicom，则到这一步时可能会出现minicom打不开的问题**。

注意：

1. 如果原来进入的是GigabitEthernet接口，则下面的实验也需要进入GigabitEthernet
2. 若遇到TcpTest报错“I/O错误”，重启TcpTest即可

![image-20211121211914740](img/image-20211121211914740.png)

![image-20211121211933354](img/image-20211121211933354.png)



![image-20211121211947847](img/image-20211121211947847.png)

### UDP实验

这个实验需要在WindowsXP上进行，所以需要预约WindowsXP：

![image-20211121212047491](img/image-20211121212047491.png)

![image-20211121212056662](img/image-20211121212056662.png)

![image-20211121212107034](img/image-20211121212107034.png)

本实验截图较少，也比较简单做，我的截图如下：

![image-20211122003614091](img/image-20211122003614091.png)