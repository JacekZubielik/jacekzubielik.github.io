---
title: W pełni zautomatyzowana instalacja wysokiej dostępności K3S
date: 2022-11-20 9:33:00
categories: [wirtualizacja,k3s-playbook]
tags: [wirtualizacja,vagrant,libvirt,k3s]
---

Konfiguracja k3s jest trudna. Dlatego dziś zbudujemy klaster K3s o wysokiej dostępności przy użyciu `etcd`, `MetalLB`, `kube-vip`, `Libvirt`, `Vagrant` `Ansible`. Zautomatyzujemy cały proces, dając Ci łatwy, powtarzalny sposób na utworzenie klastra k3s, który możesz uruchomić w kilka minut na własnym PC.

![A screenshot](/assets/posts/k3s-playbook/container_evolution.svg)

## Przygotowanie.

Vagrant z Libvirt na Linux Wymagania wstępne:

- Zainstalowany Vagrant
- Zainstalowano Libvirt i QEMU-KVM
- Instalacja wtyczki libvirt dla Vagrant

