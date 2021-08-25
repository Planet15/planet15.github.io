---
layout: post
title:  "/proc/net/dev에 관한 설명"
date:   2019-10-29
last_modified_at: 2019-10-29
categories: [linux]
tags: [linux]
---

RHEL7, CENTOS7 기준으로 아래와 같이 network stat의 값이 파일 위치가 바뀌었습니다.

기존에는 **/net/core/dev.c** (RHEL6, CENTOS6 기준) 파일에 있었지만, **net-procfs.c** 로 옮겼으며, 여기 [commit] 에 해당 합니다.

```c
net: move procfs code to net/core/net-procfs.c
```

[commit]: https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/commit/?h=v3.10&id=900ff8c6321418dafa03c22e215cb9646a2541b9

**dev_seq_printf_stats** 함수를 통해 아래의 총 17개의 값으로 표현을 해주고 있습니다.

```c
/net/core/net-procfs.c (RHEL7, CENTOS7 기준)

static void dev_seq_printf_stats(struct seq_file *seq, struct net_device *dev)
{
	struct rtnl_link_stats64 temp;
	const struct rtnl_link_stats64 *stats = dev_get_stats(dev, &temp);

	seq_printf(seq, "%6s: %7llu %7llu %4llu %4llu %4llu %5llu %10llu %9llu "
		   "%8llu %7llu %4llu %4llu %4llu %5llu %7llu %10llu\n",
		   dev->name /* [0] */, stats->rx_bytes /* [1] */, stats->rx_packets /* [2] */,
		   stats->rx_errors /* [3] */,
		   stats->rx_dropped + stats->rx_missed_errors /* [4] */,
		   stats->rx_fifo_errors /* [5] */,
		   stats->rx_length_errors + stats->rx_over_errors +
		    stats->rx_crc_errors + stats->rx_frame_errors /* [6] */,
		   stats->rx_compressed /* [7] */, stats->multicast /* [8] */,
		   stats->tx_bytes /* [9] */, stats->tx_packets /* [10] */,
		   stats->tx_errors /* [11] */, stats->tx_dropped /* [12] */,
		   stats->tx_fifo_errors /* [13] */, stats->collisions /* [14] */,
		   stats->tx_carrier_errors +
		    stats->tx_aborted_errors +
		    stats->tx_window_errors +
		    stats->tx_heartbeat_errors /* [15] */,
		   stats->tx_compressed /* [16] */);
}
```

```c
[0]  : [face] /* dev name */
[1]  : [bytes] /* rx_bytes */
[2]  : [packtes] /* rx_packets */ 
[3]  : [errs] /* rx_errors */
[4]  : [drops] /* rx_dropped + rx_missed_errors */
[5]  : [fifo] /* rx_fifo_errors */
[6]  : [frame] /* rx_length_errors + rx_over_errors */
[7]  : [compressed] /* rx_compressed */
[8]  : [multicast] /* multicast */
[9]  : [bytes] /* tx_bytes */
[10] : [packets] /* tx_packets */
[11] : [errs] /* tx_errors */
[12] : [drop] /* tx_dropped */
[13] : [fifo] /* tx_fifo_errors */
[14] : [collos] /* collisions */
[15] : [carrier] /* tx_carrier_errors + tx_aborted_errors + */
                 /* tx_window_errors + tx_heartbeat_errors */
[16] : [compressed] /* tx_compressed */
```

각 항목에 대한 설명 입니다.

```c
bytes : The total number of bytes of data transmitted or received by the interface.
packets : The total number of packets of data transmitted or received by the interface.
errs : The total number of transmit or receive errors detected by the device driver.
drop : The total number of packets dropped by the device driver.
fifo : The number of FIFO buffer errors.
frame : The number of packet framing errors.
colls : The number of collisions detected on the interface.
compressed : The number of compressed packets transmitted or received by the device driver. 
             (This appears to be unused in the 2.2.15 kernel.)
carrier : The number of carrier losses detected by the device driver.
```

또한 추가적으로 개별 디바이스를 통해 확인 가능한 값 아래와 같습니다

```sh
ls -al /sys/class/net/<디바이스명>/statistics/
ls -al /sys/class/net/eth0/statistics/
collisions           rx_crc_errors        rx_frame_errors      rx_over_errors       tx_carrier_errors    tx_fifo_errors
multicast            rx_dropped           rx_length_errors     rx_packets           tx_compressed        tx_heartbeat_errors
rx_bytes             rx_errors            rx_missed_errors     tx_aborted_errors    tx_dropped           tx_packets
rx_compressed        rx_fifo_errors       rx_nohandler         tx_bytes             tx_errors            tx_window_errors
```

**sa** 명령에 대한 추적 할 경우, **/proc/net/dev** 추적 되는 내용을 확인이 가능 합니다.

```sh
strace /usr/lib64/sa/sa1 1 1 &> results
cat result
...
munmap(0x7fb323118000, 4096)            = 0
open("/proc/net/dev", O_RDONLY)         = 3
fstat(3, {st_mode=S_IFREG|0444, st_size=0, ...}) = 0
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fb323118000
read(3, "Inter-|   Receive               "..., 1024) = 446
read(3, "", 1024)                       = 0
close(3)                                = 0
munmap(0x7fb323118000, 4096)            = 0
open("/proc/net/dev", O_RDONLY)         = 3
fstat(3, {st_mode=S_IFREG|0444, st_size=0, ...}) = 0
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fb323118000
read(3, "Inter-|   Receive               "..., 1024) = 446
read(3, "", 1024)                       = 0
close(3)                                = 0
....
```
