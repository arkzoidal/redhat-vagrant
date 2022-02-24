# redhat-vagrant
set up a vagrant redhat based box with local dnf repo ( iso image ) 

Create the following Vagrant file

# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
    config.vm.define "rhel8" do | rb |
    end

    config.vm.provider :libvirt do |libvirt|
        libvirt.default_prefix = ""
    end

    config.vm.box = "generic/rhel8"
    config.vm.hostname = "rhel8"
    config.ssh.insert_key = true
end

Create a local yum repository

 vi /etc/yum.repos.d/rhel8.repo     

[InstallMedia-BaseOS]                                                        
name=Red Hat Enterprise Linux 8 - BaseOS                                     
metadata_expire=-1                                                           
gpgcheck=0                                                                   
enabled=1                                                                    
baseurl=file:///media/BaseOS/                                                

[InstallMedia-AppStream]                                                     
name=Red Hat Enterprise Linux 8 - AppStream                                  
metadata_expire=-1                                                           
gpgcheck=0                                                                   
enabled=1                                                                    
baseurl=file:///media/AppStream/        

mount /dev/cdrom /media   

on virsh side mount cdrom permanently

virsh attach-disk rhel8  /home/vagrant/RACROC/provision/staging/rhel-8.5-x86_64-dvd.iso hdc --type cdrom --mode readonly --persistent

Create Vagrant box

vagrant package --output arkzoidal_rhel8.box


