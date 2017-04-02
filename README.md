# nginx-sts-exporter

![Docker Repository on Quay](https://quay.io/repository/marrotte/nginx-sts-exporter/status)
![Docker Pulls](https://img.shields.io/docker/pulls/marrotte/nginx-sts-exporter.svg)

Simple server that scrapes Nginx sts stats and exports them via HTTP for Prometheus consumption.

## Table of Contents
* [Dependency](#dependency)
* [Download](#download)
* [Compile](#compile)
* [Run](#run) 
* [Dockerized](#dockerized)
  * [Environment variables](#environment-variables)
  * [Docker Build](#docker-build)
  * [Docker Run](#docker-run)
* [Metrics](#metrics)
  * [Server main](#server main)
  * [Server zones](#server zones)
  * [Upstreams](#upstreams)

## Dependency

* [nginx-module-sts](https://github.com/vozlt/nginx-module-sts)
* [Prometheus](https://prometheus.io/)
* [Golang](https://golang.org/)

## Download

Binary can be downloaded from [Releases](https://github.com/marrotte/nginx-sts-exporter/releases) page.

## Compile

This shell script above will build a temp Docker image with the binary and then
export the binary inside ./bin/ directory

``` shell
./build-binary.sh
```

## Run

``` shell
nohup /bin/nginx-sts-exporter -nginx.scrape_uri=http://localhost/status/format/json
```

## Dockerized

To Dockerize this application yo need to pass two steps the build then the containerization.

### Environment variables

This image is configurable using different env variables

Variable name | Default     | Description
------------- | ----------- | --------------
NGINX_STATUS |  http://localhost/status/format/json | Nginx JSON format status page
METRICS_ENDPOINT | /metrics  | Metrics endpoint exportation URI
METRICS_ADDR | :9913 | Metrics exportation address:port
METRICS_NS | nginx | Prometheus metrics Namespaces

## Docker Hub Image
``` shell
docker pull sophos/nginx-sts-exporter:latest
```
It can be used directly instead of having to build the image yourself.
([Docker Hub sophos/nginx-sts-exporter](https://hub.docker.com/r/sophos/nginx-sts-exporter/))

### Docker Build

``` shell
./build-binary.sh
docker build -t sts-export .
```

### Docker Run

``` shell
docker run  -ti --rm --env NGIX_HOST="http://localhost/status/format/json" --env METRICS_NS="nginx_prod1" sts-export
```

## Metrics

Documents about exposed Prometheus metrics

### Server main

**Metrics details**

Nginx data         | Name                            | Exposed informations     
------------------ | ------------------------------- | ------------------------
 **Connections**   | `{NAMESPACE}_server_connections`| status [active, reading, writing, waiting, accepted, handled]

**Metrics output example**

``` txt
# Server Connections
nginx_server_connections{status="accepted"} 70606
```

### Server zones

**Metrics details**

Nginx data         | Name                            | Exposed informations     
------------------ | ------------------------------- | ------------------------
 **Requests**      | `{NAMESPACE}_server_requests`    | code [2xx, 3xx, 4xx, 5xx, total], host _(or domain name)_
 **Bytes**         | `{NAMESPACE}_server_bytes`       | direction [in, out], host _(or domain name)_
 **Cache**         | `{NAMESPACE}_server_cache`       | status [bypass, expired, hit, miss, revalidated, scarce, stale, updating], host _(or domain name)_

**Metrics output example**

``` txt
# Server Requests
nginx_server_requests{code="1xx",host="test.domain.com"} 0

# Server Bytes
nginx_server_bytes{direction="in",host="test.domain.com"} 21

# Server Cache
nginx_server_cache{host="test.domain.com",status="bypass"} 2
```

### Upstreams

**Metrics details**

Nginx data         | Name                            | Exposed informations     
------------------ | ------------------------------- | ------------------------
 **Requests**      | `{NAMESPACE}_upstream_requests` | code [2xx, 3xx, 4xx, 5xx and total], upstream _(or upstream name)_
 **Bytes**         | `{NAMESPACE}_upstream_bytes`    | direction [in, out], upstream _(or upstream name)_
 **Response time** | `{NAMESPACE}_upstream_response` | backend (or server), in_bytes, out_bytes, upstream _(or upstream name)_

**Metrics output example**

``` txt
# Upstream Requests
nginx_upstream_requests{code="1xx",upstream="XXX-XXXXX-3000"} 0

# Upstream Bytes
nginx_upstream_bytes{direction="in",upstream="XXX-XXXXX-3000"} 0

# Upstream Response time
nginx_upstream_response{backend="10.2.15.10:3000",upstream="XXX-XXXXX-3000"} 99
```
