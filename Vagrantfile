# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.synced_folder ".", "/vagrant", mount_options: ["dmode=0755", "fmode=0644"]
  config.vm.define "ansible" do |ansible|
    ansible.vm.box = "ubuntu/jammy64"
    ansible.vm.network "private_network", type: "static", ip: "192.168.99.20"
    ansible.vm.hostname = "ansible"
    ansible.vm.provider "virtualbox" do |v|
      v.name = "ansible"
      v.memory = 1024
      v.cpus = 2
    end
    ansible.vm.provision :shell do |shell|
      shell.path = "install_ansible.sh"
      shell.args = ["master", "192.168.99.20"]
      shell.env = { 'ENABLE_ZSH' => ENV['ENABLE_ZSH'] }
    end
  end

  # MODIFICATION ICI : On passe à 2 pour avoir client1 ET client2
  clients = 2
  ram_client = 1024
  cpu_client = 2

  (1..clients).each do |i|
    config.vm.define "client#{i}" do |client|
      client.vm.box = "ubuntu/jammy64"
      client.vm.network "private_network", type: "static", ip: "192.168.99.2#{i}"
      client.vm.hostname = "client#{i}"
      client.vm.provider "virtualbox" do |v|
        v.name = "client#{i}"
        v.memory = ram_client
        v.cpus = cpu_client
      end
      client.vm.provision :shell do |shell|
        shell.path = "install_ansible.sh"
        shell.args = ["node", "192.168.99.20"]
      end
    end
  end
end