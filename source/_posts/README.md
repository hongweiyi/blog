title: README
---


# 建站 

使用 [hexo](https://hexo.io/zh-cn/)，快速开始如下：

```
$ npm install hexo-cli -g
$ hexo init blog
$ cd blog
$ npm install
$ hexo server
```


# 主题

使用了 [NexT](http://theme-next.iissnan.com/)，快速开始如下：

```
$ cd your-hexo-site
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```


由于墙的存在，建议直接到项目的 [github 主页](https://github.com/iissnan/hexo-theme-next) 直接下载 zip 包。


# 部署到 github pages

安装 git 部署插件即可，参考: [部署](https://hexo.io/zh-cn/docs/deployment.html)

* 安装插件


```
$ npm install hexo-deployer-git --save
```

* 在 `_config.yml` 中修改 deploy 内容

```
deploy:
  type: git
  repo: https://github.com/hongweiyi/hongweiyi.github.io.git
```

* 部署

```
$ hexo deploy
```


