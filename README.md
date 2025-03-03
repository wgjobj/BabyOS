# BabyOS

![logo](https://github.com/notrynohigh/BabyOS/raw/master/doc/2.png)

​        为小型项目而生，一个如孩童般需要集体喂养的弱操作系统。为什么称它为弱操作系统，因为相比已有的嵌入式操作系统，这个显得比较弱鸡。这里姑且称之为操作系统吧，其本质是一份代码集中营。

------

# 适用项目

> ​        BOS: 当前项目是否需要使用像FreeRTOS等操作系统？
>
> ​                                                                                                                                        U: 需要！
>
> ​        BOS：不好意思，我可能不适合你。
>
> ​                                                                                                                    U: 开玩笑的，不需要用操作系统
>
> ​        BOS: 那您可以尝试使用我哦！

​             说一说写这么个东西的原因，大概就知道这份BOS有哪些功能。

​             ................

​        某天、一位猿说，现在对项目就只有两个要求，功耗和开发时间。99.874%产品是电池供电，功耗是重点考虑对象。其次产品的功能都不复杂，项目之间也有很多重复的部分，是否有套代码能够减少重复的工作，加速产品demo的开发。

​             ..............:)

​        **功耗**，为减小功耗，对于外设的操作原则是，唤醒外设，操作完成后进入睡眠。这样的操作形式和文件的操作很类似，文件的操作步骤是打开到编辑到关闭。决定将外设的操作看作是对文件的操作进行。每个外设打开后返回一个描述符，后续代码中对外设的操作都是基于这个描述符进行。关闭外设后回收描述符。那么在外设的驱动中，打开和关闭操作可以执行对设备的唤醒和睡眠。利用描述符来操作外设还有一个好处是，当更换外设后，只需更换驱动接口，业务部分的代码不需要变动。

​        **快速开发**，小型项目的开发中，有较多使用率高的功能模块，例如：UTC、错误管理、电池电量、存储数据、上位机通信、固件升级等等。将这些功能都做成独立的模块，不依赖于硬件。再配合一份配置文件，每次根据功能需求选择当前项目使用的功能模块。简单来说就是搭积木。

​       BOS  0.0.1版本驱动只加入模拟串口和华邦flash。存在的功能模块如下表所示。

| 序号 | 功能模块   | 说明                                                   |
| ---- | ---------- | ------------------------------------------------------ |
| 1    | 电量检测   | 支持设置阈值用于判断电量状态，正常或者低电量           |
| 2    | 校验计算   | 支持CRC32，累加和，异或和校验                          |
| 3    | 错误管理   | 支持两种等级的错误管理                                 |
| 4    | 事件管理   | 支持事件触发某个操作                                   |
| 5    | MODBUS协议 | 支持MODBUS协议RTU传输组包和解析                        |
| 6    | 私有协议   | 协议格式：[header\|id\|len\|cmd\|data\|crc]            |
| 7    | 固件升级   | 支持固件升级（私有协议基础上完成新固件的接收与存储）   |
| 8    | 数据存取   | 支持三种数据存储方式（定时存储、单次存储、连续存储）   |
| 9    | 发送管理   | 管理发送数据的秩序，防止未发送完成时又有新数据请求发送 |
| 10   | UTC        | 支持UTC的转换，UTC起始时间为2000-1-1 0：0：0           |

  **电量检测** 需要提供ADC转换的接口，设置阈值，判断当前电池的状态

  **校验计算** 提供几种校验方式，在需要时使用

  **错误管理** 提供两种等级的错误管理，低等级错误是只反馈一次异常，高等级错误是需要手动清除管理单元的异常信息，否则间隔时间到后就反馈异常。系统异常时，提交异常至管理单元并指定相同错误的最小间隔，防止同一个异常短时间多次被提交

  **事件管理** 支持注册一个功能函数，在需要时去触发执行

  **MODBUS** 支持RTU协议的组包发送和数据解析

  **私有协议** 支持私有协议，格式:  头（1字节）|ID（4字节）|Len（1字节）|Cmd（1字节）|数据|CRC（1字节）

  **固件升级** 支持基于私有协议接收和存储新固件

  **数据存取** 支持三种数据存取，定时存储（例如每小时存储一次、每天存储一次这样的场景），单次存储（加校验存储数据，例如存储配置数据），连续存储（flash上连续存储固定大小的数据，支持获取已存储数据的数量，例如存储异常日志以及获取已存储日志的数量）

  **发送管理** 支持管理数据的发送，保证数据发送完成后新的发送请求才能成功

  **UTC**  支持UTC和年月日时分秒的转换，UTC起始时间是2000-1-1 0：0：0

​        部分功能模块需要使用具体外设，创建功能模块实例时需要指定具体设备。例如数据存取，创建功能模块时指定其对应的flash设备。

​       如果某个产品的功能是每小时采集一次数据存储并上报，支持上位机取历史数据。物联网领域中有许多这样功能简单的终端设备，完成这个功能只需要添加2、6、8、9、10。如果需要监测设备异常状态则加入3，需要固件升级加入7，电池供电加入1。

​       

# 使用方法

##   1、添加文件

​        bos/core/src目录文件全部添加至工程

​        bos/driver/src选择需要的添加至工程

​        bos/hal/ 添加至工程，根据具体平台进行修改

##   2、选择功能模块

​        对于b_config.h进行配置，根据自己的需要选择功能模块。

![opt](https://github.com/notrynohigh/BabyOS/raw/master/doc/1.png)

##   3、列出需要使用的设备

​           找到b_device_list.h，在里面添加使用的外设。例如当前项目只需要使用flash和模拟串口，那么添加如下代码：    

```c
//           设备        驱动接口      描述
B_DEVICE_REG(W25QXX, bW25X_Driver, "flash")
B_DEVICE_REG(SUART, SUART_Driver, "suart")
```

##   4、使用设备

​    

```c
bInit();    //初始化，外设的初始化会在此处调用
//下面举例使用设备
int fd;
fd = bOpen(SUART, BCORE_FLAG_RW);    //其中SUART是在b_device_list.h中添加的设备
if(fd >= 0)
{
     bWrite(fd, (uint8_t *)"hello world\r\n", strlen("hello world\r\n")); //发送字符串
     bClose(fd);
}
```

   如果一个设备被打开正在使用，那么无法再次被打开。

##   5、使用功能模块

```c
int sdb_no;
sdb_no = bSDB_Regist(0, 1, W25QXX);//创建B类数据存储实例，指定设备W25QXX，获的功能模块实例IDsdb_no
//sdb_no大于等于0则有效
int bSDB_Write(int no, uint8_t *pbuf);
int bSDB_Read(int no, uint8_t *pbuf);
//读写函数传入实例ID sdb_no
```

# 喂养

​       之所以称其为需要喂养的弱操作系统，因为驱动和功能模块都需要一点点加入进去，加入的越多，后续开发起来就越快。



