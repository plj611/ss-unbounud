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

### Update 2021 Feb	

​	After the system is setup and firing up docker-compose to turn up the services, the unbound service may not run due to the missing of unbound/a-records.conf, unbound/forward-records.conf and unbound/srv-records.conf. 

​	That can be checked by checking the docker-compose logs. To correct this, add the files

​	unbound/a-records.conf

```
# A Record
     #local-data: "somecomputer.local. A 192.168.1.1"
# PTR Record
     #local-data-ptr: "192.168.1.1 somecomputer.local."     
```

​	unbound/forward-records.conf

```
forward-zone:
    # Forward all queries (except those in cache and local zone) to
    # upstream recursive servers
    name: "."
    # Queries to this forward zone use TLS
    forward-tls-upstream: yes

    # https://dnsprivacy.org/wiki/display/DP/DNS+Privacy+Test+Servers

    # Cloudflare
    forward-addr: 1.1.1.1@853#cloudflare-dns.com
    forward-addr: 1.0.0.1@853#cloudflare-dns.com
    #forward-addr: 2606:4700:4700::1111@853#cloudflare-dns.com
    #forward-addr: 2606:4700:4700::1001@853#cloudflare-dns.com

    # CleanBrowsing
    forward-addr: 185.228.168.9@853#security-filter-dns.cleanbrowsing.org
    forward-addr: 185.228.169.9@853#security-filter-dns.cleanbrowsing.org
    # forward-addr: 2a0d:2a00:1::2@853#security-filter-dns.cleanbrowsing.org
    # forward-addr: 2a0d:2a00:2::2@853#security-filter-dns.cleanbrowsing.org

    # Quad9
    # forward-addr: 9.9.9.9@853#dns.quad9.net
    # forward-addr: 149.112.112.112@853#dns.quad9.net
    # forward-addr: 2620:fe::fe@853#dns.quad9.net
    # forward-addr: 2620:fe::9@853#dns.quad9.net

    # getdnsapi.net
    # forward-addr: 185.49.141.37@853#getdnsapi.net
    # forward-addr: 2a04:b900:0:100::37@853#getdnsapi.net

    # Surfnet
    # forward-addr: 145.100.185.15@853#dnsovertls.sinodun.com
    # forward-addr: 145.100.185.16@853#dnsovertls1.sinodun.com
    # forward-addr: 2001:610:1:40ba:145:100:185:15@853#dnsovertls.sinodun.com
    # forward-addr: 2001:610:1:40ba:145:100:185:16@853#dnsovertls1.sinodun.com
```

​	unbound/srv-records.conf

```
# SRV records
# _service._proto.name. | TTL | class | SRV | priority | weight | port | target.
```



