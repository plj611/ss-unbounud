# Running TCP Unbound DNS queries

### Introduction

[Unbound](https://nlnetlabs.nl/projects/unbound/about/) is a recursive and caching DNS resolver and can answer TCP DNS queries, it is available in many Linux distributions and in Openwrt which makes it a perfect choice for proof of setup to eliminate the UDP footprints in connections.

This document describes how to setup the server.

## Getting Started

### Prerequisites

- Docker, docker-compose. Install the version for your linux distribution.
- Linux

### Server folder structure

```
├── ss
│   └── config.json
├── sshost.yaml
├── unbound
   ├── a-records.conf
   ├── client.cnf
   ├── dev
   │   ├── null
   │   ├── random
   │   └── urandom
   ├── forward-records.conf
   ├── server.cnf
   ├── srv-records.conf
   ├── unbound.conf
   ├── unbound.conf.example
   ├── unbound.pid
   ├── unbound_control.key
   ├── unbound_control.pem
   ├── unbound_server.key
   ├── unbound_server.pem
   └── var
      └── root.key
```

### Setup

1. Git clone [repo](https://github.com/plj611/ss-unbounud.git) into your local directory. 

2. Create a ./ss/config.json for shadowsocks. The following keys value is provided and should not be modified (except "password" and "method").

   ```json
   { 
           "server": "10.0.0.5",
           "server_port": 8388,
           "local_port": 1080,
           "password": "<fill in value>",
           "method": "<fill in value>",
           "nameserver": "10.0.0.6",
           "mode": "tcp",
           "timeout": 60,
           "fastopen": true
   }
   ```

   Password and method must match the client's value. A complete description for the json config can be found in [here](https://manpages.debian.org/testing/shadowsocks-libev/shadowsocks-libev.8.en.html).

3. Setup the expose port for the shadowsocks service under the ports session in the sshost.yaml.

   ```yaml
     shadowsocks: 
       image: shadowsocks/shadowsocks-libev 
       environment:
   # Environment variables can be checked using docker inspect shadowsocks container
   #        - PASSWORD=Abarfoo!
   #        - METHOD=aes-256-cfb
   #        - DNS_ADDRS=10.0.0.6 
           - ARGS=-v -c /opt/config.json
           - TZ=Asia/Shanghai
       command: ["/bin/sh", "-c", "exec ss-server $${ARGS}"]
       ports: 
           - "7005:8388"
       networks:
           vnss:
               ipv4_address: 10.0.0.5 
       volumes:
           - ${PWD}/ss:/opt
   ```

   In here, tcp/7005 is exposed and should be opened in the linux host firewall. This is also the port that must match the client's value.

4. Open the port in step 3 in the linux host.

### Start up the server

```
# docker-compose -f sshost.yaml up -d
```

