---
layout: post
title: "[Stalk Project] Jenkins & Ansible을 활용한 CI/CD 구축기"
subtitle: "Jenkins와 Ansible의 필요성을 고려하여 Stalk 프로젝트의 CI/CD를 구축한 사례입니다."
categories: devlog
tags: ci cd jenkins ansible
---

> Jenkins를 통해 빌드 및 테스트를 자동화하고, Ansible을 통해 서버 배포를 자동화한 사례입니다. 

<!--more-->

- 목차
  - [개요](#-개요)
  - [Jenkins를 선택한 이유](#-jenkins를-선택한-이유)
  - [Stalk 프로젝트 CI/CD Pipeline](#-stalk-프로젝트-cicd-pipeline)
    - [Jenkins 서버 구성](#-jenkins-서버-구성)
      - [1. 서버에 Jenkins 설치](#1-서버에-jenkins-설치)
      - [2. Jenkins 서버에 접속하여 docker 설치](#2-jenkins-서버에-접속하여-docker-설치)
      - [3. Jenkins 각종 설정](#3-jenkins-각종-설정)
      - [4. Ansible Server 구축](#4-ansible-server-구축)


## 🌱 개요

프로젝트를 생성하고 서버에 배포하기 위해서는 다양한 환경 구성이 필요합니다.

- jar 혹은 war 파일로 빌드 (Spring 프로젝트 한정)
- Tomcat 등 외장 WAS 사용 (필요시)
- env 파일 관리
- 빌드된 파일을 실행하기 위한 환경 구성 (ex. jdk 설치 등)

등 여러가지가 있을 것입니다. 이것을 서버 배포할 때마다 개발자가 하나하나 확인하는 것은 시간과 노동의 낭비일 것입니다.

CI/CD 구축은 위와 같은 사항들을 자동화합니다. 또한, 프로젝트 코드를 테스트하고 실패한다면 배포를 진행하지 않는다던지, 다중화된 서버에 일관적인 명령을 수행할 수 있는 
간단한 오케스트레이션으로 활용한다던지 등 수많은 이점이 있습니다.

-------------

## 🌱 Jenkins를 선택한 이유

Jenkins와 Github Actions의 비교는 많은 사람들이 가질 수 있는 흥미로운 주제입니다.

<a href="https://ibb.co/HnbVwST"><img src="https://i.ibb.co/5BS6p0h/jenkins-vs-github-action.png" alt="jenkins-vs-github-action" border="0"></a>
###### 🔍 둘에 대한 비교는 더 좋은 글이 많으므로 패스합니다.

오픈소스에 무료로 사용할 수 있는 CI/CD 툴의 강자 Jenkins였지만, 최근 Github Actions이 빠르게 치고 올라오는 모습을 볼 수 있습니다. 빅테크 기업도 채용할만한 사이즈가 된 Github Action.. 이 두가지 중 왜 
Jenkins를 사용했는지에 대한 이유를 소개합니다.

- CI/CD Item을 한 곳에서 관리 가능

Jenkins의 장점은 하나의 서버에서 여러 파이프라인을 관리할 수 있다는 점입니다. 반면 Github Actions는 각 프로젝트에 워크플로우를 독립적으로 관리하여 유연성이 높지만, 여러 프로젝트의 CI/CD 상태를 한눈에 파악하기 어렵습니다.

Stalk 프로젝트에서는 MSA 아키텍처를 채택하고 있으며, 각 서비스의 CI/CD 파이프라인을 하나의 Jenkins 서버에서 관리하고 있습니다. 이를 통해 전체 프로젝트의 CI/CD 상태를 한 눈에 파악하고, 문제 발생 시 빠르게 대응할 수 있습니다.

> 최근 Jenkins에서 Github Action으로 CI/CD를 옮겨가는 추세인 것 같습니다. ([카카오엔터프라이즈가 GitHub Actions를 사용하는 이유](https://tech.kakao.com/posts/516))
> 이전 프로젝트인 Tosstock에서 Github Actions로 간단하게 CI/CD를 구현해본 경험은 Github Actions가 얼마나 많은 리소스를 절감해주는지 다시 한번 느끼게 해주었습니다.
> 
> Stalk 프로젝트에서는 참고 문헌이 보다 더 많은 Jenkins를 통해 CI/CD를 배포할 때 어떤 부분을 고려해야 하는지를 근본적으로 파악하고 개선하고자 합니다. Jenkins로 CI/CD를 구축한 이후, Github Actions의 필요성이 느껴진다면 마이그레이션 할 계획입니다. 이를 통해 CI/CD 관리의 유연성과 확장성을 극대화할 것입니다.

--------

## 🌱 Stalk 프로젝트 CI/CD Pipeline

현재는 컴퓨팅 서버의 부족으로 라즈베리파이로 단일 서버를 구축한 환경입니다. 내부적으로 docker를 통해 마치 별개의 IP를 할당받은 서버처럼 동작하도록 구현하였습니다.

그 중, CI/CD를 위해 Jenkins, Docker, Ansible을 활용했습니다.

<a href="https://ibb.co/sgBVmx7"><img src="https://i.ibb.co/zmk4n9d/stalk-cicd.jpg" alt="stalk-cicd" border="0"></a>

각 단계들의 대략적인 동작 방식은 이렇습니다.

- Github에 특정 Branch에 커밋될 때, Gtihub Webhook이 동작
- Jenkins가 Github으로부터 Repository를 clone
- Jenkins가 프로젝트를 Test / Build
- Build된 프로젝트와 환경 설정 등 dockerfile로 만들어 docker hub에 push
- docker image 파일 실행할 서버에 ansible로 접근 및 서버 구축

이렇게 큰 틀을 잡고 pipeline을 구축하면서 다양한 이슈들을 해결한 사례들을 소개합니다.

---

### 🥕 Jenkins 서버 구성

Stalk 프로젝트에서 Jenkins는 Docker 이미지를 기반으로 실행됩니다. 프로젝트의 CD 중에 Jenkins에서 빌드된 프로젝트와 Dockerfile을 Docker Hub에 push해야하는 요구 조건이 있었습니다. 

여기서 흔히 DinD, DooD 라고 불리는 Docker 이슈가 발생하면서 Jenkins 환경 구성 중 추가적인 설정이 필요해졌습니다. 해당 이슈는 Docker 컨테이너 안에서 Docker 컨테이너를 실행시키는 과정에서 발생하는데, Docker 아키텍처로 인해 
발생하는 이슈입니다.

- DinD 방식으로 Docker 컨테이너를 구현해야하는 단계 중 호스트 도커 컨테이너를 `--privileged mode`로 실행
- 이것으로 호스트 도커는 해당 머신에서 할 수 있는 모든 컨테이너와 파일 시스템에 접근할 수 있기 때문에 보안에 굉장히 취약

Docker 아키텍처로 인해 dind와 dood 방식 중 dood 방식으로 문제를 해결하기로 결정하였습니다.

> 참고 자료
>
> - [DinD(Docker in Docker) & DooD(Docker out of Docker)](https://velog.io/@wjdalsckd45/DinDDocker-in-Docker-DooDDocker-out-of-Docker) <br>
> - [DinD(docker in docker)와 DooD(docker out of docker)](https://jordy-torvalds.tistory.com/entry/%ED%8D%BC%EC%98%A8-%EA%B8%80-DinDdocker-in-docker%EC%99%80-DooDdocker-out-of-docker)

#### 1. 서버에 Jenkins 설치

해당 커멘드를 사용해 Docker가 설치되어 있는 서버에 실행시킵니다.

```shell
docker run -d \n
-u root \n
-v jenkins_home:/var/jenkins_home \n
-v /var/run/docker.sock:/var/run/docker.sock
-p 8080:8080 -p 50000:50000 \n
--restart=on-failure \n
jenkins/jenkins:lts-jdk17
```

- -u 옵션을 사용하여 docker cli 관련된 명령어를 sudo 없이 (관리자 권한)으로 실행합니다.
- `/var/run/docker.sock:/var/run/docker.sock`은 dood에서 핵심 옵션입니다.
  - 호스트 docker.sock과 내부 docker 컨테이너에서 sock과 마운트
  - DooD 방식은 호스트에 있는 Docker Daemon과 통신할 수 있도록 하는 옵션

#### 2. Jenkins 서버에 접속하여 Docker 설치

- `curl -fsSL https://get.docker.com -o get-docker.sh`
- `sh get-docker.sh`

#### 3. Jenkins 각종 설정

- Github Webhook을 위한 Credential 등록
- Publish Over SSH 플러그인 설치
- Ansible Server Publish over SSH 등록
- Item 생성 및 pipeline script 작성

```shell
pipeline {
    agent any
    
    stages {
        stage('Github clone') {
            steps {
                git branch: 'main', credentialsId: 'github_webhook_token',
                url: 'https://github.com/Joypple22/stalk.git'
            }
        }
        
        stage('Project Build') {
            steps {
                sh '''
                    ./gradlew clean build
                '''
            }
        }
        
        stage('Docker Image Build') {
            steps {
                sh '''
                    docker build -f /var/jenkins_home/workspace/stalk-main/script/docker/server-main/Dockerfile -t waterkite94/stalk-main /var/jenkins_home/workspace/stalk-main
                '''
            }
        }
        
        stage('Push Docker Image to Hub') {
            steps {
                sh '''
                    docker push waterkite94/stalk-main
                '''
            }
        }
        
        stage('ssh publish ansible server') {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'ansible-server', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'ansible-playbook -i inventory playbooks/stalk-main-server.yml', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '.', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
        }
    }
}
```

해당 pipeline으로 순차적으로 git clone -> test & build -> docker image build -> ansible-server에 ssh 접속 command 실행 으로 동작합니다.

pipeline의 장점은 해당 step들을 단계적으로 실행하고 성공여부에 따라 다양한 분기점을 만들 수 있다는 점입니다.

#### 4. Ansible Server 구축

ansible은 Jenkins가 CI/CD 완료한 이후, 빌드된 프로젝트 도커 이미지를 각 프로젝트를 실행할 서버에 SSH 접속하여 실행합니다. 프로젝트를 실행할 서버는 운영체제와 docker만 설치되어 있다면 언제든지 ansible을 통해 
서버 구축을 자동화하는 간단한 오케스트레이션으로 활용합니다.

###### ❗️ ansible 서버 또한 해당 라즈베리파이에서 docker 컨테이너로 실행됩니다.

- ubuntu 운영체제 이미지로 docker 컨테이너 생성
- 각종 필요한 패키지 설치 (vim, openssh, sshpass, ansible 등)
- inventory, ansible.cfg, ansible-playbook.yml 생성

```shell
# inventory

[stalk-main]
host1 ansible_host=xxx.xxx.xxx.xx1 ansible_port=xxxx ansible_user=username ansible_ssh_pass=password
host2 ansible_host=xxx.xxx.xxx.xx2 ansible_port=xxxx ansible_user=username ansible_ssh_pass=password
...
```

```shell
# .ansible.cfg

[defaults]
inventory = ./inventory
...
```

```shell
# ansible-playbook.yml

- hosts: stalk-main
  become: true

  tasks:
    - name: docker stop stalk-main server
      command: docker stop stalk-main-server
      ignore_errors: yes

    - name: docker rm stalk-main server
      command: docker rm stalk-main-server
      ignore_errors: yes

    - name: docker remove stalk-main image
      command: docker rmi waterkite94/stalk-main
      ignore_errors: yes 

    - name: docker pull stalk-main server image
      command: docker pull waterkite94/stalk-main

    - name: docker execute stalk-main server
      command: docker run -d -it -p 8080:8080 --name stalk-main-server waterkite94/stalk-main
```

ansible.playbook.yml은 jenkins에서 프로젝트를 빌드한 이후 최종적으로 실행해야할 yml 파일입니다.

Jenkins에서 CI (Continuous Integration), CD(Continuous Delivery)을 마쳤다면, ansible을 통해 CD(Continuous Deployment)를 진행하였습니다. yml 파일은 이 CD 작업을 위한 스크립트로 
docker 기반으로 실행되고 있는 stalk-main-server을 중지, 삭제, 새로운 docker image로 빌드 및 실행을 맡고 있습니다.

ansible을 활용하는 가장 큰 이유 중 하나는 바로 inventory입니다. 서버가 여러개라면 해당 yml 파일을 하나씩 실행해야하는 번거로움을 inventory를 통해 yml파일을 실행시킬 IP와 Port를 등록함으로 해당 컴퓨팅 
서버에 한번에 yml 파일을 실행하도록 할 수 있습니다. ansible은 해당 기능을 통해 간단한 오케스트레이션이 가능한 툴입니다.

Jenkins pipeline에서 마지막에 실행될 step은 바로 ansible 서버에 ssh 접속을 한 이후 ansible-playbook.yml을 실행하기 위함입니다.

이로써 Jenkins와 Ansible, Docker를 활용한 CI/CD 작업이 마무리 되었습니다.
