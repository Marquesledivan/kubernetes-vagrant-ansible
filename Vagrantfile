# -*- mode: ruby -*-
# vi: set ft=ruby :

# NUM_NODES=1
NUM_NODES = ENV['NUM_NODES'] || 2
RUNTIME = ENV['RUNTIME'] || "install-playbook"

Vagrant.configure(2) do |config|

    config.ssh.insert_key = false
    config.vm.box = "bento/ubuntu-20.04"
    config.vm.box_version = "202008.16.0"

    # using cache for apt
    if Vagrant.has_plugin?("vagrant-cachier")
        config.cache.synced_folder_opts = {
            owner: "_apt",
            group: "_apt",
            mount_options: ["dmode=777", "fmode=666"]
        }
        config.cache.scope = :box
    end

    # setting up master host
    config.vm.define "k8smaster", primary: true do |master|
      master.vm.hostname = "k8smaster"
      master.vm.network "private_network", ip: "192.168.7.10"
      config.vm.synced_folder ".", "/vagrant"
      master.vm.box_check_update = true
      master.vm.provider :virtualbox do |vb|
        vb.name = "k8smaster"
        vb.gui = true
        vb.customize ["modifyvm", :id, "--natnet1", "192.3/16"]
        vb.customize ["modifyvm", :id, "--memory", "2048"]
        vb.customize ["modifyvm", :id, "--cpus", "2"]
      end
      master.vm.provision "ansible" do |ansible|
          ansible.playbook = "./k8s-" + RUNTIME + "/playbook.yml"
          ansible.extra_vars = {
            node_ip: "192.168.7.10",
        }
      end
    end

    # setting up the nodes hosts
    (1..NUM_NODES).each do |node_number|

        node_name = "k8snode#{node_number}"

        config.vm.define node_name do |node|
            node.vm.hostname = node_name
            counter = 10 + node_number
            node_ip = "192.168.7.#{counter}"
            node.vm.network "private_network", ip: node_ip
            node.vm.provider :virtualbox do |vb|
               vb.name = node_name
               vb.gui = true
               vb.customize ["modifyvm", :id, "--natnet1", "192.#{counter}/16"]
               vb.customize ["modifyvm", :id, "--memory", "1024"]
               vb.customize ["modifyvm", :id, "--cpus", "2"]
            end
            node.vm.provision "ansible" do |ansible|
                ansible.playbook = "./k8s-" + RUNTIME + "/playbook-node.yml"
                ansible.extra_vars = {
                    node_ip: "192.168.7.#{counter}",
                }
            end
        end
    end
end