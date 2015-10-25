## 使用 deploy-asset 的 CLI

## .darc

## 调试


## 安装

```bash
npm install --global deploy-asset
```

## 使用

### 命令行

```bash

da <folder> [options]

```  

**请使用 `da --help` 来查看所有支持的配置**


#### 使用命令行的例子：（要先配置好 uploaders）

* 发布一个单独的图片文件到七牛上去

  `da path/to/example.png --uploader qiniu`

* 发布当前目录下所有的 png 图片

  `da "*.png"`  注意如果使用了通配符，一定要加上引号，否则很容易出错

* 发布当前目录下的 index.html 文件

  如果 index.html 引用了 app.js 和 app.css，而 app.css 引用了 hero.png 图片，即它们关系是
    ```
    index.html
      | -- scripts
            | -- app.js
      | -- styles
            | -- app.css
      | -- images
            | -- hero.png
    ```

  在这种情况下，你还是只需要执行 `da index.html`，它会自动找到文件的引用关系，将它们上传到服务器，并根据服务器的路径，修改它们的引用关系    

  执行后的结果可能是：
    ```
    ✓         index.html => http://7u2rjq.com1.z0.glb.clouddn.com/index-d85f46.html
    ✓     scripts/app.js => http://7u2rjq.com1.z0.glb.clouddn.com/app-2bd0f1.js
    ✓     styles/app.css => http://7u2rjq.com1.z0.glb.clouddn.com/app-d41d8c.css
    ✓    images/hero.png => http://7u2rjq.com1.z0.glb.clouddn.com/hero-d41d8c.png
    ```


### Node 脚本

```javascript
var da = require('deploy-asset')

da(file_or_folder, options, function(err, fileMap) {
  // ...
})
```

<!-- 

// 查看配置参数 [https://qiu8310.github.io/deploy-asset/global.html#da](https://qiu8310.github.io/deploy-asset/global.html#da)


### Grunt

[grunt-deploy-asset](https://github.com/qiu8310/grunt-deploy-asset)



## Options

[Click Here To See All Options](https://qiu8310.github.io/deploy-asset/global.html#da)
 
-->

## 配置文件

* 配置文件使用的库是 [rc](https://github.com/dominictarr/rc)，它会自动在你项目所在的目录及各级父目录和当前用户根目录下查找 `.darc` 文件，将所有找到的文件合并（最先找到的会覆盖后面找到的）。

* 此项目主要需要配置的选项是 `uploaders`，其它都可以通过命令行配置，下面是一个 ftp 的配置例子：

  在你的用户根目录新建一个 `.darc` 文件，并填入下面代码（注意去掉代码中的注释）
  ```json
  {
    "uploader": "ftp_stage",  // 默认的 上传器
    "uploaders": {
      "ftp_stage": {
        "alias": "ftp",   // 这个一定要写，告诉系统这是个 ftp 的配置
        "host": "....",
        "user": "...",
        "pass": "...",
        "port": 21,
        "baseUrl": "http://www.you_server/welcome/abc", // 这里只是个例子，根据你的实际情况修改
        "destDir": "welcome/abc"
      },
      "ftp_prod": {
        "alias": "ftp", 
        "host": "....",
        "user": "...",
        "pass": "...",
        "port": 21,
        "baseUrl": "...",
        "destDir": "..."
      }
    }
  }
  ```
  这样配置好了之后你就可以使用 `da .` 来将文件发布到 stage 的 ftp 上，而运行 `da . --uploader ftp_prod` 来将文件发布到 production 环境上去。


* 另外支持的 七牛 和 又拍云的配置格式如下（可以将所有相关的配置都放在 `uploaders` 中，这里是为了说明才分开说的）：

  ```json
  {
    "uploaders": {
      "qiniu": {
        "alias": "qiniu",  // 如果 alias 的值，和它对应在 uploaders 下的 key 的值一样（此处都是 qiniu），则可忽略 alias
        "ak": "...",
        "sk": "...",
        "domain": "...",
        "bucket": "..."
      },
      "upyun": {
        "operator": "...",
        "password": "...",
        "bucket": "...",
        "domain": "...",
        "destDir": "",  // 可以不填，默认就会上传到 bucket 的根目录下
        "endpoint": "", // 可以不填，默认是 v0
        "apiVersion": "" // 可以不填，默认是 legacy
      }
    }
  }
  ```



## 注意事项

* 如果在 JS 或 JSON 中要引用某个文件，只要将它放在字符串中，保证带有至少一个路径（`/` 或 `\`），并且有后缀名，如 `.js`

* __使用命令行时如果用了 glob，一定要加引号__ 

  比如，如果你想排除所有图片文件用此命令：`da --excludes *.png`，但 *.png 会被 shell 解析成可能的 a.png b.png c.png，所以你的命令变成了：
  `da --excludes a.png b.png c.png` 并不是你想要的结果。因而，需要加上引号写成 `da --excludes '*.png'`


## 调试

使用的是 [npmlog](https://github.com/isaacs/npmlog)，支持 6 种不同的日志级别，
分别是： `silly`， `verbose`， `info`， `warn`， `error`， `silent`，越后面的
输出的日志信息就越少。

可以通过以下两种方式来设置日志级别：

* 通过参数： `--log-level` 或 `-l`，如 `--log-level=verbose` 或 `-l verbose`，
  另外支持三种简写形式，`--verbose`，`--info`，`--silent`

* 通过环境变量：类似于 [debug](https://github.com/visionmedia/debug) 的写法，如
  `DEBUG=da:verbose` 或 `DEBUG=da:info`
