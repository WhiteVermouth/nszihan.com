---
title: "Deploy and Debug Tomcat with Docker in IDEA"
date: "2018-09-28 19:37:37"
---

## Build Docker Image

Create `docker-compose.yml` and edit like following codes:

```yml
tomcat:
  image: "tomcat:latest"
  ports:
    - "127.0.0.1:8000:8000"
    - "127.0.0.1:8888:8080"
  environment:
    - JPDA_ADDRESS=8000
    - JPDA_TRANSPORT=dt_socket
  volumes:
    - /path/to/war/file/directory:/usr/local/tomcat/webapps
  container_name: tomcat-literature
  command: "/usr/local/tomcat/bin/catalina.sh jpda run"
  restart: always
```

The key point in the config file is the two environment variables:

```
JPDA_ADDRESS=8000
JPDA_TRANSPORT=dt_socket
```

### Caveat

You may set the `JPDA_ADDRESS` value `*:8000`, which makes `Tomcat` listen on `0.0.0.0` other than `127.0.0.1`.

## Run Docker Container

In the directory of `docker-compose.yml`, execute command:

```
docker-compose up -d
```

## Debug in IDEA

1. add a Remote Run Configuration

![](/images/idea/docker-tomcat-remote.png)

2. set a breakpoint in your program anywhere you like

3. swith to Run Configuration which setup in first step, and click debug
