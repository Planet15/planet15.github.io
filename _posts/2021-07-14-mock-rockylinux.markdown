---
layout: post
title:  "mock를 이용하여 rockylinux 패키지 리빌드 체험 하기"
date:   2021-07-14
last_modified_at: 2021-07-14
categories: [linux]
tags: [linux]
---



**mock 란?**
Mock은 패키지를 빌드하기 위한 도구입니다. 
빌드 호스트가 가지고 있는 것과 다른 아키텍처와 다른 Fedora , RHEL 및 Mageia 버전 및 여러 버젼에 대해서 
패키지를 빌드할 수 있습니다. Mock은 chroot를 안정적으로 구성 후 그 안에 패키지를 빌드합니다. 

**CENTOS7/RHEL7 이상의 머신 만 있으면 쉽게 패키지에 대해서 빌드가 가능합니다.**

1.CentOS 7 이상의 머신에서 아래와 같이 epel-release 패키지 및 mock를 설치를 진행 합니다.

```sh
# sudo yum install epel-release
# sudo yum install mock
```

2.현재 RockyLinux는 8.4를 릴리즈 되었으며, 빌드 환경에 맞게 아래와 같이 mock에 관련된 설정들을 가져 옵니다. 제가 build를 편하게 테스트 하기 위해서 설정 파일에 대해서 수정을 한 상태 이며, 정식으로 build 하는 경우, 식별키에 대해서 작업이 꼭 필요 합니다. **본 문서는 rebuild에 대한 테스트를 하기위해서 쉽게 구성한 내용임을 알려 드립니다.**

```sh
# git clone https://github.com/Planet15/ncloud_infra_example.git
# cd ncloud_infra_example/rockylinux/
# ls
buildsys-macros-repo  mock_config
```

3.두개의 디렉토리가 존재 하며, 사용에 대해서 설명 하겠습니다.
buildsys-macros-repo 는 현재 rockylinux 8.4 릴리즈 버젼을 수정하기 위해서 사용하는 사용자 리파지토리 입니다. 
mock_config 는 아래와 같이 3개의 파일이 존재 하며,

```sh
# cd mock_config/
# ls
rocky8.cfg  rockybuild8.tpl  rockycentos-8.tpl
# cp rocky8.cfg  /etc/mock/.
# cp rockybuild8.tpl  rockycentos-8.tpl /etc/mock/templates/.
```

rocky8.cfg 파일은 build 하기전에 사용되는 일련의 작업들을 구성한 설정 파일 입니다.
상위에 rockybuild8.tpl 와 rockycentos-8.tpl 파일을 참조 받고 있으며, 
저는 /etc/mock/rocky8.cfg 
/etc/mock/templates/ 디렉토리에 rockybuild8.tpl 와 rockycentos-8.tpl 복사 하였습니다.

아래는 rockylinux8.cfg에 대한 build에 필요한 내용을 명시적으로 선언 합니다.
**따로 수정할 내용은 없으며, rockylinux8.cfg 파일을 그대로 이용**합니다.

```sh
include('templates/rockycentos-8.tpl')
#include('templates/epel-8.tpl')
include('templates/rockybuild8.tpl')

config_opts['root'] = 'epel-8-x86_64'
config_opts['target_arch'] = 'x86_64'
config_opts['legal_host_arches'] = ('x86_64',)

# Match "groupinstall build" present on Koji:
#config_opts['chroot_setup_cmd'] = ('groupinstall build')

config_opts['chroot_setup_cmd'] += ('  buildsys-macros-el8  centpkg-minimal  scl-utils-build  ')

# temp. for gegl build:
#config_opts['chroot_setup_cmd'] += ('  libgexiv2-devel ')


# Work around maven pkgs that expect a conf file:
config_opts['files']['/etc/java/maven.conf'] = " "

# work around for various packages needing python3 defined:
config_opts['macros']['__python'] = '%{__python3}'

# Perl workaround:
config_opts['macros']['perl_bootstrap'] = '1'



# For building xmlgraphics-common:
#config_opts['files']['/etc/profile.d/mystuff.sh'] = """
#export JAVA_HOME=/
#"""
```

