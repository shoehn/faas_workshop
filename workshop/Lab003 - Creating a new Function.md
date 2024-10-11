---
date: "Friday, October 11th 2024, 10:07:55 am"
modified: "Friday, October 11th 2024, 3:45:54 pm"
---

In this lab, we will begin to create our own functions. You will see that this is easy after the complicated setup of the infrastructure. Depending on the complexity of your algorithms, not every function will be easy to implement, but once you have the code, integrating with OpenFaaS and deploying the function is easy.

Before you start implementing functions and writing code, I recommend that you define a structure in your file system. You could create a new project folder and a new subfolder, which you could call **lab003** for now.

Create the new folder (`-p` will also create the parent folder *tutorials* if it does not already exist) and then change to the new folder.

``` sh
mkdir -p tutorials/lab003
cd tutorials/lab003
```

## Generate the new function

A function in OpenFaaS is a file that contains the source code of the function to be executed when invoked within the platform. OpenFaaS -- as you have seen in the background document -- provides a lot of functionality such as scalability, automatic provisioning, or security. Therefore, we need to provide some metadata with the function. To make it much easier to specify this correctly, the `faas-cli` tool comes with functionality to automatically generate empty functions from templates.

Let's see what functions are available:

``` sh
faas-cli new --list
No language templates found.

Download templates:
  faas-cli template pull download the standard templates
  faas-cli template store list Show the community template store
```

In a newly installed `faas-cli` environment, we need to download the predefined language templates first. The command tells us how to do this.

``` sh
faas-cli pull template
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

Great. There are now a bunch of language templates available. In this workshop we will focus on `python3` because you all learned it during your studies :LiSmile:.

> 📓 INFO
>
> If you are familiar with another programming language, e.g. JavaScript or Go,
> you can use it in the assignment as well. There is no need to stick to Python.
> JavaScript is executed in a NodeJS environment in OpenFaaS, so you would
> one of the `node' templates to get started.

### One more thing

