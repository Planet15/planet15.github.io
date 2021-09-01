---
layout: post
title:  "Ansible - Resolve 'couldn't resolve module' 에러 해결"
date:   2021-09-01
last_modified_at: 2021-09-01
categories: [ansible]
tags: [ansible]
---

1. 아래와 같이 playbook을 테스트 하려고 했으나, couldn't resolve module/action 에러가 발생하였습니다.
```sh
$ sudo ansible-playbook init-rocky-openqa-developer-host.yml --check
...
ERROR! couldn't resolve module/action 'ansible.posix.firewalld'. 
This often indicates a misspelling, missing collection, or incorrect module path.
...
```

2. 아래와 같은 playbook 구문에서 에러가 나는 것을 확인 하였습니다. 이에러는 
```
- name: Permit traffic for {{ item }} service
  ansible.posix.firewalld:
    service: "{{ item }}"
    permanent: true
    state: enabled
  loop:
    - httpd
    - openqa-vnc
```
3. 에러의 내용을 판단하여 firewalld 모듈을 포함하는 컬렉션이 ansible이 설치된 노드에 설치가 되어있지 않으며, ansible.posix.firewalld는 ansible.posix 컬렉션에 있습니다. [ansible.posix.firewalld]
```sh
$ sudo ansible-galaxy collection install ansible.posix
Process install dependency map
Starting collection install process
Installing 'ansible.posix:1.3.0' to '/root/.ansible/collections/ansible_collections/ansible/posix'
```

[ansible.posix.firewalld]: https://docs.ansible.com/ansible/latest/collections/ansible/posix/firewalld_module.html

4. 설치 후, 모듈 에러가 해결 된것을 확인 할수 있습니다.
```sh
$ sudo ansible-playbook init-rocky-openqa-developer-host.yml --check
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match
'all'

PLAY [Rocky OpenQA Runbook] ********************************************************************************************
TASK [Gathering Facts] *************************************************************************************************ok: [localhost]

TASK [Check if ansible cannot be run here] *****************************************************************************ok: [localhost]

TASK [Verify if we can run ansible] ************************************************************************************ok: [localhost] => {
    "changed": false,
    "msg": "We are able to run on this node"
}
...
```