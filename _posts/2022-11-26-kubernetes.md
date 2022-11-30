---
title: Kubernetes - początki
date: 2022-11-26 9:33:00
categories: [wirtualizacja,k3s-playbook]
tags: [wirtualizacja,vagrant,libvirt,k3s,kubectl]
---

## Instalacja kubectl:


```shell
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```


## kube config

Aby uzyskać dostęp do klastra Kubernetes i skopiować konfigurację kube, uruchom lokalnie:

```shell
scp vagrant@192.168.122.222:~/.kube/config ~/.kube/config 
```

> user: vagrant, pass: vagrant
{: .prompt-tip }


Pobieranie węzłów

```shell
kubectl get nodes
```

## Wdrażanie przykładowego obciążenia nginx

```shell
kubectl apply -f example/deployment.yml
```

Sprawdź, czy został wdrożony

```shell
kubectl describe deployment nginx
```

Wdrażanie przykładowej usługi nginx z Load Balancer

```shell
kubectl apply -f example/service.yml
```

Sprawdź usługę i upewnij się, że ma adres IP z `metal lb` zgodnie z definicją w `inventory/my-cluster/group_vars/all.yml`

```shell
kubectl describe service nginx
```
Odwiedź ten adres URL lub curl

```shell
curl http://192.168.20.200
```

Powinieneś zobaczyć stronę powitalną nginx. Możesz to wyczyścić, uruchamiając

```shell
kubectl delete -f example/deployment.yml
kubectl delete -f example/service.yml
```


# Apt update i upgrade
`playbooks/apt.yml`

```yml
- hosts: "*"
  become: yes
  tasks:
    - name: apt
      apt:
        update_cache: yes
        upgrade: 'yes'
```
Apt update i upgrade dla wszystkich hostów. Odpalamy:

```shell
ansible-playbook ./playbooks/apt.yml -i ./inventory/hosts.ini

```

```log
PLAY [*] ****************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************
ok: [k3s-04.anotherlife.pl]
ok: [k3s-02.anotherlife.pl]
ok: [k3s-03.anotherlife.pl]
ok: [k3s-05.anotherlife.pl]
ok: [k3s-01.anotherlife.pl]

TASK [apt] **************************************************************************************************************************************************
ok: [k3s-01.anotherlife.pl]
ok: [k3s-03.anotherlife.pl]
ok: [k3s-02.anotherlife.pl]
ok: [k3s-05.anotherlife.pl]
ok: [k3s-04.anotherlife.pl]

PLAY RECAP **************************************************************************************************************************************************
k3s-01.anotherlife.pl      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k3s-02.anotherlife.pl      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k3s-03.anotherlife.pl      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k3s-04.anotherlife.pl      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k3s-05.anotherlife.pl      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
# Qemu-guest-agent

`playbooks/qemu-guest-agent.yml`

```yml
- name: install latest qemu-guest-agent
  hosts: "*"
  tasks:
    - name: install qemu-guest-agent
      apt:
        name: qemu-guest-agent
        state: present
        update_cache: true
      become: true
```

```shell
ansible-playbook ./playbooks/qemu-guest-agent.yml -i ./inventory/hosts.ini
```

```log
PLAY [install latest qemu-guest-agent] *******************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************
ok: [k3s-01.anotherlife.pl]
ok: [k3s-05.anotherlife.pl]
ok: [k3s-02.anotherlife.pl]
ok: [k3s-03.anotherlife.pl]
ok: [k3s-04.anotherlife.pl]

TASK [install qemu-guest-agent] **************************************************************************************************************************************************
changed: [k3s-04.anotherlife.pl]
changed: [k3s-05.anotherlife.pl]
changed: [k3s-01.anotherlife.pl]
changed: [k3s-03.anotherlife.pl]
changed: [k3s-02.anotherlife.pl]

PLAY RECAP ***********************************************************************************************************************************************************************
k3s-01.anotherlife.pl      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k3s-02.anotherlife.pl      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k3s-03.anotherlife.pl      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k3s-04.anotherlife.pl      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k3s-05.anotherlife.pl      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

# zsh

`playbooks/zsh.yml`

