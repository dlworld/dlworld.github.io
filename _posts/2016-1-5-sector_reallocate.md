---
layout: post
title: sector reallocate
categories: [disk]
tags: disk
---




### 系统日志提示磁盘介质故障

```
[root@node51 ~]# dmesg 
[ 1827.393889] ata2.00: exception Emask 0x0 SAct 0x0 SErr 0x0 action 0x0
[ 1827.393928] ata2.00: BMDMA stat 0x24
[ 1827.393950] ata2.00: failed command: READ DMA
[ 1827.393977] ata2.00: cmd c8/00:08:88:bd:a0/00:00:00:00:00/e9 tag 2 dma 4096 in
         res 51/40:00:8f:bd:a0/40:03:06:00:00/e9 Emask 0x9 (media error)
[ 1827.394051] ata2.00: status: { DRDY ERR }
[ 1827.394072] ata2.00: error: { UNC }
[ 1827.400297] ata2.00: configured for UDMA/100
[ 1827.400336] sd 1:0:0:0: [sdc]  
[ 1827.400355] Result: hostbyte=DID_OK driverbyte=DRIVER_SENSE
[ 1827.400384] sd 1:0:0:0: [sdc]  
[ 1827.400403] Sense Key : Medium Error [current] [descriptor]
[ 1827.400437] Descriptor sense data with sense descriptors (in hex):
[ 1827.402034]         72 03 11 04 00 00 00 0c 00 0a 80 00 00 00 00 00 
[ 1827.403672]         09 a0 bd 8f 
[ 1827.405282] sd 1:0:0:0: [sdc]  
[ 1827.406799] Add. Sense: Unrecovered read error - auto reallocate failed
[ 1827.408319] sd 1:0:0:0: [sdc] CDB: 
[ 1827.409817] Read(10): 28 00 09 a0 bd 88 00 00 08 00
[ 1827.411361] end_request: I/O error, dev sdc, sector 161529231
[ 1827.412891] ata2: EH complete
```

硬盘一般都有一部分预留的扇区用于重新分配给损坏的扇区。但重新分配只有在写操作失败时才触发，而读操作失败一般只会抛出I/O错误，极少情况会重新分配。上面的日志显示自动分配失败（Unrecovered read error - auto reallocate failed），这种情况有可能时异常断电导致的。

解决方法是强制重新分配损坏的扇区，通过hdparm工具向损坏的扇区发写命令就可实现。

### 先检查磁盘重新分配的扇区数量：

```
[root@node51 ~]# smartctl -a /dev/sdc | grep -i reallocated
  5 Reallocated_Sector_Ct   0x0033   200   200   140    Pre-fail  Always       -       0
196 Reallocated_Event_Count 0x0032   200   200   000    Old_age   Always       -       0
```

### 确认日志中的161529231扇区确已损坏：

```
[root@node51 ~]# hdparm --read-sector 161529231 /dev/sdc
/dev/sdc:
reading sector 161529231: FAILED: Input/output error
```

### 向磁盘损坏扇区发写命令，

```
[root@node51 ~]# hdparm --write-sector 161529231 --yes-i-know-what-i-am-doing /dev/sdc
/dev/sdc:
re-writing sector 161529231: succeeded
```

### 再读，扇区已经可用

```
[root@node51 ~]# hdparm --read-sector 161529231 /dev/sdc
/dev/sdc:
reading sector 161529231: succeeded
0000 0000 0000 0000 0000 0000 0000 0000
0000 0000 0000 0000 0000 0000 0000 0000
```

### 再次检查磁盘重新分配的扇区数量

```
[root@node51 ~]# smartctl -a /dev/sdc | grep -i reallocated
  5 Reallocated_Sector_Ct   0x0033   200   200   140    Pre-fail  Always       -       0
196 Reallocated_Event_Count 0x0032   200   200   000    Old_age   Always       -       0
```
﻿
重新分配的数量应该增加，实际没有，原因待查。但系统日志不再报错。

