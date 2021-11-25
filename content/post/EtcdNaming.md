---
categories:
- program
date: "2021-03-10T15:05:11Z"
title: Etcd服务发现实现
typora-root-url: ..
---

本文介绍`gRPC`接入`etcd`，实现服务注册与服务发现。

<!--more-->

服务端实现：

```go
import (
    "flag"
    "log"
    "net"
    
    "github.com/golang/protobuf/proto"
    "go.etcd.io/etcd/client/v3"
    "go.etcd.io/etcd/client/v3/concurrency"
    "go.etcd.io/etcd/client/v3/naming/endpoints"
    "google.golang.org/grpc"
)

const service = "/my-service"
var addr string
var endpoint string = "127.0.0.1:2379"

func main() {
    flag.StringVar(&addr, "a", "", "addr")
    flag.Parse()
    // etcd建立连接
    client, err := clientv3.New(clientv3.Config{Endpoints: []string{endpoint}})
    if err != nil {
        log.Fatalln(err)
    }
    session, err := concurrency.NewSession(client)
    if err != nil {
        log.Fatalln(err)
    }
    // 服务注册
    em, err := endpoints.NewManager(client, service)
    if err != nil {
        log.Fatalln(err)
    }
    em.AddEndpoint(client.Ctx(), service+"/"+addr, endpoints.Endpoint{Addr: addr}, clientv3.WithLease(session.Lease()))
    defer em.DeleteEndpoint(client.Ctx(), service+"/"+addr)
    // gRPC服务
    l, err := net.Listen("tcp", addr)
    if err != nil {
        log.Fatalln(err)
    }
    log.Println("listen:", addr)
    server := grpc.NewServer()
    // pb.RegisterTestServer(server, &Test{})
    server.Serve(l)
}
```

**客户端实现：**

直接使用`etcd`实现的`Resolver`

```go
import (
    "log"
    
    "github.com/golang/protobuf/proto"
    "go.etcd.io/etcd/client/v3"
    "go.etcd.io/etcd/client/v3/naming/resolver"
    "google.golang.org/grpc"
    "google.golang.org/grpc/balancer/roundrobin"
)
const service = "/my-service"
var endpoint string = "127.0.0.1:2379"

func main() {
    // etcd建立连接
    client, err := clientv3.New(clientv3.Config{Endpoints: []string{endpoint}})
    if err != nil {
        log.Fatalln(err)
    }
    // gRPC服务发现和负载均衡
    etcdResolver, err := resolver.NewBuilder(client)
    if err != nil {
        log.Fatalln(err)
    }
    conn, err := grpc.Dial(etcdResolver.Scheme()+":///"+service, grpc.WithInsecure(), grpc.WithResolvers(etcdResolver), grpc.WithBalancerName(roundrobin.Name))
    if err != nil {
        log.Fatalln(err)
    }
    // do(conn)
}
```

同时实现了服务发现和负载均衡。