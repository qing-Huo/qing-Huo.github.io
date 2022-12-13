## 数据链路层

基本概念

```
结点：
    主机，路由器
链路：
    网络中两个节点之间的 物理通道，链路的传输介质有双绞线，光纤和微波
    分为有线链路，无线链路
数据链路：
    网络中两个节点之间的 逻辑通道，把实现控制数据传输 协议 的硬件和
    软件加到链路上就构成数据链路
帧：
    链路层的协议数据单元，封装网络层数据报
数据链路层负责通过一条链路从另一个节点向另一个物理链路直接相连的
相邻节点传送数据报
```

数据链路层功能概述：

数据链路层在物理层提供服务的基础上```向网络层提供服务```，其最基本的服务是将源自网络层来的数据```可靠```地差u念书到相邻节点的目标机网络层。其主要作用是```加强物理层传输原始比特流的功能```，将物理层提供的可能出错的物理连接改造成```逻辑上无差错的数据链路```，使之对网络层表现为一条无差错的链路。

- 为网络层提供服务
    * 无确认无连接服务
    * 有确认无连接服务
    * 有确认面向连接服务(有连接一定有确认)
- 链路管理
    * 即建立的连接，维持，释放(用于面向连接的服务)
- 组帧
- 流量控制(限制发送方)
- 差错控制(帧错/位错)

#### 封装成帧和透明传输

```封装成帧```就是在一段数据的前后部分添加首部和尾部，这样就构成了一个帧。

接收端在受到物理层上较的比特流后，就能根据首部和尾部的标记，从受到的比特流中识别帧的开始和结束

首部和尾部包含许多的控制信息，他们的一个重要作用：```帧定界(确定帧的界限)```

```帧同步```：接收方应当能从接收到的二进制比特流中区分出帧的起始和终止

- 组帧的4种方法：
    * 字符计数法
    * 字符（节）填充法
    * 零比特填充法
    * 违规编码法
- 透明传输

```
透明传输 是指不管所传数据是什么样的比特组合，都应当能够在链路上传送。
因此，链路层就 “看不见” 有什么妨碍数据传输的东西
当所传数据中的比特组合恰巧与某一个控制信息完全一样时，就必须采取适当的
措施，使收方不会将这样的数据误认为是某种信息。这样才能保证数据链路层
的传输是透明的
```
- 字符计数法
    * 帧首部使用一个计数字段(第一个字节，8位)来表明帧内字符数
    * 缺点是 鸡蛋装载一个篮子里
- 字符填充法

```
数据首部有SOH,尾部有EOT
SOH(Start of Header)
EOT(End of Transmission)
当传送的帧是由文本文件组成时(文本文件的字符都是从键盘输入，都是ASCII码).不管从键盘上输入什么字符都可以放在帧里传过去，即```透明传输```
当传送的帧由非ASCII码的文本文件组成(二进制代码的程序或图像),就要采用```字符填充方法实现透明传输```
```

- 零比特填充法
    
```
在发送端，扫描整个信息字段，只要连续5个1,就立即填入1个0
在接收端受到一个帧时，先找到标志字段确认边界，再用硬件对比比特流进行
扫面，发现连续5个1时，就把后面的0删除
保证了透明传输：
    在传送的比特流中可以传送任意比特组合，而不会引起对帧边界的判断错误
note: 零比特填充法，数据帧的首尾依旧有标志位
```

- 违规编码法

```
由于曼彻斯特编码中会对一个比特分为两半进行表示(低-高，高-低)
所以可以用 高-高，低-低 来定界帧的起始和终止
由于字节计数法中Count字段的脆弱性(其值若有差错将导致严重后果)及字符填充
实现上的复杂性和不兼容性，目前普遍使用的帧同步法是
    比特填充和违规编码法
```
## 差错控制

- 差错从何而来？

```
概括来说，传输中的差错都是由于噪声引起的
全局性：
    由于线路本身电气特性所产生的 随机噪声(热噪声)，是信道固有的，随机存在
    解决办法：提高信噪比来减少或避免干扰(对传感器下手)
局部性：
    外界特性的短暂原因所造成的 冲击噪声，是产生差错的主要原因
    解决办法：通常利用编码技术来解决
```
- 数据链路的编码

```
数据链路中差错控制是针对 比特错
解决方案有
    检错编码(奇偶校验码， 循环冗余码CRC)
    纠错编码(海明码)
数据链路层编码和物理层的数据编码与调制 不同，
物理层编码针对的是 单个比特，解决传输过程中比特的同步等问题
比如曼彻斯特编码。
而数据链路层的编码针对的是 一组比特，它通过冗余码的技术实现了
一组二进制比特串在传输过程中是否出现了差错
```
- 冗余编码

```
在数据发送之前，先按某种关系 附加 上一定的冗余位，构成一个符合某一个
规则的码字后在发送
当要发送的有效数据变化时，相应的冗余位也随之变化，使码字遵从不便的规则。
接受端根据受到的码字是否仍符合原规则，从而判断是否出错
```






