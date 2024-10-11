---
date: "Friday, October 11th 2024, 10:07:55 am"
modified: "Friday, October 11th 2024, 3:45:54 pm"
---

In this lab we will start creating our own functions. You will see that this now, after the complicated setup of the infrastructure straightforward. Depending on the complexity of your algorithms not every function is easily implemented, but if you have the code, integrating into OpenFaaS and deploying the function is easy.

Before you start implementing functions and writing code, I recommend to define a structure in your file system. You might make a new project folder and a new subfolder which you might call **lab003** for now.

Make the new folder (`-p` also creates the parent folder *tutorials* if it does not exist yet) and then change to the new folder.

``` sh
mkdir -p tutorials/lab003
cd tutorials/lab003
```

## Generate the new Function

A function in OpenFaaS is a file that contains the source code of the function to be executed once it is invoked within the platform. OpenFaaS -- as you have seen in the background document -- provides a lot of functionality such as scalability, automatic deployment or security. Therefore, we need to provide some metadata with the function. To make it a lot easier to correctly specify that, the `faas-cli` tool comes with a functionality to automatically generate empty functions from templates.

Let's try out which functions are available:

``` sh
faas-cli new --list
No language templates were found.

Download templates:
  faas-cli template pull           download the default templates
  faas-cli template store list     view the community template store
```

In a newly installed `faas-cli` environment we need to download the pre-defined language templates first. The command tells us how to do that.

``` sh
faas-cli template pull
```

Let's try again:

``` sh
$ faas-cli new --list
Languages available as templates:
- bun
- csharp
- dockerfile
- go
- java11
- java11-vert-x
- node
- node18
- node20
- php7
- php8
- python27
- python3
- python3-debian
- ruby
```

Great. There a bunch of language templates available now. In this workshop we will focus on `python3`, because you have all learned it during your studies :LiSmile:.

> 📓 INFO
>
> If you are familiar with another programming language, e.g. JavaScript or Go,
> you can use that in the assignment, too. There is no need to stick to python.
> JavaScript will be executed in a NodeJS environment in OpenFaaS, so you would
> need to choose of the `node` templates to get started.

### One more thing

