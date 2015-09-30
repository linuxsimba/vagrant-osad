# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "trusty64"
  # vagrant issues #1673..fixes hang with configure_networks
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"
  config.vm.provider :libvirt do |domain|
    domain.memory = 256
    domain.nested = true
  end

  # deployment server
  config.vm.define :deployserver do |node|
    node.vm.hostname = 'deployosad'
    node.vm.synced_folder '.', '/vagrant', :disabled => true
    node.vm.network :private_network,
      :ip => '169.2.2.2/24', # bogus IP so tha vagrant-libvirt can create virt_network
      :auto_config => false,
      :libvirt__forward_mode => 'veryisolated',
      :libvirt__dhcp_enabled => false,
      :libvirt__network_name => 'host_mgmt'
    node.vm.network :private_network,
      :ip => '169.2.3.3/24',
      :auto_config => false,
      :libvirt__forward_mode => 'veryisolated',
      :libvirt__dhcp_enabled => false,
      :libvirt__network_name => 'container_mgmt'
    node.vm.provision :ansible do |ansible|
      ansible.playbook = 'deployserver.yml'
      ansible.extra_vars = {
        openstack_release: 'kilo',
        ubuntu_repo: 'http://192.168.50.1:3142/ubuntu'
      }
    end
    node.vm.network "forwarded_port", guest: 3142, host: 8090
  end

  # Repo servers
  config.vm.define :reposerver do |node|
    node.vm.hostname = 'reposerver'
    node.vm.synced_folder '.', '/vagrant', :disabled => true
    node.vm.network :private_network,
      :auto_config => false,
      :libvirt__forward_mode => 'veryisolated',
      :libvirt__dhcp_enabled => false,
      :libvirt__network_name => 'host_mgmt'
    node.vm.network :private_network,
      :auto_config => false,
      :libvirt__forward_mode => 'veryisolated',
      :libvirt__dhcp_enabled => false,
      :libvirt__network_name => 'container_mgmt'
    node.vm.provision :ansible do |ansible|
      ansible.playbook = 'reposerver.yml'
      ansible.extra_vars = {
        apt_url: 'http://192.168.50.1:3142',
        openstack_release: 'kilo'
      }
    end
  end


  # LoadBalancing VM
  config.vm.define :haproxy do |node|
    node.vm.hostname = 'haproxy'
    node.vm.synced_folder '.', '/vagrant', :disabled => true
    node.vm.network :private_network,
      :auto_config => false,
      :libvirt__forward_mode => 'veryisolated',
      :libvirt__dhcp_enabled => false,
      :libvirt__network_name => 'host_mgmt'
    node.vm.network :private_network,
      :auto_config => false,
      :libvirt__forward_mode => 'veryisolated',
      :libvirt__dhcp_enabled => false,
      :libvirt__network_name => 'container_mgmt'
    node.vm.provision :ansible do |ansible|
      ansible.playbook = 'haproxy.yml'
      ansible.extra_vars = {
        apt_url: 'http://192.168.50.1:3142',
        openstack_release: 'kilo'
      }
    end
    node.vm.network "forwarded_port", guest: 443, host: 8081
  end

  # OSAD Server
  # Minimum requirement is 4GB of RAM and 2 CPU for an install to finish
  # in a reasonable about of time. About 3 hours from scratch.
  # TODO make memory or cpus a vagrant variable and disk variable
  config.vm.define :stackserver do |node|
    node.vm.provider :libvirt do |domain|
      domain.memory = 4096
      domain.cpus = 2
      # for cinder block services whenever i get to it.
      domain.storage :file, :size => '40G'
    end
    node.vm.hostname = 'stackserver'
    node.vm.synced_folder '.', '/vagrant', :disabled => true
    # eth1
    node.vm.network :private_network,
      :auto_config => false,
      :libvirt__forward_mode => 'veryisolated',
      :libvirt__dhcp_enabled => false,
      :libvirt__network_name => 'host_mgmt'
    # eth2
    node.vm.network :private_network,
      :auto_config => false,
      :libvirt__forward_mode => 'veryisolated',
      :libvirt__dhcp_enabled => false,
      :libvirt__network_name => 'container_mgmt'
    # eth3
    node.vm.network :private_network,
      :ip => '169.3.4.20/24', # bogus Ip so it can create libvirt network
      :auto_config => false,
      :libvirt__forward_mode => 'veryisolated',
      :libvirt__dhcp_enabled => false,
      :libvirt__network_name => 'vxlan_tunnel'
    # eth4
    node.vm.network :private_network,
      :ip => '169.2.4.10/24', # bogus IP so it can create libvirt network
      :auto_config => false,
      :libvirt__forward_mode => 'veryisolated',
      :libvirt__dhcp_enabled => false,
      :libvirt__network_name => 'for_tenants'
    node.vm.provision :ansible do |ansible|
      ansible.playbook = 'stackserver.yml'
      ansible.extra_vars = {
        apt_url: 'http://192.168.50.1:3142',
        openstack_release: 'kilo'
      }
    end
  end

  # OSAD Server
  # in a reasonable about of time. About 3 hours from scratch.
  # TODO make memory or cpus a vagrant variable and disk variable
  config.vm.define :stackserver2 do |node|
    node.vm.provider :libvirt do |domain|
      domain.memory = 2048
      domain.cpus = 2
      # for cinder block services whenever i get to it.
      domain.storage :file, :size => '40G'
    end
    node.vm.hostname = 'stackserver2'
    node.vm.synced_folder '.', '/vagrant', :disabled => true
    #eth1
    node.vm.network :private_network,
      :auto_config => false,
      :libvirt__forward_mode => 'veryisolated',
      :libvirt__dhcp_enabled => false,
      :libvirt__network_name => 'host_mgmt'
    #eth2
    node.vm.network :private_network,
      :auto_config => false,
      :libvirt__forward_mode => 'veryisolated',
      :libvirt__dhcp_enabled => false,
      :libvirt__network_name => 'container_mgmt'
    #eth3
    node.vm.network :private_network,
      :auto_config => false,
      :libvirt__forward_mode => 'veryisolated',
      :libvirt__dhcp_enabled => false,
      :libvirt__network_name => 'vxlan_tunnel'
    #eth4
    node.vm.network :private_network,
      :auto_config => false,
      :libvirt__forward_mode => 'veryisolated',
      :libvirt__dhcp_enabled => false,
      :libvirt__network_name => 'for_tenants'
    node.vm.provision :ansible do |ansible|
      ansible.playbook = 'stackserver.yml'
      ansible.extra_vars = {
        apt_url: 'http://192.168.50.1:3142',
        openstack_release: 'kilo'
      }
    end
  end


  ## Tenant router
  config.vm.define :tenantrouter do |node|
    node.vm.provider :libvirt do |domain|
      domain.memory = 128
    end
    node.vm.synced_folder '.', '/vagrant', :disabled => true
    node.vm.hostname = 'tenantrouter'
    node.vm.network :private_network,
      :auto_config => false,
      :libvirt__forward_mode => 'veryisolated',
      :libvirt__dhcp_enabled => false,
      :libvirt__network_name => 'host_mgmt'
    node.vm.network :private_network,
      :ip => '169.2.4.10/24',
      :auto_config => false,
      :libvirt__forward_mode => 'veryisolated',
      :libvirt__dhcp_enabled => false,
      :libvirt__network_name => 'for_tenants'
    node.vm.provision :ansible do |ansible|
      ansible.playbook = 'tenantrouter.yml'
      ansible.extra_vars = {
        apt_url: 'http://192.168.50.1:3142',
        openstack_release: 'kilo'
      }
    end
  end
end
