---
categories:
- program
date: "2021-11-18T14:38:00Z"
title: confd+etcd实现nginx配置管理
---

`confd`是一个轻量级的配置管理工具，可以通过查询后端存储系统来实现第三方系统的动态配置管理，如`Nginx`、`Tomcat`、`HAproxy`、`Docker`配置等。

`confd`能够查询和监听后端系统的数据变更，结合配置模版引擎动态更新本地配置文件，保持和后端系统的数据一致，并且能够执行命令或者脚本实现系统的`reload`或者重启。

<!--more-->

##### 1、创建`confd`所需目录

`confd`配置文件默认在`/etc/confd`中，可以通过参数`-confdir`指定。目录中包含两个子目录，分别是：`conf.d`、`templates`

```shell
mkdir -p /etc/confd/{conf.d,templates}
```

##### 2、创建`confd`配置文件

`confd`会先读取`conf.d`目录中的配置文件（`toml`格式），然后根据文件指定的模板路径去渲染模板。

```shell
vim /etc/confd/conf.d/nginx.toml
```

内容为如下，其中`nginx.conf.tmpl`文件为`confd`的模板文件，`keys`为模板渲染成配置文件所需的配置内容，`/usr/local/nginx/conf/nginx.conf`为生成的配置文件

```toml
{%- raw -%}
[template]
src = "nginx.conf.tmpl"
dest = "/usr/local/nginx/conf/nginx.conf"
keys = [
"/nginx/conf",
]
check_cmd = "/usr/local/nginx/sbin/nginx -t -c {{ .src }}"
reload_cmd = "/usr/local/nginx/sbin/nginx -s reload"
{% endraw %}
```

##### 3、创建模板文件

拷贝`Nginx`原始的配置，增加对应的渲染内容

```shell
cp /usr/local/nginx/conf/nginx.conf /etc/confd/templates/nginx.conf.tmpl
vim /etc/confd/templates/nginx.conf.tmpl
```

增加内容为：

```ini
...
{%- raw -%}
{{- $data := json(getv "/nginx/conf")}}
{{- range $data.blackList}}
deny {{.}};
{{- end}}
{% endraw %}
...
```

##### 4、在`etcd`上创建所需的配置

使用`etcdctl`添加配置如下：

```shell
etcdctl set /nginx/conf {"blackList":["10.0.1.104","10.0.1.103"]}
```

##### 5、启动`confd`

启动`confd`，从`etcd`获取配置，渲染`Nginx`配置文件。`backend`设置成`etcd`，`node`指定访问的`etcd`服务地址，`watch`让confd支持动态监听

```shell
confd -backend etcd -node http://127.0.0.1:2379 -watch
```

##### 6、查看`Nginx`配置文件，验证`Nginx`启动

查看生成的`/usr/local/nginx/conf/nginx.conf`配置文件是否存在如下内容

```ini
...
deny 10.0.1.104;
deny 10.0.1.103;
...
```

curl命令访问`Nginx`，验证是否返回正常。`http`响应状态码为200说明访问`Nginx`正常

```shell
curl http://$IP:8000/ -i
HTTP/1.1 200 ok
...
```

##### 7、查看本机`IP`，加到`etcd`配置文件黑名单中

假设本机`IP`为10.1.70.20，将本机的`IP`加入到`Nginx`黑名单

```shell
etcdctl set /nginx/conf {"blackList":["10.0.1.104","10.0.1.103","10.1.70.20"]}
```

##### 8、查看`Nginx`配置文件，验证黑名单是否生效

查看生成的`/usr/local/nginx/conf/nginx.conf`配置文件是否存在如下内容

```ini
...
deny 10.0.1.104;
deny 10.0.1.103;
deny 10.1.70.20;
...
```

`curl`命令访问`Nginx`，访问应该被拒绝，返回403

```shell
curl http://$IP:8080/ -i
HTTP/1.1 403 Forbidden
...
```

