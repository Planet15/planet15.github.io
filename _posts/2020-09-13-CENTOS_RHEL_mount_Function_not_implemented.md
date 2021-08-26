---
layout: post
title:  "CENTOS/RHEL 에서 mount 를 할때 Function not implemented 에러 분석"
date:   2020-09-13
last_modified_at: 2020-09-13
categories: [linux]
tags: [linux]
---

안녕하세요? 오늘은 centos7에서   mount 시 Function not implemented 에러에 대해 어떤 이슈 인지 확인 하는 시간을 가져 보도록 하겠습니다.

재현 방법은 새로운 디스크를 할당 받은 후, mkfs.xfs에서  block size에 대해서 64K로 지정을 한 다음, block size가 64K로 구성된 디스크에 대해서 마운트를 시도 합니다.

​
mount 시 **Function not implemented** 라는 에러를 확인 할수 있습니다. 

```sh
# mkfs.xfs -b size=64k /dev/xvdb
meta-data=/dev/xvdb              isize=512    agcount=4, agsize=2048000 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=65536  blocks=8192000, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=65536  ascii-ci=0 ftype=1
log      =internal log           bsize=65536  blocks=4000, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=65536  blocks=0, rtextents=0
 
 
[root@leo-lab ~]# mount -t xfs /dev/xvdb /mnt/
mount: mount /dev/xvdb on /mnt failed: Function not implemented
```

에러 내용이 궁금하여, 아래와 같이 흐름을 따라 가면서 분석을 해보았습니다.
dmesg를 통해서, 아래와 같은 메세지가 출력되는 것을 확인 할수 있습니다.

```sh
# dmesg
...
[6037019.890621] XFS (xvdb): File system with blocksize 65536 bytes. Only pagesize (4096) or less will currently work.
[6037019.890648] XFS (xvdb): SB validate failed with error -38.
....
```
메세지의 의미를 파악 하기 위해서, **linux-3.10.0-1127.13.1.el7.x86_64** 의 커널 기준으로 확인 결과,  xfs를 mount 하려고 할때, **xfs_mount_validate_sb** 라는 함수가 **호출**이 되며,  리눅스 시스템의 **PAGE_SIZE 와 비교 하는 구문**이 포함이 되어있습니다.  **메세지 의미는 파일 시스템은 65535(64K) 이지만, PAGE 크기가 4096 이니 4096 혹은 그 이하로 낮게 해라는 커널에서 출력 하는 메세지**를  확인이 가능 합니다.

```c
# vim fs/xfs/libxfs/xfs_sb.c
....
/*
 * Check the validity of the SB found.
 */
STATIC int
xfs_mount_validate_sb(
        xfs_mount_t     *mp,
        xfs_sb_t        *sbp,
        bool            check_inprogress,
        bool            check_version)
{
....
        /*
         * Until this is fixed only page-sized or smaller data blocks work.
         */
        if (unlikely(sbp->sb_blocksize > PAGE_SIZE)) { /* <---------- */
 
                xfs_warn(mp,
                "File system with blocksize %d bytes. "
                "Only pagesize (%ld) or less will currently work.",
                                sbp->sb_blocksize, PAGE_SIZE);
                return -ENOSYS; /*
                                 * #define ENOSYS          38  Function not implemented
                                */
 
        }
....
}
....
```

x86_64 기준으로 페이지 크기 (PAGE_SIZE)는 (1 << PAGE_SHIFT) 즉, 2 ^ 12 = 4096 의 값이 나오는 것이 확인 가능 하였습니다

```c
# vim include/asm-generic/page.h
....
/* PAGE_SHIFT determines the page size */
....
#define PAGE_SHIFT      12
#ifdef __ASSEMBLY__
/* 따라서 실제로 비트를 왼쪽 (1 << PAGE_SHIFT)으로 이동하면 2 ^ 12 = 4096 값이 나옵니다.*/
/* echo "2^12" | bc */
#define PAGE_SIZE       (1 << PAGE_SHIFT)
#else
#define PAGE_SIZE       (1UL << PAGE_SHIFT)
#endif
#define PAGE_MASK       (~(PAGE_SIZE-1))
....
```
아래와 같이 쉘에서 리눅스 쉘 계산기 bc 를 이용 하여 계산이 가능합니다.

```sh
# bc --version
bc 1.06.95
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006 Free Software Foundation, Inc.

# rpm -qf /usr/bin/bc
bc-1.06.95-13.el7.x86_64

# echo "2^12" | bc
4096
```

사용중인 시스템에서는 아래와 같이 PAGESIZE를 확인이 가능 합니다.
현재 제가 사용중인 VM 인스턴스는 x86_64 이며 PAGESIZE는 4096  입니다.

```sh
# uname -m
x86_64
# getconf PAGESIZE
4096
```