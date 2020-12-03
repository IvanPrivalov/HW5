# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME']
ENV["LC_ALL"] = "en_US.UTF-8"

MACHINES = {
  :server => {
        :box_name => "centos/8.2",
#        :box_version => "1804.02",
        :ip_addr => '10.0.0.41',
    :disks => {
        :sata1 => {
            :dfile => home + '/VirtualBox VMs/sata1.vdi',
            :size => 10240,
            :port => 1
        },
        :sata2 => {
            :dfile => home + '/VirtualBox VMs/sata2.vdi',
            :size => 1024, # Megabytes
            :port => 2
        },
        :sata3 => {
            :dfile => home + '/VirtualBox VMs/sata3.vdi',
            :size => 1024, # Megabytes
            :port => 3
        },
        :sata4 => {
            :dfile => home + '/VirtualBox VMs/sata4.vdi',
            :size => 1024,
            :port => 4
        },
		:sata5 => {
            :dfile => home + '/VirtualBox VMs/sata5.vdi',
            :size => 1024,
            :port => 5
        },
		:sata6 => {
            :dfile => home + '/VirtualBox VMs/sata6.vdi',
            :size => 1024,
            :port => 6
        },
		:sata7 => {
            :dfile => home + '/VirtualBox VMs/sata7.vdi',
            :size => 1024,
            :port => 7
        }
    }
  },
}

Vagrant.configure("2") do |config|

#    config.vm.box_version = "1804.02"
    MACHINES.each do |boxname, boxconfig|
  
        config.vm.define boxname do |server|
		config.vm.box = 'centos/8.2'
		config.vm.box_url = 'https://cloud.centos.org/centos/8/x86_64/images/CentOS-8-Vagrant-8.2.2004-20200611.2.x86_64.vagrant-virtualbox.box'
		config.vm.box_download_checksum = '698b0d9c6c3f31a4fd1c655196a5f7fc224434112753ab6cb3218493a86202de'
		config.vm.box_download_checksum_type = 'sha256'	
            
			server.vm.box = boxconfig[:box_name]
            server.vm.host_name = boxname.to_s
  
            #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset
  
            server.vm.network "private_network", ip: boxconfig[:ip_addr]
  
            server.vm.provider :virtualbox do |vb|
                    vb.customize ["modifyvm", :id, "--memory", "256"]
                    needsController = false
            boxconfig[:disks].each do |dname, dconf|
                unless File.exist?(dconf[:dfile])
                  vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
                                  needsController =  true
                            end
  
            end
                    if needsController == true
                       vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
                       boxconfig[:disks].each do |dname, dconf|
                           vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
                       end
                    end
            end
  
			server.vm.provision "shell",
				name: "Setup zfs",
				path: "setup_zfs.sh"
  
        end
    end
		config.vm.define "client" do |client|
			client.vm.box = 'centos/8.2'
			client.vm.host_name = 'client'
			client.vm.network :private_network, ip: "10.0.0.40"
			client.vm.provider :virtualbox do |vb|
				vb.memory = "1024"
				vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
			end
		end
  end
  
