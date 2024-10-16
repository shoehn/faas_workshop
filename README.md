---
date: "Wednesday, October 9th 2024, 3:14:13 pm"
modified: "Wednesday, October 9th 2024, 3:34:21 pm"
---

<figure>
<img src="media/80f5855ea8c622b202e86e88c11f2ce606418649.png"
title="wikilink" alt="workshop_banner.png" />
</figure>

## OpenFaaS Workshop

Welcome to the OpenFaaS workshop! In this workshop, we will guide you through setting up a development environment for working with serverless functions using OpenFaaS. You will learn how to create, deploy, and manage functions, using a local setup on your own machine.

### Toolchain Overview

The workshop toolchain includes:

- **Vagrant:** A tool for building and managing virtual machine environments.
- **Virtualization Backend:** Either QEMU (recommended for macOS) or VirtualBox for Windows users.
- **OpenFaaS:** A framework for building serverless functions that run on Docker.

We provide step-by-step setup instructions for both Silicon-based macOS and Windows environments.

### Setup Instructions

Please follow the appropriate guide for your operating system:

1.  [**Setup for Silicon Mac**](src/branch/master/Install%20Vagrant%20on%20Windows.md)
    - Instructions for setting up Vagrant on an Apple Silicon (M1, M2) Mac.
2.  [**Setup for Windows**](src/branch/master/Install%20Vagrant%20on%20Windows.md)
    - Instructions for setting up Vagrant on a Windows system.

Each guide will walk you through installing the necessary dependencies, setting up the virtualization backend, and preparing the environment for the workshop.

### How to Use the OpenFaaS Instance in the Cloud

For the rest of the module, I will provide an instance of OpenFaaS in the cloud. This means you **do not need to install or set up OpenFaaS on your own computer**. You will only need to install a simple tool (called the CLI, or command line interface) to create and manage your functions on the cloud server.

#### Step 1: Install the Command Line Tool (CLI)

The CLI is a tool that helps you interact with OpenFaaS. Depending on your operating system (MacOS, Linux, or Windows), the installation steps are different.

##### For MacOS and Linux Users

1. Open the **Terminal** application.
2. Type the following command and press **Enter**:

``` shell
curl -sSLf https://cli.openfaas.com | sh
```

This command will install the faas-cli tool, which is what you’ll use to interact with the OpenFaaS cloud instance (as we did already in the workshop).

##### For Windows Users

You’ll need to install the faas-cli tool in two steps:

1. **Open PowerShell** (or Git Bash, if you have that installed).
2. Run the following command to install arkade, which will help you install faas-cli:

``` powershell
curl -sLS https://get.arkade.dev | sh
```

3. After arkade is installed, use it to install faas-cli by running:

``` powershell
arkade get faas-cli
```

#### Step 2: Connect to the Cloud Server

Once you’ve installed the faas-cli, you need to tell it to connect to the correct OpenFaaS server, which is hosted in the cloud (not on your own computer).  

##### For MacOS and Linux Users

1. Open your **Terminal**.
2. Run this command to set the server address:

```shell
export OPENFAAS_URL=<IP>:8080
```

Replace `<IP>` with the actual IP address of the cloud server. You can find this IP address in Moodle or Teams (it’s not shared publicly for security reasons).

##### For Windows Users

1. Open **PowerShell**.
2. Run the following command to set the server address:

```powershell
[Environment]::SetEnvironmentVariable("OPENFAAS_URL", "<IP>:8080", "User")
```

Again, replace `<IP>` with the actual IP address of the OpenFaaS server, which is provided in Moodle or Teams.

**Note:** This setup will only last as long as your terminal (or PowerShell) is open. If you close and reopen it, you’ll need to run the above commands again to reconnect to the server.

#### Step 3: Log in to the Server

Now that you’ve connected to the correct server, you need to log in to it. Use the following command:

``` shell
faas-cli login --password <password>
```

You’ll find the password in Moodle or Teams, just like the IP address.

#### Step 4: Create and Deploy Your Functions 

Now that you’re logged in, you can follow the workshop tutorials to create and deploy your own functions. **Remember:** you don’t need a local instance of OpenFaaS, everything will run on the cloud server!

### Important Tips

- **Function Naming:** In the workshop, we discussed the importance of choosing a unique function name with a specific prefix. When deploying your function, check the functions already deployed by other groups using faas-cli. Make sure to choose a unique name to avoid conflicts, as all groups will be working in the same namespace.
- **Reconnecting:** If you close your terminal or restart your computer, remember to re-enter the server connection command (the one with export OPENFAAS_URL or SetEnvironmentVariable) and log in again with the faas-cli login command.


## License

This workshop content is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. You are free to share and adapt the material, as long as you give appropriate credit and distribute your contributions under the same license.