```yml
- name: install latest zsh on all hosts
  hosts: "*"
  tasks:
    - name: install zsh
      apt:
        name: zsh
        state: present
        update_cache: true
      become: true
```

```shell
ansible-playbook ./playbooks/zsh.yml -i ./inventory/hosts.ini
```

```log
PLAY [install latest zsh on all hosts] *******************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************
ok: [k3s-03.anotherlife.pl]
ok: [k3s-04.anotherlife.pl]
ok: [k3s-01.anotherlife.pl]
ok: [k3s-05.anotherlife.pl]
ok: [k3s-02.anotherlife.pl]

TASK [install zsh] ***************************************************************************************************************************************************************
changed: [k3s-02.anotherlife.pl]
changed: [k3s-01.anotherlife.pl]
changed: [k3s-04.anotherlife.pl]
changed: [k3s-03.anotherlife.pl]
changed: [k3s-05.anotherlife.pl]

PLAY RECAP ***********************************************************************************************************************************************************************
k3s-01.anotherlife.pl      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k3s-02.anotherlife.pl      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k3s-03.anotherlife.pl      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k3s-04.anotherlife.pl      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k3s-05.anotherlife.pl      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

# timezone

`playbooks/timezone.yml`

```yml
- name: Set timezone and configure timesyncd
  hosts: "*"
  become: yes
  tasks:
  - name: set timezone
    shell: timedatectl set-timezone Europe/Warsaw

  - name: Make sure timesyncd is stopped
    systemd:
      name: systemd-timesyncd.service
      state: stopped

  - name: Copy over the timesyncd config
    template: src=../templates/timesyncd.conf dest=/etc/systemd/timesyncd.conf

  - name: Make sure timesyncd is started
    systemd:
      name: systemd-timesyncd.service
      state: started
```

`templates/timesyncd.conf`

```conf
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.
#
# Entries in this file show the compile time defaults.
# You can change settings by editing this file.
# Defaults can be restored by simply deleting this file.
#
# See timesyncd.conf(5) for details.

[Time]
NTP=192.168.20.1
FallbackNTP=time.cloudflare.com
#RootDistanceMaxSec=5
#PollIntervalMinSec=32
#PollIntervalMaxSec=2048
```

```shell
ansible-playbook ./playbooks/timezone.yml -i ./inventory/hosts.ini
```

```log
PLAY [Set timezone and configure timesyncd] **************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************
ok: [k3s-04.anotherlife.pl]
ok: [k3s-03.anotherlife.pl]
ok: [k3s-01.anotherlife.pl]
ok: [k3s-02.anotherlife.pl]
ok: [k3s-05.anotherlife.pl]

TASK [set timezone] **************************************************************************************************************************************************************
changed: [k3s-02.anotherlife.pl]
changed: [k3s-05.anotherlife.pl]
changed: [k3s-04.anotherlife.pl]
changed: [k3s-01.anotherlife.pl]
changed: [k3s-03.anotherlife.pl]

TASK [Make sure timesyncd is stopped] ********************************************************************************************************************************************
changed: [k3s-03.anotherlife.pl]
changed: [k3s-02.anotherlife.pl]
changed: [k3s-05.anotherlife.pl]
changed: [k3s-04.anotherlife.pl]
changed: [k3s-01.anotherlife.pl]

TASK [Copy over the timesyncd config] ********************************************************************************************************************************************
changed: [k3s-03.anotherlife.pl]
changed: [k3s-02.anotherlife.pl]
changed: [k3s-01.anotherlife.pl]
changed: [k3s-04.anotherlife.pl]
changed: [k3s-05.anotherlife.pl]

TASK [Make sure timesyncd is started] ********************************************************************************************************************************************
changed: [k3s-02.anotherlife.pl]
changed: [k3s-04.anotherlife.pl]
changed: [k3s-01.anotherlife.pl]
changed: [k3s-03.anotherlife.pl]
changed: [k3s-05.anotherlife.pl]

PLAY RECAP ***********************************************************************************************************************************************************************
k3s-01.anotherlife.pl      : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k3s-02.anotherlife.pl      : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k3s-03.anotherlife.pl      : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k3s-04.anotherlife.pl      : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k3s-05.anotherlife.pl      : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```