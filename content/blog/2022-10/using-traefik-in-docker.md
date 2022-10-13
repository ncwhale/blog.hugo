---
title: "Using traefik as proxy in docker"
date: 2022-10-13T20:39:26+09:00
draft: true
tags: ["Tool", "traefik", "docker", "docker-compose"]
categories: ["Tool", "教程"]
isCJKLanguage: true
toc:
  enable: true
  auto: true
---

# 使用 Traefik 作为前端反代替换 nginx

## 前言

[Traefik](https://traefik.io/) 是一组 go 写的网络代理，不过这次我们只用它  Proxy 这一部分来替换 nginx 作为网页前端反代使用喵~（其实它还可以作为 HAProxy 的替换喵）

优点：

* `Drop in config` 完善支持，也就是你只需要设置 `watch: true` 然后直接修改目录里的配置文件，在不重启的前提下立即就能反应到服务配置里，这个比 `nginx -t` 反复重启可是进步太多了喵；
* `Label config` 通过直接给服务打 `label` 自动配置代理，无需复杂操作；


## 服务环境

因为 `k8s` 对于个人服务器来说太过折腾（虽然咱成功部署过，但是最终还是全拆了喵），所以咱的服务器仍旧是 `docker-compose` 启动的一堆服务配置文件，然后通过 `traefik` 以及共享桥接网络来与服务通信；

## 默认部署

不折腾的方式，但是效率会打折扣（因为多加了一层 `docker-proxy` 在前面）

`traefik/docker-compose.yaml`
```YAML
version: '3'

services:
  traefik:
    # The official v2 Traefik docker image
    image: traefik:v2.9
    container_name: traefik
    restart: always

    user: "user-id:group-id"

    ports:
      - "80:80"
      - "443:443"

    # Network configure.
    networks:
      proxy:
        aliases:
          # 这里添加别名，可以让其它服务直接通过域名经过反代直接访问其它服务
          - a.mydomain.com
          - b.mydomain.com
          - c.mydomain.com
          - d.mydomain.com
          - e.mydomain.com
          - f.mydomain.com
          - traefik.mydomain.com

    volumes:
      # So that Traefik can listen to the Docker events
      - /var/run/docker.sock:/var/run/docker.sock
      # Configure files/folders to drop in.
      - ./traefik.yml:/etc/traefik/traefik.yml:ro
      - ./config:/config:ro
      # Plugins folder
      - ./plugins-storage:/plugins-storage

    labels:
      - "traefik.enable=false"
  
networks:
  proxy:
    # 因为在 traefik 目录里，所以 docker-compose up 后会创建 traefik_proxy 网桥
    driver: bridge
```

注意，这个配置文件是唯一无法热替换的喵！ 
`traefik/traefik.yml`

```YAML
entryPoints:
  web:
    address: ":80"
    # 这里可以定义所有 http 都自动转 https，根据自身需要启用
    # 为了某些边界测试方便，我用了 Drop in config 来配置这项功能
    # http:
    #   redirections:
    #     entryPoint:
    #       to: websecure
    #       scheme: https

  websecure:
    address: ":443"
    http:
      tls:
        domains:
          - main: mydomain.com
            sans:
              - "*.mydomain.com"
          - main: mydomain2.com
            sans:
              - "*.mydomain2.com"

    http2:
      maxConcurrentStreams: 1000
    http3:
      advertisedPort: 443

providers:
  docker: 
    watch: true
    # 这里就是前文提到的网桥，最好需要代理的服务都走这个网桥通信。
    network: traefik_proxy
  file:
    # Dropin config folder
    directory: "/config"
    watch: true

api:
  # 开启 Traefik 内置面板，但是不使用不安全的 8008 服务
  # 编写一个 Dropin config 可以轻松反代出来，并且可以加权限校验
  dashboard: true
  insecure: false

experimental:
  plugins:
    # 将合法的 CF 代理头转换成 X-Forward 头，以便服务获取真实客户端地址喵
    cloudflarewarp:
      modulename: github.com/BetterCorp/cloudflarewarp
      version: v1.3.3
```

## 使用 Host network

默认的 `traefik:v2.9` 镜像是无法直接使用 `network_mode: host` 然后监听 1024 以下端口（80/443）的，此时需要一个 `setcap` 操作来赋予 posix 权限喵~ 

`traefik/build/Dockerfile`:
```Dockerfile
FROM traefik:v2.9

RUN apk update && apk add libcap && setcap cap_net_bind_service=+ep /usr/local/bin/traefik
```

Tips: 如果使用 zfs 作为 docker 的存储 driver， 则可能还需要 `sudo zfs set acltype=posixacl tank/docker` 来让 zfs 启用 posix acl 支持喵~

然后可以直接修订 `traefik/docker-compose.yaml`
```YAML
version: '3'

services:
  traefik:
    build: ./build
    container_name: traefik
    restart: always

    user: "uid:gid"

    # Docker 的 cap 修订
    cap_add: 
      - NET_BIND_SERVICE
      - NET_RAW

    # Network configure.
    # 因为直接绑定了 Host 网络，所以 ports 转发设置不需要了喵。
    network_mode: host

    volumes:
      # So that Traefik can listen to the Docker events
      - /var/run/docker.sock:/var/run/docker.sock
      # Configure files/folders to drop in.
      - ./traefik.yml:/etc/traefik/traefik.yml:ro
      - ./config:/config:ro
      # Plugins folder
      - ./plugins-storage:/plugins-storage

    labels:
      - "traefik.enable=false"
  
networks:
  proxy:
    # 这个 Proxy 桥可保留（作为其它服务配置的 default way)可不保留（因为 Host 基本能直接访问所有服务）
    driver: bridge
```

## 让 DNS 配置也自动化喵~

配置 `cloudflare-companion` 工具让它自动同步 `traefik` 的状态来更新 DNS 喵~这可是解放小服务器自己运维的一大乐趣喵~

`cloudflare-companion/docker-compose.yaml`
```YAML
version: '3'

services:

  cloudflare-companion:
    image: tiredofit/traefik-cloudflare-companion:latest
    container_name: cloudflare-companion
    user: "root:root"

    volumes:
      # 如果直接跟随 traefik 这个配置可以不需要喵
      - /var/run/docker.sock:/var/run/docker.sock:ro
    env_file:
      # 大部分配置放这个文件里喵
      - cf.env
    restart: always

    labels:
      # 这个服务不需要反向代理喵~
      - "traefik.enable=false"

networks:
  default:
    # 直接用 traefik 的网络作为默认网络
    name: traefik_proxy
    external: true
```

`cf.env`

```Conf
# Traefik config.
TRAEFIK_VERSION=2
ENABLE_TRAEFIK_POLL=TRUE
TRAEFIK_POLL_URL=http://traefik.mydomain.com #这个最好指向 traefik 服务能直接访问到的内部域名/地址
TRAEFIK_POLL_SECONDS=60

# CF config.
CF_EMAIL=
CF_TOKEN=Cloudflare API Token

# domain
TARGET_DOMAIN=traefik.mydomain.com

# mydomain.com
DOMAIN1=mydomain.com
DOMAIN1_ZONE_ID=这个ID去CF页面上找喵~
DOMAIN1_PROXIED=TRUE #是否需要启用 CF 代理，个人服务建议启用喵
DOMAIN1_TARGET_DOMAIN=traefik.mydomain.com #这个域名手动配置到指向 Traefik Host IP

# mydomain2.com
DOMAIN2=mydomain2.com
DOMAIN2_ZONE_ID=同上
DOMAIN2_PROXIED=FALSE
DOMAIN2_TARGET_DOMAIN=traefik.mydomain2.com

# docker config.
DOCKER_ENTRYPOINT=unix://var/run/docker.sock
SWARM_MODE=FALSE # 这个根据 Docker 是否启用 SWARM 模式来配置
# DOCKER_HOST=tcp://198.51.100.32:2376
# DOCKER_CERT_PATH=/docker-certs
# DOCKER_TLS_VERIFY=1

# Just in case.
DRY_RUN=FALSE #如果配置 Dry run 则不会做实际修改，适合拿来测试配置文件是否正确
REFRESH_ENTRIES=TRUE #是否修改已经存在的项目，慎用喵~
```