OpenFaaS by default uses [docker hub](https://hub.docker.com/) as the repository where it pushes the functions to the network.

**Can you explain why this step cannot be avoided?**?

This is due to the architectural decision that the functions are built as containers and can scale automatically. When your instance -- i.e. a container in your cluster running the code of your function -- is fully loaded, OpenFaaS will automatically create a new instance of that function's container. Now you can imagine that this requires a central place where it can find the images of your functions.

We want to use a private repository hosted on a server instance that we own. The URL for this repository is https://registry.kokishin.de

It is quite simple to tell faasd to use this repository, but a few commands are needed to tell it how to connect to the new repository.

To do this, we need to log in using the docker client and copy the configuration to the `faasd` server. Again, this is an architectural choice:
- We have a local command line tool (called `faas-cli`) that manages the local stuff and creates the container images we use in the cloud.
- We also have a cloud-based platform -- although simulated on the same machine, it is a separate virtual machine even in this scenario -- that needs to somehow get hold of the credentials.

``` sh
$ docker login https://registry.kokishin.de

$ faas-cli registry-login --username sda2 --password <password> --server registry.kokishin.de

$ sudo mkdir /var/lib/faasd/.docker/

$ sudo cp ~/.docker/config.json /var/lib/faasd/.docker/

$ sudo systemctl restart faasd
```

In our scenario this is not so difficult because both tools are running on the same hardware. We can use docker's login command, which creates a configuration file with the encrypted password. `faasd` is implemented to use the same configuration file, so we can just copy it over.

And just to be sure, we restart the `faasd` service. This will allow it to re-read the configuration from the registry.

To actually use this registry, in the next chapter you will learn about the so-called prefix. This is an important concept for creating **namespaces**. These are needed because from now on -- if you have decided to use the private registry -- you will be putting all your functions into the same repository. The prefix will make the names unique for each function. Another architectural constraint: each function must correspond to a unique image name in the registry.

For my preferred namespace `shoehn` this would look like this:

``` sh
--prefix=registry.kokishin.com/shoehn
```

Note the domain name and the login name after the slash.

### Hello World

There is no programming tutorial without a `hello world` example. We do not want to reinvent the wheel and follow this convention.

First, you need to create the files needed for an OpenFaaS function. `faas-cli` will do this for us. **Important**: replace the shoehn part in the prefix with your unique one! It will fail if the names are reused.

``` sh
$ faas-cli new --lang python3 hello-world --prefix=registry.kokishin.de/shoehn
Folder: hello-world created.
  ___ _____ ____
 / _ \ _ __ ___ _ __ | ___|_ _ __ _/ ___|
| | | | '_ \ / _ \ '_ \| |_ / _` |/ _` \___ \
| |_| | |_) | __/ | | | _| (_| | (_| |___) |
 \___/| .__/ \___|_| |_|_| \__,_|\__,_|____/
      |_|

Function created in folder: hello-world
Written stack file: hello-world.yml

Notes:
You have created a Python3 function using the Classic Watchdog.

To include third-party dependencies, create a requirements.txt file.

For high throughput applications, we recommend using the python3-flask
or python3-http templates.
```

> 📓 INFO
>
> Just to be clear, it is not necessary to generate the files for the
> function. You can create them manually and implement them, or copy them from another
> function and start from there.
> In the programming context, you will sometimes see the term **scaffolding**.
> for generating an empty function, which we just did above.

This command line tool generated a bunch of files for us.

``` sh
~/proj/lab003$ tree
.
├── hello-world
├── __init__.py
│ ├── handler.py
│ └── requirements.txt
└── hello-world.yml
```

There is a yaml file `hello-world.yml` (yaml is a typical format for configuration files) that contains the **configuration** for the function.

``` sh
~/proj/lab003$ cat hello-world.yml
Version: 1.0
Provider:
  name: openfaas
  gateway: http://127.0.0.1:8080
Functions:
  hello world:
    language: python3
    handler: ./hello-world
    image: registry.kokishin.com/shoehn/hello-world:latest
```

A quick recap of what is in the configuration file:

- The name of the function is represented by the key under `functions`, i.e. `hello-world`.
- The language is represented by the `lang` field.
- The folder to build from is called `handler`, **this must be a folder** not a file
- The name of the docker image to use is in the `image` field.

The `hello-world` folder contains a **python module** that implements the actual function in a file called `handler.py`.

``` Python
def handle(req):
    """handle a request to the function
    Args:
        req (str): request body
    """

    return req
```

As you can see, this function only returns the input. Any values returned to stdout are then passed back to the calling program. We will see what this does in a minute.

> 📓 INFO
>
> `__init__.py` and `requirements.txt` are currently empty and not needed in this
> lab. These are standard Python concepts that we will see in the next lab.

### Deploying the function

Now it is time to make the platform work. We want to deploy the function. This involves three steps:
- Build the code, that is, create a Docker image that can be run on the platform with our function in it.
- Push the image to a central location so the platform can access it.
- Deploy the function, i.e. tell the platform what the function is and where to find the image to run the function.

There are two commands to do this with `faas-cli` (actually it is the same thing, but there is a shortcut `up` that does the three steps in one command).

``` sh
$ faas-cli up -f hello-world.yml
```

This will run all the commands to make your function available on the platform.

Just in case something fails, it might be good to know about the separate commands.

``` sh
$ faas-cli build -f hello-world.yml

$ faas-cli push -f hello-world.yml

$ faas-cli deploy -f hello-world.yml
```

So, if you see some error messages and are not sure where they are coming from, try the three steps in order and you will easily see which command is failing.

### Now for the fun part: Running Your Function

The last thing we need to cover is how to run (invoke is the term in OpenFaaS) the new function.

We can do this using the `faas-cli` command.

``` sh
$ faas-cli invoke hello-world
Read from STDIN - press (Ctrl + D) to stop.
test
test
```

This takes a little getting used to, but it is a convention in Linux. The tool reads the data we want to send to our function from standard input. If we don't specify an input, we can type it in. To let the program know when we are done typing, we press Ctrl+D.

We can try this Linux convention and use a program to generate the input for us. For ease of use in the demonstration, we'll use `echo', which just echoes what's in the quotes of the it argument.

``` sh
echo "Hello, world! | faas-cli invoke hello-world
Hello, World!
```

Note the pipe symbol `|`, which is used on Linux systems to concatenate programs on the command line.

This will get boring pretty quickly because the function does nothing interesting, but one more thing needs to be mentioned.

``` sh
$ curl -X POST http://127.0.0.1:8080/function/hello-world -d 'Hello World
Hello World
```

Yes, the result is boring. But notice that we are using a command-line tool that reads data from a URL on the network. This means that from here on you can use any method that can query a REST API on the network to use you function on the input data you provide.

Now we will move on to the next chapter and build a function that does something more interesting.
