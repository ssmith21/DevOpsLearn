# Summaries
2) What is docker
3) Why docker and containers
5) Virtual machines vs docker containers
26) A first Summary

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
   * It's worth noting that now we can change COPY . /app ---> COPY . ./ since the second "." will be replaced by /app since copy now works relative to the WORKDIR.
* RUN : tells us a command we have to run. After copying all the local files in the image (such as package.json), we need to run npm install so we say:
   * RUN npm install <-- BUT theres a gotcha: By default, all the commands will be executed in the working directory of our Docker image. By default, its the root folder in our docker file system. If we wrote COPY . /app then we want to run npm install inside /app, so we need to specify this. So we give it the WORKDIR argument.
   * RUN node server.js <-- We don't want to include this right away, since every time we create an image, it will try to run a server, but we only want to run the server when we execute the container. So we use the CMD command to specify that we only want to run *node server.js* when we run the container.
* CMD : Tells us commands to run once we run the container, similar to RUN but doesn't execute when we create the container.
   * CMD ["node","server.js"] runs node server.js
* EXPOSE : By default, Docker has it's own server system inside the container. Our localhost and the docker server don't know about eachother since the server is INSIDE the docker container. The EXPOSE command lets docker know that we want to expose a certain port to our local system. 
   * EXPOSE 80
 
 
```
FROM node

WORKDIR /app

COPY . /app

RUN npm install

EXPOSE 80

CMD ["node","server.js"]
```

 * Now that we've created the docker file run the following commands in CMD:
   * docker build . <-- tells docker to build a new *custom image* based on a docker file. The "." just tells docker that the Dockerfile is in our current dir.
   * docker run <id> <-- now we instantiate the node container in which we can access the application inside the container. Notice that in this case, the output will be nothing and the command will never finish. This is because the CMD "node server.js" never finishes, since its just a command whic runs a server. Once we close the server, it will finish. BUT this will not work, even though we exposed the port 80, we need to tell docker which *internal docker port* our *local port* is accessible from.
   * docker run -p localport:internal <id>
   * docker run -p 3000:80 <id> <-- We can access the exposed internal docker port 80 on our local machine via localhost 3000.   
                                    
                                    
## Images are read-only
* Say we want to make a change in our server.js file. Stopping and restarting the container wont show the changes. Instead we have to re-build the image.
* Recall what we did to get the node program running, we built the image from the dockerfile, which copies all the files into the */app* docker internal directory.
* So to see the changes, we have to rebuild the image. The image takes a snapshot of our source code at the point of time where we call *COPY . /app*.
   * make changes to the code
   * docker build .
   * docker run -p 3000:80 <id>
                                    
## Images are layer-based
* When we build an image, or rebuild it, only the instructions where something changed and all instructions afterwards are re-evaluated. This is because docker uses a cache, so it knows the result of all the code up until the changed code will be the same. Every instruction represent a layer in our docker instruction. Each layer is cached. 

 ## Stopping and restarting containers
* Every time we do *docker run <image>* we build a new container based on the image then run the container. But what if we don't want to build a new container because the image didn't change?
    * *docker ps -a* to get all the past containers we ran, pick a container to run.
    * *docker start <container id>* to run the container we chose.

 ## Interactive mode for docker
* Say we have a python program which generates a random number every time we enter something in the terminal: rng.py, and in rng.py we have int(input(...)) which takes an input. If we try to run the container it won't work because of an EOF error.
 ```
 FROM python
 WORKDIR /app
 COPY . /app
 CMD ["python","rng.py"]
 ```
 * Then we simply write *docker run -i -t <image id>* ,where the -i launches the container in interactive mode and -t launches a docker pseudo-terminal. We can also combine the -i and -t and write *docker run -it <image id>*

## Naming and tagging containers and images
* *docker run -p 8000:30 --name <inputContainerName> <image id>*    <-- assigns a name to a docker container
* image tags are comprised of 2 parts <name : tag>
   * name: Defines a group images, maybe more specialized images. Example: node
   * tag: Defines a specialized image within a group of images. Example: 14
   * Example, in the dockerfile we often write *FROM node* which simply pulls the most recent version of the node image. But if we want to pull node version 14, we would write *FROM node:14*
   * If we run docker ps we also see that there are *name* and *tag* columns, to specify these for a certain container given a certain image id, we write *docker build -t name:tag*
                                                                        
# 3) Containers and Networks
* What do we mean by networks? How we can connect multiple containers, how we can connect containers to our localhost (ex: send http requests to another service), how we can access the internet from our container.
## Definitions: Networks and requests
* Say we have a container with a node application. Say the application needs to communicate to some external API, 
