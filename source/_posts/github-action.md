---
title: Github Action 作业报告
date: 2023/05/15 15:00
tags:
- 课程作业
categories:
- 课程作业
---

## Actions

1. [Hexo 页面自动构建](https://github.com/DarkHighness/darkhighness.github.io/blob/main/.github/workflows/pages.yml)
2. [WakaTime 自动更新](https://github.com/DarkHighness/DarkHighness/blob/master/.github/workflows/wakatime.yml)
3. [贪吃蛇 Commit History 生成](https://github.com/DarkHighness/DarkHighness/blob/master/.github/workflows/snake.yml)
4. [3D Commit History 生成](https://github.com/DarkHighness/DarkHighness/blob/master/.github/workflows/profile-3d.yml)

> 
> 以下内容复用自 https://github.com/X-lab2017/oss101/issues/69
> 

**[Platane/snk](https://github.com/Platane/snk)** 是一个用以根据你的 Github  Contributions Graph 生成一个 “贪吃蛇游戏” 的仓库。它的输出看起来像是这样: 
![Sample](https://raw.githubusercontent.com/DarkHighness/DarkHighness/master/dist/github-snake.svg)

而要在你的仓库中，实现每日动态更新，可以采用 Github Actions的方式，模板看起来像是这样:

```yaml
name: Snake Readme

on:
  workflow_dispatch:
  schedule:
    - cron: "0 6 * * *"

jobs:
  update-readme:
    name: Generate Snake SVG
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: Platane/snk@v2
        with:
          # github user name to read the contribution graph from (**required**)
          # using action context var `github.repository_owner` or specified user
          github_user_name: ${{ github.repository_owner }}

          # list of files to generate.
          # one file per line. Each output can be customized with options as query string.
          #
          #  supported options:
          #  - palette:     A preset of color, one of [github, github-dark, github-light]
          #  - color_snake: Color of the snake
          #  - color_dots:  Coma separated list of dots color.
          #                 The first one is 0 contribution, then it goes from the low contribution to the highest.
          #                 Exactly 5 colors are expected.
          outputs: |
            dist/github-snake.svg
            dist/github-snake-dark.svg?palette=github-dark
            dist/ocean.gif?color_snake=orange&color_dots=#bfd6f6,#8dbdff,#64a1f4,#4b91f1,#3c7dd9
      - name: Commit & Push
        run: |
          git config user.name github-actions
          git config user.email github-actions@github.com
          git add -A .
          git commit -m "snake profiles"
          git push
```

一份 Github Actions 的文件主要定义了：
1.  `on`: 在何种情况下触发这一 workflow，它可以是某种事件，如: `push`, `fork`，也可以是一个 cron 格式的时间；
2. `jobs`: 这个 workflow 需要执行的任务，在这个例子中，`Generate Snake SVG` 定义了一个在 `ubuntu-latest` 环境上执行的任务，它共分为 3 个步骤:
     1.  `actions/checkout@v3`: 将当前仓库拉取到环境中；
     2. `Platane/snk@v2`: 生成 `github_user_name` 的对应 ”贪吃蛇“ 图片;
     3. `Commit & Push`: 将图片 commit 并 push 到仓库中;

---

如果觉得 Github Actions 调试困难，陷入不断的 Push ，手动触发，然后修改的循环，不妨试试: **[nektos/act](https://github.com/nektos/act)**