# -*- mode: ruby -*-
# vi: set ft=ruby :

# NOTE: vagrant-libvirt needs to run in series (not in parallel) to avoid
# trying to create the network twice... eg: vagrant up --no-parallel
# alternatively, you can just create the vm's one at a time manually...

domain = 'local'
box = 'mkutsevol/xenial'
cpus = 2
ram = 1024

node_components = [
  {:hostname => 'vm0',  :ip => '172.16.32.10', :box => box, :fwdhost => 8140, :fwdguest => 8140, :cpus => cpus, :ram => ram},
  # {:hostname => 'vm1', :ip => '172.16.32.11', :box => box},
  # {:hostname => 'vm2', :ip => '172.16.32.12', :box => box},
]

Vagrant.configure("2") do |config|
  node_components.each do |node|
    config.vm.define node[:hostname] do |node_config|
      node_config.ssh.username = 'root'
      node_config.ssh.password = "vagrant"

      node_config.vm.box = node[:box]
      node_config.vm.hostname = node[:hostname] + '.' + domain

      node_config.vm.network :public_network

      node_config.vm.network :private_network,
                             ip: node[:ip],
                             libvirt__forward_mode: 'nat',
                             libvirt__dhcp_enabled: true
      if node[:fwdhost]
        node_config.vm.network :forwarded_port, guest: node[:fwdguest], host: node[:fwdhost]
      end

      node_config.vm.synced_folder "./www", "/var/www", create: true, group: "www-data", owner: "www-data"
      node_config.vm.synced_folder ".", "/vagrant", type: 'rsync'

      memory = node[:ram] ? node[:ram] : 256;
      cpus = node[:cpus] ? node[:cpus] : 4;
      node_config.vm.provider :libvirt do |libvirt, override|
        libvirt.driver = "kvm"

        # comment out host to connect directly with qemu:///system
        # libvirt.host = "localhost"
        libvirt.connect_via_ssh = false     # also needed
        libvirt.username = "abelardojara"

        # System configuration
        libvirt.storage_pool_name = "default"
        libvirt.cpus = cpus.to_s
        libvirt.memory = memory.to_i
        libvirt.nested = true

        # Only available with KVM
        libvirt.cpu_mode = "host-passthrough"
      end

      # Update puppet
      node_config.vm.provision "shell", inline: <<-SHELL
          if [ ! -f /usr/sbin/puppet ]; then
                cd /tmp
                wget http://apt.puppetlabs.com/puppetlabs-release-pc1-trusty.deb
                dpkg -i /tmp/puppetlabs-release-pc1-trusty.deb
                apt-get update
                apt-get install -y puppet-agent
                ln -fs /opt/puppetlabs/bin/puppet /usr/sbin/puppet
          fi
      SHELL

      # Custom setup script
      node_config.vm.provision :shell do |s|
        s.path = "provision/setup.sh"
      end

      # node_config.vm.provision :puppet do |puppet|
      #   puppet.manifests_path = 'provision/manifests'
      #   puppet.module_path = 'provision/modules'
      # end

    end
  end
end
