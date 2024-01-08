---
title: "GitHub Repository Sync"
date: 2021-03-14T20:00:00+08:00
draft: false
authors: ["senzyo"]
tags: [Git,GitHub Actions]
featuredImagePreview: ""
summary: 一个本地 Git Repository 使用多个 Remote。使用 GitHub Actions 从上游同步已经 Fork 的 Repository。
---

{{< admonition type=tip title="原文已删除" open=true >}}
原文为《GitHub与Gitee双向同步》, 但是我不用 Gitee 了, 而且原文的一些配置已经过时, 干脆删掉。
{{< /admonition >}}

## 多个Remote

让一个本地 Git Repository 使用多个 Remote, `git push` 时, 同时推送到多个 Remote, 参考 `git remote set-url --add <name> <newurl>`, 运行命令, 比如: 

```bash
git remote set-url --add origin git@gitlab.com:senzyo/blog.git
```

使用 `git remote -v` 查看得到类似信息: 

```bash
origin  git@github.com:senzyo/blog.git (fetch)
origin  git@github.com:senzyo/blog.git (push)
origin  git@gitlab.com:senzyo/blog.git (push)
```

这样在 `git push` 时, 就会同时推送到两个源。

## 同步已经Fork的Repo

1. 打开 [https://github.com/settings/tokens](https://github.com/settings/tokens) 生成个人密钥: personal access token (classic)。
2. 注意: 作用域 (scopes) 勾选 the `repo` and `workflow`。
3. 生成 token 并复制。
4. 在你专门用于运行 GitHub Actions 来同步的 Repository 中点击 **Settings** → **Secrets and variables** → **Actions** → **New repository secrets**。
5. `Name` 填写 `PERSONAL_TOKEN`, `Secret` 粘贴刚才的 token。
6. 按照你的需要编辑下面的 GitHub Actions 文件。

```yaml
name: Sync Upstream

on:
  schedule:
    - cron: "0 16 * * *"
  workflow_dispatch:
  push:
    branches:
      - master

jobs:
  git-config:
    name: Git config
    runs-on: ubuntu-latest
    steps:
      - name: Git config
        run: |
          git config --global user.name "github-actions[bot]"
          git config --global user.email "github-actions[bot]@users.noreply.github.com"

  github-readme-stats:
    needs: git-config
    name: Sync github-readme-stats
    runs-on: ubuntu-latest
    steps:
      - name: Sync github-readme-stats
        run: |
          git clone https://${{ secrets.PERSONAL_TOKEN }}@github.com/senzyo/github-readme-stats.git
          cd github-readme-stats
          git remote add upstream https://github.com/anuraghazra/github-readme-stats.git
          git pull upstream master
          git push origin master

  mind-map:
    needs: git-config
    name: Sync mind-map
    runs-on: ubuntu-latest
    steps:
      - name: Sync mind-map
        run: |
          git clone https://${{ secrets.PERSONAL_TOKEN }}@github.com/senzyo/mind-map.git
          cd mind-map
          git remote add upstream https://github.com/wanglin2/mind-map.git
          git pull upstream main
          git push origin master

  rename:
    needs: git-config
    name: Sync rename
    runs-on: ubuntu-latest
    steps:
      - name: Sync rename
        run: |
          git clone https://${{ secrets.PERSONAL_TOKEN }}@github.com/senzyo/rename.git
          cd rename
          git remote add upstream https://github.com/JasonGrass/rename.git
          git pull upstream main
          git push origin master

  subconverter-modify:
    needs: git-config
    name: Sync subconverter-modify
    runs-on: ubuntu-latest
    steps:
      - name: Sync subconverter-modify
        run: |
          git clone https://${{ secrets.PERSONAL_TOKEN }}@github.com/senzyo/subconverter-modify.git
          cd subconverter-modify
          git remote add upstream https://github.com/asdlokj1qpi23/subconverter.git
          git pull upstream master
          git push origin master
```

