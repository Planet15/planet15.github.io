---
layout: post
title:  "CentOS에서 opensl 1.1.1를 rpm 형태로 build 해서 사용하기"
date:   2021-08-27
last_modified_at: 2021-08-27
categories: [linux]
tags: [linux]
---

1. 빌드 테스트를 위해서 [openssl111] 를 다운 받아 와서 빌드를 시작 합니다.
검색 엔진에서 openssl11-1.1.1g-3.el7.src.rpm 를 검색 합니다. 
src.rpm은 소스RPM이라는 뜻이며, 파일명에서 보시는 내용과 같이 el7 RedHat 
Enterprise Linux 7 기반 이라는 사실을 알수 있습니다.

2. [rpmbuild] man 페이지 내용과 같이 rebuild 명령은 주어진 소스 패키지를 설치하고 준비, 컴파일 및 설치를 수행하는 옵셥 입니다.
```sh
rpmbuild --rebuild|--recompile 소스패키지명... 
```
따라서 아래와 같이 rebuild 옵셥을 통해 수행합니다.
```sh
# rpmbuild --rebuild openssl11-1.1.1g-3.el7.src.rpm
```



3. 아래와 같이 의존성 이슈가 발생 하게 됩니다.
총 4개가 발생 하였으며, lksctp-tools-devel, perl(Test::More), perl(Module::Load::Conditional), devtoolset-8-toolchain 입니다.
```sh
warning: group mock does not exist - using root
warning: user mockbuild does not exist - using root
warning: group mock does not exist - using root
warning: user mockbuild does not exist - using root
warning: group mock does not exist - using root
warning: user mockbuild does not exist - using root
warning: group mock does not exist - using root
error: Failed build dependencies:
        lksctp-tools-devel is needed by openssl11-1:1.1.1g-3.el7.x86_64
        perl(Test::More) is needed by openssl11-1:1.1.1g-3.el7.x86_64
        perl(Module::Load::Conditional) is needed by openssl11-1:1.1.1g-3.el7.x86_64
        devtoolset-8-toolchain is needed by openssl11-1:1.1.1g-3.el7.x86_64
```
4. lksctp-tools-devel 의존성은 의존성 이슈에 나온것과 같이 해당 패키지를 설치 합니다.
```sh
# yum install lksctp-tools-devel -y
```
5. perl에 관련된 [perl-Test-Simple] 내용이며, rpm으로 묶인 패키지를 설치 합니다.
```sh
# yum install perl-Test-Simple
```


6.  perl에 관련된 [perl-Module-Load-Conditional] 내용이며, rpm으로 묶인 패키지를 설치 합니다.
```sh
# yum install perl-Module-Load-Conditional
```


7. [Red-Hat-Developer-Toolset-8.0] 를 설치 해야 하지만, 기본적으로 centos 7에서 지원하는 리파지토리에는 없는 것으로 확인 합니다.
```sh
# yum install devtoolset-8-toolchain
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * epel: ftp.iij.ad.jp
 * remi-php74: ftp.riken.jp
 * remi-safe: ftp.riken.jp
No package devtoolset-8-toolchain available.
Error: Nothing to do
```



8. centos-release-scl 리파지토리를 설치 합니다.
```sh
# yum install centos-release-scl -y
```

9. yum-config-manager명령어를 통해 rhel-server-rhscl-7-rpms 리파지토리를 활성화 시킵니다. 
```sh
# yum-config-manager --enable rhel-server-rhscl-7-rpms
```

