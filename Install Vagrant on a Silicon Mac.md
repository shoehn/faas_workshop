---
date: "Wednesday, October 9th 2024, 2:26:06 pm"
modified: "Wednesday, October 9th 2024, 2:34:09 pm"
---

## Silicon Mac Setup Instructions

### 1. Install Homebrew

Homebrew is a package manager that will help us install the necessary tools easily. Run the following command in the Terminal to install Homebrew:

``` sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

If this doesn't work, visit [brew.sh](https://brew.sh/) for more installation help.

## 2. Install Vagrant

Vagrant will help us manage virtual machines. Use Homebrew to install it by running these commands in the Terminal:

``` sh
brew tap hashicorp/tap  

brew install hashicorp/tap/hashicorp-vagrant
```

## 3. Install Rosetta

We need **Rosetta** to run some applications that are not yet compatible with the newer Apple Silicon (M1 and M2) chips. Run this command to install or update Rosetta:

``` sh
/usr/sbin/softwareupdate --install-rosetta --agree-to-license
```

## 4. Choose a Virtualization Backend

To run virtual machines, we need a backend. There are two options available: **QEMU** (recommended) or **VMware Fusion**.

### Option 1: Use QEMU (Recommended)

QEMU is an open-source virtualization backend. To install QEMU, run:

``` sh
brew install qemu
```

### Option 2: Use VMware Fusion (Optional)

You can use **VMware Fusion** as an alternative backend. It is available for free for personal use but is not open source. If you choose this option, follow these steps:

#### 1. Download VMware Fusion

[VMware Fusion Free Download](https://blogs.vmware.com/teamfusion/2024/05/fusion-pro-now-available-free-for-personal-use.html)

#### 2. Install the Vagrant VMware Plugin

``` sh
vagrant plugin install vagrant-vmware-desktop
```

For more details on setting up VMware Fusion, see the [Vagrant VMware Utility Installation Guide](https://developer.hashicorp.com/vagrant/docs/providers/vmware/vagrant-vmware-utility#vagrant-vmware-utility-installation).

## Alternative Setup Instructions

If you prefer to use a different toolchain for managing virtual machines on your Apple Silicon Mac (e.g., Parallels, VMware), you can skip the setup instructions above. In this case, prepare a virtual machine running **Ubuntu Linux 22.04 LTS** and make sure **port 8080** is accessible on your host machine.
After setting up the virtual machine, install **Docker** and **Git**. Then, follow these steps to complete the environment setup:

### 1. Install faasd

```sh
curl -sfL https://raw.githubusercontent.com/openfaas/faasd/master/hack/install.sh | bash -s -
```
  
### 2. Install the OpenFaaS CLI

```sh
curl -sSLf https://cli.openfaas.com | sh
```

### 3. Log in to OpenFaaS using the generated password

```sh
cat /var/lib/faasd/secrets/basic-auth-password | /usr/local/bin/faas-cli login --password-stdin
```

This alternative allows you to use your preferred virtualization software while still participating fully in the work
shop.
