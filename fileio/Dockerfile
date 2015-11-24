FROM    alpine:3.2

RUN apk update && \
    apk add socat && \
    rm -r /var/cache/

COPY . /src
RUN cd /src
RUN echo "Some text here." > /src/hello.txt;

CMD socat TCP4-LISTEN:9001 SYSTEM:'cat /src/hello.txt'
