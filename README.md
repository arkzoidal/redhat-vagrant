# redhat-vagrant
## set up a vagrant redhat based box with local dnf repo ( iso image ) 

Create the following Vagrant file

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

add this option to the Vagrant file

**libvirt.storage :file, :device => :cdrom, :bus => :ide, :type => :raw, :path => "/home/vagrant/RACROC/provision/staging/rhel-8.5-x86_64-dvd.iso"**

add entry to /etc/fstab

/dev/cdrom    /media   udf,iso9660 user,auto,exec,utf8   0   0

Create Vagrant box

vagrant package --output arkzoidal_rhel8.box
vagrant box add arkzoidal_rhel8.box --name Arkzoidal/rhel8_ora

## Create Vagrant box from KVM

adduser vagrant
sudo visudo -f /etc/sudoers.d/vagrant
vagrant ALL=(ALL) NOPASSWD:ALL

dnf install -y openssh-server

mkdir -p /home/vagrant/.ssh
chmod 0700 /home/vagrant/.ssh
wget --no-check-certificate \
https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub \
-O /home/vagrant/.ssh/authorized_keys
chmod 0600 /home/vagrant/.ssh/authorized_keys
chown -R vagrant /home/vagrant/.ssh

vi /etc/ssh/sshd_config 
PubKeyAuthentication yes
AuthorizedKeysFile %h/.ssh/authorized_keys
PermitEmptyPasswords no
PasswordAuthentication no

service ssh restart

shutdown the VM 



/var/lib/libvirt/images/
cp /var/lib/libvirt/images/test.img  /test 

create two file metadata.json and Vagrantfile in /test do entry in metadata.json

{
  "provider"     : "libvirt",
  "format"       : "qcow2",
  "virtual_size" : 40
}


Vagrant.configure("2") do |config|
         config.vm.provider :libvirt do |libvirt|
         libvirt.driver = "kvm"
         libvirt.host = 'localhost'
         libvirt.uri = 'qemu:///system'
         end
config.vm.define "new" do |custombox|
         custombox.vm.box = "custombox"       
         custombox.vm.provider :libvirt do |test|
         test.memory = 1024
         test.cpus = 1
         end
         end
end

qemu-img convert -f raw -O qcow2  test.img  ubuntu.qcow2

mv ubuntu.qcow2 box.img 

tar cvzf custom_box.box ./metadata.json ./Vagrantfile ./box.img 

vagrant box add --name custom custom_box.box
