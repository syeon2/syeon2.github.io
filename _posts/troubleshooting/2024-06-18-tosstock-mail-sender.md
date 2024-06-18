---
layout: post
title: "[Troubleshooing] @Async을 활용한 이메일 수신 API 개선 방안"
subtitle: "비동기 처리와 Thread Pool, AOP의 개념을 활용한 성능 개선 사례입니다."
categories: devlog
tags: troubleshooting async thread_pool aop
---

> 비동기 처리와 Thread Pool을 이해하고 @Async를 통해 약 90%의 성능을 개선시켰습니다.

<!--more-->

- 목차

## 🌱 문제 인식

Tosstock 프로젝트는 회원가입을 진행할 때 이메일을 통해 인증번호를 부여받습니다.

이 때, 회원의 사용자 경험을 높이기 위해 다음과 같은 구현 조건이 있습니다.

- <strong>이메일 전송 버튼을 누르고 클라이언트와 서버 간의 통신이 완료될 때까지 로딩 스피너가 돌아갑니다.</strong>
- <strong>한번의 통신이 완료될 때까지 최소한으로 중복으로 이메일이 발송되지 않게 동작합니다.</strong>
- <strong>최대한 로딩 스피너가 돌아가는 시간을 최소로 하여 유저 요청이 잘 진행되고 있다는 것을 인지시킵니다.</strong>

이러한 조건들이 있다고 가정합니다.

기존 Tosstock 프로젝트에서 읹증번호 요청 API의 속도는 4s가 걸렸습니다. 다양한 이론들을 활용하여 최대한 응답속도를 빠르게 하는 트러블 슈팅을 진행합니다.

-----

## 🌱 트러블 슈팅

### 🥕 기존 코드
```java
@Service
@RequiredArgsConstructor
public class MailService implements SendAuthCodeUseCase {

    private final JavaMailSender javaMailSender;

    @Override
    public boolean dispatchAuthCodeToEmail(String toEmail) {
        //...
        sendMail(toEmail, authCode);
        //...
        return true;
    }

    private void sendMail(String toEmail, String authCode) {
        //...
        javaMailSender.send(emailForm);
        //...
    }
}
```

- `implementation 'org.springframework.boot:spring-boot-starter-mail`에서 제공하는 JavaMailSender을 이용합니다.

### 🥕 실행 결과

<a href="https://ibb.co/fHnXkLX"><img src="https://i.ibb.co/Ttbw4dw/prev-sender.png" alt="prev-sender" border="0"></a>

---

## 🌱 개선 방안

`javaMailSender.send()` 메서드는 기본적으로 동기적으로 실행됩니다. 이를 비동기적으로 실행할 수 있다면 어느정도 성능을 개선할 수 있을 것 같다고 판단했습니다.

Spring에서는 기본적으로 다양한 어노테이션을 제공합니다. 그 중 @Bean으로 등록되어져 있는 클래스의 메서드를 비동기적으로 실행할 수 있는 `@Async`를 사용하여 성능을 개선하였습니다.

```java
@Service
public class AuthCodeMailService implements SendAuthCodeUseCase {
	
    private final AuthCodeMailUtil authCodeMailUtil;

    @Override
    public boolean sendAuthCodeToEmail(String toEmail) {
        //...
        saveAuthCodePort.save(toEmail, authCode);
		//...
    }
}

//-------------------------

@Slf4j
@Component
@RequiredArgsConstructor
public class AuthCodeMailUtil {

    private final JavaMailSender javaMailSender;

    @Async(value = "threadPoolTaskExecutorForMail")
    public void sendMailByJavaMailSender(String toEmail, String title, String text) {
        SimpleMailMessage emailForm = AuthCodeMailUtil.createEmailForm(toEmail, title, text);

        try {
            javaMailSender.send(emailForm);
        } catch (Exception e) {
            log.error("---Exception Auth Code Mail Sender---");
        }
    }
}
```

- @Async 어노테이션은 Proxy 구조를 기반으로 동작합니다.
- 한 클래스 내부에서 `@Async`로 되어 있는 메서드에 별도의 Thread를 부여하여 비동기적으로 처리합니다.
- 비동기적으로 처리하게 되면 `javaMailSender.send()`는 별도의 예외 처리를 두어야합니다.
  - Spring의 Exception Handler는 이메일 요청이 완료되기 전에 API 요청을 완료하기 때문
  - `javaMailSender.send()`는 비즈니스 코드가 아닌 라이브러리 메서드라서 클라이언트가 라이브러리의 문제를 인지할 필요까지는 없다고 판단
  - 라이브러리 문제는 개발진이 바로 파악할 수 있도록 Slack 등과 같은 커뮤니케이션 툴과 연동하여 알림을 받도록 합니다.


