<!--yml
category: 未分类
date: 2024-05-27 14:53:05
-->

# Using my new Raspberry Pi to run an existing GitHub Action

> 来源：[https://blog.frankel.ch/raspberry-pi-github-action/](https://blog.frankel.ch/raspberry-pi-github-action/)

GitHub Actions depend on Docker being installed on the runner. Because of this, I thought jobs ran in a dedicated image: it’s plain wrong. Whatever you script in your job happens on the running system. Case in point, the initial script installed Python and Poetry.

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

In the context of a temporary container created during each run, it makes sense; in the context of a stable, long-running system, it doesn’t.

Raspbian, the Raspberry default operating system, already has Python 3.11 installed. Hence, I had to downgrade the version configured in Poetry. It’s no big deal because I don’t use any specific Python 3.12 feature.

pyproject.toml

```
[tool.poetry.dependencies]
python = "^3.11"
```

Raspbian forbids the installation of any Python dependency in the primary environment, which is a very sane default. To install Poetry, I used the regular APT package manager:

```
sudo apt-get install python-poetry
```

The next was to handle secrets. On GitHub, you set the secrets on the GUI and reference them in your scripts via environment variables:

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

It allows segregating individual steps so that a step has access to only the environmental variables it needs. For self-hosted runners, you set environment variables in an existing `.env` file inside the folder.

```
jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - name: Update README
        run: poetry run python src/main.py --live
```

If you want more secure setups, you’re on your own.

Finally, the architecture is a pull-based model. The runner constantly checks if a job is scheduled. To make the runner a service, we need to use out-of-the-box scripts inside the runner folder:

```
sudo ./svc.sh install
sudo ./svc.sh start
```

The script uses `systemd` underneath.