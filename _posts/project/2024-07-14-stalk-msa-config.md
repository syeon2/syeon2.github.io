---
layout: post
title: "Spring Cloud Config로 분산된 마이크로 서비스 설정 중앙 관리"
subtitle: "Spring Cloud Config를 활용하여 여러 마이크로 서비스의 설정 관리한 사례르 소개합니다."
categories: devlog
tags: msa spring cloud config
---

> Spring Cloud Config를 활용하여 여러 마이크로 서비스의 설정 관리한 사례르 소개합니다.

<!--more-->

- 목차

  


## 🌱 여는 글

> Spring Cloud Config는 분산 시스템에서 외부 설정을 지원하는 서버 측과 클라이언트 측 기능을 제공합니다. <br />
> Config Server를 통해 모든 환경에 걸쳐 애플리케이션의 외부 속성을 중앙에서 관리할 수 있는 중앙 집중화된 장소를 제공받습니다. <br />
> 📚 [공식문서](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/)

공식 문서에 따르면, Spring Cloud Config는 분산된 서비스의 환경 설정을 중앙에서 관리해주는 솔루션 도구입니다.

개인적으로는 잘 사용하면 정말 효율적인 도구라고 생각하지만, 사전의 구성 계획 없이는 오히려 스파게티 관리가 될 수 있겠다는 생각이 들었습니다.

Stalk 프로젝트를 진행하면서 필요한 환경 구성은 다음과 같습니다.

- test
- dev (local)
- prod


작업 환경의 복잡성이 증가한다는 것은 곧 잘못된 환경 구성으로 인해 프로그램이 예상치 못한 동작을 하거나 다운될 수 있음을 의미합니다.

```
src/main/resources
├── application-dev.yml
├── application-test.yml
├── application.yml
└── bootstrap.yml
```

<br />

프로젝트 환경 구성을 설정하는 도중 중복되는 환경 구성을 하나로 통합하고 나머지는 분리하는 작업이 생각보다 복잡하다는 것을 느끼고 최대한 단순화시키려 했습니다. 때문에 local에서는 application.yml에 기본 환경 구성 설정만 두고 
test, dev(local) 설정 파일을 별도로 두어 관리하여 여러 이점들을 얻었습니다.


- prod만 spring cloud config에 두어 최대한 환경 설정의 얽힘이 없도록 합니다.
- 서비스를 중단(다운)시키지 않고 변경된 설정 파일을 적용시킬 수 있습니다.
- 설정파일 변경으로 일어난 장애 상황을 보다 쉽게 파악할 수 있습니다.
- 다중화된 서비스 서버에 변경된 설정파일을 반영하는 작업을 자동화할 수 있습니다.


특히, 무중단으로 설정 파일을 적용하고, 다중화된 서버에 변경된 설정을 자동으로 반영하는 것이 Spring Cloud Config의 큰 장점이라고 판단해 해당 솔루션 도구를 사용하였습니다.

---
<br />

## 🌱 Stalk 프로젝트의 Spring Cloud Config

### 🥕 Spring Cloud Config 운영 계획

   - dev(local), test 설정파일은 각 서비스 디렉토리에서 관리
   - prod 설정파일을 Spring Cloud Config에서 관리

   - 여러 마이크로 서비스에서 공통으로 사용될 수 있는 설정 값(ex. jwt.token) 같은 경우는 application-prod.yml에 선언
   - 나머지 개별 설정 파일은 `servicename-prod.yml` 형식으로 관리

<img src="https://i.ibb.co/ypg9cqP/7344814-D-AB32-4-E1-A-800-A-1-B328394-FBC8.jpg" alt="7344814-D-AB32-4-E1-A-800-A-1-B328394-FBC8" border="0">

> Stalk 프로젝트에서는 Spring Cloud Config를 통해 개발과 테스트 환경의 설정을 각각 로컬에서 관리하고, 운영 환경에서는 중앙 집중화된 Config Server를 이용해 무중단으로 설정을 업데이트하고 있습니다.<br />
> 
> 이를 통해 설정 관리의 효율성을 극대화하고, 잠재적인 장애 상황을 예방할 수 있습니다.

