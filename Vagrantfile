# Author: Edwin Heerschap


# Returns true if provisioned, false otherwise.
# Approach taken from https://stackoverflow.com/questions/24855635/check-if-vagrant-provisioning-has-been-done
def provisioned?(vm_name='default', provider='virtualbox')
  return File.exist?(".vagrant/machines/#{vm_name}/#{provider}/action_provision")
end


Vagrant.configure("2") do |config|
  
  if provisioned?

    ########################################
    ## CREDENTIALS HERE
    ########################################
    config.ssh.username="virtualhideout"
    config.ssh.private_key_path="VirtualHideoutVM-private-key"
    
    ########################################
    ## SYNC FOLDERS HERE
    ########################################
    #config.vm.synced_folder "/home/user/", "/home/virtualhideout/synced"
  end
  
  config.vm.box="VirtualHideoutBootstrap.box"   
  config.ssh.shell="ash"

  config.vm.provider "virtualbox" do |v|
    v.name = "VirtualHideoutVM"
  end


  #######################################################################
  ## PROVISIONING CODE. ADVISED TO NOT EDIT.
  #######################################################################
  
  # Stops default mount. BootStrap box does not have required libraries for folder sharing.
  config.vm.synced_folder ".", "/vagrant", disabled: true

  if !provisioned?
    config.ssh.username="virtualhideout"
    config.ssh.password="virtual"
  end

   config.vm.provision "shell", inline: <<-SHELL
      sudo su - root
      echo -e 'http://dl-cdn.alpinelinux.org/alpine/v3.11/main\nhttp://dl-cdn.alpinelinux.org/alpine/edge/main\nhttp://dl-cdn.alpinelinux.org/alpine/edge/community\nhttp://dl-cdn.alpinelinux.org/alpine/edge/testing' >> /etc/apk/repositories	
      apk update && apk upgrade
      apk add alpine-sdk linux-headers virtualbox-guest-additions virtualbox-guest-modules-virt sudo
   SHELL
  
  # Reloading for correct install of virtualbox-guest-addtions & virtualbox-guest-modules-virt
  config.vm.provision :reload

    # Adding Vagrant file directory for SSH key installation
    config.vm.provision :host_shell do |host_shell|
      dirname = File.dirname(__FILE__)
      host_shell.inline = 'VBoxManage sharedfolder add VirtualHideoutVM --name sshkeys --hostpath "' + dirname + '" --transient'    
    end

  config.vm.provision "shell", inline: <<-SHELL
     sudo modprobe -a vboxsf
     mkdir /home/virtualhideout/sshkeys
     mount -t vboxsf sshkeys /home/virtualhideout/sshkeys
     touch /home/virtualhideout/.ssh/authorized_keys
     chmod 0700 /home/virtualhideout/.ssh && chmod 0600 /home/virtualhideout/.ssh/authorized_keys
     cat /home/virtualhideout/sshkeys/VirtualHideoutVM-public-key >> /home/virtualhideout/.ssh/authorized_keys
     sudo setup-xorg-base xf86-video-vboxvideo xfce4 xfce4-terminal dbus-x11 lightdm-gtk-greeter xf86-input-mouse xf86-input-keyboard xf86-video-vboxvideo
     sudo rc-service dbus start
     sudo rc-update add dbus
     cd /home/virtualhideout/
     wget -O /home/virtualhideout/linux-5.4.30.tar.xz https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/linux-5.4.30.tar.xz
     tar -xf linux-5.4.30.tar.xz && rm linux-5.4.30.tar.xz
     sudo chown virtualhideout linux-5.4.30
     sudo apk add openssl openssl-dev bison flex elfutils elfutils-dev ncurses ncurses-dev
     mkdir build
     sudo cp /boot/config-virt /home/virtualhideout/build/.config
     make -j4 -C /home/virtualhideout/linux-5.4.30 O=/home/virtualhideout/build
     sudo echo "rc_need=udev-settle" > /etc/conf.d/networking && lbu ci
     sudo umount /home/virtualhideout/sshkeys
     rmdir /home/virtualhideout/sshkeys
  SHELL

  config.vm.provision "shell", inline: <<-SHELL
    echo "\033[1;32m\u001b[40m The error message is intentional. It complains because the machine is being powered down. Run 'vagrant up' again for the fully provisioned machine.\033[0m\n"   
    sudo poweroff
  SHELL

end
