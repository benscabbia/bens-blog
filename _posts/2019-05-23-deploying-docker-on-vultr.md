---
layout: post
title: Deploying docker on Vultr
description: This post will show you the general concept of deploying your docker-compose application to a cloud provider (Vultr) via docker-machine.
categories: devops
tags: [vultr, cloud, docker]
excerpt_separator: <!--more-->
---

I have been working away on my little - fully dockerized - project (ebaymanager), and decided it was time to try and deploy the app on Vultr! This post will go through the necessary steps and how to deploy your dockerized app on a cloud provider.

<!--more-->

To follow along, get yourself a domain and then sign up to vultr. Here is my [referral link](https://www.vultr.com/?ref=7988290) - we both get \$10 credit, but there are better ones occasionally so feel to drop me a comment and I'll happily look!

Now that's out the way, the way I run my app is with docker compose: `docker-compose up`.

> I will provide the source code once I have finished developing, but for now, you could follow along by deploying any app that's been dockerized and configure to run with docker-compose.

It's easy locally, and wouldn't it just be great to run `docker-compose up` but on a server directly... but how do we do this? [Docker machine](https://docs.docker.com/machine/)!

> Docker Machine is a tool that lets you install Docker Engine on virtual hosts, and manage the hosts with docker-machine commands. It enables you to provision multiple remote Docker hosts. Docker Machine also serves as a secondary purpose, it enables docker to be installed on older Macs/Window system.

If you have Docker already installed, it will already include Docker Machine (along with Docker Compose). You can run `docker-machine version` to see if you have it installed. If you haven't got it installed, you can install it [here](https://docs.docker.com/machine/install-machine/)

### Docker Engine vs Docker Machine

When we say Docker, we refer to Docker Engine, the client-server application made up of the Docker daemon, a REST API that specifies interfaces for interacting with the daemon, and a command line interface (CLI) client that talks to the daemon through the REST API wrapper. When we do `docker run image` we are interacting with the CLI , more info [here](https://docs.docker.com/machine/overview/).

Docker machine is a provisioning tool for managing dockerized hosts (hosts with docker on them). Typically, you install docker machine locally, which you can then use to create docker engine on remote servers.

## Setup

To get started, we need to install a driver to enable us to connect our docker machine to vultr. Do do this:

```
curl -L https://github.com/janeczku/docker-machine-vultr/releases/download/v1.4.0/docker-machine-driver-vultr-Darwin-x86_64 -o /usr/local/bin/docker-machine-driver-vultr && chmod +x /usr/local/bin/docker-machine-driver-vultr
```

This will install the driver and add the certificates to:

```
Creating CA: /Users/ben/.docker/machine/certs/ca.pem
Creating client certificate: /Users/ben/.docker/machine/certs/cert.pem
```

Now we can create an instance which we will use to run our app:

```
// 8 = london, 215 = ubuntu16 x64
docker-machine create --driver vultr --vultr-api-key=<API-KEY> --vultr-region-id=8 --vultr-os-id=215 ubuntu-machine
```

For full configuration, refer to [this page](https://github.com/janeczku/docker-machine-vultr).

When I tried to run the command above, I got the error:

> Error creating machine: Error waiting for machine to be running: Maximum number of retries (60) exceeded

Turns out there is a [github issue](https://github.com/janeczku/docker-machine-vultr/issues/41) for this, and it appears to be an SSH issue where the connection drops after a period of time. Last time I checked, there didn't seem to be a fix, but with help from a friend, I was able to [solve the issue](https://github.com/janeczku/docker-machine-vultr/issues/41#issuecomment-478247699) via brute force.

Once you have the machine created, we're nearly ready to deploy and run our app! The general workflow:

1. Connect to docker-machine Vultr Instance: `docker-machine env ubuntu`. Every docker command will now run directly on the remote host (the Vultr instance)
2. Run your `docker-compose up` command
3. To disconnect from our docker-machine, you do: `docker-machine env -u`

It's quite simple, but a little laborious. If you're using fish, you can create these couple of fish functions:

```
❯ cat ~/.config/fish/functions/dock.fish
function dock
eval (docker-machine env \$argv)
end

~
❯ cat ~/.config/fish/functions/undock.fish
function undock
eval (docker-machine env -u)
end
```

Now you can simply do: `dock ubuntu` and once you're done `undock` :).

Before we run our app, it's better to try and run something simple. Assuming you have the functions, you can do:

```
dock ubuntu
docker run hello-world
```

You should get a little hello world message - assuming everything works. Next, we can try and actually preview something in browser:

```
docker run -d -p 8000:80 --name webserver kitematic/hello-world-nginx
```

Navigate to your domain (you will need to configure your DNS A record) and you should see it there.

Now the final challenge is to get your app running. Assuming you have configured everything correctly, you should be able to do:

```
docker-compose -f docker-compose.staging.yml  up -d
```

Your app should now hopefully be running :).
