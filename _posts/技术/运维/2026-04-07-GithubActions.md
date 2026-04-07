---
layout: default2
title: "Github Actions使用"
date: 2026-04-07
categories: ['运维']
tags: ['Github Actions', 'CI/CD']
author: LiGen
---
# 1.Actions使用

[官方工作流文档](https://docs.github.com/zh/actions)

以我现有Java项目和Vue项目为例，编写可用Github Actions配置

## 1.1 常用命令


### name
定义工作流的名称，显示如下
![](/assets/img/yunwei/githubactions1.png)

### on
定义这个流水线在什么时候触发
```yml
on: push
on: ['push', 'pull_request']
```
[所有事件定义](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows)

### env
定义本流水线的环境变量，这些环境变量在后续的流程中皆可使用，以`${{env.xxx}}`的形式直接使用

### jobs
jobs任务是流水线的核心，定义了一个工作流具体的行为
```yml
jobs:
    job_name: jobname就是下面显示的名称，这是流水线的一个环节
```
![](/assets/img/yunwei/githubactions2.png) 


### job.runs-on
这个很好理解，像是docker镜像一样，这个工作流基于什么环境进行构建 
- [系统镜像](https://docs.github.com/zh/actions/reference/workflows-and-actions/workflow-syntax#choosing-github-hosted-runners)
- 还可以使用自定义镜像
```
runs-on: ubuntu-lasted
```

### job.strategy.matrix
版本矩阵，这个定义也算是一种环境变量，可以在后续的工作流中使用`${{ matrix.version }}`
```yml
jobs:
  example_matrix:
    strategy:
      matrix:
        version: [10, 12, 14]
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.version }}
```


### job.steps
步骤定义，显示如下图，一个job里每个步骤的行为和名称
![](/assets/img/yunwei/githubactions3.png) 

### job.steps.uses
定义使用哪个软件镜像或者版本或者仓库
```
steps:
  # Reference a specific commit
  - uses: actions/checkout@8f4b7f84864484a7bf31766abe9204da3cbe65b3
  # Reference the major version of a release
  - uses: actions/checkout@v5
  # Reference a specific version
  - uses: actions/checkout@v5.2.0
  # Reference a branch
  - uses: actions/checkout@main
  # Reference a docker iamge
  - uses: docker://alpine:3.8
```

### job.steps.run
执行命令行
```
- name: Install Dependencies
  run: npm install
- name: Clean install dependencies and build
  run: |
    npm ci
    npm run build
```


## 1.2 Java项目
以下流水线定义了
> 1.名称 Java CI with Maven and Docker
> 2.dev分支推送、master-jdk17分支的pull_request请求合并
> 3.定义环境变量，Docker镜像名称，项目名称
> 4.定义jobs，名称为build
> 5.基于ubuntu最新稳定版进行构建
> 6.定义了版本矩阵，当前使用java 17进行代码构建
> 7.1 actions/checkout@v4检出代码
> 7.2 actions/setup-java@v4 使用java action，指定版本
> 7.3 执行打包命令
> 7.4 提取版本号并设置为环境变量
> 7.5 后续上传文件、推送docker镜像
```yml
name: Java CI with Maven and Docker

on:
  push:
    branches: [ dev ]
  pull_request:
    branches: [ master-jdk17 ]
    types: [closed]

env:
  DOCKER_IMAGE_NAME: bearli/ligen  # 替换为您的 Docker Hub 用户名和镜像名
  APP_NAME: schola-server

jobs:
  build:

    runs-on: ubuntu-latest

    strategy:
      matrix:
        java: [ '17' ]

    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK ${{ matrix.java }}
        uses: actions/setup-java@v4
        with:
          java-version: ${{ matrix.java }}
          distribution: 'temurin'
          cache: 'maven'

      - name: Build with Maven
        run: mvn -B package --file pom.xml -Dmaven.test.skip=true

      # 从 pom.xml 提取版本号
      - name: Extract Maven version
        id: maven-version
        run: |
          # 从 pom.xml 中提取版本号
          VERSION=$(mvn help:evaluate -Dexpression=project.version -q -DforceStdout)
          echo "Extracted version: $VERSION"
          echo "version=$VERSION" >> $GITHUB_OUTPUT

      # 调试步骤：列出整个项目中的jar文件
      - name: Find and list JAR files
        run: |
          find . -name "*.jar" -type f | head -20

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: ${{ env.APP_NAME }}
          path: ${{ env.APP_NAME }}/target/*.jar
          retention-days: 7

      # 🔥 新增：Docker 镜像构建和推送
      - name: Log in to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Extract metadata for Docker
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.DOCKER_IMAGE_NAME }}
          tags: |
            ${{ env.APP_NAME }}:${{ steps.maven-version.outputs.version }}
            ${{ env.APP_NAME }}:latest

      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .  # Dockerfile 所在目录
          file: ./Dockerfile  # Dockerfile 路径
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}

```

## 1.3 Vue项目
> 1.前面的不再介绍
> 2.actions/upload-artifact@v4 将文件打包上传，文件地址为dist-prod，打包完成的文件名为dist-prod.zip
```yml
name: Build Vue Project

on:
  push:
    branches: [dev]
  pull_request:
    branches: [master-self]
    types: [closed]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      # 1. 检出代码
      - name: Checkout code
        uses: actions/checkout@v4

      # 2. 设置 Node.js 环境（根据你的项目调整版本）
      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: '8'

      # 3. 安装依赖
      - name: Install dependencies
        run: pnpm install

      # 4. 执行打包（使用你习惯的命令）
      - name: Build project
        run: pnpm run build:prod

      # 5. （可选）上传构建产物到 GitHub Artifacts
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: dist-prod
          path: dist-prod/
          retention-days: 7
```