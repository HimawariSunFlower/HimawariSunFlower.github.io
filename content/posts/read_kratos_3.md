---
title: "go-kratos 微服务必须的功能及第三方库"
date: 2022-09-13T15:08:57+08:00
draft: false
categories: ["go-kratos"]
---
  

服务发现   [consul](https://github.com/hashicorp/consul)
```
    基于name和id来区分不同注册
    如果name或id相同，会互相覆盖
```         
<br/>

链路追踪   [jaeger](https://github.com/jaegertracing/jaeger)
```
kratos的中间件通过[open-telemetry]集成数据再传递给jaeger  
```
<br/>

Metrics [open-telemetry](https://github.com/open-telemetry/opentelemetry-go) + [Grafana](https://github.com/grafana/grafana) + [Prometheus](https://github.com/prometheus/prometheus)

<br/>

...更多待添加