title: SSH 资料
date: 2015-12-30 20:19:00
categories: 自学资料
tags: [SSH]
---

## 原理

### SSH 原理

1. 远程主机收到用户的登录请求，把自己的公钥发给用户
2. 用户使用这个公钥，将登录密码加密后，发送回来
3. 远程主机用自己的私钥，解密登录密码。如果密码正确，就同意用户登录

### 免密原理

1. 将用户的公钥存在远程主机
2. 登录时，远程主机通过用户的公钥加密一段随机的字符串发送回去
3. 用户用自己的私钥解密字符串后返回。如果字符串正确，就同意用户登录

### 为什么有 known_hosts

1. 恶意拦截登录请求，冒充远程主机，将伪造的公钥发送给用户
2. 用户拿到公钥后，加密密码发送过来。恶意拦截者就可以拿到密码

为了避免上面的情况，加上 known_hosts，可以防止以后使用的时候被恶意拦截，但是无法防止第一次被恶意拦截（第一次一般会把公钥指纹打印出来问一下，个人觉得作用不大，没人会看的）

## 操作

### 免询问

[ssh: automatically accept keys](http://askubuntu.com/questions/123072/ssh-automatically-accept-keys)

* ssh -oStrictHostKeyChecking=no user@host

> 注：这个在 mac 上不生效

或者在 ~/.ssh/config 中加

```
Host *
  StrictHostKeyChecking no
```

> 注：如果服务器变更了，这个 known_hosts 需要删掉重新生成。你也可以将 known_hosts 这个文件指向 /dev/null。使用 `-oUserKnownHostsFile=/dev/null`

### 免密

在需要免密登录其它机器的机器上执行：

* ssh-keygen -t rsa
* ssh user@host 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub

> `'cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub` 的作用是，将本地的公钥文件 `~/.ssh/id_rsa.pub`，重定向追加到远程文件 authorized_keys 的末尾
