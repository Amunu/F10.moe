title: mongodb + robomongo + mongoose
date: 2014-11-11 16:21:34
tags: [mongodb, robomongo, mongoose, node]
---

最近一直在忙着重构，拎个文档就拎了十天左右的时间0. 0， 趁现在还没有 issues 过来的时间，赶紧过来把毕业设计要搭的东西给搭了。

### MongoDB linux

- 在 [MongoDB 官网](http://www.mongodb.org/downloads) 下载对应的版本

- 解压

- MongoDB的数据存储在data目录的db目录下，但是这个目录在安装过程不会自动创建，所以你需要手动创建data目录，并在data目录中创建db目录。 ** 注意：请将data目录创建于根目录下(/)。**

```sh

➜  /  sudo mkdir data
➜  /  cd data 
➜  /data  sudo mkdir db

```

- 执行mongo安装目录中的bin目录执行mongod命令来启动mongdb服务。 `./code/mongodb/bin/mongod` (我安装在了 code 文件夹里)

- MongoDB运行端口使用默认的27017，你可以在端口号为28017访问[web用户界面](http://localhost:28017)。


### robomongo 

robomongo 是比较常用的 MongoDB 的可视化工具，这里就用这个好了0. 0

- 在 [robomongo 官网](http://robomongo.org/) 下载对应版本， 我下的是 `.tar.gz` 的版本

- 解压

- 进入解压文件夹后

```sh
  $ cd bin
  $ chmod u+x robomongo.sh
  $ ./robomongo.sh
```

### mongoose

- $ npm install mongoose

这只就不具体说了0. 0， [戳这里](https://github.com/learnboost/mongoose)

就这样结束了么0。0 这篇会不会被吐槽太水了, 好饿！

