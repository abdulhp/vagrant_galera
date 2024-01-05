# -*- mode: ruby -*-
# vi: set ft=ruby :

LB_NODE_IPS = [
  "192.168.56.11"
]

DB_NODE_IPS = [
  "192.168.56.21",
  "192.168.56.22",
  "192.168.56.23",
  "192.168.56.24",
  "192.168.56.25",
]

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  
  DB_NODE_IPS.each_with_index do |ip, index|
    vm_hostname = "vm-db-vbox-#{index}"
    box_hostname = "db-vbox-#{index}"
    
    config.vm.define box_hostname do |node|
      node.vm.hostname = vm_hostname
      node.vm.network "private_network", ip: ip
      
      node.vm.provider "virtualbox" do |vb|
        vb.name = box_hostname  
      end

      node.vm.provision "init_mariadb", type: "shell", run: "once", inline: <<-SHELL
        apt-get -y update
        apt-get install -y -q --no-install-recommends apt-utils make autoconf
        apt-get clean -y -q
        
        apt-key adv --fetch-keys 'https://mariadb.org/mariadb_release_signing_key.asc'
        apt-add-repository 'deb [arch=amd64,arm64,ppc64el] https://download.nus.edu.sg/mirror/mariadb/repo/10.4/ubuntu/ focal main'
        apt-get -y -q update

        apt-get install -y nginx mariadb-server

        systemctl stop mariadb.service

        ufw allow 3306,4567,4568,4444/tcp
        ufw allow 4567/udp
      SHELL

      if Range.new(0, 2).include? index 
        node.vm.provision "config_galera", type: "shell", run: "once", inline: <<-SHELL
          cp /vagrant/galera.cnf /etc/mysql/conf.d/galera.cnf 

          sed -i 's/THIS_HOSTNAME/#{vm_hostname}/' /etc/mysql/conf.d/galera.cnf
          sed -i 's/ALL_IPS/#{DB_NODE_IPS[..2].join(',')}/' /etc/mysql/conf.d/galera.cnf
          sed -i 's/THIS_IP/#{ip}/' /etc/mysql/conf.d/galera.cnf
        SHELL

        if index == 0
          node.vm.provision "start_galera", type: 'shell', run: 'always', inline: 'galera_new_cluster'
        else
          node.vm.provision "start_mariadb", type: 'shell',  run: 'always', inline: 'service mariadb start'
        end
      else
        node.vm.provision "config_galera", type: "shell", run: "once", inline: <<-SHELL
        cp /vagrant/galera.cnf /etc/mysql/conf.d/galera.cnf 

        sed -i 's/THIS_HOSTNAME/#{vm_hostname}/' /etc/mysql/conf.d/galera.cnf
        sed -i 's/ALL_IPS/#{DB_NODE_IPS.join(',')}/' /etc/mysql/conf.d/galera.cnf
        sed -i 's/THIS_IP/#{ip}/' /etc/mysql/conf.d/galera.cnf
      SHELL
      end
    end
  end

  LB_NODE_IPS.each_with_index do |ip, index|
    vm_hostname = "vm-lb-vbox-#{index}"
    box_hostname = "lb-vbox-#{index}"

    config.vm.define box_hostname do |node|
      node.vm.hostname = vm_hostname
      node.vm.network "private_network", ip: ip
      node.vm.network "forwarded_port", guest: 80, host: 3000
      node.vm.network "forwarded_port", guest: 8404, host: 8404
      
      node.vm.provider "virtualbox" do |vb|
        vb.name = box_hostname  
      end

      node.vm.provision "init_haproxy", type: "shell", run: "once", inline: <<-SHELL
        apt-get -y update
        apt-get install -y -q --no-install-recommends software-properties-common
        apt-get clean -y -q
        
        add-apt-repository ppa:vbernat/haproxy-2.6
        apt-get -y -q update

        apt-get -y install haproxy
      SHELL

    end
  end

end