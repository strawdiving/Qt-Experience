## MAVLink v1.0详解——结构

本文针对 MAVLink v1.0版本，协议版本：3。

MAVLink是为微型飞行器MAV（Micro Air Vehicle）设计的（LGPL）开源的通讯协议。是无人飞行器和地面站（Ground Control Station ，GCS）之间，以及无人飞行器之间通讯常用的协议。APM、PIXHAWK飞控，Mission
Planner、QGroundControl地面站均使用了MAVLink协议进行通讯。

[MAVLink源码下载地址（现已更新至v2.0）][1]

[用户手册][2]

[1]: https://github.com/mavlink/qgroundcontrol
[2]: https://docs.qgroundcontrol.com/en/

### MAVLink源文件结构

![structure]()

- common文件夹：原始的MAVLink消息，包括各种消息的头文件

  common.h：定义MAVLink各个消息包中用到的枚举类型，各个消息包对应的CRC—EXTRA值、LENGTH值，包含（include）各个消息包的头文件

- autopilotmega，ASLUAV，pixhawk等文件夹：这些文件夹包含各个飞控自定义的MAVLink消息类型 

- checksum.h：计算校验码的代码

- mavlink_conversions.h：dcm，欧拉角，四元数之间的转换

- mavlink_helper.h：提供各种便捷函数：

  1) 将各个消息包补充完整并发送。将数据载荷加上消息帧的头部，如sysid和compid等，计算校验码，补充成为完整的mavlink消息包后发送；

  2) MAVLink消息包解析

- mavlink_types.h：定义用到的各种基本结构体（如mavlink_message_t）、枚举类型（如 mavlink_param_union_t）

### MAVLink消息包结构

　MAVLink传输时，以消息包作为基本单位，数据长度为8~263字节。消息数据包的结构如下：

![message]()



|         | 字节索引 | 内容         | 值         | 解释                                                         |
| ------- | -------- | ------------ | ---------- | ------------------------------------------------------------ |
| STX     | 0        | 包起始标志   | v1.0：0xFE | 包的起始标志，在v1.0中以“FE”作为起始标志（FE=254）           |
| LEN     | 1        | 有效载荷长度 | 0~255      | 有效载荷数据的字节长度（N）                                  |
| SEQ     | 2        | 包的序列号   | 0~255      | 每发送完一个消息，内容加1。用以检测丢包情况                  |
| SYS     | 3        | 系统ID编号   | 1~255      | 发送本条消息包的系统/飞行器的编号。用于消息接收端在同一网络中识别不同的MAV系统 |
| COMP    | 4        | 部件ID编号   | 0~255      | 发送本条消息包的部件的编号。用于消息接收端在同一系统中区分不同的组件，如IMU和飞控 |
| MSG     | 5        | 消息包ID编号 | 0~255      | 有效载荷中消息包的编号。该id定义了有效载荷内放的是什么类型的消息，以便消息接收端正确地解码消息包 |
| PAYLOAD | 6 - N+6  | 有效载荷数据 | 0~255字节  | 要用的数据放在有效载荷里。内容取决于message id。             |
| CKA     | N+7      | 校验和低字节 |            | 16位校验码；ITU X.25/SAE AS-4哈希校验（CRC-16-CCITT），不包括数据包起始位，从第一个字节到有效载荷（字节1 ..(N+6)）进行crc16计算（还要额外加上一个MAVLINK_CRC_EXTRA字节），得到一个16位校验码 |
| CKB     | N+8      | 校验和高字节 |            |                                                              |

注：校验（checksum）功能。加入MAVLINK_CRC_EXTRA，当两个通讯终端之间（飞行器和地面站，或飞行器和飞行器）使用不同版本的MAVLink协议时，双方计算得到的校验码会不同，则不同版本的MAVLink协议之间将无法通讯。MAVLINK_MESSAGE_CRCS中存储了每种消息包对应的MAVLINK_CRC_EXTRA。这个 MAVLINK_CRC_EXTRA在用python生成MAVLink代码时在common.h头文件中自动生成。

MAVLink数据包的结构在mavlink_types.h中用mavlink_message_t结构体定义：

```c++
typedef struct __mavlink_message {  
  uint16_t checksum; /// CRC
  uint8_t magic;   /// STX
  uint8_t len;     /// LEN
  uint8_t seq;     /// SEQ
  uint8_t sysid;   /// SYS
  uint8_t compid;  /// COMP
  uint8_t msgid;   /// MSG
  uint64_t payload64[(MAVLINK_MAX_PAYLOAD_LEN+MAVLINK_NUM_CHECKSUM_BYTES+7)/8]; 
 } mavlink_message_t; 
```

### MAVLINK Common Message Set

MAVLink通用消息集可以在《MAVLLINK Common Message set specifications》文档中查看。这些消息定义了通用消息集，这是大多数地面控制站和自动驾驶仪实现的参考消息集，头文件包含在common文件夹中。

分为两部分：MAVLink Type Enumerations(MAVLink类型枚举 )和MAVLink Messages（MAVLink消息包）。

#### MAVLink Type Enumerations

