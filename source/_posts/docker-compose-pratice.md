title: Docker Compose 最佳实践
date: 2015-10-08 20:00:00
categories: 最佳实践
tags: [Docker, Docker Compose]
---

## docker-compose 小功能

* daemon 模式

`docker-compose up -d`

* 设置 container 的名字

显式设置 container 的名字：[Issue 讨论](https://github.com/docker/compose/pull/1711), [yml.md: container_name](https://github.com/docker/compose/blob/master/docs/yml.md#container_name)

    * 设置了这个的话，scale 能力就无法使用了
    * container 名字默认格式：${PROJECT}-${NAME}-${sacle_num}

## docker-compose 的问题

* 无法给 yml 传递参数 [Issue 讨论](https://github.com/docker/compose/issues/1377)，好像有[解决方案](https://github.com/docker/compose/blob/master/docs/yml.md#variable-substitution)，不过不好用
* 无法给 container 之间加依赖 [Issue](https://github.com/docker/compose/issues/374)
* 仅能控制多个 container 的启动和关系，如果有初始化任务（如DB 初始化），还需要额外写脚本文件
* docker-compose 和 docker-swarm 集成还在开发中：https://github.com/docker/compose/blob/master/SWARM.md
