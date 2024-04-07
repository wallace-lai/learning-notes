# PCAP文件格式与读取方法

作者：wallace-lai <br/>
发布：2024-04-03 <br/>
更新：2023-04-03 <br/>

【pending】

The libpcap file format is the main capture file format used in TcpDump / WinDump, snort, and many other networking tools. It is fully supported by Wireshark/TShark, but they now generate pcapng files by default.

## Overview

This file format is a very basic format to save captured network data. As the libpcap library became the "de facto" standard of network capturing on UN*X, it became the "common denominator" for network capture files in the open source world (there seems to be no such thing as a "common denominator" in the commercial network capture world at all).

Libpcap, and the Windows port of libpcap, WinPcap, use the same file format.

**Although it's sometimes assumed that this file format is suitable for Ethernet networks only, it can serve many different network types**, examples can be found at the Wireshark's Supported Capture Media page; all listed types are handled by the libpcap file format.

The proposed file extension for libpcap based files is: .pcap

Wireshark handles all capture file I/O in the wiretap library. You'll find further details about the libpcap file format in the wiretap/libpcap.c and .h files.

## File Format

There are some variants of the format "in the wild", the following will only describe the commonly used format in its current version 2.4. This format version hasn't changed for quite a while (at least since libpcap 0.4 in 1998), so it's not expected to change except for the PCAPng file format mentioned below.

The file has a global header containing some global information followed by zero or more records for each captured packet, looking like this:

- Global Header

- Packet Header

- Packet Data

- Packet Header

- Packet Data

- ...

A captured packet in a capture file does not necessarily contain all the data in the packet as it appeared on the network; the capture file might contain at most the first N bytes of each packet, for some value of N. The value of N, in such a capture, is called the "snapshot length" or "snaplen" of the capture. N might be a value larger than the largest possible packet, to ensure that no packet in the capture is "sliced" short; a value of 65535 will typically be used in this case.

## Global Header

This header starts the libpcap file and will be followed by the first packet header:

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

未完待续...