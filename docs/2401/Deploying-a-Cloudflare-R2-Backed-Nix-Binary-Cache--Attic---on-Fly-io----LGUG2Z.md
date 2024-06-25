<!--yml

类别：未分类

日期：2024-05-27 14:51:46

-->

# 在 Fly.io 上部署基于 Cloudflare R2 的 Nix 二进制缓存（Attic！）:: LGUG2Z

> 来源：[https://lgug2z.com/articles/deploying-a-cloudflare-r2-backed-nix-binary-cache-attic-on-fly-io/](https://lgug2z.com/articles/deploying-a-cloudflare-r2-backed-nix-binary-cache-attic-on-fly-io/)

我曾试过在我位于德国的 Hetzner 专用服务器上运行 [Attic Nix 二进制缓存](https://github.com/zhaofengli/attic) 几次，但是与西雅图的 Xfinity 的连接问题和延迟总是让我感到沮丧。

今天早上我注意到 [一个评论](https://github.com/zhaofengli/attic/issues/1#issuecomment-1368527062)，是 Zhaofeng 在存储库问题跟踪器上的评论。

> 作为一个 NixOS 爱好者，我不情愿地承认我一直在 fly.io 上运行我的实例 😛

我不确定这条评论是否仍然有效，但是嘿，如果 Zhaofeng 正在或曾经在 [fly.io](https://fly.io/) 上运行他的二进制缓存，那我们也没理由不能，对吧？

## 存储[⌗](#storage)

虽然 Attic 支持本地存储，但我想我会使用 Cloudflare 的 R2 作为存储后端，既是尝试 [免费套餐](https://developers.cloudflare.com/r2/pricing/)（10GB）的借口，也因为如果我决定将服务器从 fly.io 迁移，这将很容易搬迁。

创建新的 R2 存储桶并在 [Cloudflare 控制台](https://dash.cloudflare.com) 上获取针对存储桶的读写 API 令牌非常简单。

## [服务器配置](#server-configuration)

一旦我们有了凭据，我们就可以根据 [示例配置](https://github.com/zhaofengli/attic/blob/e6bedf1869f382cfc51b69848d6e09d51585ead6/server/src/config-template.toml) 为 Attic 服务器组装一个配置文件。

```
listen = "[::]:8080" token-hs256-secret-base64 = "<generate this with openssl rand 64 | base64 -w0>"   [database] url = "sqlite:///data/attic.db?mode=rwc"   [storage] bucket = "<your bucket name>" type = "s3" region = "auto" endpoint = "https://<your cloudflare account id>.r2.cloudflarestorage.com"   [storage.credentials] access_key_id = "<your access key id>" secret_access_key = "<your secret access key>"   [chunking] nar-size-threshold = 65536 min-size = 16384 avg-size = 65536 max-size = 262144   [compression] type = "zstd"   [garbage-collection] interval = "12 hours" 
```

需要注意的一点是，我们正在将 SQLite 数据库文件存储在 `/data/attic.db` - 这将是我们即将创建的一个 fly 卷。

你应该在你的配置存储库中 `git-crypt` 这个文件（或使用 `sops` 或 `age` 加密），因为它包含敏感凭据。

**不确定如何处理 NixOS 配置存储库中的机密？我有一篇关于[在 NixOS 中处理机密](https://lgug2z.com/articles/handling-secrets-in-nixos-an-overview/)的大文章！**

## Dockerfile[⌗](#dockerfile)

幸运的是，我们可以扩展的自动发布的 Docker 镜像可供我们使用。这里没什么要做的，除了引入我们的配置文件，当我们启动 `atticd` 时引用它，并确保它以 `monolithic` 模式运行。

```
FROM ghcr.io/zhaofengli/attic:latest COPY ./server.toml /attic/server.toml EXPOSE 8080 CMD ["-f", "/attic/server.toml", "--mode", "monolithic"] 
```

## Fly 配置[⌗](#fly-config)

部署拼图的最后一块是为我们的 `atticd` 实例组装一个 `fly.toml` 文件。

让我们首先确保我们在所需区域创建了一个卷。

```
fly volume create atticdata -r sea -n 1 
```

卷可以被我们随意命名，我们只需要确保在我们的 `fly.toml` 文件的 `[mounts]` 部分更新 `source`。

```
app = "<pick your own fly app name>" primary_region = "sea"   [mounts]  source = "atticdata" destination = "/data"   [http_service]  internal_port = 8080 force_https = true auto_stop_machines = true auto_start_machines = true min_machines_running = 0 processes = ["app"]   [services.concurrency]  type = "connections" hard_limit = 1000 soft_limit = 1000 
```

就这样了，该是 `fly deploy` 的时候了！（我使用 `--ha=false`，因为我不想创建多台机器）

## 生成一个阁楼令牌[⌗](#generating-an-attic-token)

我们需要在新部署的 fly.io 机器上执行一个命令来生成一个“godmode”令牌供我们使用。让我们从获取机器 ID 开始。

```
fly machine list --json |  jq --raw-output '.[].id' 
```

然后，我们可以通过 `fly machine exec` 在 fly.io 机器上调用 `atticadm` 命令生成我们的令牌。

```
fly machine exec <your machine id> 'atticadm make-token --sub "<your preferred username>" --validity "10y" --pull "*" --push "*" --create-cache "*" --configure-cache "*" --configure-cache-retention "*" --destroy-cache "*" --delete "*" -f /attic/server.toml' 
```

我们需要将这个令牌保存在一个安全的地方，最好是加密的。我将我的加密令牌存储在我的 NixOS 配置 monorepo 中，使用 `sops-nix` 进行加密，并在授予解密权限的机器上解密并挂载到 `/run/secrets/attic/token`。

## 登录到阁楼[⌗](#logging-into-attic)

这部分非常简单！我们可以在配置中随心所欲地命名服务器；这里使用“fly”即可。最后两个参数是为我们的 fly.io 应用生成的 URL 和我们的访问令牌。

```
# if pkgs.attic is installed in environment.systemPackages attic login fly https://<your fly app name>.fly.dev $(cat /run/secrets/attic/token)   # if pkgs.attic is not installed, we can always run it directly from the flake nix run github:zhaofengli/attic#default login fly https://<your fly app name>.fly.dev $(cat /run/secrets/attic/token) 
```

## 推送到我们的阁楼缓存[⌗](#pushing-to-our-attic-cache)

现在，我们在 fly.io 上运行着 `atticd` 缓存服务器，并且我们的计算机已经使用允许我们推送的令牌进行了身份验证，让我们设置一个缓存并推送我们的整个(!) 系统配置。

注意：在我的情况下，256MB 的 RAM 不足以跟上我想要推送的一切！我建议将 RAM 扩展至至少 512MB，如果可能的话，甚至扩展至 1GB，使用 `fly scale memory 512`。下面的片段假定你已将 RAM 扩展至 512MB，*应该*能够处理三个并发作业（`-j 3`），而不会因为内存不足而导致机器重新启动 `atticd` 进程。

```
# if pkgs.attic is installed in environment.systemPackages attic cache create system -j 3 attic push system /run/current-system -j 3   # if pkgs.attic is not installed, we can always run it directly from the flake nix run github:zhaofengli/attic#default cache create system -j 3 nix run github:zhaofengli/attic#default push system /run/current-system -j 3 
```

## 配置 NixOS 使用我们的阁楼二进制缓存[⌗](#configuring-nixos-to-use-our-attic-binary-cache)

在配置 NixOS 使用我们的新二进制缓存之前，有两件事情我们需要做。

首先，我们需要获取我们 `system` 缓存的公钥。

接下来，我们需要创建一个包含我们的 `attic` 令牌的 `netrc` 文件。格式如下：

```
machine <your fly app name>.fly.dev
password <your attic token> 
```

由于这个文件再次包含敏感信息，如果你想将其存储在配置仓库中，我建议对其进行加密。在下面的示例中，我通过 `sops-nix` 添加了 `netrc` 文件内容，它会将解密后的内容挂载到授予解密权限的机器上的 `/run/secrets/attic/netrc`。

```
{  nix = { settings = { substituters = [ "https://nix-community.cachix.org?priority=41" # this is a useful public cache! "https://numtide.cachix.org?priority=42" # this is also a useful public cache! "https://<your fly app name>.fly.dev/system?priority=43" ]; trusted-public-keys = [ "nix-community.cachix.org-1:mB9FSh9qf2dCimDSUo8Zy7bkq5CX+/rkCWyvRCYg3Fs=" "numtide.cachix.org-1:2ps1kLBUWjxIneOy1Ik6cQjb41X0iXVXeHigGmycPPE=" "<your cache public key>" ];   netrc-file = config.sops.secrets."attic/netrc".path; }; }; } 
```

现在，当我们的 NixOS 机器重新构建时：

+   首先将检查官方的 NixOS 缓存（优先级为40）

+   接下来，将检查 `nix-community` 的公共缓存（优先级为41）

+   接下来，将检查 `numtide` 的公共缓存（优先级为42）

+   最后，我们的私有缓存（优先级为43）将被检查

+   如果根本没有缓存命中，包将从源代码构建

目前我们所拥有的是相当酷的。

**在下一篇文章中，我们将使这更加酷炫**，通过设置 GitHub Actions 作业[在我们推送新提交时构建我们的每个 NixOS 系统配置，并将这些构建的输出推送到我们的私有 `system` 缓存](https://hachyderm.io/@LGUG2Z/111767749455052075)!

如果你有任何问题或意见，你可以通过[Twitter](https://twitter.com/LGUG2Z)和[Mastodon](https://hachyderm.io/@LGUG2Z)联系我。

如果你对我阅读的内容以及提出类似这样解决方案感兴趣，你可以订阅我的[软件开发 RSS 订阅](https://notado.app/feeds/jado/software-development)。

如果你想看我边写代码边解释我在做什么，你也可以[订阅我的 YouTube 频道](https://www.youtube.com/channel/UCeai3-do-9O4MNy9_xjO6mg?sub_confirmation=1)。

如果你觉得这篇内容有价值，或者你是[`komorebi`](https://github.com/LGUG2Z/komorebi)或者我的[NixOS 初始模板](https://github.com/LGUG2Z)的快乐用户，请考虑在[GitHub](https://github.com/sponsors/LGUG2Z)上赞助我或在[Ko-fi](https://ko-fi.com/lgug2z)上给我打赏。
