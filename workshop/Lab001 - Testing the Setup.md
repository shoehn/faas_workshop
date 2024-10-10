---
date: "Thursday, October 10th 2024, 9:50:08 am"
modified: "Thursday, October 10th 2024, 10:01:51 am"
---

## Login to the virtual machine

First we want to check, if the machine is running. Vagrant has pre-configured the login using a secure freshly generated ssh key for us. It wraps the command in a a convenient way.

``` sh
vagrant ssh
```

This brings you immediately to the shell in the virtual machine. There might be a message saying

> \*\*\* System restart required \*\*\*

Ignore that for now. You can at a later opportunity reboot the machine with a simple:

``` sh
sudo reboot
```

It takes some time, that we will save for now.

In case you got the message, that the login to the faasd service failed during the installation of the machine. It said something like:

> dial tcp 127.0.0.1:8080: connect: connection refused

Then you might need to re-issue the login command, that we already issued during the setup:

``` sh
sudo cat /var/lib/faasd/secrets/basic-auth-password | /usr/local/bin/faas-cli login --password-stdin
```

> \[!info\]
> The error arises from a so-called race condition: the server has not yet fully started, when the installation script calls the login. Since the server is not ready, it refuses the connection and the call fails.
> That might be fixed, but since it is a minor issue, we just need to re-issue the command manually here.

## Check the GUI of the webapp

Navigate to port 8080 on your local machine: <http://127.0.0.1:8080>

You will need to login to the application. The password was automatically generated during the setup of the toolchain. It can be retrieved from the command line in the virtual machine:

``` sh
sudo cat /var/lib/faasd/secrets/basic-auth-password
```
