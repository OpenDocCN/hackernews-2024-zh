<!--yml
category: 未分类
date: 2024-05-27 13:00:35
-->

# Cache is King: A guide for Docker layer caching in GitHub Actions - Blacksmith

> 来源：[https://blacksmith.sh/blog/cache-is-king-a-guide-for-docker-layer-caching-in-github-actions](https://blacksmith.sh/blog/cache-is-king-a-guide-for-docker-layer-caching-in-github-actions)

A majority of the GitHub Action workflows we see at Blacksmith use Docker. Whether it is to setup containers to run tests against, or build custom Docker images and push them to a registry, Docker is everywhere. It is also common knowledge that for a healthy Continuous Integration (CI) pipeline, one has to think through an effective caching strategy - there are few things as mind-numbing as watching tests rebuild undifferentiated dependencies on every run.

A few weeks ago, I was helping a customer setup their Docker workflows, and started reading about the best way to enable caching in GitHub Actions. Surprisingly, I found *at least* three ways to achieve this, each with its own quirks. This blog is a one-stop shop with everything I learned and will help you choose the most suitable caching strategy for your workflows.

## Docker caching 101

Docker images are best visualized as a stack of layers, where each layer represents a set of filesystem changes resulting from an instruction in the `Dockerfile`. Each layer only contains the changes from the layer before it. This ensures that there is no duplication of data across layers. A big advantage of the layered structure is that the intermediate filesystems (layers) can be re-used in subsequent Docker builds. This re-using of layers is what forms the basis of Docker caching, and results in much faster, incremental builds, instead of always building the image from scratch.

The best `Dockerfile` s are the ones that stack layers in order from least frequently to most frequently mutated. Consolidating the layers that are least likely to change as the base of every Docker build, allows for the most effective re-use of layers when building subsequent Docker images. There are several, much more informed blogs out there about `Dockerfile` best practices, so that’s the last I’ll speak of it.

## **Starting simple**

If you are familiar with how a Docker build and push step is written in GitHub Actions, feel free to skip over to the next section that talks about caching.

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

This is what a vanilla GitHub Action workflow that builds and pushes a Docker image looks like. This workflow assumes the `Dockerfile` is in the root of the repository, and uses two popular Docker actions:

*   `[setup-buildx-action](https://github.com/docker/setup-buildx-action)` to configure [Docker Buildx](https://github.com/docker/buildx) to create a builder instance for running the image build

*   `[build-push-action](https://github.com/docker/build-push-action)` to build and push the Docker image using the builder configured in the previous step

Notably, there is no caching occurring in this workflow - so, every time the job is run, Docker will have to build the image from scratch!

## Docker caching with GitHub Actions cache

The first caching strategy we are going to explore is caching Docker layer blobs to the native Github Actions [cache](https://github.com/actions/cache). This approach is the simplest to reconcile, and I’d recommend it as a good starting point to build intuition. There are, however, some limitations to this approach as your codebase scales.

*   Each GitHub repository is only given 10GB of cache space, after which, the oldest entries in the cache are evicted. If your Docker image is reasonably large, or has several layers, you will likely run into this limit and not reap the benefits of effective caching.

*   GitHub’s cache is only scoped to the development branch running the Docker build. Sharing the cached layers across your organization, or with other build systems, is not possible with this approach.

Here is how you would edit the above workflow file to enable caching backed by GitHub’s cache.

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

Let’s take a second to understand `cache-from` and `cache-to`.

*   `cache-from` points to the source where the build process will attempt to find a cache to use while building the Docker image

*   `cache-to` specifies the destination where the cache generated during the build process will be stored

In the above workflow, the first run will be an uncached run, since there is no cache at `cache-from` to import. At the end of this run, cache blobs will be written to `cache-to`. Subsequent runs will be able to leverage these cached Docker layers, and just like that, you’re running much faster than you were!

## Docker caching with a registry backed cache

The second flavour of Docker caching we’ll look at uses a Docker registry as the cache backend. A Docker registry is a storage and distribution system for Docker images. There are two ways you can leverage a registry backed cache, each with its own strengths and limitations.

### 1\. Inline cache

An inline cache is the simplest registry backed cache to setup. This method embeds the build cache artifacts directly into the Docker image. Let’s modify our workflow file before diving in.

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

As you can see, our `cache-to` now points to a different cache backend. The `type=inline` instructs the Docker builder to embed cache artifacts into the Docker image, and push them to the same location as the Docker image Subsequent runs of this workflow will use the image referenced in `cache-from` as their base, to significantly speed-up Docker builds.

So… why is this better than the GitHub Action’s cache backend?

*   You are not subject to GitHub’s cache size limits and eviction policies.

*   You can re-use the cache artifacts across your organization or with other build systems.

So… why aren’t we done yet?

For this we need to take a slight detour to understand “multi-stage” Docker builds. A [multi-stage](https://docs.docker.com/build/building/multi-stage/) Docker build is one where the `Dockerfile` is organized into multiple stages. Each stage has its own base image and set of instructions. The author has complete flexibility to decide what artifacts from the previous stage should be copied over to the next stage. For example, you can have a `Build` stage that includes all the tools and dependancies, but have your `Final` stage only have the necessary pieces to run your application.

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

Having only the necessary files and dependencies copied into the final production image allows for it to be much smaller and more secure, than if the entire `Dockerfile` were to be written in a single stage.

The `inline` cache only supports `mode=min` . This mode only caches the layers of the final stage, not any of the intermediate layers in a multi-stage Docker build. This significantly reduces the chances of a cache hit in subsequent Docker builds.

Another shortcoming of the `inline` cache is that since it embeds the cache artifacts into the Docker image, it can significantly inflate the size of your Docker image. Enter, our final caching solution.

### 2\. Registry cache

The registry cache backend is the `inline` cache++. This backend pushes cache artifacts as a separate image than the Docker image to a dedicated location in the registry.

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

Contrary to the `inline` cache, the registry cache supports caching of intermediate layers in a multi-stage Docker build when configured with `mode=max` . This increases the chances of cache hits on subsequent builds. It also supports a host of [options](https://docs.docker.com/build/cache/backends/registry/) to control compression type, levels, and naming of the cache image etc. At the cost of some additional complexity to set up, this is the best-in-class caching solution for Docker builds.

At Blacksmith, we are always thinking of ways to boost the performance of your CI jobs. Keep your eyes peeled for out-of-the-box Docker layer caching with zero code changes, collocated Docker mirrors, and beefy remote builders from us in the near future! If you need help optimizing your workflows, or want to be notified as new features drop, email us at [hello@blacksmith.sh](mailto:hello@blacksmith.sh).