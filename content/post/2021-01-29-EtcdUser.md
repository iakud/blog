---
categories:
- program
date: "2021-01-29T16:08:07Z"
title: Etcd用户和权限设置
typora-root-url: ..
---

**创建root用户**

```bash
etcdctl --user root user add root
etcdctl --user root user grant-role root root
```

添加root用户的时候会要求指定密码
<!--more-->

**创建普通用户**

```bash
etcdctl --endpoints=http://127.0.0.1:2379 --user=root:password user add akai
```

**添加角色**

```bash
etcdctl --endpoints http://127.0.0.1:2379 --user=root:password role add normal
```

**角色授权**

```shell
etcdctl --endpoints http://127.0.0.1:2379 --user=root:password role grant-permission --prefix=true nurmal readwrite /path
```

**用户绑定角色**

```shell
etcdctl --endpoints http://127.0.0.1:2379 --user=root:password user grant-role akai normal
```

**验证**

```shell
etcdctl --endpoints=http://127.0.0.1:2379 --user=akai:password put /test 'hello world'
etcdctl --endpoints=http://127.0.0.1:2379 --user=akai:password get /test
/test
hello world
```

