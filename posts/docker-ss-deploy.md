---
title: "利用Docker部署shadowsocks和simple-obfs"
date: "2018-09-14 14:59:06"
---

## 简介

> `Shadowsocks-libev`: Shadowsocks-libev is a lightweight secured SOCKS5 proxy for embedded devices and low-end boxes.

> `Simple-obfs`: Simple-obfs is a simple obfusacting tool, designed as plugin server of shadowsocks.

> `Docker`: Docker is an open platform for developers and sysadmins to build, ship, and run distributed applications, whether on laptops, data center VMs, or the cloud.

## 编写 Dockerfile

首先编写 `shadowsocks-libev` 的 `Dockerfile`

```dockerfile
#
# Dockerfile for shadowsocks-libev
#

FROM alpine

RUN set -ex \
    # Build environment setup
    && apk add --no-cache --virtual .build-deps \
    autoconf \
    automake \
    build-base \
    c-ares-dev \
    libev-dev \
    libtool \
    libsodium-dev \
    linux-headers \
    mbedtls-dev \
    pcre-dev \
    git \
    # Build and install
    && cd /tmp \
    && git clone https://github.com/shadowsocks/shadowsocks-libev.git \
    && cd shadowsocks-libev && git submodule update --init --recursive \
    && ./autogen.sh \
    && ./configure --prefix=/usr --disable-documentation \
    && make install \
    && apk del .build-deps \
    # Runtime dependencies setup
    && apk add --no-cache \
    rng-tools \
    $(scanelf --needed --nobanner /usr/bin/ss-* \
    | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
    | sort -u) \
    && rm -rf /tmp/*

ENTRYPOINT ["ss-server"]
```

接下来是 `simple-obfs` 的 `Dockerfile`

```dockerfile
#
# Dockerfile for shadowsocks-libev-simple-obfs
#

FROM alpine

RUN set -ex \
    # Build environment setup
    && apk add --no-cache --virtual .build-deps gcc autoconf make libtool automake zlib-dev openssl asciidoc xmlto libpcre32 libev-dev g++ linux-headers git \
    # Build and install
    && cd /tmp \
    && git clone https://github.com/shadowsocks/simple-obfs.git \
    && cd simple-obfs \
    && git submodule update --init --recursive \
    && ./autogen.sh \
    && ./configure --prefix=/usr --disable-documentation \
    && make && make install \
    && apk del .build-deps \
    # Runtime dependencies setup
    && apk add --no-cache \
    $(scanelf --needed --nobanner /usr/bin/obfs-* \
    | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
    | sort -u) \
    && rm -rf /tmp/*

ENTRYPOINT ["obfs-server"]
```

## 运行 docker build 和 docker push

在 `shadowsocks-libev` 和 `simple-obfs` 的 `Dockerfile` 目录下分别运行

```
   docker build ./Dockerfile -t vermouthx/shadowsocks-libev
   docker build ./Dockerfile -t vermouthx/simple-obfs
```

将 build 好的 image push 到 docker cloud

```
   docker push vermouthx/shadowsocks-libev
   docker push vermouthx/simple-obfs
```

## 运用 Ansible 实现 VPS 自动化部署

关于`Ansible`：`Ansible`是一个自动化运维工具，具体可看[官网](https://www.ansible.com/)

编写 `Ansible Playbook`的相关`tasks`，关键代码如下

```yaml
- name: deploy shadowsocks-libev
  become: yes
  docker_container:
    name: ss-server
    image: vermouthx/shadowsocks-libev
    volumes:
      - "/etc/shadowsocks-libev:/etc/shadowsocks-libev"
    entrypoint: ss-server
    command: "-c /etc/shadowsocks-libev/config.json"
    ports:
      - "9999:9999/udp"
    user: nobody
    networks:
      - name: shadowsocks
    state: "{{ container_state }}"
    recreate: yes
  tags: ss
- name: deploy simple-obfs
  become: yes
  docker_container:
    name: obfs-server
    image: vermouthx/simple-obfs
    volumes:
      - "/etc/simple-obfs:/etc/simple-obfs"
    command: "-c /etc/simple-obfs/config.json"
    ports:
      - "9999:8139"
    user: nobody
    networks:
      - name: shadowsocks
    state: "{{ container_state }}"
    recreate: yes
  tags: obfs
```

运行该 `playbook` 即可完成`shadowsocks`+`simple-obfs`的部署

`Dockerfile`的源码：https://github.com/WhiteVermouth/shadowsocks-docker
