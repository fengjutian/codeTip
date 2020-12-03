# Nodejs on Docker

这个例子的目的是告诉你如何把一个Node.js应用程序放入到Docker容器。 该指南旨在用于开发，而不是用于生产部署。 该指南还假定你有安装Docker的经验，且对Node.js应用程序是如何构建的,有一个基本的了解.

在本指南的第一部分，我们将用Node.js创建一个简单的Web应用程序, 然后我们将为该应用程序建立一个Docker映像， 最后我们将运行这个镜像作为一个容器。

Docker允许你把一个应用和它的所有依赖打包成一个标准化单元,称为一个容器(container),用于软件开发. 每一个容器(container)都是一个精简到基本版本的Linux操作系统。 镜像(image)是你加载到一个容器中的软件。

创建 Node.js 应用程序
首先，创建一个新目录,所有文件将会位于其中。 在此目录中，创建一个package.json文件，描述你的应用程序及其依赖：

`
{
  "name": "docker_web_app",
  "version": "1.0.0",
  "description": "Node.js on Docker",
  "author": "First Last <first.last@example.com>",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.13.3"
  }
}
`


然后，创建一个server.js文件,用来定义了一个使用Express.js框架的Web应用程序：

`
'use strict';

const express = require('express');

// Constants
const PORT = 8080;

// App
const app = express();
app.get('/', function (req, res) {
  res.send('Hello world\n');
});

app.listen(PORT);
console.log('Running on http://localhost:' + PORT);
`

在接下来的步骤中,我们将看看你如何在一个Docker容器内运行这个应用程序,使用官方Docker镜像. 首先，你需要为你的应用程序构建一个Docker镜像。


创建一个 Dockerfile

创建一个名为Dockerfile一个空文件：

`
$ touch Dockerfile
`

在你喜欢的文本编辑器中打开Dockerfile
我们需要做的第一件事就是定义要从什么镜像构建我们想要构建的。 这里我们将使用最新的LTS(长期支持)版argon ,可用于Docker Hub的node：

`
FROM node:argon
`

接下来，我们创建一个目录用于在镜像中保存应用程序的代码， 这将是你的应用程序的工作目录：

`# Create app directory
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
`

此镜像自带的Node.js和NPM已经安装了,所以接下来我们需要做的就是使用npmbinary,来安装你的应用程序的依赖：

`# Install app dependencies
COPY package.json /usr/src/app/
RUN npm install
`

要打包在Docker镜像内的你的应用程序的源代码,使用COPY指令:

`# Bundle app source
COPY . /usr/src/app
`

你的应用程序绑定到8080端口，所以你将使用EXPOSE指令把它映射,由docker守护程序(daemon)：

EXPOSE 8080
最后但同样重要,用CMD定义运行你的应用程序的命令，它定义了你的运行时。 在这里，我们将使用基础的npm start这将运行node server.js来启动你的服务器：

CMD [ "npm", "start" ]
PS:(npm start === node server.js)"scripts": { "start": "node server.js" }
现在你的Dockerfile应该看起来像这样的：

FROM node:argon

`# Create app directory
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
`

`# Install app dependencies
COPY package.json /usr/src/app/
RUN npm install
`

`# Bundle app source
COPY . /usr/src/app
`

`EXPOSE 8080
CMD [ "npm", "start" ]
`
构建你的镜像
进入到Dockerfile所在的目录并运行以下命令来构建Docker镜像。 -t标志可以让你标记你的镜像，以便更容易发现以后使用docker images命令：

`$ docker build -t <your username>/node-web-app .`

你的镜像现在将会被Docker列出：

`$ docker images`

# Example
REPOSITORY                      TAG        ID              CREATED
node                            argon      539c0211cd76    3 weeks ago
<your username>/node-web-app    latest     d64d3505b0d2    1 minute ago
运行镜像
运行你的镜像,带有-d容器将运行在独立模式下，让容器在后台运行。-p标志,将一个公共端口重定向到容器内的一个私有端口。运行你之前构建的镜像：

`$ docker run -p 49160:8080 -d <your username>/node-web-app`

打印你的应用程序的输出：

`# Get container ID
$ docker ps
`

`# Print app output
$ docker logs <container id>
`

`# Example
Running on http://localhost:8080
`
如果你需要进容器内可以使用exec命令：

`# Enter the container
$ docker exec -it <container id> /bin/bash
`

测试
要测试你的应用程序，获取的你的应用程序的Docker映射端口：

`$ docker ps`

# Example
ID            IMAGE                                COMMAND    ...   PORTS
ecce33b30ebf  <your username>/node-web-app:latest  npm start  ...   49160->8080
在上面的例子中，Docker将容器的内部的8080端口映射到你的机器上的49160端口。

现在你可以使用curl调用你的应用程序(如果需要的话,安装命令：sudo apt-get install curl)：

`$ curl -i localhost:49160`

`HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 12
Date: Sun, 02 Jun 2013 03:53:22 GMT
Connection: keep-alive`

Hello world
我们希望该教程帮助了你在Docker上启动和运行一个简单的Node.js应用程序。

你可以找到有关的Docker和Node.js on Docker更多信息,在以下的地方：

官方 Node.js Docker 镜像

Node.js Docker 最佳实践指南

官方 Docker 文档

Docker Tag on StackOverflow

Docker Subreddit

Docker for Windows

Docker Hub

cloud docker