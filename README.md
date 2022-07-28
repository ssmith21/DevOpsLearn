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
* Docker is made up of 2 core concepts:
  * container: Unit of software which we run
  * image: *Template* for the containers. Contains code, required tools and runtimes. The image executes the code.
* We can use images to create multiple containers. 
* For example, say we want to run a NodeJS app. We define the image once, but can run multiple containers. The containers are the unit of software running the NodeJS app.

## Finding / Creating images
* Use docker hub
* Say we want to run a node application container, on docker if we search *node* we can find the image we want to run.
* In a terminal, run *docker run node*
  * Creates a container based on the image that was published.
  * We will run the container so that we can interact with the NodeJS interactive shell. This is great, because it allows us to interact with Node without having to install Node on our machine.
  * What we've done is we've created a container based off the image, but at the moment the container isn't doing much since by default, the container is isolated from the environment. So inside the container, there's an interactive shell which is running, but it's not exposed to us as a user (since it's inside the container). We can tell the container was created by running *docker ps -a*
  * But now we want to access the interactive shell which is inside the container by running *docker run -it node*, (the -it tells docker we want to run an interactive session).
* RECAP: we pull the node image from docker hub. We now have a node container with an interactive shell which is not expose to us. We run the command *docker run -it node*. By using the docker run command, we create a concrete container instance which exposes the interactive shell to us.
 
## Building an image with Dockerfile
* Contains the instructions for docker which we want to execute when we create our own image.
* FROM : allows us to build an image up from another image, we don't usually build an image from scratch.
   * Ex: FROM: node *the based on an image which runs on our local machine OR if it doesn't runs it from docker hub.*
* COPY : Tells docker which files should go into our image.
   * COPY . . specifies the path outside the container. The first "." is the folder which contains all the folder files. The second "." is the path inside the image where the image should be stored. COPY (local path) (destination)
   * COPY . /app will copy all the files/folders from our local system into an "internal docker filesystem named app".
* WORKDIR : Tells docker where to run all the commands. 
   * WORKDIR /app : When we pass a RUN argument, docker will run the commands inside /app
* RUN : tells us a command we have to run. After copying all the local files in the image (such as package.json), we need to run npm install so we say:
   * RUN npm install <-- BUT theres a gotcha: By default, all the commands will be executed in the working directory of our Docker image. By default, its the root folder in our docker file system. If we wrote COPY . /app then we want to run npm install inside /app, so we need to specify this. So we give it the WORKDIR argument.
