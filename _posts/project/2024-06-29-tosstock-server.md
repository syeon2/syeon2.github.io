---
layout: post
title: "[Stalk Project] 아기자기한 라즈베리파이 서버 구축"
subtitle: "AWS 클라우드를 대체해 사용할 서버를 직접 구축한 사례입니다."
categories: devlog
tags: troubleshooting server cs network
image:
  path: https://i.ibb.co/w0mxVQ6/rasberrypi.jpg
---

> AWS 클라우드의 비용 절감 및 Jenkins, Redis 등의 인프라 툴을 사용하기 위해 라즈베리파이로 서버를 구축한 사례입니다.

<!--more-->

- 목차
  - [서버 구축 배경](#-서버-구축-배경)
  - [서버 스팩](#-서버-스팩)
  - [Troubleshooting 포트포워딩](#-troubleshooting-포트포워딩)
    - [문제 인식](#-문제-인식)
    - [문제 해결](#-문제-해결)
  - [결론](#-결론)

## 🌱 서버 구축 배경

Tosstock 프로젝트는 AWS 프리티어 계정으로 각종 서비스를 사용하여 서버를 구축하여 사용했습니다. 하지만 프리티어 계정으로는 사용에 여러 한계에 직면하였습니다.

- 작은 AWS EC2의 메모리
  <a href="https://ibb.co/5MFKgZs"><img src="https://i.ibb.co/dWgmwZk/ec2-image.png" alt="ec2-image" border="0"></a>

프리티어가 사용할 수 있는 컴퓨팅 서버는 t2.micro이고 1GiB의 메인 메모리로 사용할 수 있습니다.

원래 Tosstock 프로젝트는 Jenkins를 사용하여 CI/CD를 구축하려고 했으나 EC2안에서 Docker 컨테이너로 실행한 Jenkins가 빌드 중 계속 다운되는 현상이 발생했습니다.

구글링을 통해 간혹 프리티어 계정으로 만들어진 EC2 인스턴스에서 Jenkins가 다운된다는 여러 글을 접하였고 급하게 Github Action을 통해 CI/CD를 구축하였습니다.

> Github Action으로 CI/CD를 구축하면서 느낀 점은 확실히 참조할만한 레퍼런스의 수가 Jenkins가 훨씬 많다는 것이었습니다.  
> 
> 또한, 규모가 커질 프로젝트를 고려하여 여러 CI/CD 아이템들을 관리해야할 상황이 생길 것을 고려하였습니다. Jenkins는 이러한 고민을 해결해 줄 수 있는 
> 툴이라고 판단하여 꼭 구축해보고 싶었습니다.

- ElastiCache 서버 개수의 한계

프리티어는 하나의 VPC에 하나의 ElasiCache만 생성하는 것을 무료로 제공하고 있습니다.

Tosstock은 Redis를 사용하는 도메인이 많은 프로젝트입니다. 현재는 Refresh Token을 저장하는 용도로만 ElastiCache를 사용하고 있고 나머지는 
서버가 있는 EC2의 Docker 컨테이너에 Redis를 띄워 사용하고 있습니다.

모든 도메인에서 사용하고 있는 Redis를 localhost가 아닌 외부 IP를 통해 사용하여 현장감을 주고 싶었습니다.

- <strong>비용</strong>

역시나 클라우드의 비용이 걸렸습니다. 거의 20-30일 정도 Tosstock 프로젝트 서버를 구축해서 유지해왔는데 약 $2.23, 한화로 3천원 가량이 부과되었습니다.

ElastiCache를 사용하기 위해서는 VPC를 꼭 만들고 ElastiCache를 사용할 각 컴퓨팅 리소스들을 묶어주어야하는데, 프리티어 계정이어도 VPC 비용은 부과됩니다.

> 전부터 개인 서버가 있었으면 좋겠다는 생각과 함께 지금이 서버를 만들기에 좋은 타이밍이지 않을까 판단하였습니다. 이왕 개인 서버를 구축할거 스펙도 조금 고려해서 
제대로 만들어보자 했습니다. ~~(사실 서버 구축의 제일 큰 목적은 사심..🥹)~~

---

## 🌱 서버 스팩
- 컴퓨터 : 라즈베리파이5 (라즈비안 OS, Linux Debian 기반)
- CPU : 쿼드 코어 (ARM Cortex-A76), 2.4GHz
- RAM : 8GB
- Memory : Samsung 980 M.2 NVMe (1TB)

OS는 우분투와 고민하다 결국 라즈베리파이 전용 OS인 라즈비안 OS를 선택했습니다. Linux Debian 기반의 OS이지만 서버로 구현하고자 하는 목표에 필요한 Docker를 지원하기 때문에 
호환성을 고려하여 선택했습니다.

커스텀을 고려한 부분은 HDD, SDD 같은 하드 디스크 메모리였습니다. 구축할 서버의 특성상 DB에 접근, 읽기 및 쓰기 작업이 빈번합니다. 라즈베리파이가 기본으로 제공하는 
MicroSD 지원보다 SSD를 사용하는 것이 구축할 서버의 목적에 적합한 선택지라고 판단하였습니다.


<table>
  <tr>
    <th>성능 항목</th>
    <th>SanDisk 32 GB micro SD카드</th>
    <th>980 Pro (PCIe 2.0)</th>
    <th>980 Pro (PCIe 3.0-ish)</th>
  </tr>
  <tr>
    <td><strong>순차 읽기</strong></td>
    <td>82.04 MB/s</td>
    <td>416.87 MB/s</td>
    <td>797.13 MB/s</td>
  </tr>
  <tr>
    <td><strong>순차 쓰기</strong></td>
    <td>53.20 MB/s</td>
    <td>413.17 MB/s</td>
    <td>746.30 MB/s</td>
  </tr>
  <tr>
    <td><strong>4k 랜덤 읽기 (IOPS)</strong></td>
    <td>3,722 IOPS</td>
    <td>102,400 IOPS</td>
    <td>126,419 IOPS</td>
  </tr>
  <tr>
    <td><strong>4k 랜덤 쓰기 (IOPS)</strong></td>
    <td>835 IOPS</td>
    <td>55,956 IOPS</td>
    <td>50,319 IOPS</td>
  </tr>
  <tr>
    <td><strong>4k 순차 읽기</strong></td>
    <td>14,680 KB/s</td>
    <td>175,700 KB/s</td>
    <td>230,856 KB/s</td>
  </tr>
  <tr>
    <td><strong>4k 순차 쓰기</strong></td>
    <td>4,723 KB/s</td>
    <td>136,852 KB/s</td>
    <td>164,065 KB/s</td>
  </tr>
  <tr>
    <td><strong>4k 랜덤 읽기</strong></td>
    <td>14,689 KB/s</td>
    <td>60,172 KB/s</td>
    <td>27,292 KB/s</td>
  </tr>
  <tr>
    <td><strong>4k 랜덤 쓰기</strong></td>
    <td>2,933 KB/s</td>
    <td>156,278 KB/s</td>
    <td>204,744 KB/s</td>
  </tr>
</table>

###### 참고 자료 : [라즈베리파이 MicroSD vs SSD](https://blog.naver.com/roboholic84/223313400266)

> 블로깅으로 열심히 만든 웹 서버가 만들어졌습니다.
> 
> 1차적으로는 라즈베리파이의 SSH 접근을 허용하여 같은 네트워크 망에서만 서버에 접근할 수 있었습니다. 궁극적인 목표는 외부망에서도 서버에 접근할 수 있고, 
> 허용된 포트로 접근하여 API call 및 API Docs를 제공, DB, 인프라 등에 접근할 수 있도록 하는 것입니다.

<p align="center" style="color:gray">
  <img style="margin:50px 0 10px 0" src="https://i.ibb.co/w0mxVQ6/rasberrypi.jpg" />
  작고 소중한 내 라즈베리파이🍓
</p>

---

## 🌱 Troubleshooting: 포트포워딩

### 🥕 문제 인식
개인 서버를 만들고 와이파이에 연결하는 것만으로는 외부에서 접근할 수 없습니다.

같은 망 안에서는 같은 공유기에 연결되어 있는 기기들간 ssh 인증을 통한 접근이 가능합니다. 하지만 외부에서는 현재 기기가 네트워크 통신을 위해 가지고 있는 IP로 접근이 불가합니다.

> 공유기에는`DHCP`(Dynamic Host Configuration Protocol) 라는 프로토콜로 호스트에게 IP 주소, 각종 설정 정보들을 제공하는 기능을 가지고 있습니다.
> 
> 대표적으로 공유기가 가지고 있는 IP가 아닌 공유기 내부에서만 사용할 수 있는 사설 IP 주소를 각 호스트에게 할당합니다. 해당 IP는 공유기 내부에서만 유효함으로 보안성을 가지고 
> 있으며, 전세계적으로 IP 주소 고갈의 이슈를 방지하기 위한 기술로도 설명할 수 있습니다.

<a href="https://ibb.co/5206FhM"><img src="https://i.ibb.co/259yN3n/tosstock-network1.jpg" alt="tosstock-network1" border="0"></a>

### 🥕 문제 해결

공유기 환경을 전제로 다음과 같은 단계를 통해서 문제를 해결하였습니다.

- 라즈베리파이에 고정 IP를 할당
- 포트포워딩

제 라즈베리파이는 Wireless Lan(Wifi)을 통해 통신을 주고 받습니다. 보통, 통신 기기가 공유기로부터 와이파이 연결을 할 때 부여받는 IP는 때에 따라 달라집니다.

이후에 작업할 포트포워딩은 공유기 IP와 외부로부터 접근할 수 있는 IP와 Port를 할당하고, 외부 IP와 고정 IP로 등록한 IP와 맵핑하여 내부 IP와 Port에 접근할 수 있습니다.

이러한 이유로 먼저 라즈베리파이에 고정 IP를 할당하고, 외부 포트와 맵핑할 내부 IP, 내부 포트를 입력해주면 외부에서 해당 서버로 접근할 수 있는 파이프라인이 완성됩니다.

<a href="https://ibb.co/QkP9dBK"><img src="https://i.ibb.co/RN32ynD/tosstock-network2.jpg" alt="tosstock-network2" border="0"></a>

> 💡 포트포워딩이란?
> 
> 가정에서 흔히 사용하는 공유기는 보안을 위해 외부로부터의 접근은 차단합니다. 하지만 포트 포워딩은 지정된 포트만을 개방해서 본래 공인 IP와 개방된 포트를 입력하면, 
> 내부 사설 IP와 지정된 내부 포트에 맵핑되어 통신을 가능하도록 합니다.
> 
> 흔히 DMZ라는 설정으로도 알려져 있지만, DMZ는 하나의 공유기에 하나의 사설 IP, 즉 1:1로 모든 포트를 맵핑하여 접근가능하도록 합니다. 이 기능은 기기의 노출을 최소화시켜 
> 칩입자로부터 공격 범위를 최소화하기 때문에 보다 안정적입니다.

--------------------

## 🌱 결론

이것으로 작고 소중한 개인 서버가 생겼습니다. 비록 서버가 하나밖에 없지만 꼭 나만의 서비스를 만들어 배포해보고 싶다는 마음이 커져갑니다 :)