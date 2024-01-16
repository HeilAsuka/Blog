+++
title="使用 Cloudflared、Traefik 和 Docker 来托管网站而无需打开端口"
[extra]
author = "HeilAsuka"
[taxonomies]
tags = ["Cloudflare", "Traefik", "Docker", "自托管"]

+++
这是原始英文页面
```
https://gero.dev/blog/cloudflared-traefik-docker
```

用这种方法部署有以下几个好处
- 隐藏服务器ip
- 你甚至可以用家里的电脑部署
- 让cloudflare帮你过滤流量

你需要会创建cloudflare tunnel，下面的内容不想写了，我直接贴配置文件,你需要提前创建`cloudflaretunnel`这个network，用下面这个命令创建`sudo docker network create cloudflaretunnel`


`docker-compose.yml`
```yml
version: "3.9"
services:
  tunnel:
    container_name: cloudflared-tunnel
    image: cloudflare/cloudflared
    restart: always
    command: tunnel run
    environment:
      - TUNNEL_TOKEN=<YOUR TUNNEL TOKEN>
    networks:
      - cftunnel-transport

  traefik:
    image: traefik:2.9
    container_name: traefik
    restart: always
    networks:
      - cftunnel-transport
      - cloudflaretunnel
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./traefik.yaml:/traefik.yaml:ro
      - ./certificates.yaml:/certificates.yaml:ro
      - ./origin-certificates/:/origin-certificates:ro

networks:
  cftunnel-transport:
  cloudflaretunnel:
    external: true
```

`certificates.yaml`
```yml
tls:
  stores:
    default:
      defaultCertificate:
        certFile: /origin-certificates/mydomain.tld.pem #证书从Cloudflare下载
        keyFile: /origin-certificates/mydomain.tld.key #证书从Cloudflare下载

  certificates:
    - certFile: /origin-certificates/mydomain.tld.pem #证书从Cloudflare下载
      keyFile: /origin-certificates/mydomain.tld.key #证书从Cloudflare下载

```

`traefik.yaml`
```yml
log:
  level: DEBUG

entryPoints:
  websecure:
    address: ":443"

providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
  file:
    filename: certificates.yaml
```


测试用的`docker-compose.yml`
```yml
version: '3'

services:
  whoami:
    image: traefik/whoami
    container_name: whoami
    restart: always
    labels:
      - traefik.enable=true
      - traefik.http.routers.whoami.rule=Host(`用你自己的域名替换`)
      - traefik.http.routers.whoami.entrypoints=websecure
      - traefik.http.routers.whoami.tls=true
      - traefik.http.routers.whoami.service=whoami
      - traefik.http.services.whoami.loadbalancer.server.port=80 #端口是docker镜像开放的端口
    networks:
      - cloudflaretunnel

networks:
  cloudflaretunnel:
    external: true
```


{{ comment(text='想说啥就说，憋着不好。') }}