---
<br />

### 🥕 서비스 환경 설정 .yml 파일 관리

- private git repository에 저장하고 ssh 통신으로 로드 및 업데이트

> Git 관련 값들에 대한 암/복호화를 위해 Jasypt 라이브러리를 활용하였습니다.

private git repository로부터 설정 값을 받기 위해서는 다음과 같은 설정 값이 필요합니다.

```yaml
spring:
  cloud:
    config:
      server:
        git:
          default-label: main
          uri: git@github.com:syeon2/stalk-config-files.git
          ignore-local-ssh-settings: true
          strict-host-key-checking: false
          host-key: AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBEmKSENjQEezOmxkZMy7opKgwFB9nkt5YRrYMjNuG5N87uRgg6CLrbo5wAdT/y6v0mKV0U2w0WZ2YB/++Tpockg=
          host-key-algorithm: ecdsa-sha2-nistp256
          private-key: |
            ----BEGIN EC PRIVATE KEY-----
            .....
            -----END EC PRIVATE KEY-----
          passphrase: secretkey

```

private-key와 passphrase 값은 deploy keys를 등록한 private git repository에 접근할 때 필요한 값이기에 꼭 필요합니다.

보안 이슈로 해당 값들을 암호화해야했습니다. 하지만 private-key 값의 길이와 키 값 특성상 .env 파일을 제외한 intellij 시스템 변수, JVM 환경 변수, 운영체제 시스템 변수 등에는 등록하는 것이 
제한적이었습니다.

이를 해결하기 위해 Jasypt 라이브러리를 활용하여 private-key와 passphrase와 같은 중요한 값들을 평문 암호로 변환하여 암호화하였습니다.

```yaml
encrypt:
  key: ${JASYPT_PASSWORD}
```

bootstap.yml 에 해당 설정 값이 존재한다는 것을 명시해두고 Jasypt를 사용하기 위한 코드를 작성해야합니다.

```kotlin
@Configuration
class JasyptConfig {
    companion object {
        val PASSWORD = System.getenv("JASYPT_PASSWORD") ?: "none"
        const val ALGORITHM = "PBEWithMD5AndDES"
    }

    @Bean("jasyptStringEncryptor")
    fun stringEncryptor(): StringEncryptor {
        val encryptor = PooledPBEStringEncryptor()
        val config = SimpleStringPBEConfig()

        config.password = PASSWORD
        config.algorithm = ALGORITHM
        config.setPoolSize("1")
        config.stringOutputType = "base64"
        config.setKeyObtentionIterations("1000")
        config.setSaltGeneratorClassName("org.jasypt.salt.RandomSaltGenerator")
        config.setIvGeneratorClassName("org.jasypt.iv.NoIvGenerator")

        encryptor.setConfig(config)

        return encryptor
    }
}
```

