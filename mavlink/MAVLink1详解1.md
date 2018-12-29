## MAVLink v1.0详解——源码

本文针对 MAVLink v1.0版本，协议版本：3。

MAVLink是为微型飞行器MAV（Micro Air Vehicle）设计的（LGPL）开源的通讯协议。是无人飞行器和地面站（Ground Control Station ，GCS）之间，以及无人飞行器之间通讯常用的协议。APM、PIXHAWK飞控，Mission
Planner、QGroundControl地面站均使用了MAVLink协议进行通讯。​  

### MAVLink消息解码

当使用MavLink协议提供的方法封装消息包时，会根据所使用的MSG获取到该类别MSG消息的LEN信息，同时软件（地面站或飞行控制软件）会根据自身状态信息填写SYS、COMP。信息填写完毕生成数据包时，封装方法会自动添加STX，并在上一次发送消息包所使用的SEQ上加1作为本次发送的SEQ写入，当SEQ超过255时，会回到0并重新开始计数。CKA、CKB会在PAYLOAD信息写入后、封装完成之前，根据CRC[[Fe1\]](file:///C:\Users\Administrator\Documents\%E5%9C%B0%E9%9D%A2%E7%AB%99MAVLink%E5%8D%8F%E8%AE%AE%E9%80%9A%E4%BF%A1%E5%88%86%E6%9E%90.docx#_msocom_1) （CyclicRedundancy Check）循环冗余校验码算法计算得出并自动写入包内。

**mavlink_parse_char** —— 便捷函数，用于完整地解析MAVLink消息包。一次解析一个字节，如果成功解码一个消息包，则返回一个完整的消息包以及解码状态。

函数的返回值可能为：

- 0——MAVLINK_FRAMING_INCOMPLETE，没有消息可以被解码，返回0
- 1——MAVLINK_FRAMING_OK，成功解码且CRC校验成功，返回1
- 2——MAVLINK_FRAMING_BAD_CRC，CRC校验失败，返回2

```c
1.	MAVLINK_HELPER uint8_t mavlink_parse_char(uint8_t chan, uint8_t c, mavlink_message_t* r_message, mavlink_status_t* r_mavlink_status)
2.	{
3.	    uint8_t msg_received = mavlink_frame_char(chan, c, r_message, r_mavlink_status);
4.	    if (msg_received == MAVLINK_FRAMING_BAD_CRC) {
5.		    // bad CRC. 解析错误
6.		    mavlink_message_t* rxmsg = mavlink_get_channel_buffer(chan);
7.		    mavlink_status_t* status = mavlink_get_channel_status(chan);
8.		    status->parse_error++;
9.		    status->msg_received = MAVLINK_FRAMING_INCOMPLETE;
10.		    status->parse_state = MAVLINK_PARSE_STATE_IDLE;
11.		    if (c == MAVLINK_STX)
12.		    {
13.			    status->parse_state = MAVLINK_PARSE_STATE_GOT_STX;
14.			    rxmsg->len = 0;
15.			    mavlink_start_checksum(rxmsg);
16.		    }
17.		    return 0;
18.	    }
19.	    return msg_received;
20.	}
```

参数：

- chan —— 当前通道的ID编号，可以使用该函数解析不同的通道。这里的通道不是串口这样的物理信息通道，而是通信流的逻辑分区。COMM_NB是MCU（例如ARM7）上的通道数的限制，而COMM_NB_HIGH是Linux / Windows中通道数量的限制
- c —— 要解析的字符
- r_message —— 如果没有消息可以解码，为NULL，有消息解码，返回该消息数据
- r_mavlink_status —— 如果消息被解码，则等于通道的状态

应用示例：

```c
1.	#include<mavlink.h>
2.	mavlink_message_t msg;
3.	int chan=0;
4.	while(serial.bytesAvailable>0)
5.	{
6.		uint_8 byte = serial.getNextByte();
7.		if(mavlink_parse_char(chan,byte,&msg))
8.	    {
9.	      printf(Received Message from component %d of system %d”,msg.compid,msg.sysid);
10.	    }
11.	 }
```

程序流程：

mavlink_parse_char调用mavlink_frame_char，mavlink_frame_char调用mavlink_frame_char_buffer。

```c
1.	MAVLINK_HELPER uint8_t mavlink_frame_char(uint8_t chan, uint8_t c, mavlink_message_t* r_message, mavlink_status_t* r_mavlink_status)
2.	{
3.		return mavlink_frame_char_buffer(mavlink_get_channel_buffer(chan),
4.						 mavlink_get_channel_status(chan),
5.						 c,
6.						 r_message,
7.						 r_mavlink_status);
8.	}
```

将收到的字符一个个进行解码，一一对应message的各数据位，检验收到的校验码是否正确，有效载荷长度小于最大长度，且和该消息包长度一致。

若一切顺利，返回1。可得到解码后的消息，先被parse进一个内部buffer（每个通道对应一个），存储在r_message（mavlink_message_t类型）里，更新通道的状态存储到r_mavlink_status（mavlink_status_t类型）里。

若未有消息成功解码，r_message为空。

```c
	MAVLINK_HELPER uint8_t mavlink_frame_char_buffer(mavlink_message_t* rxmsg, mavlink_status_t* status, uint8_t c, mavlink_message_t* r_message, mavlink_status_t* r_mavlink_status)
2.	{
3.	        /*
4.		  默认的message crc 函数. You can override this per-system to
5.		  put this data in a different memory segment
6.		*/
7.	#if MAVLINK_CRC_EXTRA
8.	#ifndef MAVLINK_MESSAGE_CRC
9.		static const uint8_t mavlink_message_crcs[256] = MAVLINK_MESSAGE_CRCS;
10.	#define MAVLINK_MESSAGE_CRC(msgid) mavlink_message_crcs[msgid]
11.	#endif
12.	#endif
13.		/*启用此选项以检查每条消息的长度.
14.	   只有通道中只包含头文件中列出的消息类型时才使用。
15.		*/
16.	#ifdef MAVLINK_CHECK_MESSAGE_LENGTH
17.	#ifndef MAVLINK_MESSAGE_LENGTH
18.		static const uint8_t mavlink_message_lengths[256] = MAVLINK_MESSAGE_LENGTHS;
19.	#define MAVLINK_MESSAGE_LENGTH(msgid) mavlink_message_lengths[msgid]
20.	#endif
21.	#endif
22.		int bufferIndex = 0;
23.		status->msg_received = MAVLINK_FRAMING_INCOMPLETE;
24.		switch (status->parse_state)
25.		{
26.		case MAVLINK_PARSE_STATE_UNINIT:
27.		case MAVLINK_PARSE_STATE_IDLE:
28.			if (c == MAVLINK_STX)
29.			{
30.				status->parse_state = MAVLINK_PARSE_STATE_GOT_STX;
31.				rxmsg->len = 0;
32.				rxmsg->magic = c;
33.				mavlink_start_checksum(rxmsg);
34.			}
35.			break;
36.		case MAVLINK_PARSE_STATE_GOT_STX:
37.				if (status->msg_received 
38.	/* Support shorter buffers than the
39.	   default maximum packet size */
40.	#if (MAVLINK_MAX_PAYLOAD_LEN < 255)	|| c > MAVLINK_MAX_PAYLOAD_LEN
41.	#endif
42.					)
43.			{
44.				status->buffer_overrun++;
45.				status->parse_error++;
46.				status->msg_received = 0;
47.				status->parse_state = MAVLINK_PARSE_STATE_IDLE;
48.			}
49.			else
50.			{
51.				// NOT counting STX, LENGTH, SEQ, SYSID, COMPID, MSGID, CRC1 and CRC2
52.				rxmsg->len = c;
53.				status->packet_idx = 0;
54.				mavlink_update_checksum(rxmsg, c);
55.				status->parse_state = MAVLINK_PARSE_STATE_GOT_LENGTH;
56.			}
57.			break;
58.		case MAVLINK_PARSE_STATE_GOT_LENGTH:
59.			rxmsg->seq = c;
60.			mavlink_update_checksum(rxmsg, c);
61.			status->parse_state = MAVLINK_PARSE_STATE_GOT_SEQ;
62.			break;
63.		case MAVLINK_PARSE_STATE_GOT_SEQ:
64.			rxmsg->sysid = c;
65.			mavlink_update_checksum(rxmsg, c);
66.			status->parse_state = MAVLINK_PARSE_STATE_GOT_SYSID;
67.			break;
68.		case MAVLINK_PARSE_STATE_GOT_SYSID:
69.			rxmsg->compid = c;
70.			mavlink_update_checksum(rxmsg, c);
71.			status->parse_state = MAVLINK_PARSE_STATE_GOT_COMPID;
72.			break;
73.		case MAVLINK_PARSE_STATE_GOT_COMPID:
74.	#ifdef MAVLINK_CHECK_MESSAGE_LENGTH
75.		        if (rxmsg->len != MAVLINK_MESSAGE_LENGTH(c))
76.			{
77.				status->parse_error++;
78.				status->parse_state = MAVLINK_PARSE_STATE_IDLE;
79.				break;
80.		    }
81.	#endif
82.			rxmsg->msgid = c;
83.			mavlink_update_checksum(rxmsg, c);
84.			if (rxmsg->len == 0)
85.			{
86.				status->parse_state = MAVLINK_PARSE_STATE_GOT_PAYLOAD;
87.			}
88.			else
89.			{
90.				status->parse_state = MAVLINK_PARSE_STATE_GOT_MSGID;
91.			}
92.			break;
93.		case MAVLINK_PARSE_STATE_GOT_MSGID:
94.			_MAV_PAYLOAD_NON_CONST(rxmsg)[status->packet_idx++] = (char)c;
95.			mavlink_update_checksum(rxmsg, c);
96.			if (status->packet_idx == rxmsg->len)
97.			{
98.				status->parse_state = MAVLINK_PARSE_STATE_GOT_PAYLOAD;
99.			}
100.			break;
101.		case MAVLINK_PARSE_STATE_GOT_PAYLOAD:
102.	#if MAVLINK_CRC_EXTRA
103.			mavlink_update_checksum(rxmsg, MAVLINK_MESSAGE_CRC(rxmsg->msgid));
104.	#endif
105.			if (c != (rxmsg->checksum & 0xFF)) {
106.				status->parse_state = MAVLINK_PARSE_STATE_GOT_BAD_CRC1;
107.			} else {
108.				status->parse_state = MAVLINK_PARSE_STATE_GOT_CRC1;
109.			}                _MAV_PAYLOAD_NON_CONST(rxmsg)[status->packet_idx] = (char)c;
110.			break;
111.		case MAVLINK_PARSE_STATE_GOT_CRC1:
112.		case MAVLINK_PARSE_STATE_GOT_BAD_CRC1:
113.			if (status->parse_state == MAVLINK_PARSE_STATE_GOT_BAD_CRC1 || c != (rxmsg->checksum >> 8)) {
114.				// got a bad CRC message
115.				status->msg_received = MAVLINK_FRAMING_BAD_CRC;
116.			} else {
117.				// Successfully got message
118.				status->msg_received = MAVLINK_FRAMING_OK;
119.	                }
120.	                status->parse_state = MAVLINK_PARSE_STATE_IDLE;                _MAV_PAYLOAD_NON_CONST(rxmsg)[status->packet_idx+1] = (char)c;
121.	                memcpy(r_message, rxmsg, sizeof(mavlink_message_t));
122.			break;
123.		}
124.		bufferIndex++;
125.		// If a message has been sucessfully decoded, check index
126.		if (status->msg_received == MAVLINK_FRAMING_OK)
127.		{
128.			//while(status->current_seq != rxmsg->seq)
129.			//{
130.			//	status->packet_rx_drop_count++;
131.			//   status->current_seq++;
132.			//}
133.			status->current_rx_seq = rxmsg->seq;
134.			// 初始条件: 如果还没有收到包, 则丢包计数是未定义undefined。
135.			if (status->packet_rx_success_count == 0)      status->packet_rx_drop_count = 0;
136.			//将该包记为收到
137.			status->packet_rx_success_count++;
138.		}
139.		r_message->len = rxmsg->len; // Provide visibility on how far we are into current msg
140.		r_mavlink_status->parse_state = status->parse_state;
141.		r_mavlink_status->packet_idx = status->packet_idx;
142.		r_mavlink_status->current_rx_seq = status->current_rx_seq+1;
143.		r_mavlink_status->packet_rx_success_count = status->packet_rx_success_count;
144.		r_mavlink_status->packet_rx_drop_count = status->parse_error;
145.		status->parse_error = 0;
146.	
147.		if (status->msg_received == MAVLINK_FRAMING_BAD_CRC) {
148.			/*
149.			 CRC结果错误. 我们现在需要用the one on the wire来覆盖msg CRC，这样如果caller 决定转发forward 该消息，无论如何		  mavlink_msg_to_send_buffer() 不会覆盖checksun校验和。
150.			 */
151.			r_message->checksum = _MAV_PAYLOAD(rxmsg)[status->packet_idx] | (_MAV_PAYLOAD(rxmsg)[status->packet_idx+1]<<8);
152.		}
153.		return status->msg_received;
154.	}
```

通讯以字节流的形式到达，调用一次read()不能确保读取的数据是完整帧，需要将每次读取的数据整合在一起，对整合后的数据分析，按照定义的帧格式，通过帧头尾将帧从字节流中抽取出来。

参考：连续读取到串口中，读取串口一般不能一次读完，建议在事件中，把数据加到缓冲区，触发另一段代码从缓冲区里检查是否存在一个完整的数据帧。先在缓冲区中查找第一个帧头部，如果找到，检查后面数据是否够，够了就是一个完整的数据帧，把该帧从缓冲整个取出。

1. 依次读取每一位，指针一直向前；

2. 更新checksum;

3. 检查checksum；

4. 成功接收一个消息包后，status.msg_received返回值为1

当收到完整的消息包时，将消息包复制到* r——message中，并将通道的状态复制到* r_mavlink_status中。

消息包解析器的状态机parse_state：

```c
    MAVLINK_PARSE_STATE_UNINIT=0,
    MAVLINK_PARSE_STATE_IDLE,
    MAVLINK_PARSE_STATE_GOT_STX,
    MAVLINK_PARSE_STATE_GOT_SEQ,
    MAVLINK_PARSE_STATE_GOT_LENGTH,
    MAVLINK_PARSE_STATE_GOT_SYSID,
    MAVLINK_PARSE_STATE_GOT_COMPID,
    MAVLINK_PARSE_STATE_GOT_MSGID,
    MAVLINK_PARSE_STATE_GOT_PAYLOAD,
    MAVLINK_PARSE_STATE_GOT_CRC1,
    MAVLINK_PARSE_STATE_GOT_BAD_CRC1
```

![mavlink_status]()