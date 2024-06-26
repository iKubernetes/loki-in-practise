# Loki Simple Scalable Mode 部署示例

前提：部署有可用的docker-ce和docker-compose。

You can use this `docker-compose` setup to run Docker for development or in production.

## Features

- Running in [Simple Scalable Deployment](https://grafana.com/docs/loki/latest/fundamentals/architecture/deployment-modes/#simple-scalable-deployment-mode) mode with 3 replicas for `read` and `write` targets
- Memberlist for [consistent hash](https://grafana.com/docs/loki/latest/fundamentals/architecture/rings/) ring
- [Minio](https://min.io/) for S3-compatible storage for chunks & indexes
- nginx gateway which acts as a reverse-proxy to the read/write paths
- Promtail for logs
  - 带有三个日志生成器示例：loggen-json、loggen-apache-combined和loggen-apache-common，使用三个不同的job分别收集三个容器的日志
    - loggen-json：JSON格式的HTTP访问日志
    - loggen-apache-combined：文本格式的HTTP访问日志，combined格式
    - loggen-apache-commin：文本格式的HTTP访问日志，common格式
  - 还有一个job用于收集loki-read、loki-write和loki-backend相关容器的日志，日志格式为logfmt
- 支持多租户(tenant1 as the tenant ID)
- Configuration for interactive debugging (see [Debugging](#debugging) section below)
- Prometheus for metric collection

## Diagram

The below diagram describes the various components of this deployment, and how data flows between them.

```mermaid
graph LR
    Grafana --> |Query logs| nginx["nginx (port: 8080)"]
    Promtail -->|Send logs| nginx

    nginx -.-> |read path| QueryFrontend
    nginx -.-> |write path| Distributor

    subgraph LokiRead["loki -target=read"]
        QueryFrontend["query-frontend"]
        Querier["querier"]

        QueryFrontend -.-> Querier
    end

    subgraph Minio["Minio Storage"]
        Chunks
        Indexes
    end

    subgraph LokiWrite["loki -target=write"]
        Distributor["distributor"] -.-> Ingester["ingester"]
        Ingester
    end

    Querier --> |reads| Chunks & Indexes
    Ingester --> |writes| Chunks & Indexes
```

## Getting Started

Simply run `docker-compose up` and all the components will start.

It'll take a few seconds for all the components to start up and register in the [ring](http://localhost:8080/ring). Once all instances are `ACTIVE`, Loki will start accepting reads and writes. All logs will be stored with the tenant ID `docker`.

All data will be stored in the `.data` directory.

The nginx gateway runs on port `8080` and you can access Loki through it.

Prometheus runs on port `9090`, and you can access all metrics from Loki & Promtail here.

Grafana runs on port `3000`, and there are Loki & Prometheus datasources enabled by default.

## Endpoints

- [`/ring`](http://localhost:8080/ring) - view all components registered in the hash ring
- [`/config`](http://localhost:8080/config) - view the configuration used by Loki
- [`/memberlist`](http://localhost:8080/memberlist) - view all components in the memberlist cluster
- [all other Loki API endpoints](https://grafana.com/docs/loki/latest/api/)

## Debugging

First, you'll need to build a Loki image that includes and runs [delve](https://github.com/go-delve/delve).

Run `make loki-debug-image` from the root of this project. Grab the image name from the output (it'll look like `grafana/loki:...-debug`) and replace the Loki images in `docker-compose.yaml`.

Next, view the `docker-compose.yaml` file and uncomment the sections related to debugging.

You can follow [this guide](https://blog.jetbrains.com/go/2020/05/06/debugging-a-go-application-inside-a-docker-container/) to enable debugging in GoLand, but the basic steps are:

1. Bind a host port to one of the Loki services
2. Add a _Go Remote_ debug configuration in GoLand and use that port
3. Run `docker-compose up`
4. Set a breakpoint and start the debug configuration
5. Build/debug something awesome :)



## 版权声明

本项目由[马哥教育](http://www.magedu.com/)开发，允许自由转载，但必须保留马哥教育及相关的一切标识。另外，商用需要征得马哥教育的书面同意。