# Author: Edwin Heerschap
# Builds an alpine linux distribution for the purpose of kernel module development.

# Returns true if provisioned, false otherwise.
# Approach taken from https://stackoverflow.com/questions/24855635/check-if-vagrant-provisioning-has-been-done
def provisioned?(vm_name='default', provider='virtualbox')
  return File.exist?(".vagrant/machines/#{vm_name}/#{provider}/action_provision")
end

Vagrant.configure("2") do |config|
   
  ##########################################
  ## General settings
  ##########################################

    sshKeyName="VirtualHideoutVMKey"    

    ########################################
    ## Processors and memory setting.
    ########################################

    vmMemoryMb = 2048
    vmCoreCount = 4

  if provisioned?
    ########################################
    ## SYNC FOLDERS HERE
    ########################################
    #config.vm.synced_folder "/home/user/", "/home/virtualhideout/synced"
    config.vm.synced_folder "/home/edwin/Documents/dev/SneakyHideout/VirtualHideout/devdrivers/", "/home/virtualhideout/synced"
  end

    #######################################
    ## VM 1st Adapter MAC Address
    #######################################
    config.vm.base_mac="0800279AD647"

  ##########################################
  ## Below shouldn't be touched - Sensitive settings
  ##########################################

  config.vm.box="VirtualHideoutBootstrap.box"   
  config.ssh.shell="ash"

  config.vm.provider "virtualbox" do |v|
    v.name = "VirtualHideoutVM"
  end

  config.vm.provider "virtualbox" do |v|
    # If changing the number of processors, any
    # make commands -j parameter will need to
    # be changed

    v.memory = vmMemoryMb
    v.cpus = vmCoreCount
  end

  # Directroy of Vagrantfile
  dirname = File.dirname(__FILE__)

  if provisioned?
    config.ssh.username="virtualhideout"
    config.ssh.private_key_path="#{dirname}/#{sshKeyName}" 
  else
    config.ssh.username="virtualhideout"
    config.ssh.password="virtual"
  end

  #######################################################################
  ## PROVISIONING CODE. ADVISED TO NOT EDIT.
  #######################################################################
  
  # Stops default mount. BootStrap box does not have required libraries for folder sharing.
  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.provision :host_shell do |host_shell|
    host_shell.inline = 'if [ -f ' + dirname + '/' + sshKeyName + ' ]; then rm ' + dirname + "/" + sshKeyName + ' && rm ' + dirname + "/" + sshKeyName + '.pub; fi; ssh-keygen -f ' + dirname + "/" + sshKeyName + ' -P ""'
  end

   config.vm.provision "shell", inline: <<-SHELL
      sudo su - root
      echo -e 'http://dl-cdn.alpinelinux.org/alpine/v3.11/main\nhttp://dl-cdn.alpinelinux.org/alpine/v3.11/main\nhttp://dl-cdn.alpinelinux.org/alpine/v3.11/community' >> /etc/apk/repositories	
      apk update && apk upgrade
      apk add alpine-sdk linux-headers virtualbox-guest-additions virtualbox-guest-modules-virt sudo linux-virt-dev
   SHELL
  
  # Reloading for correct install of virtualbox-guest-addtions & virtualbox-guest-modules-virt
  config.vm.provision :reload

  # Adding Vagrant file directory for SSH key installation
  config.vm.provision :host_shell do |host_shell|
      host_shell.inline = 'VBoxManage sharedfolder add VirtualHideoutVM --name sshkeys --hostpath "' + dirname + '" --transient'    
  end
  
  config.vm.provision "shell", inline: <<-SHELL
     sudo modprobe -a vboxsf
     mkdir /home/virtualhideout/sshkeys
     mount -t vboxsf sshkeys /home/virtualhideout/sshkeys
     touch /home/virtualhideout/.ssh/authorized_keys
     chmod 0700 /home/virtualhideout/.ssh && chmod 0600 /home/virtualhideout/.ssh/authorized_keys
  SHELL
  
  config.vm.provision "shell", inline: 'cat /home/virtualhideout/sshkeys/' + sshKeyName + '.pub >> /home/virtualhideout/.ssh/authorized_keys'

  config.vm.provision "shell", inline: <<-SHELL
     sudo setup-xorg-base xfce4 xfce4-terminal dbus-x11 lightdm-gtk-greeter xf86-input-mouse xf86-input-keyboard xf86-video-vmware
     sudo rc-service dbus start
     sudo rc-update add dbus
     cd /home/virtualhideout/
     ln -s /lib/modules/5.4.34-0-virt/build/ build
  SHELL
     
   config.vm.provision "shell", inline: <<-SHELL
     sudo umount /home/virtualhideout/sshkeys
     rmdir /home/virtualhideout/sshkeys
  SHELL

  config.vm.provision "shell", inline: <<-SHELL
    modprobe -a vboxsf
    echo "\033[1;32m\u001b[40m Provisioning completed and powering down VM.  Run 'vagrant up' again for the fully provisioned machine.\033[0m\n"   
    sudo poweroff
  SHELL

end

