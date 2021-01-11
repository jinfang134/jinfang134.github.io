---
layout: post
title: 为私有项目集成ci/cd服务
date: 2021-01-11 12:04:36
tags:
---
CI/CD 是一种通过在应用开发阶段引入自动化来频繁向客户交付应用的方法。CI/CD 的核心概念是持续集成、持续交付和持续部署。作为一个面向开发和运营团队的解决方案，CI/CD 主要针对在集成新代码时所引发的问题（亦称：“集成地狱”）。

很多收费的CI/CD对开源项目都很友好(比如支持github的travis-ci)，可以提供免费的CI/CD服务，具体的服务商列表，可以参考[这里](https://github.com/ripienaar/free-for-dev#ci-and-cd)，但是对于私有项目，则要么付费，要么服务受到了限制,结合国内的网络环境，我选取了以下几种持续集成服务作为对比，最后我选择了coding作为我的ｃｉｃｄ方案，1000分钟也满足了我的需求
<!-- more -->

| 服务       | 优点 |                             缺点 |
|------------|--------|---------------------------------|
| coding.net | 国内服务提供商，网速快，提供样例配置，上手很快，兼容jenkins的pipeline语法，并提供一些针对性的扩展，自带docker仓库及其它制品库，源代码支持github,gitlab,gitee等多种第三方仓库   |　免费时长较少，1000分钟／月                                 |
| gitlab.com | 可以直接从仓库里创建pipeline,集成非常方便，2000min/月     | 国内访问需要科学上网，网速比较慢 |
| github.com |   集成github的action，不限项目数量，2000min/月  |  网络较慢，特殊时期可能无法访问, |

下面以一个java springboot+gradle项目为例子，来说明如何集成到项目当中，进行项目构建和发布到远程服务器当中．

构建过程非常简单
```mermaid
graph LR

    A[检出] -->|Gitee| B(测试)
    B --> C(编译)
    C -->D[发布到远程服务器]
```

## coding
类似于jenkins的配置，在项目根目录下添加文件`Jenkinsfile`
```groovy
pipeline {
  agent any
  stages {
    stage('检出') {
      steps {
        checkout([$class: 'GitSCM',
        branches: [[name: GIT_BUILD_REF]],
        userRemoteConfigs: [[
          url: GIT_REPO_URL,
          credentialsId: CREDENTIALS_ID
        ]]])
      }
    }
    stage('单元测试') {
      post {
        always {
          junit 'build/test-results/**/*.xml'
        }
      }
      steps {
        sh './gradlew test'
      }
    }
    stage('编译') {
      steps {
        echo '开始构建'
        sh './gradlew clean build -x test'
      }
    }
  
    stage('部署到远端服务') {
      steps {
        script {
          def remoteConfig = [:]
          remoteConfig.name = "aliyun"
          remoteConfig.host = "${REMOTE_HOST}"
          remoteConfig.port = "${REMOTE_SSH_PORT}".toInteger()
          remoteConfig.allowAnyHosts = true

          withCredentials([
            sshUserPrivateKey(
              credentialsId: "${REMOTE_CRED}",
              keyFileVariable: "privateKeyFilePath"
            ),
            usernamePassword(
              credentialsId: "${CODING_ARTIFACTS_CREDENTIALS_ID}",
              usernameVariable: 'CODING_DOCKER_REG_USERNAME',
              passwordVariable: 'CODING_DOCKER_REG_PASSWORD'
            )
          ]) {
            // SSH 登陆用户名
            remoteConfig.user = "${REMOTE_USER_NAME}"
            // SSH 私钥文件地址
            remoteConfig.identityFile = privateKeyFilePath

            sshPut(
              remote: remoteConfig,
              // 本地文件或文件夹
              from: '/build/libs/app-1.0.1.jar',
              // 远端文件或文件夹
              into: '/home/ubuntu/workspace/app/app.jar'
            )

            sshCommand(
              remote: remoteConfig,
              command: "service daotong-app restart",
              sudo: true,
            )

            echo "部署成功，请到 http://${REMOTE_HOST}:8080 预览效果"
          }
        }

      }
    }
  }
  
}
```


## gitlab
在项目根目录下添加`.gitlab-ci.yml`文件，并添加如下内容

```yml
image: java:8

before_script:
  ##
  ## Install ssh-agent if not already installed, it is required by Docker.
  ## (change apt-get to yum if you use an RPM-based image)
  ##
  - 'command -v ssh-agent >/dev/null || ( apt-get update -y && apt-get install openssh-client -y )'

  ##
  ## Run ssh-agent (inside the build environment)
  ##
  - eval $(ssh-agent -s)

  ##
  ## Add the SSH key stored in SSH_PRIVATE_KEY variable to the agent store
  ## We're using tr to fix line endings which makes ed25519 keys work
  ## without extra base64 encoding.
  ## https://gitlab.com/gitlab-examples/ssh-private-key/issues/1#note_48526556
  ##
  - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -

  ##
  ## Create the SSH directory and give it the right permissions
  ##
  - mkdir -p ~/.ssh
  - echo "$SSH_KNOWN_HOSTS" >> ~/.ssh/known_hosts
  - chmod 700 ~/.ssh
  - chmod 644 ~/.ssh/known_hosts


stages:
  - build
  - test
  - deploy

test-job:
  stage: test
  script:
    - /gradlew clean test

build-job:
  stage: build
  script:
    - echo "build"
    - ./gradlew clean build -x test


deploy-prod:
  stage: deploy
  script:
    - echo "deploy starting"
    - scp build/libs/app-*.jar ubuntu@$SSH_HOST:~/workspace/app/app.jar
    - ssh ubuntu@$SSH_HOST "sudo service app restart"
    - echo "deploy sucess"
  only:
    - master
```