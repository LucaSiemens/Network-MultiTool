# Network-Multitool
A multitool for container/network testing and troubleshooting, forked from https://github.com/Praqma/Network-MultiTool.

The container image contains lots of tools, as well as a `nginx` web server, which listens on port `80` and `443` by default. The web server helps to run this container-image in a straight-forward way, so you can simply `exec` into the container and use various tools.

## Tools included
* Nginx Web Server (port 80, port 443) - customizable ports!
* wget, curl
* dig, nslookup
* ip, ifconfig, route, tcptraceroute, mtr
* arp
* ps, netstat
* iperf
* tcpdump, tshark
* jq
* mgen
* tsn test app

**Note:** The SSL certificates are generated for 'localhost', are self signed, and placed in `/certs/` directory. During your testing, ignore the certificate warning/error. While using curl, you can use `-k` to ignore SSL certificate warnings/errors.


# How to use this image? 

## Build as regular docker container
1. Clone this repo
2. Move to the directory: ```cd Network-MultiTool```
3. Clone the tsn app repo
4. Optional: Create a tsn log output directory somewhere on your local machine (referred to as <tsn_logs> in the following)
5. Optional: Create a directory somewhere on your local machine that will be used as another shared directory (referred to as <shared_dir> in the following)
6. Build the image:
```
$ docker build -t network-tools .
```
7. Run the image:
```
$ docker run --cpus=2 --memory=256m -d --name=network-multitool --net=host --privileged -v <tsn_logs>:/app/out -v <shared_dir>:/shared_dir -e HTTP_PORT=1180 -e HTTPS_PORT=11443 -t network-tools
```
8. Verify it's working:
```
$ curl http://localhost:1180
-e Praqma Network MultiTool (with NGINX) - ubuntu - *.*.*.*
$ curl -k https://localhost:11443
-e Praqma Network MultiTool (with NGINX) - ubuntu - *.*.*.*
```
9. Enter the container with an interactive shell:
```
$ docker exec -it <container_id> bash
```

# FAQs (unedited from original project)
## Why this multitool runs a web-server?
Well, normally, if a container does not run a daemon/service, then running it (the container) involves using *creative ways / hacks* to keep it alive. If you don't want to suddenly start browsing the internet for "those creative ways", then it is best to run a small web server in the container - as the default process. 

This helps you when you are using Docker. You simply execute:
```
$ docker run  -d praqma/network-multitool
```

This also helps when you are using kubernetes. You simply execute:
```
$ kubectl run multitool --image=praqma/network-multitool
```


The multitool container starts as web server. Then, you simply connect to it using:
```
$ docker exec -it some-silly-container-name /bin/sh 
```

Or, on Kubernetes:
```
$ kubectl exec -it multitool-3822887632-pwlr1  -- /bin/sh
```

This is why it is good to have a web-server in this tool. Hope this answers the question! Besides, I believe that having a web server in a multitool is like having yet another tool! Personally, I think this is cool! [Henrik](https://www.linkedin.com/in/henrikrenehoegh/) thinks the same!


## I can't find a tool I need for my use-case?
We have tried to put in all the most commonly used tools, while keeping it small and practical. We can't have all the tools under the sun, otherwise it will end up as [something like this](https://www.amazon.ca/Wenger-16999-Swiss-Knife-Giant/dp/B001DZTJRQ).  

However, if you have a special need, for a special tool, for your special use-case, then I would recommend to simply build your own docker image using this one as base image, and expanding it with the tools you need.

## Why not use LetsEncrypt for SSL certificates instead of generating your own?
There is absolutely no need to use LetsEncrypt. This is a testing tool, and validity of SSL certificates does not matter.

## Why use a daemonset when troubleshooting host networking on Kubernetes?
One could argue that it is possible to simply install the tools on the hosts and get over with it. However, we should keep the infrastructure immutable and not install anything on the hosts. *Ideally* we should never `ssh` to our cluster worker nodes. Some of the reasons are:

* It is generally cumbersome to install the tools since they might be needed on several hosts.
* New packages may conflict with existing packages, and *may* break some functionality of the host.
* Removing the tools and dependencies after use could be difficult, as it *may* break some functionality of the host.
* By using a `daemonset`, it makes it easier to integrate with other resources. e.g. Use volumes for packet capture files, etc.
* Using the `daemonset` provides a *'cloud native'* approach to provision debugging/testing tools.
* You can `exec` into the `daemonset`, without needing to SSH into the node.
