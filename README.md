# 使用GitHub Actions实现CI/CD

## 概述
GitHub Actions来设置一个基本的CI/CD（持续集成/持续部署）管道。以一个基于Node.js的应用程序为例，展示如何在每次推送代码时自动运行测试，并在特定条件下自动部署应用程序。

## 步骤1：创建GitHub仓库
1. 如果还没有GitHub仓库，先去[GitHub](https://github.com)创建一个新的仓库。

## 步骤3：配置CI/CD管道
1. 在你的GitHub仓库中，创建一个名为`.github/workflows`的目录，并在其中创建一个YAML文件，例如`ci-cd.yml`。这个文件将定义你的CI/CD流程。

### 示例 `.github/workflows/ci-cd.yml`
```yaml
name: CI/CD Pipeline

#作用：定义工作流的名称，便于识别和管理。
#解释：这个工作流的名称为 CI/CD Pipeline

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

#作用：定义触发工作流的事件。
#解释：push：当代码被推送到指定分支时触发。
#     branches：指定触发的分支，这里是 main。
#     pull_request：当创建或更新拉取请求时触发。
#     branches：指定触发的分支，这里是 main。

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '14'

    - name: Cache node modules
      uses: actions/cache@v3
      with:
        path: node_modules
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: npm test

#作用：定义工作流中的各个作业（job）。
#解释：在这个工作流中，有两个作业：build-and-test 和 deploy。

  deploy:
    needs: build-and-test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'

#作用：定义构建和测试作业。
#解释：runs-on: ubuntu-latest：指定作业在最新版本的 Ubuntu 虚拟机上运行。
#     steps:：定义作业中的各个步骤。

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

#作用：检出代码。
#解释：uses: actions/checkout@v3：使用 actions/checkout 动作检出代码，版本为 v3。

    - name: Set up Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '14'

#作用：设置 Node.js 环境。
#解释：uses: actions/setup-node@v3：使用 actions/setup-node 动作设置 Node.js 环境，版本为 v3。
#     with:：传递参数给动作。
#     node-version: '14'：指定 Node.js 版本为 14

    - name: Cache node modules
      uses: actions/cache@v3
      with:
        path: node_modules
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-

#作用：缓存 node_modules 目录。
#解释：uses: actions/cache@v3：使用 actions/cache 动作缓存文件，版本为 v3。
#     with:：传递参数给动作。
#     path: node_modules：指定要缓存的路径为 node_modules。
#     key: ${到}：生成缓存的唯一键，包括操作系统、Node.js 版本和 package-lock.json 的哈希值。
#     restore-keys: |：指定恢复缓存时的备选键。

    - name: Install dependencies
      run: npm ci

    - name: Deploy to server
      env:
        SERVER_USER: ${{ secrets.SERVER_USER }}
        SERVER_HOST: ${{ secrets.SERVER_HOST }}
        SERVER_PATH: ${{ secrets.SERVER_PATH }}
      run: |
        echo "Deploying to server..."
        # 在这里添加你的部署命令，例如使用 scp、ssh 等
        scp -r . ${SERVER_USER}@${SERVER_HOST}:${SERVER_PATH}
