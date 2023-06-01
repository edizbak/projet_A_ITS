# -*- mode: ruby -*-
# vi: set ft=ruby :
# To enable zsh, please set ENABLE_ZSH env var to "true" before launching vagrant up 
#   + On windows => $env:ENABLE_ZSH="true"
#   + On Linux  => export ENABLE_ZSH="true"

Vagrant.configure("2") do |config|
  config.vm.define "mediawiki" do |mediawiki|
    mediawiki.vm.box = "geerlingguy/centos7"
	mediawiki.vm.box_download_insecure=true
    mediawiki.vm.network "private_network", type: "static", ip: "192.168.99.10"
    mediawiki.vm.hostname = "mediawiki"
    mediawiki.vm.provider "virtualbox" do |v|
      v.name = "mediawiki"
      v.memory = 2048
      v.cpus = 2
    end
    mediawiki.vm.provision :shell do |shell|
      shell.path = "install_mediawiki.sh"
      shell.args = ["master", "192.168.99.10"]
      shell.env = { 'ENABLE_ZSH' => ENV['ENABLE_ZSH'] }
      
    end
  end
  clients=2
  ram_client=2048
  cpu_client=2
  (1..clients).each do |i|
    config.vm.define "client#{i}" do |client|
      client.vm.box = "geerlingguy/centos7"
      client.vm.network "private_network", type: "static", ip: "192.168.99.1#{i}"
      client.vm.hostname = "client#{i}"
      client.vm.provider "virtualbox" do |v|
        v.name = "client#{i}"
        v.memory = ram_client
        v.cpus = cpu_client
      end
      client.vm.provision :shell do |shell|
        shell.path = "install_mediawiki.sh"
        shell.args = ["node", "192.168.99.10"]
      end
    end
  end
end