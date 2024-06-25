<!--yml

category: 未分类

date: 2024-05-27 14:52:23

-->

# Browsh

> 来源：[`www.brow.sh/`](https://www.brow.sh/)

**Browsh 是一个完全现代化的基于文本的浏览器。** 它可以渲染任何现代浏览器可以渲染的内容；HTML5，CSS3，JS，视频甚至 WebGL。它的主要目的是在远程服务器上运行，并通过 SSH/Mosh 或浏览器中的 HTML 服务访问，以显着减少带宽，从而提高浏览速度并降低带宽成本。

## 下载（v1.8.0）

Browsh 可作为单个静态二进制文件在所有主要平台上使用。唯一的依赖是最近版本 57+的 Firefox。

最新版本

|

[版本存档](https://github.com/browsh-org/browsh/releases) 也可用 Docker 镜像：

`docker run -it browsh/browsh`

## 实时 SSH 演示

**暂时离线**

只需将你的 SSH 客户端指向

*brow.sh*

，例如;

`ssh brow.sh`

。无需验证。该服务仅用于演示，会话持续 5 分钟并被记录。

请注意，SSH 实际上是一个效率非常低的协议，为了获得最佳结果，请将 Browsh 与你自己的服务器一起安装，以及[Mosh](https://mosh.org/)。

## 浏览器中的服务

**暂时离线**

+   [html.brow.sh](https://html.brow.sh) 使用非常基本的图形和 HTML 锚标签。虽然这个服务看起来与终端客户端相似，但它目前还没有完全相同的功能。

+   [text.brow.sh](https://text.brow.sh) 仅使用纯文本，更适合与`curl`一起使用。

[`www.youtube.com/embed/zqAoBD62gvo`](https://www.youtube.com/embed/zqAoBD62gvo)

视频

## 捐赠

Browsh 目前由以下人员维护和资助

[一个人](http://tombh.co.uk)

如果你希望 Browsh 继续帮助那些因网络缓慢和/或昂贵而困扰的人，请考虑

捐赠

。
