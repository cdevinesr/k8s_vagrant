# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  node_count = 3

  config.vm.define "c1-master1", primary: true do |master|
    if Vagrant.has_plugin?("vagrant-vbguest")
      master.vbguest.auto_update = false
    end

    master.vm.hostname = "c1-master1"
    master.vm.box = "ubuntu/bionic64"
    master.vm.network "private_network", ip: "172.16.94.10"
    master.vm.network "forwarded_port", guest: 22, host: 2222, host_ip: "0.0.0.0", id: "ssh", auto_correct: true

    master.vm.provider "virtualbox" do |vb|
      vb.customize [ 'modifyvm', :id, '--nictype1', 'virtio' ]
      vb.name = "Kubernetes Cluster 1 Master 1"
      vb.memory = 2048
      vb.cpus = 2
    end

    master.vm.provision "shell", inline: <<-SHELL
      mkdir -p /etc/ansible
      echo $'[local]\\nlocalhost ansible_connection=local\\n' > /etc/ansible/hosts
    SHELL

    master.vm.provision "ansible_local" do |ansible|
      ansible.become = true
      ansible.compatibility_mode = "2.0"
      ansible.install_mode = "pip"
      ansible.inventory_path = "/etc/ansible/hosts"
      ansible.limit = "local"
      ansible.playbook = "ansible/k8s.yml"
      ansible.extra_vars = {
        ansible_python_interpreter: "/usr/bin/python3",
        k8s_master: true,
        node_ip: "172.16.94.10"
      }
    end
  end

  (1..node_count).each do |i|
    config.vm.define "c1-node#{i}" do |node|
      if Vagrant.has_plugin?("vagrant-vbguest")
        node.vbguest.auto_update = false
      end

      node.vm.hostname = "c1-node#{i}"
      node.vm.box = "ubuntu/bionic64"
      last_octet = i + 10
      node.vm.network "private_network", ip: "172.16.94.#{last_octet}"

      node.vm.provider "virtualbox" do |vb|
        vb.customize [ 'modifyvm', :id, '--nictype1', 'virtio' ]
        vb.name = "Kubernetes Cluster 1 Node #{i}"
        vb.memory = 2048
        vb.cpus = 2
      end

      node.vm.provision "shell", inline: <<-SHELL
        mkdir -p /etc/ansible
        echo $'[local]\\nlocalhost ansible_connection=local\\n' > /etc/ansible/hosts
      SHELL

      node.vm.provision "ansible_local" do |ansible|
        ansible.become = true
        ansible.compatibility_mode = "2.0"
        ansible.install_mode = "pip"
        ansible.inventory_path = "/etc/ansible/hosts"
        ansible.limit = "local"
        ansible.playbook = "ansible/k8s.yml"
        ansible.extra_vars = {
          ansible_python_interpreter: "/usr/bin/python3",
          k8s_master: false,
          node_ip: "172.16.94.#{last_octet}"
        }
      end
    end
  end
end
