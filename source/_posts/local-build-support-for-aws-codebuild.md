---
title: '[AWS] CodeBuild 로컬 테스트 및 디버깅'
date: 2019-08-25 15:49:12
tags:
  - aws
  - codebuild
  - docker

thumbnailImage: thumbnail.png
thumbnailImagePosition: left
autoThumbnailImage: yes
coverSize: full
coverMeta: in
coverImage: cover.png

---

이번 글에서는 로컬에서 Docker 로 AWS CodeBuild 환경을 구현하여 로컬에서 테스트 할 수 있는 방법에 대해 소개합니다.

<!-- excerpt -->

* [배경](#배경)

* [로컬 환경 구현하기](#로컬-환경-구현하기)

* [참고글](#참고글)

## 배경

회사 배포 방식을 AWS CodeBuild 로 production 배포 환경을 구현한 상태였습니다. 그러나 가끔 배포가 실패하는데 디버깅을 할 수 있는 환경이 없어서 매번 AWS 콘솔에서 직접 수동으로 테스트 하는것이 번거로웠습니다. 

그래서 로컬에서 AWS CodeBuild 하는 방법을 찾았고 소개하려 합니다. CodeBuild 에 필요한 buildspec.yml 작성 방법을 공부하는 입장에서도 매우 유용할 것입니다. 


{% alert warning %}
테스트는 MacOS 에서 진행하였습니다.
{% endalert %}
## 로컬 환경 구현하기

로컬 CodeBuild 환경에 대한 이해를 하기 위해 3가지 요소를 설명합니다.

<br>

- AWS CodeBuild local agent

  쉽게 말해서 빌드 서버 입니다. 빌드 하는 곳은 모두 빌드 서버라고 할 수 있습니다. AWS 에서는 CodeBuild 라는 서비스로 빌드 서버를 제공합니다.

- AWS CodeBuild image

  CodeBuild agent 가 프로젝트를 빌드하기 위해 필요한 이미지입니다. 
  각 프로젝트 개발 환경은 다릅니다. python, nodejs, java 등등 개발 언어 또는 OS 환경에 따라 빌드 환경을 구성해야 합니다. AWS는 여러 빌드 환경을 제공하고 있습니다.

- Project code

  빌드할 코드 파일입니다. 저는 예제로 생성한 [vue.js 기본 템플릿](https://github.com/devhaks/aws/tree/blog/local-build-support-for-aws-codebuild)을 빌드 하겠습니다.

#### 1) 로컬에 Docker 설치

  - Docker를 설치하지 않은 경우, [Docker 초급](https://devhaks.github.io/2019/06/30/docker-1/)을 참고하세요.
  - 로컬이 없다면, [AWS Cloud9](https://aws.amazon.com/ko/cloud9/) (Web browser IDE)을 이용하여 docker 설치 없이 다음 단계로 진행 가능합니다.

#### 2) CodeBuild image 빌드하기

  - 로컬에 다음 저장소를 다운로드 합니다.

  ```bash
  $ git clone https://github.com/aws/aws-codebuild-docker-images
  ```

  - 프로젝트에 맞는 CodeBuild image 를 생성합니다. 
  - 제가 예시로 생성한 vue.js 는 NPM 기반이기 때문에 nodejs 용 빌드 이미지를 생성하겠습니다. 

  ```bash
  $ cd aws-codebuild-docker-images
  
  # aws-codebuild-docker-images/
  # 지원하는 이미지 목록 입니다. 디렉토리명이 왜 unsupported_images 인지는 잘 모르겠네요.
  $ ls -l ubuntu/unsupported_images/
    total 0
    drwxr-xr-x   3 lewis  staff    96B  8  4 22:58 android-java-6
    drwxr-xr-x   3 lewis  staff    96B  8  4 22:58 android-java-7
    drwxr-xr-x   4 lewis  staff   128B  8  4 22:58 android-java-8
    drwxr-xr-x   5 lewis  staff   160B  8  4 22:58 docker
    drwxr-xr-x   5 lewis  staff   160B  8  4 22:58 dot-net
    drwxr-xr-x   7 lewis  staff   224B  8  4 22:58 golang
    drwxr-xr-x   7 lewis  staff   224B  8  4 22:58 java
    drwxr-xr-x  10 lewis  staff   320B  8  4 22:58 nodejs
    drwxr-xr-x   5 lewis  staff   160B  8  4 22:58 php
    drwxr-xr-x   8 lewis  staff   256B  8  4 22:58 python
    drwxr-xr-x   7 lewis  staff   224B  8  4 22:58 ruby
    drwxr-xr-x   3 lewis  staff    96B  8  4 22:58 ubuntu-base

  $ cd ubuntu/unsupported_images/nodejs/10.14.1
  
  # aws-codebuild-docker-images/ubuntu/unsupported_images/nodejs/10.14.1/
  $ ls -l
  total 32
  -rw-r--r--  1 lewis  staff  7325  8  4 22:58 Dockerfile
  -rwxr-xr-x  1 lewis  staff   439  8  4 22:58 dockerd-entrypoint.sh
  -rw-r--r--  1 lewis  staff    51  8  4 22:58 ssh_config

  # build 명령어 도움말 출력하여 빌드를 위한 -t 옵션 설명보기
  $ docker build --help 
  ...
  -t, --tag list        Name and optionally a tag in the 'name:tag' format
  ...

  # codebuild 이미지 생성하기
  # build -t 옵션 설명대로 name은 'aws/codebuild/nodejs', tag는 '10.14.1' 로 설정합니다.
  # 마지막 '.' 표시는 기본적으로 Dockerfile 이 존재하는 경로를 의미합니다. 
  # 현재 경로에 Dockerfile 이 있으므로 '.' 로 적습니다.
  $ docker build -t aws/codebuild/nodejs:10.14.1 .
  Sending build context to Docker daemon  11.26kB
  ...
  Successfully built 03c41ff10894
  Successfully tagged aws/codebuild/nodejs:10.14.1

  # docker 이미지 목록 출력
  $ docker images
  REPOSITORY            TAG       IMAGE ID      CREATED       SIZE
  aws/codebuild/nodejs  10.14.1   03c41ff10894  8 weeks ago   1.34GB
  ```

#### 3) AWS CodeBuild local agent 이미지 다운로드

 - 이 이미지는 실제 AWS CodeBuild agent 처럼 동일한 환경을 가진 이미지 입니다.
 - [aws-codebuild-local](https://hub.docker.com/r/amazon/aws-codebuild-local) 이미지를 다운로드 합니다. 

  ```bash
  $ docker pull amazon/aws-codebuild-local:latest --disable-content-trust=false
  ...

  $ docker images # 이미지 목록 출력 
  REPOSITORY                  TAG       IMAGE ID      CREATED       SIZE
  amazon/aws-codebuild-local  latest    b0bdf3d66f0e  2 months ago  563MB
  ```

#### 4) project 빌드하기

  - [vue.js 기본 템플릿](https://github.com/devhaks/aws/tree/blog/local-build-support-for-aws-codebuild)을 다운로드 합니다.

  ```bash
  $ git clone git@github.com:devhaks/aws.git
  $ cd aws
  # 브랜치 변경
  aws $ git checkout blog/local-build-support-for-aws-codebuild
  ```
  - 로컬 빌드를 하기 위해 프로젝트에 [codebuild_build.sh](https://raw.githubusercontent.com/aws/aws-codebuild-docker-images/master/local_builds/codebuild_build.sh) 파일을 다운로드 하고 실행 권한을 변경합니다.
  ```bash
  aws $ chmod +x codebuild_build.sh
  ```

  - 프로젝트의 buildspec.yml 살펴보기

  ```yml
  version: 0.2
  
  phases:
    install:  # 빌드에 필요한 패키지 설치
      commands:
        - npm install  # 설치 명령어
    pre_build:  # 빌드 전처리: 코드 검사 
      commands:
        - npm run lint
    build:  # 빌드 실행
      commands:
        - npm run build
    post_build: # 빌드 후처리
      commands:
        - rm -rf node_modules .git # 빌드 결과물 용량을 최적화 하기 위한 패키지 및 .git 삭제
        - ls -al
  artifacts:  # 결과물 설정
    files:
      - '**/*'  # 모든 파일을 결과물로 생성
  ```

  - local build 테스트하기
  
  ```bash
  # aws/codebuild/nodejs 의 image_id => 03c41ff10894
  # 현재 경로의 artifacts 디렉토리에 결과물 저장하도록 설정
  aws $ ./codebuild_build.sh -i 03c41ff10894 -a ./artifacts

  Creating network "agent-resources_default" with the default driver
  Creating volume "agent-resources_source_volume" with local driver
  Creating volume "agent-resources_user_volume" with local driver
  Creating agent-resources_agent_1 ... done
  Creating agent-resources_build_1 ... done
  ...
  agent_1  | [Container] 2019/09/30 03:54:28 Phase complete: POST_BUILD State: SUCCEEDED
  agent_1  | [Container] 2019/09/30 03:54:28 Phase context status code:  Message: 
  agent_1  | [Container] 2019/09/30 03:54:28 Expanding base directory path: .
  agent_1  | [Container] 2019/09/30 03:54:28 Assembling file list
  agent_1  | [Container] 2019/09/30 03:54:28 Expanding .
  agent_1  | [Container] 2019/09/30 03:54:28 Expanding artifact file paths for base directory .
  agent_1  | [Container] 2019/09/30 03:54:28 Assembling file list
  agent_1  | [Container] 2019/09/30 03:54:28 Expanding **/*
  agent_1  | [Container] 2019/09/30 03:54:28 Found 21071 file(s)
  agent_1  | [Container] 2019/09/30 03:54:37 Preparing to copy secondary artifacts
  agent_1  | [Container] 2019/09/30 03:54:37 No secondary artifacts defined in buildspec
  agent_1  | [Container] 2019/09/30 03:54:37 Phase complete: UPLOAD_ARTIFACTS State: SUCCEEDED
  agent_1  | [Container] 2019/09/30 03:54:37 Phase context status code:  Message:
  agent-resources_build_1 exited with code 0

  # 빌드 결과 확인
  aws $ ls -l artifacts
  total 71056
  -rw-r--r--  1 lewis  staff  35887169  9 30 12:54 artifacts.zip
  ```
  
  - codebuild_build.sh 사용법
  ```bash
  $ codebuild_build.sh [-i image_name or image_id] [-a artifact_output_directory] [options]
  ```

  {% alert warning %}
  주의할 점은 `image_name` 에 `/` 가 포함된 경우 docker hub 의 이미지를 검색하기 때문에 `image_id` 를 사용합니다. 
  {% endalert %}

## 참고글

[AWS 공식 문서 - 로컬 빌드 테스트](https://aws.amazon.com/ko/blogs/devops/announcing-local-build-support-for-aws-codebuild/)