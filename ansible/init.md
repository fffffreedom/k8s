```
---

#disable selinux
- name: Get selinux status
  command: getenforce
  register: selinux_status
  ignore_errors: True
  changed_when: false

- name: Get selinux status
  command: setenforce 0
  when: selinux_status.stdout != 'Disabled'

- lineinfile:
    path: /etc/selinux/config
    regexp: 'SELINUX='
    line: 'SELINUX=disabled'

#add harbor dns record to /etc/hosts
- lineinfile:
    path: /etc/hosts
    regexp: 'registry.work.com'
    line: "{{ registry_ip }} registry.work.com"

#disable firewalld
- name: Stop firewalld
  service: name=firewalld state=stopped
  tags: configure

- name: disable firewalld
  service: name=firewalld enabled=no

#set sysctl
- lineinfile:
    path: /etc/sysctl.conf
    regexp: 'vm.overcommit_memory'
    line: 'vm.overcommit_memory = 1'

#set sysctl
- lineinfile:
    path: /etc/sysctl.conf
    regexp: 'vm.oom_dump_tasks'
    line: 'vm.oom_dump_tasks = 1'

#set sysctl
- lineinfile:
    path: /etc/sysctl.conf
    regexp: 'vm.oom_kill_allocating_task'
    line: 'vm.oom_kill_allocating_task = 0'

#set sysctl
- lineinfile:
    path: /etc/sysctl.conf
    regexp: 'vm.panic_on_oom'
    line: 'vm.panic_on_oom = 0'

#set sysctl
- lineinfile:
    path: /etc/sysctl.conf
    regexp: 'kernel.panic'
    line: 'kernel.panic = 10'

#set sysctl
- lineinfile:
    path: /etc/sysctl.conf
    regexp: 'kernel.keys.root_maxbytes'
    line: 'kernel.keys.root_maxbytes = 25000000'

#set sysctl
- lineinfile:
    path: /etc/sysctl.conf
    regexp: 'kernel.keys.root_maxkeys'
    line: 'kernel.keys.root_maxkeys = 1000000'

- name: exec sysctl command
  command: sysctl -p

#swap off
- name: Remove swapfile from /etc/fstab
  mount:
    name: swap
    fstype: swap
    state: absent

- name: Disable swap
  command: swapoff -a

#set yum repo
- name: set  cloud repo
  template: src=cloudrepo.repo.j2 dest=/etc/yum.repos.d/cloudrepo.repo
  tags: install

#update yum cache
- name: install updates
  yum: name=vim update_cache=yes

- name: Install rpm
  yum: name={{ item }}
  with_items:
    - conntrack-tools
    - socat
```
