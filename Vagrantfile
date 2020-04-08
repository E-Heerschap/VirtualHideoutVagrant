

def provisioned?(vm_name='default', provider='virtualbox')
  File.exist?(".vagrant/machines/${vm_name}/#{provider}/action_provision")
end

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.

  config.vm.synced_folder ".", "/vagrant", disabled: true
  
  if provisioned?
    config.ssh.username="virtualhideout"
    config.ssh.private_Key_path="VirtualHideoutVM-private-key"
  else
    config.ssh.username="virtualhideout"
    config.ssh.password="virtual"
    config.vm.box="VirtualHideoutBootstrap.box"   
  end

  #config.vm.base_mac = "08002775A476"
  
  config.ssh.shell="ash"
  
  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  config.vm.provider "virtualbox" do |v|
    v.name = "VirtualHideoutVM"
  end

   config.vm.provision "shell", inline: <<-SHELL
      sudo su - root
      echo -e 'http://dl-cdn.alpinelinux.org/alpine/v3.11/main\nhttp://dl-cdn.alpinelinux.org/alpine/edge/main\nhttp://dl-cdn.alpinelinux.org/alpine/edge/community\nhttp://dl-cdn.alpinelinux.org/alpine/edge/testing' >> /etc/apk/repositories	
      apk update && apk upgrade
      apk add alpine-sdk linux-headers virtualbox-guest-additions virtualbox-guest-modules-virt sudo
      mkdir /home/virtualhideout/mnt
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
     echo "rc_need=udev-settle" > /etc/conf.d/networking && lbu ci
  SHELL
  
   config.vm.provision :reload

end
