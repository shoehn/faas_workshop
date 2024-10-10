---
date: "Thursday, October 10th 2024, 10:57:42 am"
modified: "Thursday, October 10th 2024, 11:07:33 am"
---

## Learn about the CLI

### List the deployed functions

This will show the functions, how many replicas you have and the invocation count.

``` sh
$ faas-cli list

Function                        Invocations     Replicas
cows                            1               1
figlet                          2               1
```

In my case you see the `cows`and the `figlet`functions that I have installed as samples from the store.

You can use the `--verbose`flag or `-v`for short to get more information on your funtions.

``` sh
$ faas-cli list -v

Function                        Image                                           Invocations     Replicas        CreatedAt
cows                            ghcr.io/openfaas/cows:latest                    1               1               2024-10-10 08:42:56.288543292 +0000 UTC
figlet                          ghcr.io/openfaas/figlet:latest                  2               1               2024-10-10 08:49:04.818864813 +0000 UTC
```

You can now see the Docker image along with the names of the functions.

### Invoke a function

The most interesting part is, of course, invoking our functions to make them do something for real.

To try it out choose one of the functions that you saw on the list. I will choose cows, since that fits well to Switzerland.

``` sh
$ faas-cli invoke cows
Reading from STDIN - hit (Control + D) to stop.
         |\/|
         (oo)
  /-------VV
 / |     ||
*  ||----||
   ^^    ^^
  Lycownthropy
```

You'll now be asked to type in some text. Hit Control + D when you're done.

Alternatively you can use a command such as `echo` or `curl` as input to the `invoke` command which works through the use of pipes.

``` sh
$ echo "Lycownthropy" | faas-cli invoke cows
         (__)    (__)   (__)    (__)
         (oo)    (oo)   (oo)    (oo)
  /-------\/      \/     \/      \/-------\
 / |     ||  ===================  ||     | \
*  ||----||   || |--|   |--| ||   ||----||  *
   ^^    ^^   -- ^  ^   ^  ^ --   ^^    ^^
          A game of cowntract bridge
```

> 📔 INFO
>
> If your *gateway is not* deployed at [http://127.0.0.1:8080](http://127.0.0.1:8080/) then you will need to specify the alternative location. There are several ways to accomplish this:
>
> 1.  Set the environment variable `OPENFAAS_URL` and the `faas-cli` will point to that endpoint in your current shell session. For example: `export OPENFAAS_URL=http://openfaas.endpoint.com:8080`. This is already set in [Lab 1](https://github.com/openfaas/workshop/blob/master/lab1.md) if you are following the Kubernetes instructions.
> 2.  Specify the correct endpoint inline with the `-g` or `--gateway` flag: `faas deploy --gateway http://openfaas.endpoint.com:8080`
> 3.  In your deployment YAML file, change the value specified by the `gateway:` object under `provider:`.
