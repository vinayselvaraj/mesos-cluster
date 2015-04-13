# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|

  config.vm.define "master" do |master|
    master.vm.box = "chef/centos-7.0"
    master.vm.network "private_network", ip: "172.28.128.100"
    master.vm.synced_folder ".", "/vagrant", disabled: true
    master.vm.provision "shell", inline: <<-SHELL
        sudo yum -y update
        sudo rpm -Uvh http://repos.mesosphere.io/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm        
        sudo yum -y install mesos marathon mesosphere-zookeeper docker
        
        sudo systemctl stop mesos-slave.service
        sudo systemctl disable mesos-slave.service
        
        sudo echo "172.28.128.100" > /etc/mesos-master/hostname
        sudo echo "172.28.128.100" > /etc/mesos-master/ip
        
        sudo systemctl enable docker
        sudo systemctl start docker
        
        sudo systemctl start zookeeper
        sudo systemctl start mesos-master
        sudo systemctl start marathon
    SHELL
  end
  
  (1..3).each do |i|
    config.vm.define "slave#{i}" do |node|
      private_ip = "172.28.128.11#{i}"
      node.vm.box = "chef/centos-7.0"
      node.vm.network "private_network", ip: private_ip
      node.vm.synced_folder ".", "/vagrant", disabled: true
      node.vm.provision "shell", inline: <<-SHELL
        sudo yum -y update
        sudo rpm -Uvh http://repos.mesosphere.io/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm
        sudo yum -y install mesos docker
        
        sudo systemctl stop mesos-master.service
        sudo systemctl disable mesos-master.service
        
        sudo echo "zk://172.28.128.100:2181/mesos" > /etc/mesos/zk
        sudo echo #{private_ip} > /etc/mesos-slave/hostname
        sudo echo #{private_ip} > /etc/mesos-slave/ip
        sudo echo "docker,mesos" > /etc/mesos-slave/containerizers
        
        sudo systemctl enable docker
        
        sudo systemctl start docker
        sudo systemctl start mesos-slave
      SHELL
    end
  end

end
