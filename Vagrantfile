# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|
 config.vm.box = "ubuntu/focal64"
 config.vm.network "forwarded_port", guest: 80, host: 8080
 config.vm.provider "virtualbox" do |vb|
 vb.memory = "1024"
 vb.cpus = 1
 vb.name = "nginx - webserver"
 end
 config.vm.provision "ansible_local" do |ansible|
 ansible.playbook = "playbook.yml"
 end
end