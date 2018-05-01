# -*- mode: ruby -*-
# vi: set ft=ruby ts=2 sw=2 tw=0 et :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

# cluster configure
boxes = {
  "arch"      => { :box => "archlinux/archlinux",      :cpu => "1", :ram => "1024" },
}

Vagrant.configure("2") do |config|
  #if Vagrant.has_plugin?("vagrant-cachier")
  #  config.cache.scope = :box
  #  config.cache.auto_detect = true
  #  config.cache.synced_folder_opts = {
  #    type: :nfs,
  #    mount_options: ['rw', 'vers=3', 'tcp', 'nolock']
  #  }
  #end
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.no_remote   = true
    config.vbguest.auto_update = false
  end
  if Vagrant.has_plugin?('vagrant-registration')
    config.registration.username = ENV['RHELuser']
    config.registration.password = ENV['RHELpass']
  end
  if Vagrant.has_plugin?('vagrant-proxyconf')
    config.proxy.http           = ENV['http_proxy']
    config.proxy.no_proxy       = ENV['no_proxy']
    config.proxy.enabled        = { svn: false, docker: false, git: false }
  end
  # box
  config.vm.box_check_update  = false
  # ssh
  config.ssh.insert_key       = false
  config.ssh.forward_agent    = true
  # synced folders
  config.vm.synced_folder ".", "/vagrant", disabled: true

  boxes.each_with_index do |(hostname, info), index|
    config.vm.define hostname do |cfg|
      cfg.vm.box     = info[:box]
      cfg.vm.box_url = info[:url]

      # virtualbox
      cfg.vm.provider :virtualbox do |vb|
        vb.name   = "#{hostname}"
        vb.cpus   = info[:cpu]
        vb.memory = info[:ram]
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      end

      # network
      #cfg.vm.network :private_network, ip: "#{info[:ip]}", nictype: "virtio"
      cfg.vm.network :private_network, type: :dhcp

      # provision
      if index == boxes.size - 1
        cfg.vm.provision :shell do |sh|
          sh.path = 'bootstrap.sh'
        end
        cfg.vm.provision :ansible do |ansible|
          ansible.compatibility_mode = "2.0"
          ansible.playbook           = "tests/main.yml"
          ansible.limit              = "all"
          ansible.extra_vars         = { http_proxy: ENV['http_proxy'] } 

          if File.file?("roles/requirements.yml") # download requirements if there ia a requirements file
            ansible.galaxy_role_file  = "roles/requirements.yml"
          end
          ansible.galaxy_roles_path  = ".roles"
          # ansible.verbose            = "vv"
          ansible.raw_arguments      = [
            "--diff",
          ]
        end
      end
    end
  end
end
