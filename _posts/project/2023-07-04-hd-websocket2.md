---
layout: post
title: "[Hot-Dealicious #2] WebSocket 성능 테스트"
subtitle: "WebSocket 성능 테스트"
categories: project
tags: project Network WebSocket
---

> 다양한 툴을 사용하여 WebSocket의 성능을 테스트해본 경험을 이야기합니다.

<!--more-->

## 🌱 WebSocket의 성능을 분석해보자.
이번 프로젝트의 주요한 기능은 WebSocket을 활용하여 실시간으로 라이더의 위치정보를 클라이언트에게 전송하는 것이다. 나름 대규모 시스템을 염두하고 코드를 작성하고 
있어서 이 서버가 WebSocket에 대한 부하를 어느정도까지 커버할 수 있을지 궁금했고, Trouble shooting할 영역이 있는지 확인할 수 있는 근거를 마련할 수 있겟다고 생각했다.

또 코드를 작성하면서 내가 작성한 코드가 어느정도 성능을 낼 수 있을지에 대한 메트릭 자료도 제대로 활용할 수 있는 기회라고 생각하여 주저없이 성능 분석에 대한 스터디를 시작했다.

---
## 📚 JMeter를 사용한 부하 테스트
## 🌱 JMeter로 테스트하기까지
현재 프로젝트에 적용되어 있는 WebSocket 라이브러리는 Spring WebSocket과 Stomp를 조합하여 사용하고 있다. 단순 WebSocket을 테스트하기에는 Postman으로도 가능하지만 Stomp 프로토콜의 
API 테스트는 지원하지 않고 있다. 이에 Apic이라는 툴을 알게되어 단일성으로 간단하게 테스트하고 있었다.

기능이 동작하는지만 테스트하려면 Apic으로도 충분했지만 대용량 트래픽이 발생하는 경우를 테스트를 통해 간접적으로 경험해보고 싶었다. 매우 유명한 ngrinder 툴이 있었지만 WebSocket API로 많은 
부하를 줄시에는 다소 정확하지 않다는 [카카오 기술 블로그](https://tech.kakao.com/2020/06/15/websocket-part2/)를 통해 접하게 되었다. 그래서 알게된 JMeter를 통해 테스트를 진행해보았다.

Stomp 프로토콜로 WebSocket 연결을 하게 되면 다양한 커멘드를 통해 요청을 주고 받을 수 있다. 이번 테스트 환경에서는 WebSocket Connect -> Send 순으로 요청 시나리오를 만들어 다양하게 테스트해보았다.

1. 각 1번, 1000번, 5000번, 10000번의 요청 시나리오를 1초에 처리하기.
2. 10000번, 50000번의 요청 시나리오를 n초에 걸쳐 처리하기.

---
## 🌱 테스트 결과
1번, 1000번 5000번까지는 결과가 동일했다. 서버 어플리케이션도 잘 동작했고 요청도 잘 처리되었다. 문제는 5000번 이상의 요청 시나리오를 1초에 처리할 때 발생하였다.

<img src="https://wkblog-images.s3.ap-northeast-2.amazonaws.com/project/hot-dealicious/websocket-jmeter.png">
<img src="https://wkblog-images.s3.ap-northeast-2.amazonaws.com/project/hot-dealicious/websocket-jemter2.png">

적은?량의 처리할 때와는 다르게 10000건 이상의 요청을 처리할 때는 Error 발생률이 증가했다. 위의 상황만보더라도 거의 3분의 1이 요청 실패로 이어저 대책을 마련해야했다.
먼저 에러가 발생할 상황들을 가정해보았다.

1. WebSocket은 서버와 클라이언트가 커넥션을 계속 유지해야하는 네트워크 프로토콜이다. 그렇기 때문에 서버와 클라이언트가 네트워크 통신을 할 때 사용하는 소켓을 계속 유지하고 있어야한다.
소켓은 보통 네트워크 TCP/UDP Layer에서 어플리케이션 Layer로 올라오게 될 때 파일 디스크립터에 할당된다. 각 프로세스마다 할당될 수 있는 파일 디스크립터는 제한되어 있다고 알고 있어 이에 대한 문제가 아닐까 생각했다.
2. 두번째 가정 상황은 OOME가 발생했을 경우이다. 파일 디스크립터가 충분히 만들어질 수 있는 요청일지라도 소켓을 메모리에 저장하는 메모리가 Max Heap을 넘어 OOME가 발생하게 된다면 이 상황으로 인해 요청에 에러가 
발생할 수도 있겠대고 생각했다.
3. CPU와 Thread의 문제도 고려해보았다. 동시다발적으로 WebSocket을 생성할 때 동시성 이슈가 발생할 수도 있지 않을까 생각해보았다.(사실 WebSocket은 일반적으로 동시성 이슈를 고려하지 않아도 된다고 한다.) 또 혹여나 많은 요청이 한꺼번에 처리해야하는 상황에서 gc에 부하가 걸려 hang이 되는 상황도 고려해보았다.

### 🟤 서버의 메모리/Thread/CPU 확인
1. 먼저 VisualVM을 통해 서버의 여러 리소스를 확인해보았다.
<img src="https://wkblog-images.s3.ap-northeast-2.amazonaws.com/project/hot-dealicious/websocket-vm.png" />
당시 상황을 캡쳐한 메트릭이다. CPU를 먼저 보자. 만일 Race Condition으로 발생한 이슈라면 CPU 가용률이 순간적으로 올라가고 jvm thread의 상태는 lock이 되어야 한다. 하지만 위의 상황을 보았을 때 처리를 위해 CPU 처리량이 
높아진 것은 확인할 수 있지만, fastThread를 통해 dump를 떠서 Thread 상태를 확인해보았을 때는 lock이 걸린 thread는 존재하지 않았다.
<br />
<br />
또한 Heap 메모리도 안정적으로 확장된 것이 보인다. Max Heap memory 안에서 요청을 잘 처리하고 있는 것을 확인해볼 수 있었다.
<br />
<br />

2. 혹여나 서버의 처리량이 많아 GC에 hang이 걸린 것은 아닌지 확인해보았다.
<img src="https://wkblog-images.s3.ap-northeast-2.amazonaws.com/project/hot-dealicious/websocket-fastthread.png">
thread dump를 뜨고 fastThread로 메트릭을 확인해보았다. 여러 GC Thread들이 눈에 들어왔지만 모두 상태가 RUNNABLE인 것을 확인할 수 있었다. 이것을 통해 gc에 hang이 걸린 문제도 아니라고 판단이 되었다.

이것을 통해 위의 1, 2, 3번 가정 모두 현 상황과 맞지 않음을 알 수 있었다...(그럼 뭘까..?🤔 이후 계속...)
