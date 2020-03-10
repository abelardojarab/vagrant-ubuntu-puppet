# -*- mode: ruby -*-
# vi: set ft=ruby :

# NOTE: vagrant-libvirt needs to run in series (not in parallel) to avoid
# trying to create the network twice... eg: vagrant up --no-parallel
# alternatively, you can just create the vm's one at a time manually...

domain = 'local'
box = 'generic/ubuntu1604'
cpus = 2
ram = 2000

node_components = [
  {:hostname => 'vm0',  :ip => '192.168.122.20', :box => box, :fwdhost => 2222, :fwdguest => 22, :cpus => cpus, :ram => ram},
  # {:hostname => 'vm1', :ip => '192.168.122.21', :box => box},
  # {:hostname => 'vm2', :ip => '192.168.122.22', :box => box},
]

Vagrant.configure("2") do |config|
  node_components.each do |node|
    config.vm.define node[:hostname] do |node_config|
      node_config.ssh.username = 'vagrant'
      node_config.ssh.password = 'vagrant'
      node_config.ssh.insert_key = true

      node_config.vm.box = node[:box]
      node_config.vm.hostname = node[:hostname] + '.' + domain

      node_config.vm.network :private_network,
                             :autostart => true,
                             ip: node[:ip],
                             libvirt__network_name: "default",
                             libvirt__forward_mode: 'nat',
                             libvirt__dhcp_enabled: true

      if node[:fwdhost]
        node_config.vm.network :forwarded_port, guest: node[:fwdguest], host: node[:fwdhost]
      end

      #node_config.vm.synced_folder "./www", "/var/www", create: true, group: "www-data", owner: "www-data"
      #node_config.vm.synced_folder ".", "/vagrant", type: 'rsync'

      memory = node[:ram] ? node[:ram] : ram;
      cpus = node[:cpus] ? node[:cpus] : cpus;
      node_config.vm.provider :libvirt do |libvirt, override|
        libvirt.driver = "kvm"

        # comment out host to connect directly with qemu:///system
        # libvirt.host = "localhost"

        # If use ssh tunnel to connect to Libvirt.
        libvirt.connect_via_ssh = false

        # The username and password to access Libvirt. Password is not used when
        # connecting via ssh.
        libvirt.username = "root"

        # Libvirt storage pool name
        libvirt.storage_pool_name = "default"

        # System configuration
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
                wget https://apt.puppetlabs.com/puppet5-release-xenial.deb
                sudo dpkg -i puppet5-release-xenial.deb
                apt-get update
                apt-get install -y puppet-agent
                ln -fs /opt/puppetlabs/bin/puppet /usr/sbin/puppet
                apt-get install zabbix-agent openconnect nfs-common nfs-kernel-server
          fi
      SHELL

      # Custom setup script
      node_config.vm.provision :shell do |s|
        s.path = "provision/setup.sh"
      end

      # # Puppet provisioning
      # node_config.vm.provision :puppet do |puppet|
      #   puppet.manifests_path = 'provision/manifests'
      #   puppet.module_path = 'provision/modules'
      # end

    end
  end
end
