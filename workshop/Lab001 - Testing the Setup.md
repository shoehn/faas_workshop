---
date: "Thursday, October 10th 2024, 9:50:08 am"
modified: "Thursday, October 10th 2024, 10:55:49 am"
---

## Login to the virtual machine

First we want to check, if the machine is running. Vagrant has pre-configured the login using a secure freshly generated ssh key for us. It wraps the command in a a convenient way.

``` sh
vagrant@ubuntu:~$ vagrant ssh
```

This brings you immediately to the shell in the virtual machine. There might be a message saying

> \*\*\* System restart required \*\*\*

Ignore that for now. You can at a later opportunity reboot the machine with a simple:

``` shell
vagrant@ubuntu:~$ sudo reboot
```

It takes some time, that we will save for now.

> :notebook: INFO
>
> I did not implement the reboot in the provisioning script of the machine to
> save time. I did implement the update of all the packages in the image of the
> virtual machine, since these are quite old for some systems. But that does not
> matter, because they are up to date Ubuntu Linux Systems after the update of
> all packages (and a reboot).

In case you got the message, that the login to the faasd service failed during the installation of the machine. It said something like:

> dial tcp 127.0.0.1:8080: connect: connection refused

Then you might need to re-issue the login command, that we already issued during the setup:

``` shell
vagrant@ubuntu:~$ sudo cat /var/lib/faasd/secrets/basic-auth-password | /usr/local/bin/faas-cli login --password-stdin
```

> :notebook: INFO
>
> The error arises from a so-called race condition: the server has not yet fully started, when the installation script calls the login. Since the server is not ready, it refuses the connection and the call fails.
> That might be fixed, but since it is a minor issue, we just need to re-issue the command manually here.

## Check the GUI of the webapp

Navigate to port 8080 on your local machine: <http://127.0.0.1:8080>

![](../media/Screenshot%202024-10-10%20at%2010.34.25.png)

You will need to login to the application. The password was automatically generated during the setup of the toolchain. It can be retrieved from the command line in the virtual machine:

``` bash
vagrant@ubuntu:~$ sudo cat /var/lib/faasd/secrets/basic-auth-password
```

This will give you a very long and ugly string back, but don't care, just copy that one to the password field of the login dialog.

## Check out the Portal

For testing purposes, try to deploy one of the default functions.

![](../media/Screenshot%202024-10-10%20at%2010.40.02.png)

> 📔 INFO
>
> This will not work on Silicon Macs for all functions, because there are no ARM64 compatible versions of the functions in the default sample directory.

Try your function in the UI. You should see it in the left navigation bar and be able to click it.

![](../media/Screenshot%202024-10-10%20at%2010.45.05.png)

Then you can invoke it.

![](../media/Screenshot%202024-10-10%20at%2010.46.35.png)

Play around with the interface.

You will see the following fields displayed:

- Status - whether the function is ready to run. You will not be able to invoke the function from the UI until the status shows Ready.
- Replicas - the amount of replicas of your function running in the cluster
- Image - the Docker image name and version as published to the Docker Hub or Docker repository
- Invocation count - this shows how many times the function has been invoked and is updated every 5 seconds

Click *Invoke* a number of times and see the *Invocation count* increase.

### Deploy via the Function Store

You can deploy a function from the OpenFaaS store. The store is a free collection of functions curated by the community.

- Click *Deploy New Function*
- Click *From Store*
- Click *Figlet* or enter *figlet* into the search bar and then click *Deploy*

The Figlet function will now appear in your left-hand list of functions. Give this a few moments to be downloaded from the Docker Hub and then type in some text and click Invoke like we did for the Markdown function.

You'll see an ASCII logo generated like this:

``` sh
__        __   _                            _        
\ \      / /__| | ___ ___  _ __ ___   ___  | |_ ___  
 \ \ /\ / / _ \ |/ __/ _ \| '_ ` _ \ / _ \ | __/ _ \ 
  \ V  V /  __/ | (_| (_) | | | | | |  __/ | || (_) |
   \_/\_/ \___|_|\___\___/|_| |_| |_|\___|  \__\___/ 
                                                     
 ____  ____    _    ____  
/ ___||  _ \  / \  |___ \ 
\___ \| | | |/ _ \   __) |
 ___) | |_| / ___ \ / __/ 
|____/|____/_/   \_\_____|
```
