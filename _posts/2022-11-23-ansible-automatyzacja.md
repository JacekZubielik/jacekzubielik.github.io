---
title: Ansible automatyzacja wszystkiego
date: 2022-11-23 9:33:00
categories: [wirtualizacja,k3s-playbook]
tags: [wirtualizacja,vagrant,libvirt,k3s,ansible]
---

## Instalujemy wymagane zależności
```shell
sudo apt install -y software-properties-common && sudo apt install -y ansible
```

## Definicja hostów

`ansible/inventory/hosts.ini`

[hosts.ini](https://raw.githubusercontent.com/JacekZubielik/k3s-playbook/main/ansible/inventory/hosts.ini)

```ini
[master]
192.168.122.100 ansible_ssh_user=vagrant ansible_ssh_pass=vagrant host_key_checking=False
192.168.122.101 ansible_ssh_user=vagrant ansible_ssh_pass=vagrant host_key_checking=False
192.168.122.102 ansible_ssh_user=vagrant ansible_ssh_pass=vagrant host_key_checking=False

[infra]
192.168.122.110 ansible_ssh_user=vagrant ansible_ssh_pass=vagrant host_key_checking=False
192.168.122.111 ansible_ssh_user=vagrant ansible_ssh_pass=vagrant host_key_checking=False
192.168.122.112 ansible_ssh_user=vagrant ansible_ssh_pass=vagrant host_key_checking=False

[app]
192.168.122.120 ansible_ssh_user=vagrant ansible_ssh_pass=vagrant host_key_checking=False
192.168.122.121 ansible_ssh_user=vagrant ansible_ssh_pass=vagrant host_key_checking=False
192.168.122.122 ansible_ssh_user=vagrant ansible_ssh_pass=vagrant host_key_checking=False

[k3s_cluster:children]
master
infra
app
```


Sprawdzamy czy hosty `master` odpowiadają:

```shell
ansible -i ./inventory/hosts.ini master -m ping
```

```
192.168.122.101 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.122.102 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.122.100 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

Teraz hosty `infra`:

```shell
ansible -i ./inventory/hosts.ini infra -m ping
```

```
192.168.122.111 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.122.112 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.122.110 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

Na końcu hosty `app`:

```shell
ansible -i ./inventory/hosts.ini app -m ping
```

```
192.168.122.120 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.122.121 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.122.122 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

## Udostępnianie klastra

```shell
ansible-playbook ./playbooks/site.yml -i ./inventory/hosts.ini
```

Po wdrożeniu płaszczyzna kontroli będzie dostępna za pośrednictwem wirtualnego adresu IP, który jest zdefiniowany w `inventory/group_vars/all.yml` jako `apiserver_endpoint`.

```yml
apiserver_endpoint: "192.168.122.222"
```

Upewnij się, że możesz pingować swojego VIPa zdefiniowanego w `inventory/group_vars/all.yml` jako `apiserver_endpoint`

```shell
ping 192.168.122.222
```

## Usuwanie
Aby usunąć k3s z węzłów, te węzły powinny zostać zrestartowane.

```shell
ansible-playbook ./playbooks/reset.yml -i ./inventory/hosts.ini
```