OpenFaaS by default uses [docker hub](https://hub.docker.com/) as the repository where it pushes the functions on the network.

**Can you explain why this step cannot be avoided?**

This is due to the architectural decision that the functions are built as containers and that they can scale automatically. If your instance -- i.e. a container in your cluster that is running the code of your function -- is fully loaded OpenFaaS will automatically create a new instance of that function's container. Now you can imagine, that this requires a central place where it can find the images of your functions.

We want to use a private repository, that is hosted on a server instance owned by us. The URL for the repository is https://registry.kokishin.de

It is quite straightforward to tell faasd to use that registry, but a few commands are needed to tell it, how it can connect to the new registry.

To do that we need to login using the docker client and copy the config over to the `faasd` server. This is once again due to the architectural decision:
- we have a local command line tool (called `faas-cli`) that manages the local things and generates the container images we use on the cloud
- we also have a cloud based platform -- although, simulated on the same machine it is even in that scenario a separate virtual machine --, that needs to somehow get hold of the login credentials

``` sh
$ docker login https://registry.kokishin.de

$ faas-cli registry-login --username sda2 --password <password> --server registry.kokishin.de

$ sudo mkdir /var/lib/faasd/.docker/

$ sudo cp ~/.docker/config.json /var/lib/faasd/.docker/

$ sudo systemctl restart faasd
```

In our scenario it is not so difficult, because both tools are on the same hardware. We can use dockers login command, which generates a configuration file with the encrypted password. `faasd` is implemented to use the same configuration file, so that we can just copy that over.

And just to be sure, we restart the `faasd` service. So it can re-read the configuration of the registry.

To actually use that registry in the next chapter, you will learn about the so-called prefix. This is an important concept to generate **namespaces**. These are needed, because from now on -- if you did decide to use the private registry -- you will all push your functions into the same repository. The prefix will make the names unique for each function. Another restriction from the architecture: each function must correspond to a unique image name in the registry.

For my preferred namespace `shoehn` that would look like this:

``` sh
--prefix=registry.kokishin.de/shoehn
```

Notice the domain name and the login name after the slash.

### Hello World

There is no such thing as a programming tutorial without a `hello world` example. We do not want to re-invent the wheel and follow that convention.

First you need to generate the files needed for an OpenFaaS function. `faas-cli` does that for us. **Important**: replace the shoehn part in the prefix with your unique one! It will fail, if the names are re-used.

``` sh
$ faas-cli new --lang python3 hello-world --prefix=registry.kokishin.de/shoehn
Folder: hello-world created.
  ___                   _____           ____
 / _ \ _ __   ___ _ __ |  ___|_ _  __ _/ ___|
| | | | '_ \ / _ \ '_ \| |_ / _` |/ _` \___ \
| |_| | |_) |  __/ | | |  _| (_| | (_| |___) |
 \___/| .__/ \___|_| |_|_|  \__,_|\__,_|____/
      |_|

Function created in folder: hello-world
Stack file written: hello-world.yml

Notes:
You have created a Python3 function using the Classic Watchdog.

To include third-party dependencies create a requirements.txt file.

For high-throughput applications, we recommend using the python3-flask
or python3-http templates.
```

> 📓 INFO
>
> Just to make that clear: It is not necessary to generate the files for the
> function. You can manually create and implement them or copy from another
> function and start from there.
> In the programming context you will sometimes read the term **scaffolding**
> for the generation of an empty function that we just did above.

This command line tool has generated a bunch of files for us.

``` sh
~/proj/lab003$ tree
.
├── hello-world
│   ├── __init__.py
│   ├── handler.py
│   └── requirements.txt
└── hello-world.yml
```

There is yaml file `hello-world.yml` (yaml is a typical format for configuration files) that contains the **configuration** for the function.

``` sh
~/proj/lab003$ cat hello-world.yml
version: 1.0
provider:
  name: openfaas
  gateway: http://127.0.0.1:8080
functions:
  hello-world:
    lang: python3
    handler: ./hello-world
    image: registry.kokishin.de/shoehn/hello-world:latest
```

A short recap on what is in the configuration file:

- The name of the function is represented by the key under `functions` i.e. `hello-world`
- The language is represented by the `lang` field
- The folder used to build from is called `handler`, **this must be a folder** not a file
- The Docker image name to be used is under the field `image`

The `hello-world` folder contains a **python module** that implements the actual function in a file named `handler.py`.

``` python
def handle(req):
    """handle a request to the function
    Args:
        req (str): request body
    """

    return req
```

As you see, this function just returns the input. Any values returned to stdout will subsequently be returned to the calling program. We will try out what that does in a minute.

> 📓 INFO
>
> `__init__.py` and `requirements.txt` are currently empty and not needed in this
> lab. These are default python concepts that we will see in the next lab.

### Deploy the function

Now it is time to make the platform work. We want to deploy the function. This encompasses three steps:
- Build the code, i.e. make docker image that can be executed on the platform with our function in it
- Push the image to a central place, so that it can be accessed by the platform
- Deploy the function, i.e. tell the platform what the function is and where to find the image to run the function

There is two commands to do that with `faas-cli` (actually, it is the same thing, but there is a convenience shortcut `up` which does the three steps in one command).

``` sh
$ faas-cli up -f hello-world.yml
```

This executes all the commands to make your function available on the platform.

Just in case something fails, then it might be good to know about the separate commands.

``` sh
$ faas-cli build -f hello-world.yml

$ faas-cli push -f hello-world.yml

$ faas-cli deploy -f hello-world.yml
```

So, if you see some error messages and are unsure where they come from try the three stages in order and you will easily see which command fails.

### The fun part: Run your function

The last thing we need to cover is how to execute (invoke is the term in OpenFaaS) the new function.

We can do that using the `faas-cli`

``` sh
$ faas-cli invoke  hello-world
Reading from STDIN - hit (Control + D) to stop.
test
test
```

This takes a little getting used to, but it is a convention in Linux. The tool reads the data that we want to send to our function from standard input. If we don't specify an input, we can type it in. To let the program know when we are done typing, we press Ctrl+D.

We can try that Linux convention and use a program that generates the input for us. For ease of use in the demonstration we use `echo` which just echoes what's in the quotes of it argument.

``` sh
echo "Hello, World!" | faas-cli invoke  hello-world
Hello, World!
```

Notice the pipe symbol `|` that is used in Linux systems to concatenate programs on the command line.

This gets boring quite quickly, because the function does not something interesting, but one more thing must be mentioned.

``` sh
$ curl -X POST http://127.0.0.1:8080/function/hello-world -d 'Hello World'
Hello World
```

Yes, the result is boring. But notice, that we use a commandline tool that reads the data from a URL on the network. That means from here on you can use any method that can query an REST API on the network to use you function on the input data that you supply.

Now we will go on to the next chapter and build a function that does something more interesting.
