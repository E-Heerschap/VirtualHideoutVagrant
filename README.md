# VirtualHideout Vagrant Documentation

This documenation contains information specific to building a virtual box vagrant box
capable of compiling and running the VirtualHideout kernel modules. This box is intended for developers and does not represent the minimum library requirements for using the VirtualHideout tools.

## Prerequisites
- Virtual Box
- Package (HashiCorp utility for packaging virtual box)
- Vagrant Host Shell Plugin: `vagrant plugin install vagrant-host-shell`
- Vagrant Reload Plugin: `Vagrant plugin install vagrant-reload`

## Steps to build box via Vagrantfile

Unfourtunately, until an S3Bucket hosting a prepackaged box (will occur eventually...) the user MUST build the vagrant box themselves initially.

Simply run `vagrant up` in this directory to download, build and configure the VirtualHideoutVM.

It is recommended that once the VirtualHideoutVM is provisioned that it is then packaged and stored locally. Run `vagrant package --base VirtualHideoutVM --output VirtualHideoutVM.box` to package.
If you are using this box make sure you remove any preexisitng boxes named VirtualHideoutVM.box already in the vagrant box repository by `vagrant box remove VirtualHideoutVM.box`.

## Steps to build bootstrap box.
- Alpine boot iso

1. Start Virtual Box and create an "Alpine Build" VM. This VM is not the packaged VM.
Instead, this VM is used to build/setup the alpine VM to be used. However, the disk size you select for the build VM is the disk size used for the primary VM. 
Start the Alpine Build VM with the Alpine Linux Virtual distribution iso from the prerequisites.

2. Login to the alpine build VM by typing "root". You will not be asked for a password.

3. Run `setup-alpine` and follow the steps. Leaving all settings default except for name is fine. Set the name to "Virtual Hideout". Set the root password to whatever you deem acceptable.

4. Run `poweroff` to shutdown the VM when prompted to reboot the system.

5. Change the VM to boot from the disk and not the iso.

6. On the host machine run `vagrant package --base VirtualHideoutBootstrap --output VirtualHideoutBootstrap.box

7. All done building the box :)

### Notes

* The user name virtualhideout is not required. Just convention.

* The MAC Address will likely not be required for the box to work. However, the official documentation (https://www.vagrantup.com/docs/virtualbox/boxes.html) mentions this as a required step.

* For extra security, it is recommended the public/private keys in the repository should not be used. Instead, generate your own as done in step 14.

* For now the virtual machine must be named "VirtualHideoutVM".

* To increase compile time of the linux kernel (final step in the provision process) change the -jx parameter of the make command in the Vagrant file and let x be the number of cores to use for the compilation.

## Prepackaged box credentials

* Public/private key pair for the VirtualHideoutVM (default provisioned)  box are available in this repository.

* The general user with access to sudo has the credentials `username: virtualhideout password: virtual`

* The root user has the credentials `username: root password: virtualbuild`

## Trouble shooting:

* For SSH issues, the private key for the VirtualHideoutVM (default provisioned) box is available in the repository.

* If network issues occur, try building your own box and noting the MAC address of the first network adapter and adding `config.vm.base_mac="MAC_ADDRESS_HERE"` line to the Vagrant file

* The provisioning process assumes four cores are available for the compilation of the linux kernel. This may be problematic for some pcs. This can be changed by changing the -jx parameter of the make command in the Vagrant file for making the linux kernel. Replace x with the number of cores to use. -j1 is implies no parallel building.

* If a message occurs that the VM cannot be renamed and there is no VM in virtual box then the old VM folder will need to be manually deleted. Check where the virtualboxvm's are created and delete the folder "VirtualHideoutVM".

## Useful resources

* https://www.vagrantup.com/docs/vagrantfile/ssh_settings.html

* https://www.vagrantup.com/docs/boxes/base.html

* https://www.vagrantup.com/docs/virtualbox/boxes.html

## TODO

* Change hard name requirement

* Dynamically generate SSH key files.