rockybuild8.tpl 와 rockycentos-8.tpl 는 처음 build 하기 위해서 필요한 리파지토리 및 선언 변수를 지정하는 부분 으로써 기존의 파일에 저는 릴리즈 버젼을 정하기 위해서 다음을 추가 하였습니다. 
**rockybuild8.tpl 에 있는 RockyDevelCustoms 세션에 있는 baseurl 부분은 구성된 위치에 따라서 다를 수 있으니수정이 필요 합니다.**

만약 **rockylinux 버젼이 올라 가는 경우, RockyDevelCustoms 세션의 리파지토리 내용 및 rockybuild8.tpl 와 rockycentos-8.tpl 에서 바라보는 리파지토리도 바뀌**어야 합니다. 
이유는 제가 현재 정적으로 빌드 버젼을 정했기 때문입니다. 
현재는 8.4 기준으로 빌드 환경을 구성 하였습니다.

```sh
.....
[RockyDevelCustoms]
name=Customization for Rocky Builds
baseurl=file:///root/ncloud_infra_example/rockylinux/buildsys-macros-repo/
enabled=1
gpgcheck=0
priority=200
module_hotfixes=1
.....
```

4.디렉토리를 하나 만들어서 테스트를 시작 합니다.
저는 예제로 bash-4.4.19-14.el8.src.rpm 에 대해서 테스트를 하겠습니다.
rebuid 하기 위해서는 srpm 파일이 필요 하며, 패키지를 빌드 하면서 추가로 수정이 필요한 경우가 발생 할 수 있습니다.

```sh
# mkdir /root/mock
# mkdir /root/mock/rpms
# wget http://ftp.iij.ad.jp/pub/linux/centos-vault/centos/8/BaseOS/Source/SPackages/bash-4.4.19-14.el8.src.rpm
```

5.mock를 이용하여 build를 시작 합니다.
```sh
# mock -r rocky8 rebuild bash-4.4.19-14.el8.src.rpm --resultdir=/root/mock/rpms/
....
Finish: rpmbuild bash-4.4.20-1.el8_4.src.rpm
Finish: build phase for bash-4.4.20-1.el8_4.src.rpm
INFO: Done(bash-4.4.20-1.el8_4.src.rpm) Config(rocky8) 5 minutes 33 seconds
INFO: Results and/or logs in: /root/mock/rpms/
INFO: Cleaning up build root ('cleanup_on_success=True')
Start: clean chroot
Finish: clean chroot
Finish: run
```

6.완료된 파일을 --resultdir 에 정의한 디렉토리에서 확인이 가능합니다.
```sh
# cd rpms
# ls -al
total 14732
drwxr-xr-x 2 root root    4096 Jul 14 17:44 .
drwxr-xr-x 5 root root     119 Jul 14 17:38 ..
-rw-r--r-- 1 root mock 9475791 Jul 14 17:40 bash-4.4.20-1.el8_4.src.rpm
-rw-r--r-- 1 root mock 1620724 Jul 14 17:44 bash-4.4.20-1.el8_4.x86_64.rpm
-rw-r--r-- 1 root mock 1161764 Jul 14 17:44 bash-debuginfo-4.4.20-1.el8_4.x86_64.rpm
-rw-r--r-- 1 root mock  861620 Jul 14 17:44 bash-debugsource-4.4.20-1.el8_4.x86_64.rpm
-rw-r--r-- 1 root mock  115648 Jul 14 17:44 bash-devel-4.4.20-1.el8_4.x86_64.rpm
-rw-r--r-- 1 root mock 1306748 Jul 14 17:44 bash-doc-4.4.20-1.el8_4.x86_64.rpm
-rw-rw-r-- 1 root root  253809 Jul 14 17:44 build.log
-rw-rw-r-- 1 root root    1573 Jul 14 17:39 hw_info.log
-rw-rw-r-- 1 root root   21144 Jul 14 17:40 installed_pkgs.log
-rw-rw-r-- 1 root root  234088 Jul 14 17:44 root.log
-rw-rw-r-- 1 root root    1014 Jul 14 17:44 state.log
```

참고한 사이트는[ref1], [ref2], [ref3] 입니다.
[ref1]: https://github.com/rpm-software-management/mock/wiki 
[ref2]: https://wiki.rockylinux.org/en/team/development/Mock_Build_Howto
[ref3]: https://fedoraproject.org/wiki/Using_Mock_to_test_package_builds
