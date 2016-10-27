# -*- mode: ruby -*-
# vi: set ft=ruby :

# Contains setup for all VMs

VAGRANT_VERSION="2"
NODE_NUM=Integer("#{ENV['NODE_NUM']}")
Vagrant.configure(VAGRANT_VERSION) do |config|
    config.vm.synced_folder ".", "/vagrant"
    (1..NODE_NUM).each do |i|
        config.vm.define "docker#{i}", primary: true do |docker|
            docker.vm.box = "spiral/docker"
            docker.vm.hostname = "docker#{i}"
            docker.vm.provider "virtualbox" do |v|
                v.name = "docker#{i}"
                v.memory = 512
                v.cpus = 1
            end
            #docker.vm.provision "shell", path: "script/update_hosts.sh"
            # if i == 1
            #   docker.vm.network "forwarded_port", guest: 5050, host: 5050
            # end
            docker.vm.network :private_network, ip: "#{ENV['IP_ADDR_PREFIX']}.#{100+ i}", bridge: "#{ENV['NETWORK']}"
        end
    end
end
