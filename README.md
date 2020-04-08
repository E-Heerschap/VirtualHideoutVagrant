# VirtualHideout Vagrant Documentation

This documenation contains information specific to building a virtual box vagrant box
capable of compiling and running the VirtualHideout kernel modules. This box is intended for developers and does not represent the minimum library requirements for using the VirtualHideout tools.

## Prerequisites
- Virtual Box
- Alpine Linux Virtual Distribution iso
- Package (HashiCorp utility for packaging virtual box)

## Steps to build box. (Prepackaged box available - skip to 'Setup the Vagrantfile')

1. Start Virtual Box and create an "Alpine Build" VM. This VM is not the packaged VM.
Instead, this VM is used to build/setup the alpine VM to be used. However, the disk size you select for the build VM is the disk size used for the primary VM. 
Start the Alpine Build VM with the Alpine Linux Virtual distribution iso from the prerequisites.

2. Login to the alpine build VM by typing "root". You will not be asked for a password.

3. Run `setup-alpine` and follow the steps. Leaving all settings default except for name is fine. Set the name to "Virtual Hideout". Set the root password to whatever you deem acceptable.

4. Run `poweroff` to shutdown the VM when prompted to reboot the system.

5. Create a new VM. This will be the primary VM. When prompted, select the virtual hard disk created for the Alpine Build VM. This contains the boot scripts to boot into alpine. If the Alpine Build VM is started again the installtion process is restarted.

6. Login to "root" with the password used in step 2.

7. Run `echo -e 'http://dl-cdn.alpinelinux.org/alpine/v3.11/main\nhttp://dl-cdn.alpinelinux.org/alpine/edge/main\nhttp://dl-cdn.alpinelinux.org/alpine/edge/community\nhttp://dl-cdn.alpinelinux.org/alpine/edge/testing' >> /etc/apk/repositories`. This adds the main, edge community and edge main alpine repositories.

8. Run `apk update && apk upgrade`

9. Run `apk add alpine-sdk linux-headers virtualbox-guest-additions virtualbox-guest-modules-virt sudo`. This installs the general tools used to build/compile kernel modules and the virtualbox kernel modules required to share directories with the host system. The virtual box system can be created from the ISO downloaded by their FTP server. However, running this on the standard alpine distribution resolves in errors. If this method is chosen, either recompile the kernels modules manually in a different system and install them manually (not recommended) or find the packages required to build the system in debains 'build-essential' and 'dkms' packages (recommended).

10. Reboot the system.

11. Run `modprobe -a vboxsf` (Starts the kernel modules required for file sharing between host os and VM).

12. Create a virtualhideout user using `adduser virtualhideout`.

13. Add the virtualhideout user to the sudoers list by running: `echo 'virtualhideout ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers`

14. Set the password for virtualhideout: `passwd virtualhideout`

15. Switch to the virtualhideout user `su - virtualhideout`

16. On the host OS run `ssh-keygen` and produce a public/private key pair.

17. Run `mkdir ~/.ssh && touch ~/.ssh/authorized_keys`.

18. Install SSH keys.  An easy way to do this is to mount the folder containing the public key in system by running `sudo mount -t vboxsf shared_vbox_folder /mnt/folder`. Not /mnt/folder needs to be created first on the VM and the shared_vbox_folder needs to be configured in virtualbox first under devices->shared folders->shared folders settings. Once the file system is mounted, run `cat /mnt/folder/publickeyfile.pub >> /home/VirtualHideout/.ssh/authorized_keys`.

19. OpenSSH requires specific permissions for these files. Run `chmod 0700 ~/.ssh && chmod 0600 ~/.ssh/authorized_keys`

20. To set up the xfce desktop install the following: `sudo setup-xorg-base xf86-video-vboxvideo xfce4 xfce4-terminal dbus-x11 lightdm-gtk-greeter xf86-input-mouse xf86-input-keyboard xf86-video-vboxvideo`. For the curious, xf86 are packages are x-11 drivers.

21. Run `sudo rc-service dbus start`. If this runs fine add it to the boot `rc-update add dbus`.

22. Start the lightdm desktop to test. `sudo rc-service lightdm start`. Sign in to the regular user.

23. Download the linux kernel associated with the current alpine linux kernel. Open the terminal (in tool bar) and run `wget https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/linux-5.4.30.tar.xz` (or with the current used kernel).

24. Untar the download `tar -xf linux-5.4.30.tar.xz && rm linux.5.4.30.tar.xz`

25. To build the linux kernel the following will need to be installed: `sudo apk add openssl openssl-dev bison flex elfutils elfutils-dev ncurses ncurses-dev`

26. Run `cd /home/virtualhideout/linux-5.4.30.tar.xz && 0=/home/virtualhideout/build/kernel menuconfig`. Leave the settings as default. Then run `make -j4 O=/home/virtualhideout/build/kernel`.

27. If the kernel has been build successfullym remove the folder. Run `cd ~ && rm -rf linux.5.4.30`

28. Make sure the first network adapter is type NAT. Goto devices->network->networksettings in virtualbox to check.

29. Note the MAC address of the first network adapter (under advanced). This is required for the vagrant file.

30. Shut down the VM

31. Run `vagrant package --base <VM NAME IN VIRTUAL BOX> --output VirtualHideoutVM.box`. If an old box has been added to vagrant run `vagrant box remove VirtualHideoutVM.box` before packaging the box to ensure a fresh box.

32. All done building the box :)

### Notes

* The user name virtualhideout is not required. Just convention.

* The MAC Address will likely not be required for the box to work. However, the official documentation (https://www.vagrantup.com/docs/virtualbox/boxes.html) mentions this as a required step.

* For extra security, it is recommended the public/private keys in the repository should not be used. Instead, generate your own as done in step 14.

## Setup the Vagrantfile

If the Vagrantfile is being setup directly after packaging a VirtualHideoutVM box then a VagrantFile should automatically be created for you. The settings below will need to be added.

If you are NOT packaging your own VirtualHideoutVM, run `vagrant box add -f VirtualHideoutVM.box` to add the box. Ensure the settings below are correct in your Vagrantfile pulled from the repository are correct.

The required config settings in the Vagrant file are:

`config.vm.box="VirtualHideoutVM.box"
config.ssh.username="virtualhideout"
config.ssh.shell="ash"
config.ssh.private_key_path="YOUR_PATH_TO_PRIVATE_KEY"`

## Prepackaged box credentials

* Public/private key pair for the prepackaged box are available in this repository.

* The general user with access to sudo has the credentials `username: virtualhideout password: virtual`

* The root user has the credentials `username: root password: virtualbuild`

## Trouble shooting:

* For SSH issues, the private key for the pre-packaged box is available in the repository.

* If network issues occur, try building your own box and noting the MAC address of the first network adapter and adding `config.vm.base_mac="MAC_ADDRESS_HERE"` line to the Vagrant file

## Useful resources

* https://www.vagrantup.com/docs/vagrantfile/ssh_settings.html

* https://www.vagrantup.com/docs/boxes/base.html

* https://www.vagrantup.com/docs/virtualbox/boxes.html

