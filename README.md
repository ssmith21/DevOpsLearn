# 1) Intro

## Containers
* We want to have the exact same environment for development and production, this ensures that program works exactly as tested
* Don't want to uninstall and reinstall dependencies all the time.

## Virtual machines versus Docker
* We can essentially do the same thing as Docker with a VM, but the problems are the OS (discrepencies in versions, each OS is installed seperately) and the overhead;
wastes a lot of space in memory and tends to be slow.
* Docker simply runs containers on the *Docker Engine* which runs on an *Emulated Container Support*, this all runs on one OS.

## Example 1
* We have a nodeJS app which requires *NodeJS 14*. To run it, we also must run *npm install express*.
* To make things simple, we write a dockerfile, then in the terminal which sees the Dockerfile, we write *docker compose .*
*   Once it's built, we get the ID of the image and run *docker run -p 3000:3000 <ID>*
*   Now we can run localhost:3000 which we can visit. The -p 3000:3000 uses the localhost to reach the application running on port 3000 instead of the container.
*   There is no default connection between our container and our localhost. If we want to send an http request to an application running on a container, we need to open up the port on the container to which we want to communicate with.  
* To see which containers are running, open up another terminal and write *docker ps*
* To stop a container, we run *docker stop <assigned name>*

  # 2) Images and containers

  ## Images vs Containers
* Docker  -> container: Unit of software which we run
          -> image: *Template* for the containers. Contains code, required tools and runtimes. The image executes the code.
* We can use images to create multiple containers. 
* For example, say we want to run a NodeJS app. We define the image once, but can run multiple containers. The containers are the unit of software running the NodeJS app.
