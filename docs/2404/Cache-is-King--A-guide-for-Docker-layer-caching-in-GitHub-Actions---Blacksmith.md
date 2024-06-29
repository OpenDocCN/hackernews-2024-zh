<!--yml

category: 未分类

date: 2024-05-27 13:00:35

-->

# 缓存为王：GitHub Actions 中 Docker 层次缓存的指南 - Blacksmith

> 来源：[https://blacksmith.sh/blog/cache-is-king-a-guide-for-docker-layer-caching-in-github-actions](https://blacksmith.sh/blog/cache-is-king-a-guide-for-docker-layer-caching-in-github-actions)

在 Blacksmith 看到的大部分 GitHub Action 工作流都在使用 Docker。无论是为了设置容器以运行测试，还是构建自定义 Docker 镜像并将其推送到注册表，Docker 都无处不在。众所周知，为了一个健康的持续集成（CI）流水线，必须考虑一个有效的缓存策略——在每次运行时看到测试重新构建无差别的依赖项是一件让人头疼的事情。

几周前，我在帮助一个客户设置他们的 Docker 工作流时，开始研究在 GitHub Actions 中启用缓存的最佳方式。令人惊讶的是，我找到了*至少*三种实现这一目标的方式，每种方式都有其自己的特点。本文是我学到的所有内容的一站式指南，将帮助您为工作流选择最合适的缓存策略。

## Docker 缓存 101

Docker 镜像最好被视为一组层次结构，其中每个层代表由 `Dockerfile` 中的指令引起的文件系统更改集。每个层仅包含其前一层的更改。这确保了在层次结构中没有数据重复。分层结构的一个重要优势是可以在后续的 Docker 构建中重用中间文件系统（层次），这种层次的重用是 Docker 缓存的基础，使得构建变得更快、增量化，而不是每次都从头开始构建镜像。

最好的 `Dockerfile` 是那些按从最不频繁到最频繁变化的顺序堆叠层的文件。将最不可能改变的层作为每个 Docker 构建的基础，允许在构建后续 Docker 镜像时最有效地重用这些层。有关 `Dockerfile` 最佳实践，还有许多更详细的博客文章，所以这是我将要谈论的最后一点。

## **从简单开始**

如果您熟悉在 GitHub Actions 中编写 Docker 构建和推送步骤，可以直接跳到下一节讨论缓存的部分。

```
name: Build and Push Docker Image

on:
 push:
 branches:
      - main

jobs:
 build-and-push:
 runs-on: ubuntu-latest
 steps:
      - name: Checkout code
 uses: actions/checkout@v3

      - name: Set up Docker Buildx
 uses: docker/setup-buildx-action@v3

      - name: Build and push Docker image
 uses: docker/build-push-action@v5
 with:
 context: .
```

下面是一个典型的 GitHub Action 工作流示例，用于构建并推送 Docker 镜像。此工作流假设 `Dockerfile` 位于仓库的根目录，并使用了两个流行的 Docker 动作：

+   `[setup-buildx-action](https://github.com/docker/setup-buildx-action)` 用于配置 [Docker Buildx](https://github.com/docker/buildx) 以创建一个运行镜像构建的构建器实例

+   `[build-push-action](https://github.com/docker/build-push-action)` 用于使用上一步配置的构建器构建并推送 Docker 镜像

值得注意的是，在此工作流程中没有发生缓存 - 因此，每次运行作业时，Docker 都必须从头开始构建镜像！

## 使用 GitHub Actions 缓存的 Docker 缓存

我们将要探讨的第一种缓存策略是将 Docker 层 blob 缓存到本机 GitHub Actions [cache](https://github.com/actions/cache)。这种方法最容易协调，我建议它作为建立直觉的良好起点。然而，随着代码库规模的扩展，此方法存在一些限制。

+   每个 GitHub 存储库仅提供 10GB 的缓存空间，此后，缓存中的最旧条目将被驱逐。如果您的 Docker 镜像相当大，或者有多个层次，您可能会遇到此限制，并且无法享受到有效缓存的好处。

+   GitHub 的缓存仅限于运行 Docker 构建的开发分支。不能通过此方法在整个组织或其他构建系统中共享缓存层。

这是您如何编辑上述工作流文件以启用由 GitHub 的缓存支持的缓存。

```
name: Build and Push Docker Image

on:
 push:
 branches:
      - main

jobs:
 build-and-push:
 runs-on: ubuntu-latest
 steps:
      - name: Checkout code
 uses: actions/checkout@v3

      - name: Set up Docker Buildx
 uses: docker/setup-buildx-action@v3

      - name: Build and push Docker image
 uses: docker/build-push-action@v5
 with:
 context: .
 push: true
 tags: user/app:latest
 cache-from: type=gha
 cache-to 
```

让我们花点时间来理解 `cache-from` 和 `cache-to`。

+   `cache-from` 指向构建过程中尝试寻找缓存以供使用的源。

+   `cache-to` 指定了在构建过程中生成的缓存的目标存储位置。

在上述工作流程中，第一次运行将是未缓存的运行，因为 `cache-from` 中没有缓存可导入。在此运行结束时，缓存 blob 将被写入 `cache-to`。后续运行将能够利用这些缓存的 Docker 层，就像这样，您的运行速度比以前快得多！

## Docker 使用注册表支持的缓存

我们将看一看使用 Docker 注册表作为缓存后端的第二种 Docker 缓存方式。Docker 注册表是 Docker 镜像的存储和分发系统。您可以利用两种方法来使用注册表支持的缓存，每种方法都有其优点和限制。

### 1\. 内联缓存

内联缓存是设置最简单的注册表支持的缓存。此方法将构建缓存工件直接嵌入到 Docker 镜像中。让我们在深入之前修改我们的工作流文件。

```
name: Build and Push Docker Image

on:
 push:
 branches:
      - main

jobs:
 build-and-push:
 runs-on: ubuntu-latest
 steps:
      - name: Checkout code
 uses: actions/checkout@v3

      - name: Set up Docker Buildx
 uses: docker/setup-buildx-action@v3

      - name: Build and push Docker image
 uses: docker/build-push-action@v5
 with:
 context: .
 push: true
 tags: user/app:latest
 cache-from: type=registry,ref=user/app:latest
 cache-to 
```

正如您所见，我们的 `cache-to` 现在指向不同的缓存后端。`type=inline` 指示 Docker 构建器将缓存工件嵌入到 Docker 镜像中，并将它们推送到与 Docker 镜像相同的位置。此工作流程的后续运行将使用 `cache-from` 中引用的镜像作为基础，显著加快 Docker 构建速度。

那么... 这比 GitHub Action 的缓存后端更好在哪里呢？

+   您不受 GitHub 缓存大小限制和驱逐策略的约束。

+   您可以在整个组织或其他构建系统中重复使用缓存工件。

那么... 为什么我们还没完成呢？

因此，我们需要稍微绕个弯来理解“多阶段”Docker构建。[多阶段](https://docs.docker.com/build/building/multi-stage/)Docker构建指的是`Dockerfile`组织为多个阶段的情况。每个阶段都有自己的基础镜像和一组指令。作者完全可以决定哪些前阶段的构件应复制到下一个阶段。例如，您可以有一个`Build`阶段，其中包含所有工具和依赖项，但是您的`Final`阶段只包含运行应用程序所需的必要部分。

```
 FROM node:16-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install
COPY . .
RUN npm run build

FROM node:16-alpine
WORKDIR /app
COPY --from=builder /app/build /app
CMD ["node", "server.js" 
```

只将必要的文件和依赖项复制到最终的生产镜像中，可以使其比在单阶段编写整个`Dockerfile`时更小更安全。

`inline`缓存仅支持`mode=min`。此模式仅缓存最终阶段的层，而不是多阶段Docker构建中的任何中间层。这显著减少了后续Docker构建中缓存命中的机会。

`inline`缓存的另一个缺点是，由于它将缓存生成物嵌入到Docker镜像中，这可能会显著增加您的Docker镜像大小。接下来，让我们介绍我们的最终缓存解决方案。

### 2\. 注册表缓存

注册表缓存后端是`inline`缓存++。此后端将缓存生成物推送为独立的镜像，而不是与Docker镜像合并到注册表的专用位置。

```
name: Build and Push Docker Image

on:
 push:
 branches:
      - main

jobs:
 build-and-push:
 runs-on: ubuntu-latest
 steps:
      - name: Checkout code
 uses: actions/checkout@v3

      - name: Set up Docker Buildx
 uses: docker/setup-buildx-action@v3

      - name: Build and push Docker image
 uses: docker/build-push-action@v5
 with:
 context: .
 push: true
 tags: user/app:latest
 cache-from: type=registry,ref=user/app:buildcache
 cache-to 
```

与`inline`缓存相反，注册表缓存在配置为`mode=max`时支持缓存多阶段Docker构建中的中间层。这增加了后续构建中缓存命中的机会。它还支持一系列[选项](https://docs.docker.com/build/cache/backends/registry/)，用于控制缓存镜像的压缩类型、级别和命名等。尽管设置复杂度有所增加，但这是Docker构建的最佳类缓存解决方案。

在Blacksmith，我们始终考虑提升您的CI作业的性能。请密切关注我们即将推出的无需代码更改的Docker层缓存、共同位置的Docker镜像镜像和强大的远程构建器！如果您需要优化工作流程或希望在新功能发布时收到通知，请发送电子邮件至[hello@blacksmith.sh](mailto:hello@blacksmith.sh)。
