---
date: "Thursday, October 10th 2024, 9:50:08 am"
modified: "Thursday, October 10th 2024, 10:55:49 am"
---

## Login to the virtual machine

Let’s start by checking if your virtual machine is running. Vagrant makes it easy for you to log in securely using a pre-configured SSH key. Simply run the following command in your terminal:

``` sh
vagrant ssh
```

This command will bring you to the command line of the virtual machine. If you see a message like this:

> \*\*\* System restart required \*\*\*

You can ignore it for now. If needed, you can restart the machine later with the command:

``` shell
sudo reboot
```

Restarting takes some time, so we’ll skip it for now.

> :notebook: INFO
>
> The virtual machine has been updated with the latest 
> packages to save time, but a restart wasn’t included in the
> setup. The system is up to date, and you can reboot later if 
> necessary.

### Fixing Login Issues with faasd

If you saw an error message during setup that looked like this:

> dial tcp 127.0.0.1:8080: connect: connection refused

It means the faasd service wasn’t fully ready when the login attempt happened. To fix this, just run the following command inside the virtual machine:

``` sh
sudo cat /var/lib/faasd/secrets/basic-auth-password | /usr/local/bin/faas-cli login --password-stdin
```

> :notebook: INFO
>
> This issue is caused by a “race condition,” which means the 
> server wasn’t fully started when the script tried to log in. 
> It’s a minor problem, and running the command above will fix 
> it.

## Access the Web Interface

Open your web browser and go to: [http://127.0.0.1:8080](http://127.0.0.1:8080)

![](../media/Screenshot%202024-10-10%20at%2010.34.25.png)

You will need to log in. The password was automatically generated during the setup. To get the password, run this command inside the virtual machine:

``` bash
sudo cat /var/lib/faasd/secrets/basic-auth-password
```

The command will display a long password. Just copy it and use it to log in to the web interface.

## Try out the Portal

To test the system, try deploying one of the default functions.

![](../media/Screenshot%202024-10-10%20at%2010.40.02.png)

> 📔 INFO
>
> If you’re using a Silicon Mac, some default functions may 
> not work because they aren’t available for ARM64.


You should see your deployed function listed on the left side of the portal. Click on it, and then you can use the Invoke button to run the function.

Feel free to explore the interface and try out different functions.

![](../media/Screenshot%202024-10-10%20at%2010.45.05.png)

Then you can invoke it.

![](../media/Screenshot%202024-10-10%20at%2010.46.35.png)


You will see the following information displayed:

- **Status:** Shows if the function is ready to run. You won’t be able to invoke it until the status says “Ready.”
- **Replicas:** The number of instances of your function running.
- **Image:** The name and version of the Docker image being used.
- **Invocation Count:** How many times the function has been run. This updates every 5 seconds.

Try clicking **Invoke** several times and watch the **Invocation Count** increase.


### Deploy a Function from the OpenFaaS Store

You can also deploy functions from the OpenFaaS store, which is a collection of community-curated functions.

1. Click **Deploy New Function**.
2. Click **From Store**.
3. Search for **Figlet** (or type “figlet” in the search bar) and click **Deploy**.
 
The Figlet function will appear on the left. Wait a few moments for it to download, then enter some text and click Invoke to see an ASCII logo generated, like this:

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
