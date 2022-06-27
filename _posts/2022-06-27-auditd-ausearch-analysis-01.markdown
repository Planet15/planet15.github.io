---
layout: post
title:  "auditd에서ausearch 활용 하여 로그 분석 하기"
date:   2022-06-27
last_modified_at: 2022-06-27
categories: [linux]
tags: [linux]
---

auditd 서비스가 동작중이며, Oracle Linux 6.X, 7.X, 8.X, Red Hat Enterprise Linux 6.X, 7.X, 8.X, 
CentOS 6.x, 7.X, 8.X, Rocky Linux 8.X에서 auditd 서비스가 running 상태라면 사용 가능 하다.

ausearch 는이벤트 식별자, 키 식별자, CPU 아키텍처, 명령 이름, 호스트 이름, 그룹 이름 또는 그룹 ID, 시스템 호출, 
메시지 등과 같은 다양한 검색 기준 및 이벤트를 기반으로audit 데몬 로그 파일을 검색하는 데 사용되는 간단한 명령줄 도구이다.


ausearch는 다른 텍스트 파일처럼 볼 수 있는 /var/log/audit/audit.log 파일을 쿼리하여 확인을 한다..
```sh
# cat /var/log/audit/audit.log
```

사용 방법은 간단 하며, 특정한 프로세스에 대해서 audit에 대한검색이  가능 하다

```sh
# ps -aux | grep sshd
<snip>
root        1369  0.0  0.2 153440 10960 ?        Ss   09:57   0:00 sshd: root [priv]
<snip>

# ausearch -p 1369 
<snip>
----
time->Mon Jun 27 14:57:26 2022
type=CRYPTO_SESSION msg=audit(1656309446.213:176): pid=1369 uid=0 auid=0 ses=1 subj=system_u:system_r:sshd_t:s0-s0:c0.c1023 msg='op=start direction=from-client cipher=aes256-ctr ksize=256 mac=hmac-sha2-256 pfs=curve25519-sha256 spid=1384 suid=0 rport=55310 laddr=192.168.219.154 lport=22  exe="/usr/sbin/sshd" hostname=? addr=192.168.219.9 terminal=? res=success'
----
<snip>

```

특정 이벤트로도 확인이 가능하다. 가령, 시스템에서 promiscuous mode 이벤트에 대해서 audit 을 확인 하고 싶은 경우 
아래의 명령어롤 틍해 확인이 가능하다.

```sh
# ausearch --message ANOM_PROMISCUOUS
<snip>
time->Wed Jun 15 08:37:31 2022
type=ANOM_PROMISCUOUS msg=audit(1655249851.351:312): dev=enp0s3 prom=0 old_prom=256 auid=0 uid=72 gid=72 ses=7
<snip>
```

여러 메세지 이벤트에 대해서각 버젼 별로 확인이필요 하다RHEL v6[ref1], RHEL v7[ref2], RHEL v8 및 전버젼[ref3]

[ref1]:https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/security_guide/sec-audit_record_types
[ref2]:https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/security_guide/sec-understanding_audit_log_files
[ref3]:https://access.redhat.com/articles/4409591
