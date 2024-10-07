
# Prerequisities

For the setup of the OpenFaaS environment that you can use for experimentation in the module you need 
a linux machine running Ubuntu 22.04. For that reason we will use a virtual machine. The proposed solution
is to use vagrant and the scripts provided in this github repository that I created for the module.

There is no need to use the setup for the virtual machines provided. If you have a preferred technology use that instead.

There are **known limitations** with virtualization on mac computers that use **Apples Silicon Chipsets**. If you have
a running instance of VMWare Fusion or Parallels I recommend that you create a linux machine there. Otherwise,
you will find the instructions in the folder ```setup/mac/silicon``` in this repository. This workshop was developed on a 
Mac with a M1, so it should work and stability should be ok.

## Windows 

On windows you need to install VirtualBox and Vagrant to run the proposed toolchain.

## MacOS with a Silicon Chip (e.g. M1)

I recommend homebrew as a package manager to install everything that is needed.

You will need ```qemu``` as a virtualization hypervisor and ```vagrant``` as the frontend.

## MacOS with an Intel Chip

You need to install VirtualBox and Vagrant to run the proposed toolchain.