10. 9번 이후에 다시 [Red-Hat-Developer-Toolset-8.0] 를 설치 합니다.
```sh
# yum install devtoolset-8 -y
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * centos-sclo-rh: mirror.navercorp.com
 * centos-sclo-sclo: mirror.navercorp.com
 * epel: ftp.iij.ad.jp
 * remi-php74: ftp.riken.jp
 * remi-safe: ftp.riken.jp
centos-sclo-rh                                                                                                    | 3.0 kB  00:00:00
centos-sclo-sclo                                                                                                  | 3.0 kB  00:00:00
(1/2): centos-sclo-sclo/x86_64/primary_db                                                                         | 300 kB  00:00:00
(2/2): centos-sclo-rh/x86_64/primary_db                                                                           | 3.2 MB  00:00:00
Resolving Dependencies
...
Installed:
  devtoolset-8.x86_64 0:8.1-1.el7

Dependency Installed:
  devtoolset-8-binutils.x86_64 0:2.30-55.el7.2                         devtoolset-8-dwz.x86_64 0:0.12-1.1.el7
  devtoolset-8-dyninst.x86_64 0:9.3.2-6.el7                            devtoolset-8-elfutils.x86_64 0:0.176-1.el7
  devtoolset-8-elfutils-libelf.x86_64 0:0.176-1.el7                    devtoolset-8-elfutils-libs.x86_64 0:0.176-1.el7
  devtoolset-8-gcc.x86_64 0:8.3.1-3.2.el7                              devtoolset-8-gcc-c++.x86_64 0:8.3.1-3.2.el7
  devtoolset-8-gcc-gfortran.x86_64 0:8.3.1-3.2.el7                     devtoolset-8-gdb.x86_64 0:8.2-3.el7
  devtoolset-8-libquadmath-devel.x86_64 0:8.3.1-3.2.el7                devtoolset-8-libstdc++-devel.x86_64 0:8.3.1-3.2.el7
  devtoolset-8-ltrace.x86_64 0:0.7.91-1.el7                            devtoolset-8-make.x86_64 1:4.2.1-4.el7
  devtoolset-8-memstomp.x86_64 0:0.1.5-5.el7                           devtoolset-8-oprofile.x86_64 0:1.3.0-2.el7
  devtoolset-8-perftools.x86_64 0:8.1-1.el7                            devtoolset-8-runtime.x86_64 0:8.1-1.el7
  devtoolset-8-strace.x86_64 0:4.24-4.el7                              devtoolset-8-systemtap.x86_64 0:3.3-2.el7
  devtoolset-8-systemtap-client.x86_64 0:3.3-2.el7                     devtoolset-8-systemtap-devel.x86_64 0:3.3-2.el7
  devtoolset-8-systemtap-runtime.x86_64 0:3.3-2.el7                    devtoolset-8-toolchain.x86_64 0:8.1-1.el7
  devtoolset-8-valgrind.x86_64 1:3.14.0-16.el7                         libgfortran5.x86_64 0:8.3.1-2.1.1.el7

Complete!
```

11.  rebuild 옵셥을 통해 수행합니다.
```sh
# rpmbuild --rebuild openssl11-1.1.1g-3.el7.src.rpm
...
```

```sh
Wrote: /root/rpmbuild/RPMS/x86_64/openssl11-1.1.1g-3.el7.x86_64.rpm
Wrote: /root/rpmbuild/RPMS/x86_64/openssl11-1.1.1g-3.el7.x86_64.rpm
Wrote: /root/rpmbuild/RPMS/x86_64/openssl11-devel-1.1.1g-3.el7.x86_64.rpm
Wrote: /root/rpmbuild/RPMS/x86_64/openssl11-static-1.1.1g-3.el7.x86_64.rpm
Wrote: /root/rpmbuild/RPMS/x86_64/openssl11-debuginfo-1.1.1g-3.el7.x86_64.rpm
Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.ClYuEz
+ umask 022
+ cd /root/rpmbuild/BUILD
+ cd openssl-1.1.1g
+ /usr/bin/rm -rf /root/rpmbuild/BUILDROOT/openssl11-1.1.1g-3.el7.x86_64
+ exit 0
Executing(--clean): /bin/sh -e /var/tmp/rpm-tmp.C4clCr
+ umask 022
+ cd /root/rpmbuild/BUILD
+ rm -rf openssl-1.1.1g
+ exit 0
```

12. 정상적으로 빌드 된것을 확인 할수 있습니다.

```sh
/root/rpmbuild/RPMS/x86_64/openssl11-1.1.1g-3.el7.x86_64.rpm
/root/rpmbuild/RPMS/x86_64/openssl11-1.1.1g-3.el7.x86_64.rpm
/root/rpmbuild/RPMS/x86_64/openssl11-devel-1.1.1g-3.el7.x86_64.rpm
/root/rpmbuild/RPMS/x86_64/openssl11-static-1.1.1g-3.el7.x86_64.rpm
/root/rpmbuild/RPMS/x86_64/openssl11-debuginfo-1.1.1g-3.el7.x86_64.rpm
```

[rpmbuild]: https://linux.die.net/man/8/rpmbuild
[openssl111]: https://koji.fedoraproject.org/koji/buildinfo?buildID=1729681
[perl-Test-Simple]: https://perldoc.perl.org/Test::Simple
[perl-Module-Load-Conditional]: https://perldoc.perl.org/Module::Load::Conditional
[Red-Hat-Developer-Toolset-8.0]: https://access.redhat.com/documentation/en-us/red_hat_developer_toolset/8/html/8.0_release_notes/dts8.0_release
