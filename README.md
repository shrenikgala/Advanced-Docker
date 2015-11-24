#ADVANCE DOCKER

## name : Shrenik Gala , Unity id:sngala

1) **File IO**: You want to create a container for a legacy application. You succeed, but you need access to a file that the legacy app creates.

* Create a container that runs a command that outputs to a file.
* Use socat to map file access to read file container and expose over port 9001 (hint can use SYSTEM + cat).
* Use a linked container that access that file over network. The linked container can just use a command such as curl to access data from other container.

Following is the dockerfile for the first container :
<pre>FROM    alpine:3.2

RUN apk update && \
    apk add socat && \
    rm -r /var/cache/

COPY . /src
RUN cd /src
RUN echo "Some text here." > /src/hello.txt;

CMD socat TCP4-LISTEN:9001 SYSTEM:'cat /src/hello.txt'
</pre>
 
To run the container type the following commands :
<pre>sudo docker build -t image1 .
sudo docker run -d -it --name container1 image1
</pre>

Following is the dockerfile for the second container:
<pre>FROM ubuntu
RUN apt-get install -y curl
</pre>

To run this container as a link type following commands:
<pre>sudo docker built -t image2 .
sudo docker run --link container1:appserver --rm -it --name container2 image2
</pre>
For testing
<pre>curl appserver:9001</pre>

![Image1](https://github.com/shrenikgala/DevOpsHW4/blob/master/fileio.gif)


2) **Ambassador pattern**: Implement the remote ambassador pattern to encapsulate access to a redis container by a container on a different host.

* Use Docker Compose to configure containers.
* Use two different VMs to isolate the docker hosts. VMs can be from vagrant, DO, etc.
* The client should just be performing a simple "set/get" request.
* In total, there should be 4 containers.

Docker compose file for host1 i.e Server which will create two containers redis_server and ambassador
<pre>redis_server:
  image: crosbymichael/redis
  container_name: redis_server

ambassador:
  container_name: ambassador
  image: svendowideit/ambassador
  links:
    - redis_server:redis_server
  ports:
    - 6379:6379
</pre>

Commands to run the docker-compose.yml :
<pre>sudo docker-compose up -d</pre>

Docker compose file for host2 i.e Client which will create two containers redis_client and ambassador
<pre>
ambassador:
  image: svendowideit/ambassador
  container_name: ambassador
  expose:
   - "6379"
  environment:
    - REDIS_PORT_6379_TCP=tcp://54.183.132.22:6379 //client and server both are ec2 instances

redis_client:
  image: relateiq/redis-cli
  container_name: redis_client
  links:
    - ambassador:redis
  ports:
   - 6379:6379
</pre>

Command to run docker-compose.yml and to run the redis client after which you can perform simple SET/GET requests:
<pre>sudo docker-compose up -d
sudo docker-compose run redis_client</pre>


![Image2](https://github.com/shrenikgala/DevOpsHW4/blob/master/ambassador.gif)



3) **Docker Deploy**: Extend the deployment workshop to run a docker deployment process.

* A commit will build a new docker image.
* Push to local registery.
* Deploy the dockerized [simple node.js App](https://github.com/CSC-DevOps/App) to blue or green slice.
* Add appropriate hook commands to pull from registery, stop, and restart containers.

Post-commit Hook which will build a new docker image and push to local registry
<pre>
#!/bin/bash

sudo docker rmi ncsu-app
sudo docker rmi localhost:5000/ncsu:latest
sudo docker build -t ncsu-app .
sudo docker tag -f ncsu-app localhost:5000/ncsu:latest
sudo docker push localhost:5000/ncsu:latest
</pre>

Setting the following remotes after cloning app repo
<pre>
git remote add blue "file:///home/shrenik/Deployment/deploy/blue.git"
git remote add green "file:///home/shrenik/Deployment/deploy/green.git"
</pre>

Post receive hooks in blue.git and green.git folders respectively
<pre>
#!/bin/sh

GIT_WORK_TREE=/home/shrenik/Deployment/deploy/blue-www/ git checkout -f
sudo docker pull localhost:5000/ncsu:latest  
sudo docker stop app 
sudo docker rm app
sudo docker rmi localhost:5000/ncsu:current  
sudo docker tag -f localhost:5000/ncsu:latest localhost:5000/ncsu:current
sudo docker run -p 50100:8080 -d --name app localhost:5000/ncsu:latest 
</pre>
<pre>
#!/bin/sh

GIT_WORK_TREE=/home/shrenik/Deployment/deploy/green-www/ git checkout -f
sudo docker pull localhost:5000/ncsu:latest  
sudo docker stop app2  
sudo docker rm app2
sudo docker rmi localhost:5000/ncsu:current  
sudo docker tag -f localhost:5000/ncsu:latest localhost:5000/ncsu:current
sudo docker run -p 50101:8080 -d --name app2 localhost:5000/ncsu:latest 
</pre>

Commiting and deploying changes
<pre>
git push blue master
git push green master
</pre>

![Image3](https://github.com/shrenikgala/DevOpsHW4/blob/master/deployment.gif)
