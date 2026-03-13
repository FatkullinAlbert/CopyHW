# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"

  # Клиентская машина
  config.vm.define "client" do |client|
    client.vm.hostname = "client"
    client.vm.network "private_network", ip: "192.168.56.150"
    client.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
      vb.cpus = 1
    end
  end

  # Сервер резервного копирования
  config.vm.define "backup" do |backup|
    backup.vm.hostname = "backup"
    backup.vm.network "private_network", ip: "192.168.56.160"
    backup.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
      vb.cpus = 1
      # Добавляем второй диск размером 2GB
      unless File.exist?("./backup_disk.vdi")
        vb.customize ['createhd', '--filename', './backup_disk.vdi', '--size', 2048]
      end
      vb.customize ['storageattach', :id, '--storagectl', 'SCSI', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', './backup_disk.vdi']
    end
  end

  # Настройка Ansible
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "ansible/playbook.yml"
    ansible.inventory_path = "ansible/inventory"
    ansible.become = true
  end
end