그리고 위의 값 중에 암호화하고자 하는 값을 변환하는 [사이트](https://www.devglan.com/online-tools/jasypt-online-encryption-decryption)를 이용하였습니다. 
해당 사이트에서 값을 암호화하고 yaml 파일에 `ENC()`라는 값을 넣습니다.

```yaml
spring:
  cloud:
    config:
      server:
        git:
          default-label: main
          uri: git@github.com:syeon2/stalk-config-files.git
          ignore-local-ssh-settings: true
          strict-host-key-checking: false
          host-key: AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBEmKSENjQEezOmxkZMy7opKgwFB9nkt5YRrYMjNuG5N87uRgg6CLrbo5wAdT/y6v0mKV0U2w0WZ2YB/++Tpockg=
          host-key-algorithm: ecdsa-sha2-nistp256
          private-key: ENC(********)
          passphrase: ENC(********)
```

도중 복호화가 되지 않은 현상이 발생하였고 [git issue](https://github.com/ulisesbocchio/jasypt-spring-boot/issues/218)에 같은 문제를 해결한 사례를 참고하여 해결하였습니다.

> jasypt를 통해 어플리케이션 안에서 암/복호화를 진행할 수 있게 되었습니다. 
> 
> 장점 : JASYPT_PASSWORD로 입력한 값만 관리할 수 있어 관리 비용이 줄어듭니다. <br />
> 단점 : JASPYT_PASSWORD 하나만 탈취당하면 해당 시스템의 값들의 암호화 값이 노출됩니다.

Stalk Project에서는 pipeline으로 사용하는 Jenkins에 환경변수로 등록하여 Docker 이미지를 빌드할 때 --build-arg, ARG, ENV로 
java 실행 Dockerfile에 주입합니다. (예시_[Stalk Config Jenkinsfile](https://github.com/syeon2/stalk-config/blob/main/devops/Jenkinsfile), [Stalk Config Dockerfile](https://github.com/syeon2/stalk-config/blob/main/devops/Dockerfile))

<br />
<br />

###### 📚 참고자료
[Jasypt를 이용한 Yaml 또는 Properties 프로퍼티 설정 파일의 리소스 암호화](https://mangkyu.tistory.com/284) <br />

---

<br />

### 🥕 마이크로 서비스 운영 환경 업데이트 방법

- Spring Cloud Bus AMQP & RabbitMQ를 활용

> 자동 갱신이 아닌 수동 갱신의 방식을 선택하여 프로젝트에 적용하였습니다.

자동 갱신은 설정 파일이 업데이트 되면 자동으로 모든 서버에 변경된 사항이 적용되게 하는 방식입니다. Watcher나 Github Webhook 등을 사용한 방법으로 자동 갱신을 
구현할 수 있지만, 여기서는 수동 갱신 방법을 선택하였습니다.

- 변경 통제 및 검증 & 예기치 않은 중단 방지

자동화된 설정 변경은 의도하지 않은 변경사항이 실수로 배포될 위험을 증가시킬 수 있습니다. 수동 갱신은 휴먼 에러 등의 사항을 신중하게 관리할 수 있습니다.

- 특정 시점에 배포 가능

업데이트 후 바로 반영하는 것이 아닌 여러 마이크로 서비스 간의 협의 하에 함께 반영해야하는 상황에 필요한 이유입니다.

등 여러가지 이유들 중 위의 2가지를 고려하여 수동 갱신 방식을 선택하였습니다.

<br />

수동 방식 중 Spring Cloud Bus는 Method: Post, */acturator/busrefresh 엔드포인트를 호출하여 수동적이지만, 업데이트된 설정 파일을 모두 적용하는 도구입니다.

Kafka를 Spring Cloud Bus의 도구로 사용하는 것은 비용과 용량 측면에서 과도할 수 있다는 판단하에 rabbitmq를 택하였습니다.

[//]: # (RabbitMQ 동작 방식 (BroadCast)

<br />
<br />

###### 📚 참고자료
[Spring Cloud Bus 공식문서](https://docs.spring.io/spring-cloud-bus/reference/spring-cloud-bus/configuration.html)<br />
[[RabbitMQ] 기초 개념](https://velog.io/@sdb016/RabbitMQ-%EA%B8%B0%EC%B4%88-%EA%B0%9C%EB%85%90#rabbitmq%EB%9E%80)<br />
[Spring AMQP 맛보기 - 1. AMQP 개념](https://preamtree.tistory.com/172)

---

## 🌱 맺으며

Spring Cloud Config 도구를 활용하여 다양한 이점을 얻을 수 있었습니다. <br />
그러나 좋은 기술이라도 무턱대고 사용하면 오히려 제품에 독이 될 수 있다는 것을 이 프로젝트를 통해 깨달았습니다.

서비스를 구축할 때 사전에 어떻게 구현할 것인지에 대한 계획을 철저히 세우는 것이 더욱 탄탄한 서비스를 만들 수 있는 초석이라고 생각됩니다.