MAVLink Type Enumerations在common.h文件中定义。如下，枚举变量定义了飞行器的类型MAV_AUTOPILOT。

![enum]()

```c++
typedef enum MAV_AUTOPILOT
 2 {
 3     MAV_AUTOPILOT_GENERIC=0, /* Generic autopilot, full support for everything | */
 4     MAV_AUTOPILOT_RESERVED=1, /* Reserved for future use. | */
 5     MAV_AUTOPILOT_SLUGS=2, /* SLUGS autopilot, http://slugsuav.soe.ucsc.edu | */
 6     MAV_AUTOPILOT_ARDUPILOTMEGA=3, /* ArduPilotMega / ArduCopter, http://diydrones.com | */
 7     MAV_AUTOPILOT_OPENPILOT=4, /* OpenPilot, http://openpilot.org | */
 8     MAV_AUTOPILOT_GENERIC_WAYPOINTS_ONLY=5, /* Generic autopilot only supporting simple waypoints | */
 9     MAV_AUTOPILOT_GENERIC_WAYPOINTS_AND_SIMPLE_NAVIGATION_ONLY=6, /* Generic autopilot supporting waypoints and other simple navigation commands | */
10     MAV_AUTOPILOT_GENERIC_MISSION_FULL=7, /* Generic autopilot supporting the full mission command set | */
11     MAV_AUTOPILOT_INVALID=8, /* No valid autopilot, e.g. a GCS or other MAVLink component | */
12     MAV_AUTOPILOT_PPZ=9, /* PPZ UAV - http://nongnu.org/paparazzi | */
13     MAV_AUTOPILOT_UDB=10, /* UAV Dev Board | */
14     MAV_AUTOPILOT_FP=11, /* FlexiPilot | */
15     MAV_AUTOPILOT_PX4=12, /* PX4 Autopilot - http://pixhawk.ethz.ch/px4/ | */
16     MAV_AUTOPILOT_SMACCMPILOT=13, /* SMACCMPilot - http://smaccmpilot.org | */
17     MAV_AUTOPILOT_AUTOQUAD=14, /* AutoQuad -- http://autoquad.org | */
18     MAV_AUTOPILOT_ARMAZILA=15, /* Armazila -- http://armazila.com | */
19     MAV_AUTOPILOT_AEROB=16, /* Aerob -- http://aerob.ru | */
20     MAV_AUTOPILOT_ASLUAV=17, /* ASLUAV autopilot -- http://www.asl.ethz.ch | */
21     MAV_AUTOPILOT_ENUM_END=18, /*  | */
22 } MAV_AUTOPILOT;

```

#### MAVLink Messages

MAVLink Messages在common文件夹内每个消息包的头文件中定义。在文档中msgid以蓝色的“#”加数字的方式来表示，如心跳包的“#0”，在心跳包的头文件mavlink_msg_heartbeat.h中，MAVLINK_MSG_ID_HEARTBEAT对应心跳包的message ID，为0。

```c++
#define MAVLINK_MSG_ID_HEARTBEAT 0
```

心跳包的内容存放在payload数据载荷中。以心跳包为例，

![heartbeat]()

```c++
typedef struct __mavlink_heartbeat_t
{
 uint32_t custom_mode; /*< A bitfield for use for autopilot-specific flags.*/
 uint8_t type; /*< Type of the MAV (quadrotor, helicopter, etc., up to 15 types, defined in MAV_TYPE ENUM)*/
 uint8_t autopilot; /*< Autopilot type / class. defined in MAV_AUTOPILOT ENUM*/
 uint8_t base_mode; /*< System mode bitfield, see MAV_MODE_FLAG ENUM in mavlink/include/mavlink_types.h*/
 uint8_t system_status; /*< System status flag, see MAV_STATE ENUM*/
 uint8_t mavlink_version; /*< MAVLink version, not writable by user, gets added by protocol because of magic data type: uint8_t_mavlink_version*/
} mavlink_heartbeat_t;
```

心跳包一般用来表明发出该消息的设备是否活跃（一般以1Hz发送），消息接收端会根据是否及时收到了心跳包来判断是否和消息发送端失去了联系。

心跳包由6个数据成员组成，占用9个字节。

1. type：飞行器类型，表示了当前发消息的是什么飞行器，如四旋翼，直升机等。type的取值对应枚举类型MAV_TYPE（如四旋翼，对应数值2）
2. autopilot：飞控类型，如apm，Pixhawk等，发送心跳包的飞行器的飞控类型。autopilot的取枚举类型MAV_AUTOPILOT
3. base mode（基本模式）：飞控现在所处的飞行模式，这个参数要看各个飞控自己的定义方式，会有不同的组合、计算方式
4. custom mode（用户模式）：飞控现在所处的飞行模式，这个参数要看各个飞控自己的定义方式，会有不同的组合、计算方式
5. system status：系统状态，见MAV_STATE枚举变量
6. mavlink version：消息发送端的MAVLink版本

　其余的消息也是类似的结构，基本消息的定义可以查看官方网页的说明（具体说明以各个飞控为准），也可查看各个消息包头文件的定义。