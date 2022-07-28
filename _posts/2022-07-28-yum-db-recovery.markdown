---
layout: post
title: "Error: rpmdb open failed떄 복구 시나리오"
date: 2022-07-28
last_modified_at: 2022-07-28
categories: [linux]
tags: [linux]
---

rpm DB가 깨어진 상태와 복구가 필요한 테스트가 필요하여 시나리오를 구성 해보았다.

1. 먼저 간단히 rpm DB를 아래와 같이 깨어진 상태를 만든다.
   이미 깨어진 상태라면 이 과정은 생략 한다.

```sh
# cd /var/lib/rpm
# cat /dev/null > __db.001
# cat /dev/null > __db.002
# cat /dev/null > __db.003 
```

2. 이후 아래와 같이 yum check-update 명령을 사용 하면 Error: rpmdb open failed
에러 메세지가 확인이 가능하다.

```sh
# yum check-update
error: db5 error(11) from dbenv->open: Resource temporarily unavailable
error: cannot open Packages index using db5 - Resource temporarily unavailable (11)
error: cannot open Packages database in /var/lib/rpm
CRITICAL:yum.main:
Error: rpmdb open failed
```

3. rpm --rebuilddb 명령을사용하여 복구를 시도 한다. -vv 옵션을 통해 복구 과정을 확인 가능하다.

```sh
# rpm --rebuilddb -vv
D: rebuilding database /var/lib/rpm into /var/lib/rpmrebuilddb.1646
D: opening db environment /var/lib/rpm private:0x401
D: opening db index /var/lib/rpm/Packages 0x400 mode=0x0
D: locked db index /var/lib/rpm/Packages
D: opening db environment /var/lib/rpmrebuilddb.1646 private:0x401
D: opening db index /var/lib/rpmrebuilddb.1646/Packages (none) mode=0x42
D: opening db index /var/lib/rpmrebuilddb.1646/Packages 0x1 mode=0x42
D: disabling fsync on database
<snip>
D: closed db index /var/lib/rpm/Packages
D: closed db environment /var/lib/rpm
D: closed db index /var/lib/rpmrebuilddb.1646/Sha1header
D: closed db index /var/lib/rpmrebuilddb.1646/Sigmd5
D: closed db index /var/lib/rpmrebuilddb.1646/Installtid
D: closed db index /var/lib/rpmrebuilddb.1646/Dirnames
D: closed db index /var/lib/rpmrebuilddb.1646/Triggername
D: closed db index /var/lib/rpmrebuilddb.1646/Obsoletename
D: closed db index /var/lib/rpmrebuilddb.1646/Conflictname
D: closed db index /var/lib/rpmrebuilddb.1646/Providename
D: closed db index /var/lib/rpmrebuilddb.1646/Requirename
D: closed db index /var/lib/rpmrebuilddb.1646/Group
D: closed db index /var/lib/rpmrebuilddb.1646/Basenames
D: closed db index /var/lib/rpmrebuilddb.1646/Name
D: closed db index /var/lib/rpmrebuilddb.1646/Packages
D: closed db environment /var/lib/rpmrebuilddb.1646
```

4. 최종적으로yum clean all을 한후, yum check-update 를 확인하여 복구된 rpm DB를 확인한다.

```sh
# yum clean all
# yum check-update
```

