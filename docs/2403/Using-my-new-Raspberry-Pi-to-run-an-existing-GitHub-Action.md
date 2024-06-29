<!--yml

category: 未分类

date: 2024-05-27 14:53:05

-->

# 使用我的新树莓派来运行现有的 GitHub Action

> 来源：[https://blog.frankel.ch/raspberry-pi-github-action/](https://blog.frankel.ch/raspberry-pi-github-action/)

GitHub Actions 依赖于 Runner 上安装了 Docker。因此，我认为作业在专用镜像中运行是错误的。无论你在作业中脚本化的内容，都会在运行系统上发生。例如，最初的脚本安装了 Python 和 Poetry。

```
jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - name: Set up Python 3.x
        uses: actions/setup-python@v5
        with:
          python-version: 3.12
      - name: Set up Poetry
        uses: abatilo/actions-poetry@v2
        with:
          poetry-version: 1.7.1
```

在每次运行时创建的临时容器的情况下，这是有意义的；在稳定的长期运行系统中，这是不合适的。

Raspbian，树莓派的默认操作系统，已经安装了 Python 3.11。因此，我必须降级 Poetry 中配置的版本。这没什么大不了的，因为我没有使用任何特定的 Python 3.12 功能。

pyproject.toml

```
[tool.poetry.dependencies]
python = "^3.11"
```

Raspbian 禁止在主环境中安装任何 Python 依赖项，这是一个非常明智的默认设置。为了安装 Poetry，我使用了常规的 APT 包管理器：

```
sudo apt-get install python-poetry
```

接下来是处理密钥。在 GitHub 上，你可以在 GUI 中设置密钥，并通过环境变量在脚本中引用它们：

```
jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - name: Update README
        run: poetry run python src/main.py --live
        env:
          BLOG_REPO_TOKEN: $
          YOUTUBE_API_KEY: $
```

它允许隔离各个步骤，以便每个步骤只能访问它需要的环境变量。对于自托管的 Runner，你可以在文件夹中的现有 `.env` 文件中设置环境变量。

```
jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - name: Update README
        run: poetry run python src/main.py --live
```

如果你希望有更安全的设置，那就全靠你自己了。

最后，这种架构是一种拉取式模型。Runner 不断检查是否有作业被调度。要将 Runner 设置为服务，我们需要在 Runner 文件夹中使用现成的脚本：

```
sudo ./svc.sh install
sudo ./svc.sh start
```

脚本使用 `systemd` 作为底层。
