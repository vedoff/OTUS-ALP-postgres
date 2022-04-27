# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME']
ENV["LC_ALL"] = "en_US.UTF-8"


Vagrant.configure(2) do |config|
  # ВИРТУАЛЬНАЯ МАШИНА №1
    config.vm.define "srv01" do |srv01|
    # имя виртуальной машины
    srv01.vm.box = 'debian11/v1230.1' 
    srv01.vm.provider "virtualbox" do |vb|
      vb.name = "srv01"
    end
    # hostname виртуальной машины
    srv01.vm.hostname = "srv01"
    # настройки сети
    srv01.vm.network "private_network", ip: "192.168.56.40"
    srv01.vm.synced_folder ".", "/vagrant",  
          type: "rsync",
          rsync_auto: "true",
          rsync_exclude: [".git/",".vagrant/",".gitignore","Vagrantfile"]
          #srv01.vm.provision "shell", path: "provision/prepare-host.sh"
    end

  # ВИРТУАЛЬНАЯ МАШИНА №2
    config.vm.define "srv02" do |srv02|
    # имя виртуальной машины
    srv02.vm.box = 'debian11/v1230.1' 
    srv02.vm.provider "virtualbox" do |vbc|
      vbc.name = "srv02"
    end
    # hostname виртуальной машины
    srv02.vm.hostname = "srv02"
    # настройки сети
    srv02.vm.network "private_network", ip: "192.168.56.41"
    srv02.vm.synced_folder ".", "/vagrant",  
          type: "rsync",
          rsync_auto: "true",
          rsync_exclude: [".git/",".vagrant/",".gitignore","Vagrantfile"]
          #srv02.vm.provision "shell", path: "provision/prepare-host.sh"
    end

  # ВИРТУАЛЬНАЯ МАШИНА №3
    config.vm.define "srv03" do |srv03|
    # имя виртуальной машины
    srv03.vm.box = 'debian11/v1230.1' 
    srv03.vm.provider "virtualbox" do |vbc|
      vbc.name = "srv03"
    end
    # hostname виртуальной машины
    srv03.vm.hostname = "srv03"
    # настройки сети
    srv03.vm.network "private_network", ip: "192.168.56.42"
    srv03.vm.synced_folder ".", "/vagrant",  
        type: "rsync",
        rsync_auto: "true",
        rsync_exclude: [".git/",".vagrant/",".gitignore","Vagrantfile"]
        #srv02.vm.provision "shell", path: "provision/prepare-host.sh"
  end

  # ВИРТУАЛЬНАЯ МАШИНА №4
    config.vm.define "backup" do |backup|
    # имя виртуальной машины
    backup.vm.box = 'debian11/v1230.1' 
    backup.vm.provider "virtualbox" do |vbc|
      vbc.name = "backup"
  end
    # hostname виртуальной машины
    backup.vm.hostname = "backup"
    # настройки сети
    backup.vm.network "private_network", ip: "192.168.56.43"
    backup.vm.synced_folder ".", "/vagrant",  
        type: "rsync",
        rsync_auto: "true",
        rsync_exclude: [".git/",".vagrant/",".gitignore","Vagrantfile"]
        #srv02.vm.provision "shell", path: "provision/prepare-host.sh"

# ssh-pub add in servers
  config.vm.provision "shell", inline: <<-SHELL
  cat /vagrant/provision/vagrant-key.pub >> /home/vagrant/.ssh/authorized_keys
  SHELL

    end
end
