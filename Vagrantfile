# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :dz => {
        :box_name => "generic/ubuntu2204",
        :vm_name => "dz",
        :net => [
           ["192.168.11.150",  2, "255.255.255.0", "mynet"],
        ]
  }
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|
   
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxconfig[:vm_name]
      
      box.vm.provider "virtualbox" do |v|
        v.memory = 1024
        v.cpus = 1
       end

	  box.vm.disk :disk, size: "1GB", name: "disk1"
	  box.vm.disk :disk, size: "1GB", name: "disk2"
	  
	  box.vm.network(:forwarded_port,
                    guest: 80,
                    host: 8080,
                    host_ip: "127.0.0.1")

      box.vm.provision "shell", inline: <<-SHELL
        mkdir -p ~root/.ssh
        cp ~vagrant/.ssh/auth* ~root/.ssh
        sudo sed -i 's/\#PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
        systemctl restart sshd
		
		DISK1_PATH=$(ls /dev/disk/by-id/*-disk1 | head -n 1)
		DISK2_PATH=$(ls /dev/disk/by-id/*-disk2 | head -n 1)
		TARGET_DISKS=("/dev/sdb" "/dev/sdc")
		MOUNT_POINTS=("/mnt/disk1" "/mnt/disk2")
		for i in 0 1; do
			DISK=${TARGET_DISKS[$i]}
			MNT=${MOUNT_POINTS[$i]}
			mkdir -p "$MNT"
			mkfs.ext4 -F "$DISK"
			UUID=$(blkid -o value -s UUID "$DISK")
			echo "UUID=$UUID $MNT ext4 defaults,nofail 0 2" >> /etc/fstab
			mount "$MNT"
		done
      SHELL
    end
  end
end