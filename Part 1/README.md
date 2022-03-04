# Part 1

## 1.1
![1.1](pics/1.1.PNG)

## 1.2
![1.2](pics/1.2.PNG)

## 1.3
Commands:
* docker run devopsdockeruh/simple-web-service:ubuntu
* docker container ls
* docker start unruffled_kirch
* docker exec -it unruffled_kirch bash
* tail -f ./text.log

Scret message is: 'You can find the source code here: https://github.com/docker-hy'

## 1.4

```bash 
kosti@tietokone:~$ docker run -it ubuntu sh -c 'apt-get update; apt-get install curl -y; echo "Input website:"; read website; echo "Searching.."; sleep 1; curl http://$website;'
...
Input website:
helsinki.fi
Searching..
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>301 Moved Permanently</title>
</head><body>
<h1>Moved Permanently</h1>
<p>The document has moved <a href="https://www.helsinki.fi/">here</a>.</p>
</body></html>
```

## 1.5

```bash
kosti@tietokone:~$ docker run devopsdockeruh/simple-web-service:alpine
kosti@tietokone:~$ docker run devopsdockeruh/simple-web-service:ubuntu
kosti@tietokone:~$ docker images
REPOSITORY                          TAG       IMAGE ID       CREATED         SIZE
devopsdockeruh/simple-web-service   ubuntu    4e3362e907d5   11 months ago   83MB
devopsdockeruh/simple-web-service   alpine    fd312adc88e0   11 months ago   15.7MB
kosti@tietokone:~$ docker container ls
CONTAINER ID   IMAGE                                      COMMAND                 CREATED          STATUS         PORTS     NAMES
f230bda9b617   devopsdockeruh/simple-web-service:alpine   "/usr/src/app/server"   10 seconds ago   Up 9 seconds             recursing_ganguly
kosti@tietokone:~$ docker exec -it recursing_ganguly sh
/usr/src/app # ls
server    text.log
/usr/src/app # tail ./text.log
2022-03-04 13:16:20 +0000 UTC
2022-03-04 13:16:22 +0000 UTC
2022-03-04 13:16:24 +0000 UTC
2022-03-04 13:16:26 +0000 UTC
Secret message is: 'You can find the source code here: https://github.com/docker-hy'
```

## 1.6

Found the password from: https://hub.docker.com/r/devopsdockeruh/pull_exercise
```bash
kosti@tietokone:~$ docker run -it devopsdockeruh/pull_exercise
Give me the password: basics
You found the correct password. Secret message is:
"This is the secret message"
```

## 1.7
Dockerfile:
```Dockerfile
FROM devopsdockeruh/simple-web-service:alpine
CMD server
```
Commands:
```bash
kosti@tietokone:~$ docker build . -t web-server
kosti@tietokone:~$ docker run web-server
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /*path                    --> server.Start.func1 (3 handlers)
[GIN-debug] Listening and serving HTTP on :8080
```

## 1.8
curl.sh:
```sh
echo "Input website:"; read website; echo "Searching.."; sleep 1; curl http://$website;
```

Dockerfile:
```Dockerfile
FROM ubuntu:20.04

WORKDIR /usr/src/app

COPY curl.sh .

RUN chmod +x curl.sh
RUN apt-get update
RUN apt-get install curl -y

CMD ./curl.sh
```