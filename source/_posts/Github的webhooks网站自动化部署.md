---
title: Github的webhooks网站自动化部署
tags:
  - git
  - github
  - webhooks
  - 自动化部署
categories:
  - 文章
toc: true
date: 2019-09-09 07:56:41
---

# Github的webhooks进行网站自动化部署

![image.png](/images/2019/09/09/bb535690-d292-11e9-9a4a-2f1c09bb03b0.png)

相信很多码农都玩过了Git，如果对Git只是一知半解，可以移步LV写的 GIT常用操作总结，下面介绍到的一些关于 Git 的概念就不再赘述。

为啥想写这篇文章？主要是因为部门服务器因为安全性原因不允许SCP上传文件进行应用部署，然后有一些应用是放在Github上的，然后部署应用的步骤就变成：

1.git clone github项目 本地目录
2.配置一下应用的pm2.json并reload
3.Nginx配置一下反向代理并restart

当然如果只是一次性部署上去就不再修改的话并没啥问题，但是要是项目持续性修改迭代的话，就比较麻烦了，我们就在不断的重复着上面的步骤。作为一个码农，怎么允许不断的重复同样的工作，于是Github webhooks闪亮登场。

# 关于Github webhooks

- 必须是Github上面的项目
- 订阅了确定的事件（包括push/pull等命令）
- 自动触发

刚好符合了这几个条件，那接下来就看看如何进行网站自动化部署，主要会从下面几点来讲解：

1. 自动化shell脚本
2. 服务端实现
3. 配置github webhooks

# 自动化脚本

`auto_build.sh`

```shell
#! /bin/bash
SITE_PATH='/root/nginx/www'
cd $SITE_PATH
git reset --hard origin/master
git clean -f
git pull
git checkout master
```

Note: 在执行上面shell脚本之前我们必须第一次手动git clone项目进去

# 服务端实现

Github webhooks需要跟我们的服务器进行通信，确保是可以推送到我们的服务器，所以会发送一个带有X-Hub-Signature的POST请求，为了方便我们直接用第三方的库github-webhook-handler来接收参数并且做监听事件的处理等工作。

```shell
npm i github-webhook-handler -S
```

`index.js`

```js
var http = require('http');
var spawn = require('child_process').spawn;
var createHandler = require('github-webhook-handler');

// 下面填写的myscrect跟github webhooks配置一样，下一步会说；path是我们访问的路径
var handler = createHandler({ path: '/auto_build', secret: '' });

http.createServer(function (req, res) {
  handler(req, res, function (err) {
    res.statusCode = 404;
    res.end('no such location');
  })
}).listen(6666);

handler.on('error', function (err) {
  console.error('Error:', err.message)
});

// 监听到push事件的时候执行我们的自动化脚本
handler.on('push', function (event) {
  console.log('Received a push event for %s to %s',
    event.payload.repository.name,
    event.payload.ref);

  runCommand('sh', ['auto_build.sh'], function( txt ){
    console.log(txt);
  });
});

function runCommand( cmd, args, callback ){
    var child = spawn( cmd, args );
    var response = '';
    child.stdout.on('data', function( buffer ){ resp += buffer.toString(); });
    child.stdout.on('end', function(){ callback( resp ) });
}

// 由于我们不需要监听issues，所以下面代码注释掉
//  handler.on('issues', function (event) {
//    console.log('Received an issue event for %s action=%s: #%d %s',
//      event.payload.repository.name,
//      event.payload.action,
//      event.payload.issue.number,
//      event.payload.issue.title)
});
```

# 配置github webhooks

![image.png](/images/2019/09/09/3f507120-d294-11e9-9a4a-2f1c09bb03b0.png)

# 小结

上面就是利用Github webhooks进行网站自动化部署的全部内容了，不难发现其实这项技术还是有局限性的，那就是依赖于github，一般我们选择的都是免费github账号，所有项目都对外，一些敏感项目是不适合放置上去的。