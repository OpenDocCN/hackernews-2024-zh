<!--yml

category: 未分类

date: 2024-05-27 14:46:56

-->

# 终极 Docker 速查表 - DevOps 循环

> 来源：[`devopscycle.com/blog/the-ultimate-docker-cheat-sheet/`](https://devopscycle.com/blog/the-ultimate-docker-cheat-sheet/)

获取你的 Docker 速查表，可以选择[PDF](http://devopscycle.com/wp-content/uploads/sites/4/2023/12/the-ultimate-docker-cheat-sheet-1.pdf) 或者一个 [图片](http://devopscycle.com/wp-content/uploads/sites/4/2023/11/the-ultimate-docker-cheat-sheet-4.png) 。为了跟随本文，确保你的开发机器已经[安装了 Docker](https://docs.docker.com/get-docker/)。在这篇博客文章中，我们将编写自己的 Dockerfile，学习如何创建镜像，最终将它们作为容器运行。完整的源代码可以在[GitHub](https://github.com/aichbauer/the-ultimate-docker-cheat-sheet)上找到。

请[下载](http://devopscycle.com/wp-content/uploads/sites/4/2023/11/the-ultimate-docker-cheat-sheet-1.pdf)你的 Docker 速查表，以便跟随本文。欢迎与你的同事和朋友分享。

成为我们新[社区](https://discord.gg/7xpRbG2gY9)的第一批成员，并了解最新的 DevOps 主题（首批到达的前 42 人免费提供饼干！）。通过注册我们的通讯，不要错过任何新资源。直接在你的收件箱里获得我们的最新资源和见解！

所有这些都是相互依赖的。你需要一个 Dockerfile 来创建一个镜像，你需要一个镜像来创建一个容器。

总之：Dockerfile 是镜像的基础，而镜像用于创建容器。容器作为主机上的一个进程运行。然而，它有自己的文件系统，并与其他进程分隔开来。

要生成一个 Dockerfile，你可以创建一个纯文本文件。本文将使用命令行来完成：

```
# create a new file in your current working directory called Dockerfile
$ touch Dockerfile
# open the file in your favorite editor (we are using Visual Studio Code)
# if you do not have the code command installed, you will need to open it manually
$ code DockerfileCode language: Bash (bash)
```

一个 Dockerfile 包含了构建、启动和运行应用程序的所有指令。你需要手动执行的每个命令都写在一个单独的文件中。它首先使用一个[基础镜像](https://devopscycle.com/blog/how-do-you-choose-a-docker-base-image/)。通常这是一个小型的 Linux 发行版，比如 alpine。如果你必须执行一个二进制文件，你应该使用 `FROM [scratch](https://hub.docker.com/_/scratch/)。本文使用了一个[Fastify](https://fastify.dev/)服务器，因此它使用了一个为[Node.js](https://nodejs.org/en)配置的[alpine](https://www.alpinelinux.org/)镜像。你可以在[GitHub](https://github.com/aichbauer/the-ultimate-docker-cheat-sheet)上查看完整的代码。如果你对如何选择基础镜像感兴趣，你可以阅读我们的文章：“你如何选择 Docker 基础镜像”

现在我们将在 Visual Studio Code 中编辑 Dockerfile：

```
# the next line sets the base image for this image # the base image is also based on a Dockerfile # see: https://hub.docker.com/layers/library/node/18-alpine/images/sha256-a0b787b0d53feacfa6d606fb555e0dbfebab30573277f1fe25148b05b66fa097 # node provides official images for Node.js and # alpine: a lightweight Linux distribution to reduce image size FROM node:18-alpine 
# sets the working directory inside the image # all commands after this instruction will be # executed inside this directory WORKDIR /app 
# copies the package.json and package-lock.json # from the client (e.g., your server or your development machine) # into the /app directory inside the image # before running npm ci to # get the advantage of layer caching COPY ./package* . 
# installs all node.js dependencies # npm ci is similar to npm install but intended to be # used in continuous integration (CI) environments # it will do a clean installion based on the package-lock.json RUN npm ci 
# copies the source code into the image COPY . . 
# this runs the build command specified in the package.json RUN npm run build:server 
# the EXPOSE instruction does not actually expose the port 300 of this image # this is documentation so that we know which port we need to expose # we do this when starting the container with the --publish flag EXPOSE 3000 
# executes the server.js file that is located in the build directory CMD ["node", "./build/index.js"] Code language: Dockerfile (dockerfile)
```

您可以使用多个阶段组成一个 Dockerfile。您可以将一个阶段视为一个镜像。此外，您可以在另一个阶段中使用来自一个阶段的材料，如文件。一个新的阶段总是以 `FROM <base-image>` 行开始。如果您想了解如何选择基础镜像，您可以阅读我们的文章：[“如何选择 Docker 基础镜像”](https://devopscycle.com/blog/how-do-you-choose-a-docker-base-image/)。您可以使用 `as` 关键字来命名一个阶段。本文将使用一个简单的 HTML 和 JavaScript 文件作为我们的客户端应用程序。作为 web 服务器，我们将使用 [NGINX](https://www.nginx.com/)。您可以在 [GitHub](https://github.com/aichbauer/the-ultimate-docker-cheat-sheet) 上查看完整的代码。

一个典型的方法是有一个 `builder stage`，拥有一个更大的基础镜像。该镜像包含构建源代码所需的所有可执行文件和库。

第二阶段是 `serve stage`，拥有一个小的基础镜像。好处是减少了依赖和库。刚好足够执行和提供你的应用程序。

采用这种方法，你可以使最终的镜像既更小又更安全。随着镜像大小的减小和镜像由更少的库组成。更少的系统库意味着它具有更低的攻击面。多阶段 Dockerfiles 不仅限于此用例，也可以有多于两个阶段。

```
# lets create a new Dockerfile for our frontend application
# as we can specify a file in the build command, we name this Dockerfile.client
$ touch Dockerfile.client
# open the file in your favorite editor (we are using Visual Studio Code)
# if you do not have the code command installed, you will need to open it
$ code Dockerfile.clientCode language: Bash (bash)
```

这是一个多阶段应用程序的示例：

```
# the base image # name it builder # you can reference this stage # in other stages by this name FROM node:18-alpine as builder 
# working directory inside the image WORKDIR /app 
# copies files from the client to the image COPY ./package* . 
# run a command inside the image RUN npm ci 
# copies files from the client to the image COPY . . 
# run a command inside the container # this will create a new folder in called dist in our app directory # inside the dist directory, you will find the # final HTML and JavaScript file RUN npm run build:client 
# serve stage # slim nginx base image named as serve # will start nginx as non root user FROM nginxinc/nginx-unprivileged:1.24 as serve 
# we can now copy things from the first stage to the second # we copy the build output to the directory where nginx serves files COPY --from=builder /app/dist /var/www 
# we overwrite the default config with our own # if you take a look at the GitHub repository, you # see the .nginx directory with the nginx.conf # here we only use the port 80 # in production, you would also want to make sure # all requests, even in your internal network or Kubernetes cluster # is served via HTTPS when dealing with sensible data COPY --from=builder /app/.nginx/nginx.conf /etc/nginx/conf.d/default.conf 
# the EXPOSE instruction does not actually expose the port 80 of this image # this is documentation so that we know which port we need to expose # we do this when starting the container with the --publish flag EXPOSE 80 
# The command used when the image is started as a container # Note: for Docker containers (or for debugging), # the "daemon off;" directive which is used in this example # tells nginx to stay in the foreground. # for containers, this is useful. # best practice: one container = one process. # one server (container) has only one service. CMD ["nginx", "-g", "daemon off;"] Code language: Dockerfile (dockerfile)
```

我们使用 Docker CLI 来根据我们的 Dockerfiles 构建镜像。

```
# list the directory to make sure you are in the directory with the Dockerfile
$ ls
# if not, change the directory with "cd ./path/to/directory-with-Dockerfile"
# build an Image out of the Dockerfile in the current working directory
$ docker build .
# if you want to build another Dockerfile in this directory, use the --file flag
# e.g., --file <filename>
$ docker build --file Dockerfile.client .Code language: Bash (bash)
```

从 Dockerfile 创建镜像只需要 `docker build` 命令。如果不指定名称和标记，则只能通过其镜像 ID 引用镜像。

```
# list all local images on the client (your server or your development machine)
$ docker image ls
# find your image IDCode language: Bash (bash)
```

如果想要给镜像命名，需要在构建镜像时使用 `--tag`（简写语法：`-t`）标志。如果您正在使用像 [Docker Hub](https://hub.docker.com/) 这样的注册表，则需要这个。

要命名和标记您的镜像，请使用以下模式：`<name>:<tag>`。这通常被翻译为 `<username>/<repository>:<version>`。用户名对应于注册表的用户名。

```
# build and tag your image
# a tag consists of a name and a tag, which is separated by a colon (:)
$ docker build --tag examplename/examplerepository-server:0.1.0 .
# or
$ docker build --file Dockerfile.client --tag examplename/examplerepository-client:0.1.0 .
# list all local images
$ docker image ls
# you will see your image with a proper repository name and a tagCode language: PHP (php)
```

一个容器是一个正在运行的镜像。您可以使用 CLI 命令 `docker run <image-name>` 运行镜像：

```
# start our image
$ docker run examplename/examplerepository-server:0.1.0
# or
$ docker run examplename/examplerepository-client:0.1.0Code language: Bash (bash)
```

每个 Docker 安装都带有一个本地注册表。Docker 将您的镜像存储在这里。首先，`docker run` 命令将查看本地注册表并尝试找到镜像。由于我们在执行 `docker run` 命令的同一台机器上构建了它，这将发生在我们的镜像上。如果本地找不到镜像，则会查看 Docker Hub 注册表。您还可以从其他注册表获取镜像（例如，您的自托管注册表）。为此，您可以使用自托管注册表的 URL。

```
# try to find an image on another registry
$ docker run https://registrydomain.com/examplename/examplerepository-server:0.1.0Code language: Bash (bash)
```

如果运行镜像，它将在前台进程中启动。容器作为在执行`docker run`命令的终端中的一个进程运行。如果您关闭终端，它将立即停止容器。为了让您的容器在您的机器或服务器上运行，您可以将容器作为后台进程运行。这样，您就可以放心关闭终端。

`--detached`（简写语法：`-d`）标志将在后台进程中启动容器：

```
# run container in the background
$ docker run --detached examplename/examplerepository-server:0.1.0
# or
$ docker run --detached examplename/examplerepository-client:0.1.0Code language: Bash (bash)
```

您将看到命令退出，然后可以再次使用终端。

Docker 只会显示正在运行的容器。

```
# list all running containers
$ docker container ls
# short
$ docker psCode language: Bash (bash)
```

如果要查看所有容器，即使是已停止的容器，您需要传递`--all`（简写语法：`-a`）标志。

```
# list all stopped and running containers
$ docker container ls --allCode language: Bash (bash)
```

有时，您希望停止容器。停止容器后，它们仍然存在于系统中，您可以重新启动它们。如果希望从系统中清除容器，则需要将其删除。您只能删除已停止的容器。

```
# stop a container
$ container stop <container-id>
# start a container
$ container start <container-id>
# restart container
$ container restart <container-id>
# remove a stopped container
$ container rm <container-id>Code language: Bash (bash)
```

如果您希望一旦停止容器就删除容器，可以在启动容器时传递`--rm`标志。只有在容器是无状态的情况下才使用此选项。如果您的容器具有自己的状态，请确保使用卷来保留它。否则，当容器停止时，容器内的所有数据都将丢失。

```
# automatically remove a container after it stops
$ docker run --rm examplename/examplerepository-server:0.1.0
# or
$ docker run --rm examplename/examplerepository-client:0.1.0Code language: Bash (bash)
```

有时信号无法正确传递到容器。想象一下，您已经因为无法使用 CTRL+C 停止容器而杀死了终端。但是，如果您尝试重新启动容器，它会告诉您端口已被分配。这意味着您的旧容器仍在运行。要终止容器，请运行以下命令：

```
# kill a container
$ docker kill <conatiner-id>Code language: Bash (bash)
```

通常，Docker 容器公开一个或多个端口。您可以通过这些端口访问运行在容器内部的应用程序。要访问这些端口，您需要在容器创建期间发布这些端口。从主机系统访问容器的另一种方式是在其中执行命令。这通常用于调试或单次使用容器应用程序。

要将端口暴露给主机系统，您需要添加`--publish`（简写语法：`-p`）标志。

```
# publish ports, e.g., forward container port to a port on the host system
$ docker run --publish 3000:3000 examplename/examplerepository-server:0.1.0
# or
$ docker run --publish 80:80 examplename/examplerepository-server:0.1.0
# if you run both containers, the server and the client
# and you visit the localhost:80 in your browser
# you should see the message Hello WorldCode language: Bash (bash)
```

在上面的第一个示例中，我们将容器的端口 3000 绑定到主机系统的端口 3000 上。主机系统可以是您的开发机器或服务器。格式如下：`--publish <hostport>:<containerport>`。

有时，您希望访问容器。这对于调试很有好处。

```
# access the container
$ docker exec --interactive --tty <container-id> <shell-command>Code language: Bash (bash)
```

`--interactive --tty`（简写语法`-it`）指示 Docker 分配伪 TTY 连接。这样，容器的 stdin（标准输入）在容器中创建一个交互式 shell。

您可以执行任何容器内可能的命令。如果您运行的是 Debian 容器，您将能够列出目录：

```
# list the directory inside the container
$ docker exec --interactive --tty <container-id> lsCode language: PHP (php)
```

您甚至可以使用以下命令创建一个类似于安全外壳的连接：

```
# SSH into the container (if the `sh` command exists in the container)
$ docker exec --interactive --tty <container-id> sh
# this will keep the connection to the container open
# and you can execute multiple commands within the container
# to exit the container, run the following
$ exitCode language: Bash (bash)
```

容器内存储的数据默认情况下不会持久化。当你停止并移除一个容器时，该容器中的所有数据都会丢失。在我们的应用程序中，如果你点击网站上的按钮（[`localhost:80`](http://localhost/)），我们会将“New message”写入容器内的一个 JSON 文件中。现在如果我们停止并移除容器，所有这些消息也会被删除。

如果你想要在容器启动之间保持数据持久化，你需要使用卷（volumes）。有两种不同类型的卷：命名卷和挂载卷。命名卷完全由 Docker 处理，挂载卷由你管理。对于挂载卷，你需要指定主机系统上存储这些数据的位置。我们在运行命令中使用`--volume`（缩写`-v`）。

```
# using a named volume
# everything within this path of the container will be stored
# in a volume named <volume-name>
$ docker run --volume <volume-name>:/path/in/container <image-name>

# using a mounted volume
# everything inside the path of the container will be stored 
# in the path of the host
$ docker run --volume /path/on/host:/path/in/container <image-name>Code language: Bash (bash)
```

对于我们的应用程序，我们需要使用以下命令来启动服务器容器：

```
# create a volume called server-volume
# we store the content of /app/build/data within our container
# on our host machine (your dev machine or your server)
docker run --volume server-volume:/app/build/data --publish 3000:3000 examplename/examplerepository-server:0.1.0Code language: Bash (bash)
```

你可以通过列出它们来获取所有卷及其元数据的概述。

```
# list all volumes
$ docker volume ls

# here you will see the location where Docker will store the named volumes
# on the host machineCode language: Bash (bash)
```

在本文中，我们学习了 Dockerfile、镜像和容器之间的区别。现在你可以编写你的 Dockerfile 来创建镜像。此外，你还学会了如何启动容器并从主机系统访问它们。另外，你还学会了如何在容器启动之间持久化数据。

如果你需要帮助容器化，[随时联系我们](https://calendly.com/devopsberatung/meet)，或加入我们的新社区进行进一步问题和讨论（前 42 位到场者可获得免费饼干）！

你喜欢这篇文章吗？与你的同事和朋友分享吧。

不要错过我们的最新提示、指南和更新——现在就订阅我们的通讯！我们承诺只向你发送最相关和最有用的信息。加入我们的行列，一起探索 Docker 和更多领域的世界。
