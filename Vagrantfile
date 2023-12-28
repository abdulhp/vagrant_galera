# -*- mode: ruby -*-
# vi: set ft=ruby :

NODE_IPS = [
  "192.168.56.11",
  "192.168.56.12",
  "192.168.56.13"
]

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  
  NODE_IPS.each_with_index do |ip, index|
    vm_hostname = "vm-vbox-#{index}"
    box_hostname = "vbox-#{index}"
    
    config.vm.define box_hostname do |node|
      node.vm.hostname = vm_hostname
      node.vm.network "private_network", ip: ip
      
      node.vm.provider "virtualbox" do |vb|
        vb.name = box_hostname  
      end

      node.vm.provision "init_mariadb", type: "shell", run: "once", inline: <<-SHELL
        apt-get -y update
        apt-get install -y --no-install-recommends apt-utils make autoconf
        apt-get clean -y
        
        apt-key adv --fetch-keys 'https://mariadb.org/mariadb_release_signing_key.asc'
        apt-add-repository 'deb [arch=amd64,arm64,ppc64el] https://mirror.biznetgio.com/mariadb/repo/10.4/ubuntu focal main'
        apt-get -y update

        apt-get install -y nginx mariadb-server

        systemctl stop mariadb.service

        ufw allow 3306,4567,4568,4444/tcp
        ufw allow 4567/udp

        cp /vagrant/galera.cnf /etc/mysql/conf.d/galera.cnf 

        sed -i 's/THIS_HOSTNAME/#{vm_hostname}/' /etc/mysql/conf.d/galera.cnf
        sed -i 's/ALL_IPS/#{NODE_IPS.join(',')}/' /etc/mysql/conf.d/galera.cnf
        sed -i 's/THIS_IP/#{ip}/' /etc/mysql/conf.d/galera.cnf
      SHELL

      if index == 0
        node.vm.provision "start_galera", type: 'shell', run: 'once', inline: 'galera_new_cluster'
      else
        node.vm.provision "start_mariadb", type: 'shell',  run: 'once', inline: 'service mariadb start'
      end
    end
  end
end