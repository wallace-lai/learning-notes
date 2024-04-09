# PCAP文件解析

作者：wallace-lai <br/>
发布：2024-04-03 <br/>
更新：2023-04-09 <br/>

【pending】

## 一、PCAP文件格式
pcap文件中的数据一般按照如下的形式来组织。首先是一个全局的header结构，用于描述整个pcap文件，随后是一个又一个的数据包。数据包由两部分来描述，分别是header结构和实际的数据。

- Global Header

- Packet Header

- Packet Data

- Packet Header

- Packet Data

- ...

### 1.1 Global Header

Global Header由以下的C语言结构体给出定义。

```c
typedef struct pcap_hdr_s {
    guint32 magic_number;   /* magic number */
    guint16 version_major;  /* major version number */
    guint16 version_minor;  /* minor version number */
    gint32  thiszone;       /* GMT to local correction */
    guint32 sigfigs;        /* accuracy of timestamps */
    guint32 snaplen;        /* max length of captured packets, in octets */
    guint32 network;        /* data link type */
} pcap_hdr_t;
```

（1）`magic_number`用于指明文件格式，同时也说明了文件内容的字节序。pcap文件的`magic_number`值为`0xa1b2c3d4`。

（2）`version_major`和`version_minor`用于指明pcap文件格式的版本，当前版本为2.4.

（3）`thiszone`用于校正本地时间和GMT时间之间的差值。一般情况下，本地时间用的就是GMT时区，因此`thiszone`的值为0。

（4）`sigfigs`捕获数据所用时间戳的精度。一般值为0。

（5）`network`用于指明网络类型，一般情况下是`LINKTYPE_ETHERNET`（其值为1），即以太网。


### 1.2 Packet Header

Packet Header由以下的C语言结构体给出定义。
```c
typedef struct pcaprec_hdr_s {
        guint32 ts_sec;         /* timestamp seconds */
        guint32 ts_usec;        /* timestamp microseconds */
        guint32 incl_len;       /* number of octets of packet saved in file */
        guint32 orig_len;       /* actual length of packet */
} pcaprec_hdr_t;
```

（1）`ts_sec`和`ts_usec`指的是数据包捕获时的时间，分别是秒和毫秒。

（2）`incl_len`表示实际捕获的数据包存储所占用的大小。

（3）`orig_len`表示数据包在网络流量中的实际大小。

<p style="color: red">
问题：incl_len和orig_len有什么区别？
</p>

## 二、使用dpkt解析PCAP文件

### 2.1 dpkt官方代码案例


未完待续...