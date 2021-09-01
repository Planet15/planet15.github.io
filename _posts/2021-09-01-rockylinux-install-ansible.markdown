---
layout: post
title:  "Rocky Linux 8에 ansible 설치하기"
date:   2021-09-01
last_modified_at: 2021-09-01
categories: [linux]
tags: [linux]
---

1. Rocky Linux 의 기본 리파지토리에는 ansible 패키지를 제공하고 있지 않습니다. 따라서 아래와 같은 에러가 발생 합니다.
```sh
$ sudo yum install ansible
Last metadata expiration check: 0:58:30 ago on Wed 01 Sep 2021 01:21:25 AM UTC.
No match for argument: ansible
Error: Unable to find a match: ansible
```

2.  EPEL 리파지토리에서 설치하면 ansible 패키지를 설치할 수 있으며, 다른 종속성은 기본 repos와 appstream 을 통해 해결이 가능합니다.
```sh
$ sudo dnf install epel-release -y
Last metadata expiration check: 0:59:03 ago on Wed 01 Sep 2021 01:21:25 AM UTC.
Dependencies resolved.
======================================================================================================================== Package                         Architecture              Version                      Repository                 Size 
========================================================================================================================Installing:
 epel-release                    noarch                    8-10.el8                     extras                     22 k 

Transaction Summary
========================================================================================================================Install  1 Package


Installed:
  epel-release-8-10.el8.noarch

Complete!
```

3. EPEL 리파지토리 설치 및 활성화 이후, 다시 ansible을 설치 합니다. 
```sh
$ sudo dnf install ansible -y
Extra Packages for Enterprise Linux Modular 8 - x86_64                                   16 kB/s | 931 kB     00:57    
Extra Packages for Enterprise Linux 8 - x86_64                                          4.4 MB/s |  10 MB     00:02    
Last metadata expiration check: 0:00:03 ago on Wed 01 Sep 2021 02:21:43 AM UTC.
Dependencies resolved.
======================================================================================================================== Package                       Architecture    Version                                         Repository          Size 
========================================================================================================================Installing:
 ansible                       noarch          2.9.25-1.el8                                    epel                17 M 
Installing dependencies:
 libsodium                     x86_64          1.0.18-2.el8                                    epel               162 k 
 python3-babel                 noarch          2.5.1-5.el8                                     appstream          4.8 M 
 python3-bcrypt                x86_64          3.1.6-2.el8.1                                   epel                44 k 
 python3-cffi                  x86_64          1.11.5-5.el8                                    baseos             237 k 
 python3-cryptography          x86_64          3.2.1-4.el8                                     baseos             558 k 
 python3-jinja2                noarch          2.10.1-2.el8_0                                  appstream          536 k 
 python3-jmespath              noarch          0.9.0-11.el8                                    appstream           44 k 
 python3-markupsafe            x86_64          0.23-19.el8                                     appstream           38 k 
 python3-pip                   noarch          9.0.3-19.el8.rocky                              appstream           19 k 
 python3-ply                   noarch          3.9-9.el8                                       baseos             110 k 
 python3-pyasn1                noarch          0.3.7-6.el8                                     appstream          125 k 
 python3-pycparser             noarch          2.14-14.el8                                     baseos             108 k 
 python3-pynacl                x86_64          1.3.0-5.el8                                     epel               100 k 
 python3-pytz                  noarch          2017.2-9.el8                                    appstream           53 k 
 python3-setuptools            noarch          39.2.0-6.el8                                    baseos             162 k 
 python36                      x86_64          3.6.8-2.module+el8.4.0+597+ddf0ddea             appstream           18 k 
 sshpass                       x86_64          1.06-9.el8                                      epel                27 k 
Installing weak dependencies:
 python3-paramiko              noarch          2.4.3-1.el8                                     epel               289 k 
Enabling module streams:
 python36                                      3.6
...
Complete!
```

4. ansible 설치 후, 버젼을 확인 합니다.
```sh
$ ansible --version
ansible 2.9.25
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/vagrant/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']      
  ansible python module location = /usr/lib/python3.6/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.6.8 (default, May 19 2021, 03:00:47) [GCC 8.4.1 20200928 (Red Hat 8.4.1-1)]
  ```

5. ansible setup 모듈을 이용하여, 동작여부를 파악 합니다.
```sh
$ ansible -m setup localhost
localhost | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "10.0.2.15"
        ],
        ...
```