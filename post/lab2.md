# IP配置

登录服务器后，先该四个PC机的IP。一定要**严格按照下图中的设置，不然后果自负：**

![image-20211110233959102](https://raw.githubusercontent.com/zhtjtcz/MyImg/master/img/202111102340975.png)



同时，清空S1，S2，R1，R2上的所有内容。

先`reset saved-xxxx`，输入y

再`reboot`，先输入n，再输入y

# T1 抓包

打开 wireshark，然后点start即可进行抓包。随便挑一个包截个图，然后按报告里分析下就行。答案是固定的人。



# T2 Mac 地址

以下内容在PCA中做。

先在超级终端中看一下Mac地址表：

![image-20211110234259721](https://raw.githubusercontent.com/zhtjtcz/MyImg/master/img/202111102342772.png)



这里仿照我的报告截个图。

（1） 这里照抄

（2）不能，理由，分析过程照抄。

先清空一下Mac表：

![image-20211110234401556](https://raw.githubusercontent.com/zhtjtcz/MyImg/master/img/202111102344587.png)

然后display看一眼，真空了。



打开`cmd`，然后随便ping个什么东西。输入`ipconfig/all`，看一下PCA的Mac地址。

然后再display看一眼，发现里面多了一个Mac地址，就是PCA的，验证成功。



# T3 组网

首先点击桌面上新连线组网，依次点击PCA，PCB，S1按钮。

然后右键PCA 与 S1 选择端口进行连接，PCB同理。连接好后效果如图示：

![image-20211110165749708](https://raw.githubusercontent.com/zhtjtcz/MyImg/master/img/202111101657778.png)



记得点实验组网->提交。

然后设置S1：

![image-20211110234530795](https://raw.githubusercontent.com/zhtjtcz/MyImg/master/img/202111102345831.png)

按照表格，进行A ping B,A ping C操作，一个通，一个不通，截图记录。



# T4

照抄。



# T5

按下图配置：

![image-20211110234849722](https://raw.githubusercontent.com/zhtjtcz/MyImg/master/img/202111102348781.png)

然后，设置S1，S2，输入如下命令（**记得输入两遍，一个交换机一遍）**：

![image-20211110234939077](https://raw.githubusercontent.com/zhtjtcz/MyImg/master/img/202111102349161.png)

![image-20211110234947893](https://raw.githubusercontent.com/zhtjtcz/MyImg/master/img/202111102349965.png)



这时让A ping C，通，截图记录。同时在PCA上监听一下报文，看icmp类型。截个图。

填表。完全照抄，只需要将源Mac，目标Mac换成PCA 和 PCC 的Mac即可。每个人实验环境不一样，Mac也不一样，记得改。



# T6

display 看一下S1上的Mac表。此时会有很多，截图。

找到里面PCA 和PCC的部分，填进去。剩下的随便挑4个放进去即可。字段截图中都有的，慢慢抄。

# T9

在PCC 的R1终端中打命令：

![image-20211110235234456](https://raw.githubusercontent.com/zhtjtcz/MyImg/master/img/202111102352497.png)



 稍等一会，会不断输出东西。截两个图放进去，流程图照抄。



# T10

经过研究，我手头的报告作者**他们都没有打新的命令**，直接在T9基础上等了一小会，又截了两个图。我们仿照着做，然后流程图照抄。

链路层实验到此结束。