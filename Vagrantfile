# -*- mode: ruby -*-
# vi: set ft=ruby :

# NOTE: vagrant-libvirt needs to run in series (not in parallel) to avoid
# trying to create the network twice... eg: vagrant up --no-parallel
# alternatively, you can just create the vm's one at a time manually...

domain = 'example.com'
box = 'ubuntu/xenial64'
ram = 512

node_components = [
  {:hostname => 'puppet',  :ip => '172.16.32.10', :box => box, :fwdhost => 8140, :fwdguest => 8140, :ram => ram},
  # {:hostname => 'client1', :ip => '172.16.32.11', :box => box},
  # {:hostname => 'client2', :ip => '172.16.32.12', :box => box},
]

Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  node_components.each do |node|
    config.vm.define node[:hostname] do |node_config|
      node_config.vm.box = node[:box]
      # node_config.vm.box_url = 'https://atlas.hashicorp.com/' + node_config.vm.box
      node_config.vm.hostname = node[:hostname] # + '.' + domain

      # node_config.vm.network :public_network, :dev => "virbr0", :mode => 'bridge'
      node_config.vm.network :private_network, ip: node[:ip]
      # if node[:fwdhost]
      #   node_config.vm.network :forwarded_port, guest: node[:fwdguest], host: node[:fwdhost]
      # end

      memory = node[:ram] ? node[:ram] : 256;
      cpus = node[:cpus] ? node[:cpus] : 4;
      node_config.vm.provider :libvirt do |libvirt, override|
        libvirt.driver = "qemu"
        # leave out host to connect directly with qemu:///system
        #libvirt.host = "localhost"

        libvirt.connect_via_ssh = false     # also needed
        libvirt.username = "root"
        libvirt.storage_pool_name = "default"
        libvirt.cpus = cpus.to_s
        libvirt.memory = memory.to_s
        libvirt.nested = true

        # Only available with KVM
        # libvirt.cpu_mode = "host-passthrough"

        # override.vm.box = node[:image]
        # override.nfs.functional = node[:nfs]
      end

      node_config.vm.provision :puppet do |puppet|
        puppet.manifests_path = 'provision/manifests'
        puppet.module_path = 'provision/modules'
      end
    end
  end
end
