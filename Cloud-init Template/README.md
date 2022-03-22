# Шаблон в ProxMox VE на основе cloud-init образа Ubuntu 20.04

- Скачиваем образ Ubuntu 20.04 на сервере Proxmox
```
wget https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img
```
- Устанавливаем libguestfs-tools на сервере Proxmox
```
sudo apt update -y && sudo apt install libguestfs-tools -y
```
- Ставим в образ агента qemu, net-tools, bash-completion (и все что необходимо для базового образа)
```
virt-customize -a focal-server-cloudimg-amd64.img --install qemu-guest-agent
virt-customize -a focal-server-cloudimg-amd64.img --install net-tools
virt-customize -a focal-server-cloudimg-amd64.img --install bash-completion
```
- Создаем виртуальную машину с нужными нам параметрами и импортируем диск
```
sudo qm create 9000 --name "ubuntu-2004-cloudinit-template" --memory 1024 --cores 1 --numa 1 --net0 virtio,bridge=vmbr0
#sudo qm create 9000 --name "ubuntu-2004-cloudinit-template" --memory 1024 --cores 1 --numa 1 --net0 virtio,bridge=vmbr0 --sockets 1 --cpu cputype=kvm64  --kvm 1

sudo qm importdisk 9000 focal-server-cloudimg-amd64.img VM

sudo qm set 9000 --scsihw virtio-scsi-pci --scsi0 /vm/images/9000/vm-9000-disk-0.raw
sudo qm set 9000 --ide2 VM:cloudinit --boot c --bootdisk scsi0 --serial0 socket --vga serial0 --agent 1
sudo qm set 9000 --hotplug disk,network,usb,memory,cpu
```
- Настраиваем Cloud-Init
```
sudo qm set 9000 --ciuser alex  --ipconfig0 ip=192.168.0.250/24,gw=192.168.0.1 --nameserver 192.168.0.1 --searchdomain 192.168.0.1 --cipassword
```
   - Не забываем задать SSH public key через веб интерфейс или опцию --sshkey с укзанием файла
   ```
   sudo qm set 9000 --sshkey sshkey
   ```
- Запускаем нашу виртуальную машину
```
sudo qm start 9000
```
- После запуска перезагружаемся и конфигурируем машинку
   - Редактируем файл /etc/cloud/cloud.cfg добавляя установленные нами пакеты
	~~~
	# Install packages
	package_upgrade: true
	packages:
	 - qemu-quest-agent
	 - net-tools
	 - bash-completion
	~~~
	- настраиваем /etc/bash.bashrc /root/.bashrc и т.д.
- Выключаем машинку
```
sudo qm shutdown 9000
```
- Преобразовываем нашу машинку в шаблон
```
sudo qm template 9000
```