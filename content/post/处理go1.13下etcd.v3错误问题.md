---
title: "处理go1.13下etcd"
date: 2020-02-11T16:18:55+08:00
tags: ["go"]
---
处理etcd.v3在go 1.13错误问题
<!--more-->
# 处理go1.13下etcd.v3错误问题

在go v1.13拉取etcd会报下面的错误

> ```
> build command-line-arguments: cannot load github.com/coreos/go-systemd/journal: no matching versions for query "latest"
> ```

解决上面的问题只要将go.mod里面添加一行代码

> 　replace github.com/coreos/go-systemd => github.com/coreos/go-systemd/v22 v22.0.0

然后再调用go get go.etcd.io/etcd/clientv3来拉取相应的库，这里你会发现又会出现下面的错误

> ```
>  github.com/coreos/etcd/clientv3/balancer/resolver/endpoint
> ../../go/pkg/mod/github.com/coreos/etcd@v3.3.18+incompatible/clientv3/balancer/resolver/endpoint/endpoint.go:114:78: undefined: resolver.BuildOption
> ../../go/pkg/mod/github.com/coreos/etcd@v3.3.18+incompatible/clientv3/balancer/resolver/endpoint/endpoint.go:182:31: undefined: resolver.ResolveNowOption
> # github.com/coreos/etcd/clientv3/balancer/picker
> ../../go/pkg/mod/github.com/coreos/etcd@v3.3.18+incompatible/clientv3/balancer/picker/err.go:37:44: undefined: balancer.PickOptions
> ../../go/pkg/mod/github.com/coreos/etcd@v3.3.18+incompatible/clientv3/balancer/picker/roundrobin_balanced.go:55:54: undefined: balancer.PickOptions
> ```

后来查了很久才发现etcd 3.3.18与grpc的版本有冲突，需要将grpc降级到1.26版本

> go get google.golang.org/grpc@v1.26.0

这时候你再拉取就不会再出现任何错误了