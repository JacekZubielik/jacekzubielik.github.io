---
title: Zarządzanie maszynami wirtualnymi KVM – Vagrant i libvirt
date: 2022-11-21 10:00:00
categories: [wirtualizacja,k3s-playbook]
tags: [wirtualizacja,vagrant,libvirt,k3s]
---

Kiedy po raz pierwszy zainstalujesz `Vagrant`, są szanse, że początkowo będziesz korzystać z dostawcy `VirtualBox VM`, który jest obsługiwany od razu po wyjęciu z pudełka. Jednak w niektórych sytuacjach `VirtualBox` może nie być preferowanym hipernadzorcą. Na szczęście za pomocą wtyczki, `Vagrant` może być również używany z `KVM`. W tym poście dowiemy się, jak to działa i skorzystamy z okazji, aby dowiedzieć się trochę o `KVM`, `libvirt` i tym podobnych.

`KVM` (Kernel Virtual Machine) to moduł jądra Linuksa, który zamienia Linuksa w hipernadzorcę, wykorzystując sprzętową obsługę wirtualizacji, która jest wbudowana we wszystkie nowoczesne procesory x86 (ta funkcja nazywa się `VT-X` na procesorach Intel i `AMD-V` na procesorach AMD). Zazwyczaj KVM nie jest używany bezpośrednio, ale jest zarządzany przez [libvirt](https://libvirt.org/), który jest zbiorem składników oprogramowania, takich jak API, demon zdalnego dostępu i narzędzie wiersza poleceń `virsh` do sterowania maszynami wirtualnymi.

`Libvirt` z kolei może być używany z klientami w większości głównych języków programowania (w tym C, Java, Python i Go) i jest wykorzystywany przez wiele narzędzi do wirtualizacji, takich jak graficzny menedżer maszyn wirtualnych [ virt-manager](https://virt-manager.org/) lub `OpenStack Nova` do zarządzania maszyn wirtualnych. Istnieje również klient Ruby dla libvirt API, który czyni go gotowym do pracy z Vagrant.

Oprócz `KVM`, `libvirt` jest w stanie wykorzystać wielu innych dostawców wirtualizacji, w tym `LXC`, `VMWare` i `HyperV`. Poniższy diagram podsumowuje, w jaki sposób omawiane komponenty są powiązane.

![Libvirt](/assets/posts/k3s-playbook/lib-virt.svg)

## Tworzenie maszyn wirtualnych za pomocą Vagrant

Powyższy schemat wskazuje, że mamy do wyboru kilka metod tworzenia maszyn wirtualnych. Ponieważ `libvirt` nie jest jednym ze standardowych dostawców wbudowanych w Vagrant, będziemy musieli najpierw zainstalować wtyczkę. Zakładając, że w ogóle nie zainstalowałeś jeszcze Vagrant, oto kroki potrzebne do zainstalowania i skonfigurowania `Vagrant`, `KVM` i wymaganej wtyczki na standardowej instalacji Ubuntu. Najpierw instalujemy bibliotekę `libvirt`, `virt-manager`, `vagrant`, `packer`, a bieżącego użytkownika dodajemy do grup `libvirt` i `KVM`.


Importowanie klucza GPG repozytorium:

```shell
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
```

Dodaj repozytorium APT HashiCorp, uruchamiając poniższe polecenia w terminalu:

```shell
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt update
```

```shell
sudo apt-get update 
sudo apt-get install \
  wget \
  apt-transport-https \
  gnupg2 \
  libvirt-daemon \
  libvirt-clients \
  virt-manager \
  python3-libvirt \
  packer \
  vagrant \
  vagrant-libvirt 
sudo adduser $(id -un) libvirt
sudo adduser $(id -un) kvm
```

Musisz się wylogować (lub uruchomić `su -l`). Zauważ, że instalujemy wtyczkę [libvirt Vagrant](https://github.com/vagrant-libvirt/vagrant-libvirt) z pakietu Ubuntu, a nie bezpośrednio, dla innych dystrybucji Linuksa, możesz zainstalować za pomocą `vagrant plugin install vagrant-libvirt`. 

Potwierdź instalację, sprawdzając wersje oprogramowania:

```shell
packer -v
```

Packer 1.8.4

```shell
vagrant -v
```

Vagrant 2.3.3


## Ubuntu Vagrant Box dla KVM

Projekt Bento na Github

Użyj git do pobrania źródła Bento z Github:

```shell
git clone https://github.com/chef/bento.git
```

Szablon Packera dla Ubuntu znajduje się w katalogu _`bento/packer_templates/ubuntu/`_.
Przejdźmy do tego katalogu:

```shell
cd bento/packer_templates/Ubuntu/
```


```shell
ls
ubuntu-22.04-amd64.json
```

Następnie możesz zbudować Ubuntu Box:

```shell
packer build -only=qemu ubuntu-22.04-amd64.json
```

Importuj Box

```shell
vagrant box add ubuntu-22.04-amd64 file://../../builds/ubuntu-22.04.libvirt.box
```

Potwierdź, że box jest dostępny:

```shell
vagrant box list | grep ubuntu
```

```
ubuntu-22.04-amd64 (libvirt, 0)
```

## Instancja maszyny wirtualnej przy użyciu Vagrant Box
Projekt k3s-playbook na Github

Użyj git do pobrania źródła k3s-playbook z Github:

```shell
git clone https://github.com/JacekZubielik/k3s-playbook.git
```

Plik dla **Libvirt** znajduje się w katalogu _`k3s-playbook/vagrant/`_.
Przejdźmy tam:

```shell
cd ./k3s-playbook/vagrant/
```

```shell
vim Vagrantfile
```

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :


types = ['master', 'infra', 'app']

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'

Vagrant.require_version ">= 1.7.0"
Vagrant.configure("2") do |config|

  types.each do |t|
    3.times do |i|
      config.vm.define "k3s-#{t}-#{i}" do |config|
      config.vm.hostname = "k3s-#{t}-#{i}"
      config.vm.box = "ubuntu-22.04-amd64"
      #config.vm.box = "generic/ubuntu2204"
      config.vm.box_check_update = true
      config.ssh.insert_key = true
      config.vm.network "private_network", ip: "192.168.122.1#{ types.find_index(t) }#{ i }", :libvirt__network_name => 'playbooknet'
      config.vm.provider :libvirt do |v|
        v.memory = 4096
        v.cpus = 5
        v.default_prefix="playbook-"
        end
      end
    end
  end
end
```

W drugim wierszu widać zmienną środowiskową, która instruuje Vagranta, aby używał dostawcy `libvirt` (zamiast domyślnego dostawcy `VirtualBox`). W kolejnych kilku wierszach definiujemy maszyny wirtualne. W bloku specyficznym dla dostawcy definiujemy liczbę procesorów wirtualnych dla komputera i ilość pamięci RAM oraz ustawiamy prefiks, którego Vagrant będzie używał do budowania nazwy domeny libvirt dla maszyny wirtualnej.

Teraz powinieneś być w stanie wywołać maszynę za pomocą `vagrant up` w katalogu, w którym znajduje się plik.

Spróbujmy dowiedzieć się więcej o konfiguracji, którą stworzył dla nas Vagrant. Najpierw uruchom `virt-manager`, aby uruchomić graficznego menedżera maszyn. W przeglądzie powinna być teraz widoczna nowa maszyna wirtualna, a podczas dwukrotnego kliknięcia na komputerze powinien otworzyć się terminal.

Klikając ikonę "Informacje" lub "Wyświetl - > szczegóły", powinieneś również zobaczyć konfigurację maszyny, w tym takie rzeczy, jak podłączone dyski wirtualne i interfejsy sieciowe.

![virt-manager](/assets/posts/k3s-playbook/virtual-manager-k3s.png)

Oczywiście możemy również uzyskać te - a nawet więcej - informacje za pomocą wiersza poleceń klienta **virsh**. Najpierw uruchom `virsh list`, aby wyświetlić listę wszystkich domen (tj. Maszyn wirtualnych).

```shell
virsh list
```

```
 Id   Name                    State
---------------------------------------
 1    playbook-k3s-app-1      running
 2    playbook-k3s-app-2      running
 3    playbook-k3s-master-2   running
 4    playbook-k3s-infra-2    running
 5    playbook-k3s-app-0      running
 6    playbook-k3s-infra-1    running
 7    playbook-k3s-master-1   running
 8    playbook-k3s-infra-0    running
 9    playbook-k3s-master-0   running
 ```

Zakładając, że nie masz uruchomionych innych maszyn wirtualnych zarządzanych przez libvirt, da ci to wynik, który już widzieliśmy w menedżerze maszyn wirtualnych. Możesz pobrać podstawowe fakty na temat konkretnej maszyny za pomocą:

```shell
virsh dominfo playbook-k3s-app-1
```

```
Id:             1
Name:           playbook-k3s-app-1
UUID:           4e53fc37-e334-479e-b4e6-810a569f085d
OS Type:        hvm
State:          running
CPU(s):         5
CPU time:       32,0s
Max memory:     4194304 KiB
Used memory:    4194304 KiB
Persistent:     yes
Autostart:      disable
Managed save:   no
Security model: apparmor
Security DOI:   0
Security label: libvirt-4e53fc37-e334-479e-b4e6-810a569f085d (enforcing)
```

Narzędzie `virsh` ma szeroką gamę opcji, najlepszym sposobem na nauczenie się go jest wpisanie `virsh help` i po prostu wypróbowanie kilku poleceń. Kilka godnych uwagi przykładów to:


```
# List all storage pools
virsh pool-list
```

```
# List all virtual network interfaces attached to the VM
virsh domiflist playbook-k3s-app-1
```

```
# List all networks
virsh net-list
```

Wewnętrznie libvirt używa [plików konfiguracyjnych XML](https://libvirt.org/format.html) do utrzymywania stanu maszyn wirtualnych, sieci i obiektów pamięci masowej. Aby zobaczyć na przykład pełną konfigurację XML naszej maszyny testowej, uruchom

```
virsh dumpxml playbook-k3s-app-1
```

Na wyjściu widzimy teraz wszystkie obiekty: procesor, dyski i kontroler dysków, interfejsy sieciowe, kartę graficzną, klawiaturę itp., podłączone do maszyny. Możemy teraz zrzucić dalsze struktury XML i dane, aby głęboko zagłębić się w konfigurację. Na przykład dane wyjściowe XML dla maszyny mówią nam, że maszyna jest podłączona do sieci `vagrant-libvirt`, odpowiadającej wirtualnemu mostkowi Linux virbr1 (oczywiście libvirt używa mostów do modelowania sieci). Aby uzyskać więcej informacji na ten temat, uruchom

```
virsh net-dumpxml vagrant-libvirt
virsh net-dhcp-leases vagrant-libvirt
ifconfig virbr1
brctl show virbr1
```