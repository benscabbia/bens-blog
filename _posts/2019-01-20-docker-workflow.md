---
layout: post
title: Docker Workflow
categories: programming
tags: [docker, devops]
excerpt_separator: <!--more-->
---

It's 2019, and I've decided to finally take the time to learn and start using Docker! This post is what I wished I had when I started. The only pre-requisite to this post is that you go through parts 1-3 from the [official documentation](https://docs.docker.com/get-started/), then you should follow be able to follow without a problem! 

<!--more-->

The best way to learn is to get completely stuck in. A friend of mine created a really good [project](https://github.com/OllieDay/asp-net-core-nginx-https-docker-compose-example) with all the essentials. This post will use the project to explain how to use Docker during your development workflow. The project is a dotnet core project with an NGINX reverse proxy, but the Docker principles will be the same for any language or framework.  

## Run the project

The first step is to clone the repo:

```bash
git clone https://github.com/OllieDay/asp-net-core-nginx-https-docker-compose-example.git
```

Once downloaded, we will see a bunch of files. But before we start taking a deep dive, let's just run the project:

```bash
docker-compose up
```

You will then see something like below:

```
Creating network "asp-net-core-nginx-https-docker-compose-example_default" with the default driver
Creating example.app   ... done
Creating example.proxy ... done
Attaching to example.app, example.proxy
example.app      | Hosting environment: Development
example.app      | Content root path: /app
example.app      | Now listening on: http://[::]:80
example.app      | Application started. Press Ctrl+C to shut down.
```

Let's see if this works:

```
curl https://localhost -k  
```

Alternatively, you can navigate to your browser at [https://localhost/](https://localhost/).

> Note, you will get a  warning since it's an untrusted, self-signed certificate. 
> The purpose of the `-k` is enable [insecure connections](https://curl.haxx.se/docs/manpage.html#-k) (connections not trusted by a CA). More info [here](https://curl.haxx.se/docs/sslcerts.html). 

You will now get a response, `Hello World!`. 

Once you're done, hit `CTRL + C`. 

Now that we've run the example, we will start from the bottom and work our way up to the very top, by then, hopefully, you'll fully understand the power behind docker and why it makes development a joy!

## Dockerfile
Let's begin by looking at the Dockerfile present in `Example.App/Dockerfile`. A Dockerfile is used by docker to build an image. As a programmer, an image can be thought of as a class, and a container is an instance of that image, an object. Let's take a look and see what's happening:

```Docker
# Dockerfile creates two images to form a multi-stage build

# Image 1, First stage of our build ----------------------------------------
# Load the dotnet SDK and create an alias 'build'
FROM microsoft/dotnet:2.2.100-sdk-alpine3.8 AS build

# Set the working directory to /app. This means calling RUN, CMD, ENTRYPOINT, COPY or ADD will be set to that directory
WORKDIR /app

# Copy our csproj file and paste it to the root direct i.e. /app
COPY *.csproj .

# Restore packages
RUN dotnet restore

# Once packages are restored, we copy the rest of our files in current directory to /app
COPY . .

# Build the app DLL's to /out directory
RUN dotnet publish -c Debug -o out


# Image 2, Second stage of our build ----------------------------------------
# Load the dotnet core runtime - this is a streamlined version of the SDK. SDK allows you to both build and run your app, runtime only allows you to run the app
FROM microsoft/dotnet:2.2.0-aspnetcore-runtime-alpine3.8

# Informs Docker that our container will listen to port 80. We can specify if port listens for UDP/TCP. TCP is default 
EXPOSE 80

WORKDIR /app

# Copy the artifacts from another image called 'build' the files in /app/out and paste to our working directory: /app
COPY --from=build /app/out .

# Once the image is created and the container starts up, this is the first command that it will run. In this case, run the app
ENTRYPOINT ["dotnet", "Example.App.dll"]
```

Hopefully, you followed the comments and understand on a line by line basis what the Dockerfile is doing - but how do we use it? Let's build it:

```bash
docker build -t exampleapp .
```

Assuming you ran earlier the `docker-compose up` command, this will be fairly quick. You will see the 11/11 commands have been run, one for each line in our Dockerfile. Now type:

```bash
docker images
```

You will see 4 (or more) images, 2 of them are microsoft/dotnet, 1 which is untagged and 1 which is called exampleapp. These are the images we've just built for our app. The two dotnet ones are used by our two other images for the multi-stage build, the untagged image is used to build our app and exampleapp is used for running the app.

Let's now run the image:

```bash
docker run -p 4000:80 exampleapp
```

The command is self-explanatory. Run our image exampleapp i.e. create a container, and map our port 4000 to our containers port 80. Let's make a request:

```bash
curl http://localhost:4000
```

Once again, we will get `Hello world!`

Hopefully, this is starting to make more sense. The Dockerfile contains instructions for Docker telling it - step by step - how to build it. We then create the image via `build` and create an instance of that image - a container - via the `run` command. 

### Layers

Something that's been happening but we haven't discussed is the layers. When Docker creates images from our Dockerfile, each instruction in the Dockerfile adds a new layer which is then used by the image. Docker can then cache these layers, and rather than having to regenerate the whole image each time something changes, it can just update the modified layer. Layers can also be reused by other images, thus reducing overall disk usage and improve performance. Layers are partly the reason why Docker is extremely fast (once the images have downloaded).

If you paid close attention, in the previous image build, we noticed there were 11 instructions, each of these instructions had a random hash, this hash is the unique identifier for the layer! You can view your layers via: `docker history exampleapp`. More information [here](https://medium.com/@jessgreb01/digging-into-docker-layers-c22f948ed612).

Let's now head over our `Example.Proxy` and let's take a look at the Dockerfile:

```Docker
FROM alpine:3.8 AS generate
WORKDIR /certificates
RUN apk update && \
    apk add --no-cache openssl && \
    rm -rf /var/cache/apk/*
RUN openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout example.key -out example.crt -subj "/C=GB"

FROM nginx:1.15.8-alpine
EXPOSE 443
COPY nginx.conf /etc/nginx/nginx.conf
COPY --from=generate /certificates/example.key /certificates/example.crt /etc/nginx/ssl/
```

I won't add an explanation on what it's doing line per line, as you should now have a rough understanding on what it's doing since we've seen all of the commands except for `Run`. In this instance, by using `Run` we install the latest updates, add `openssl` and generate a certificate. It is also loading our nginx configuration, but explaining that is outside the scope of this post, but I am planning on writing a post on it. 

> Note, you can delete all your images at once via: `docker rmi -f (docker images -a -q)`. A more intelligent command is prune, which removes all unused containers, networks, images (both dangling and unreferenced), and optionally, volumes: `docker system prune --volumes -f`. More info [here](https://docs.docker.com/engine/reference/commandline/system_prune/).

## docker-compose.yml
We've successfully built and run our images independently and that's great. But what happens if we had a complicated app, with dozens of containers to run therefore dozens of images to build? That would just be painful. Fortunately, that's what docker-compose does. Docker-compose is a tool that defines and runs multi-container applications. You can use compose for any stage in your environment: production, staging, development, testing or in a CI pipeline. 

Let's take a look at our docker-compose.yml:

```Docker
version: '3.7'

services:

  # Define a service called example.app
  example.app:

  # Create an image called example.app
  image: example.app

  # Create a container called example.app
  container_name: example.app

  # When creating the image, look for a Dockerfile located in Example.App. The build: path/to/dockerfile is shortform
  build: Example.App

  # Set an environment variable which is available to our container
  environment:
    - ASPNETCORE_ENVIRONMENT=Development
      
  example.proxy:
    image: example.proxy
    container_name: example.proxy
    build: Example.Proxy
    # Map the hosts port 443 to the containers 443
    ports:
      - 443:443
```

There are a couple of interesting things happening here. Firstly, is that rather than having to use our containers IP address we can just use `example.app` everywhere. You might have noticed in our `nginx.conf` that we set the `upstream=example.app`. Docker-compose is what makes this possible. 

Secondly, the line `build: Example.App` is actually a shortform and is equivalent to:

```Docker
build:
  context: path/to/build/context
  dockerfile: path/to/dockerfile
```

The short form is convenient when your Dockerfile is nested with the project and directory. It's useful to use the long-form when and if you end up renaming your Dockerfile to something else like `Dockerfile.development` and `Dockerfile.production`. For an example, view this [project](https://github.com/OllieDay/zhongwen.world/tree/master/zhongwen.world.proxy). 

The rest should be fairly self explanatory, but effectively, using docker-compose is a 3 step process:
1. Define your apps Dockerfile so it can build the image
2. Define your services so it can run your images in an isolated environment
3. Execute `docker-compose up` so it starts and runs your whole app 

How cool is that? Lastly, do note that when you make a change to, for example, a projects source code, you will need to have that image rebuild. Unfortunately, `docker-compose up` does not rebuild images, so to force a rebuild, you have to first run: `docker-compose build` followed by `docker-compose up`. You can also combine the two by passing the `--build` flag i.e. `docker-compose up --build`. 

Hopefully, you've found this post somewhat helpful, if you have, it would be great if you could star the [project](https://github.com/OllieDay/asp-net-core-nginx-https-docker-compose-example) the post was built on! As a next step, I'd encourage you to find other docker projects, run them, take a look at how they're wired up and then try to set up your development workflow! 

In a future post, I will go into how to use Docker in production and deployments! 