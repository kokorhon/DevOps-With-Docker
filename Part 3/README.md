# Part 3

## 3.1

Link to repo with Heroku pipeline:
https://github.com/kokorhon/DevOps-pipeline

## 3.2
Not sure what to document of this but here goes :)

First I created this repository that I would then later update: https://github.com/kokorhon/DevOps-watchtower

Then I would build the image and publish it in: https://hub.docker.com/r/kostikorhonen/hello

Second I created this docker-compose.yml that would run the application image and watchtower:
```yml
version: "3"
services:
        hello:
                image: kostikorhonen/hello
                ports:
                        - "80:8080"
        watchtower:
                image: containrrr/watchtower
                volumes:
                        - /var/run/docker.sock:/var/run/docker.sock
                        - /home/kosti/.docker/config.json:/config.json
                command: --interval 10
```
Then I opened one terminal and ran the ```docker-compose up``` for this file.

Then I opened another terminal and build new docker image from this Dockerfile:
```Dockerfile
FROM ubuntu

COPY ./32script.sh .
COPY ./my_password.txt .

RUN apt-get update && \
    apt-get -qy full-upgrade && \
    apt-get install -qy curl && \
    apt-get install -qy curl && \
    curl -sSL https://get.docker.com/ | sh

RUN apt install -y git

CMD ["bash", "32script.sh"]
```
This image is used to clone/pull the git repo and then update the kostikorhonen/hello image in docker hub.

32script.sh looks like this:
```bash
git clone https://github.com/kokorhon/DevOps-watchtower.git;
cd ./DevOps-watchtower;
git pull;
cd ..;
docker build -t curling ./DevOps-watchtower;
docker run -it --name mycurl curling;
docker image tag curling kostikorhonen/hello;
cat ./my_password.txt | docker login --username kostikorhonen --password-stdin;
docker push kostikorhonen/hello;
```
The my_password.txt has my docker password in it 👀.

Now when the image is created with: ```docker build -t dockerindocker .```

And container run with: ```docker run -v /var/run/docker.sock:/var/run/docker.sock --name updatecontainer dockerindocker```

The thing should be set up and the watchtower should have noticed if there was a change to the kostikorhonen/hello image. When change is noticed watchtower reboots the application container with new information from the docker hub image.

**NOW** when we want to make new change to the kostikorhonen/hello image it should be automatically updated with wachtower after the ```updatecontainer``` has run.

I hope this fulfills the purpose of this exercise 😁

## 3.3

All that I changed was:

frontend Dockerfile:
```Dockerfile
FROM ubuntu
EXPOSE 5000
ENV REACT_APP_BACKEND_URL=http://127.0.0.1:8080/

WORKDIR ./usr/src/app

RUN apt-get update
RUN apt-get install -y curl
RUN curl -sL https://deb.nodesource.com/setup_16.x | bash
RUN apt-get install -y nodejs

COPY ./example-frontend .

RUN npm install
RUN npm run build
RUN npm install -g serve

RUN useradd -m appuser
USER appuser

CMD ["serve", "-s", "-l", "5000", "build"]
```

backend Dockerfile:
```Dockerfile
FROM golang:1.16
EXPOSE 8080

ENV REQUEST_ORIGIN=http://127.0.0.1:5000/

WORKDIR /usr/src/app

COPY . .

RUN go build

RUN test ./..

RUN useradd -m appuser
USER appuser

CMD ["./server"]
```

## 3.4

### At the start

frontend Dockerfile (566MB):
```Dockerfile
FROM ubuntu
EXPOSE 5000
ENV REACT_APP_BACKEND_URL=http://127.0.0.1:8080/

WORKDIR ./usr/src/app

RUN apt-get update
RUN apt-get install -y curl
RUN curl -sL https://deb.nodesource.com/setup_16.x | bash
RUN apt-get install -y nodejs

COPY ./example-frontend .

RUN npm install
RUN npm run build
RUN npm install -g serve

RUN useradd -m appuser
USER appuser

CMD ["serve", "-s", "-l", "5000", "build"]
```

backend Dockerfile (1.07GB):
```Dockerfile
FROM golang:1.16
EXPOSE 8080

ENV REQUEST_ORIGIN=http://127.0.0.1:5000/

WORKDIR /usr/src/app

COPY . .

RUN go build

RUN test ./..

RUN useradd -m appuser
USER appuser

CMD ["./server"]
```

### At the end

frontend Dockerfile (565MB):
```Dockerfile
FROM ubuntu
EXPOSE 5000
ENV REACT_APP_BACKEND_URL=http://127.0.0.1:8080/

WORKDIR ./usr/src/app

COPY ./example-frontend .

RUN apt-get update && apt-get install -y curl && \
    curl -sL https://deb.nodesource.com/setup_16.x | bash && \
    apt-get install -y nodejs && \
    npm install && \
    npm run build && \
    npm install -g serve && \
    useradd -m appuser

USER appuser

CMD ["serve", "-s", "-l", "5000", "build"]
```

backend Dockerfile(1.07GB):
```Dockerfile
FROM golang:1.16
EXPOSE 8080

ENV REQUEST_ORIGIN=http://127.0.0.1:5000/

WORKDIR /usr/src/app

COPY . .

RUN go build && test ./.. && useradd -m appuser

USER appuser

CMD ["./server"]
```

## 3.5

### At the start
frontend: 565MB

backend: 1.07GB

### At the end

frontend (FROM node:16-alpine): 421.26MB

backend (FROM golang:1.16-alpine): 449.6MB

## 3.6

### Part 1

frontend Dockerfile (178.41MB):
```Dockerfile
FROM node:16 as build

WORKDIR ./usr/src/app

COPY ./example-frontend .

RUN npm install && \
    npm run build

FROM node:alpine as main

COPY --from=build /usr/src/app/build /

RUN npm install -g serve && adduser -D appuser

USER appuser

ENV REACT_APP_BACKED_URL=http://127.0.0.1:8080/

EXPOSE 5000

CMD ["serve", "-s", "-l", "5000", "build"]
```

### Part 2

backend Dockerfile(13.09MB):
```Dockerfile
FROM golang:1.16-alpine as builder

ENV REQUEST_ORIGIN=http://127.0.0.1:5000/

WORKDIR /app

COPY . .

RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -ldflags="-w -s" . && adduser -D appuser

FROM scratch as main

WORKDIR /app

ENV REQUEST_ORIGIN=http://127.0.0.1:5000/

COPY --from=builder /etc/passwd /etc/passwd

COPY --from=builder /app/server /usr/bin/

USER appuser

EXPOSE 8080

CMD ["server"]
```

## 3.7

Dockerfile before optimization and security changes(911MB):
```Dockerfile
FROM node:16

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 8080

CMD ["node", "server.js"]
```

Dockerfile after optimization and security changes(112.35 MB):
```Dockerfile
FROM node:16-alpine as build

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install && adduser -D appuser

COPY . .

FROM node:16-alpine as main

WORKDIR /usr/src/app

COPY --from=build /etc/passwd /etc/passwd
COPY --from=build usr/src/app/server.js .
COPY --from=build usr/src/app/node_modules ./node_modules

USER appuser

EXPOSE 8080

CMD ["node", "server.js"]
```

## 3.8

I'll use my skip here.