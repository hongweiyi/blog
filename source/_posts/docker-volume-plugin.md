title: Docker 插件 - Volume plugins
date: 2015-10-14 22:05:00
categories: 自学资料
tags: [Docker, Docker plugins, Docker volume]
---

## Docker 插件是什么

docker 插件是 docker 提供出来的扩展机制，目前 docker 支持 volume 和 network 两种插件，由于 network 插件比较复杂而且没有好的开源项目，这里主要介绍 volume 插件。

插件是一个独立的进程和 docker daemon 运行在同一台 host 上，通过 Plugin Discovery 的机制进行插件发现，插件有几个要求：

* 插件名要求是小写
* 插件可以运行在容器内也可以运行在容器外，不过现阶段建议运行在容器外

<!-- more -->

## 插件发现

插件发现机制需要插件将自己的地址文件放在固定目录，方便 docker 发现插件进程，有三种文件可以设置：

* `.sock` 文件是 UNIX domain sockets
* `.spec` 文本文件内包含了一个 URL，比如：`unix:///other.sock`
* `.json` 文本文件包含了插件的完整 JSON 描述

UNIX domain socket 文件必须放在 `/run/docker/plugins` 目录，但是 `.spec`，`.json` 文件则可以放在 `/etc/docker/plugins` 或者 `/usr/lib/docker/plugins` 中。

无后缀的文件名决定了插件的名字，比如 `/run/docker/plugins/myplugin.sock` 的插件名就是 `myplugin`。你可以在子目录中放置地址文件，比如 `/run/docker/plugins/myplugin/myplugin.sock`。

docker 优先搜索 `/run/docker/plugins` 目录，如果没有 unix socket 的话才会去搜索 `/etc/docker/plugins` 和 `/usr/lib/docker/plugins`，如果根据指定插件名搜到了插件就会立马停止搜索。

### `.json`

JSON 格式文件示例：

```
{
  "Name": "plugin-example",
  "Addr": "https://example.com/docker/plugin",
  "TLSConfig": {
    "InsecureSkipVerify": false,
    "CAFile": "/usr/shared/docker/certs/example-ca.pem",
    "CertFile": "/usr/shared/docker/certs/example-cert.pem",
    "KeyFile": "/usr/shared/docker/certs/example-key.pem",
  }
}
```

## 插件生命周期

* 启动插件
* 启动 docker
* 停止 docker
* 停止插件


## 插件激活

运行命令 `docker run --volume-driver=foo` 即可以激活名为 `foo` 的 volume 插件，需要注意的是，插件是按需加载机制，只有被使用到了才会被激活。

## volume 插件使用

示例：

```
$ docker run -ti -v volumename:/data --volume-driver=flocker busybox sh
```

上面表示的意思是，使用 flocker 插件将 voluemname 挂载到容器的 /data 目录。

注意：volumename 一定不能以 `/` 开头。（文档说的，没看 docker 源码，我实现一个以 `/` 开头好像也没问题，应该是规范吧）

## 插件 API 设计

插件是 API 是基于 HTTP 的 JSON POST 请求，所以插件需要实现一个 HTTP 服务器并且将其 bind 到一个 UNIX socket 上。API 的版本设置在了 HTTP
头里面，现在这个头的固定值为：`application/vnd.docker.plugins.v1+json`

不过 docker 的开发人员已经提供了一个比较好的 docker volume 的扩展 API 代码，可以参考：[docker-volume-extension-api](https://github.com/calavera/dkvolume)

### `/Plugin.Activate`

请求：空

响应：

```json
{
  "Implements:" ["VolumeDriver"]
}
```

返回插件实现，表示是 volume 插件

### `/VolumeDriver.Create`

请求：

```
{
    "Name": "volume_name"
}
```

告诉插件用户想要创建一个 volume，并将用户输入的 volume 名传给插件。插件在这个时候可以不用理会这个请求，会有真正挂载的请求。

响应：

```
{
    "Err": null
}
```

如果出错返回错误字符串。

### `/VolumeDriver.Remove`

与 Create 相对应。

请求：

```
{
    "Name": "volume_name"
}
```

响应：

```
{
    "Err": null
}
```


### `/VolumeDriver.Mount`

请求：

```
{
    "Name": "volume_name"
}
```

用户请求挂载某个文件，这个请求仅会在容器启动时发送一次。

响应：

```
{
   "Mountpoint": "/path/to/directory/on/host",
   "Err": null
}
```

将 volume_name 挂载的真正挂载点返回给 docker，如果出错则返回错误字符串。


### `/VolumeDriver.Path`

请求：

```
{
    "Name": "volume_name"
}
```

响应：

```
{
   "Mountpoint": "/path/to/directory/on/host",
   "Err": null
}
```

插件需要管理 volume_name 的真正挂载地址，这个请求需要将 volume_name 挂载的真正挂载点返回给 docker，如果出错则返回错误字符串。

### `/VolumeDriver.Unmount`

请求：

```
{
    "Name": "volume_name"
}
```

表示 docker 已经不需要这个 volume 了，插件需要安全的将这个挂载从挂载点卸载。

响应：

```
{
    "Err": null
}
```


> 参考地址：[Understand Docker plugins](https://docs.docker.com/extend/plugins/)