> 💡 주의사항 (Thread Pool이란?)
> 
> @Async는 기존의 쓰레드와 달리 별도의 쓰레드를 부여하여 메서드가 동작하기 때문에 비동기적으로 처리가 가능합니다. 이 때 주의해야할 점은 별도로 부여받는 쓰레드를 고려하는 것입니다.
> 
> @Async는 메서드를 실행할 새로운 쓰레드를 제한없이 생성합니다. @Async 메서드가 실핼될 때마다 쓰레드가 실행되는데, 과도한 메서드의 호출로 생성될 쓰레드가 문제가 될 수 있습니다.
>
> 이 문제는 곧 OOME(Out of Memeory Exception)을 발생시켜 프로그램이 중단될 여지를 가집니다.
> 
> OOME 문제를 예방하기 위해서 서버 별로 Thread Pool을 두어 @Async가 발생하는 메서드가 Thread Pool에 생성되어 있는 쓰레드를 통해 동작할 수 있도록 합니다.

```java
@Configuration
public class MailAsyncConfig {

    @Bean(name = "threadPoolTaskExecutorForMail")
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("authcode-mailsender");
        executor.initialize();
        
        return executor;
    }
}
```

- @Async가 동작할 쓰레드를 @Bean으로 등록하여 생성합니다.
- Bean에 특정 이름을 부여하여 @Async가 어떤 Thread Pool에서 쓰레드를 가져다 실행할지를 지정할 수 있습니다.
- @Async로 호출될 때 생성되는 쓰레드의 로직보다 Thread Pool로 인해 미리 생성되어져 있는 쓰레드를 가져와 실행하는 매커니즘 또한 속도 개선에 긍정적인 영향을 줍니다. 

### 🥕 실행 결과

<a href="https://ibb.co/yVN6J0b"><img src="https://i.ibb.co/fQXk7MP/post-sender.png" alt="post-sender" border="0"></a>

> 4s(4000ms)가 걸리던 API의 응답속도에서 약 300ms까지 응답 속도를 개선시켰습니다. 

---

## 🌱 비동기 처리 방식으로 개선한 문제의 단점

- 클라이언트는 메일 라이브러리를 통한 예외를 응답받지 못하게 되었습니다.
  - API Response를 통해 문제를 인식할 수 없게 되었고 오직 서버 log를 통한 알림을 통해 예외 발생을 인지할 수 있습니다.
    

- 사용되어지는 쓰레드 양의 증가 (메모리)
  - 기존 로직은 하나의 API 처리시 WAS로부터 할당받아 처리하는 쓰레드만 필요하였습니다.
  - @Async로 문제를 개선한 이후 하나의 API를 처리하는데 최대 2개의 쓰레드가 필요하게 되어 메모리 사용량이 증가합니다.
  - 관리해야할 Thread Pool이 생겨 지속적인 모니터링이 필요하게 되었습니다. 

=> Slack 같은 커뮤니케이션 툴을 통해 알림을 받도록 합니다. 오히려 비즈니스 코드의 문제가 아니라 라이브러리 자체의 결함으로 파악하여 빠르게 서버를 점검할 수 있습니다.

=> 이메일 인증이 많은 요청을 일으키는 API가 아니라는 점, 이메일을 보낸 이후 Redis에 인증번호를 저장하는 비즈니스 로직을 제외하고는 특별한 고성능의 로직이 존재하지 않기 때문에 2개의 쓰레드가 동시에 존재하는 시간은 작을 것이라 판단하여 적용하게 되었습니다.

---

## 🌱 결론

비교적 간단한 작업으로 문제를 해결한 사례였습니다. 하지만 문제를 해결하고, 유지보수와 안정성 있도록 운영하기 위해 다양한 이론적 지식들이 기반 되어야하는 트러블슈팅이었습니다.

현업에서 작업했다면 팀원 간 해당 문제 해결의 장점과 단점을 고려한 충분한 논의가 필요했기에 개발자의 커뮤니케이션 스킬이 중요하다는 것을 다시 한번 느끼게 되는 트러블 슈팅이었습니다.

---

[//]: # (#### 🥕 참고 자료)

[//]: # ()
[//]: # (- @Async와 AOP)

[//]: # (- Thread Pool이란)

[//]: # (- 동기와 비동기)