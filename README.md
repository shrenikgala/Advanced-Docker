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
 
To run the type the following commands :
<pre> sudo docker build -t image1 .
sudo docker run -d -it --name container1 image1
</pre>
