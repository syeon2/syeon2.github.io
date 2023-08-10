---
layout: post
title: "ClassLoader의 다양한 활용 (feat. jjwt library)"
subtitle: "ClassLoader 활용 방법에 대해 스터디하는 글입니다."
categories: devlog
tags: java_spring JVM classLodaer
---

> JVM의 클래스로더를 알아보고 코드에서 어떻게 사용되는지 생각해보기 위해 Deep Dive합니다.


<!--more-->

### 🟤 ClassLoader를 왜 스터디해야할까?
JVM을 스터디하면서 Runtime Data Area, Execution Engine은 스터디하는 목적에 대해 분명했다. Java의 메모리 관리에 대해 스터디하고 싶다면 Runtime Data Area는 
꼭 스터디해야할 영역이다. Execution Engine은 Java 코드가 컴파일되어 있는 바이트코드를 해석하는 인터프리터, JIT 컴파일러 등이 있지만, 역시나 Java 개발자들 사이에서 
중요하게 생각하는 Garbage Collection이 Execution Engine 영역 안에 있기 때문에 스터디해야하는 영역이다. 하지만 클래스로더는 제일 처음 Java 바이트코드가 실행될 때 
JVM 안에서 진입하게 되는 영역이지만 어떻게 활용되는지는 감이 안왔다. 그저 코드가 실행되는 것만을 지켜볼 뿐..

원티드 사전 과제를 진행하면서 JWT의 라이브러리가 3개로 나뉘어져있고 인터페이스, 구현체 등이 각각의 개별적인 라이브러리에 코드로 구현되어 있었다. 내부를 확인해보니 ClassLoader를 사용하는 듯한 
코드를 발견했는데, 이것을 계기로 클래스로더가 어떤 역할을 하는지 확인하고 싶었다.

---------

## 🌱 클래스로더를 사용한 다양한 기술

### 1. Singleton
클래스로더를 설명하다가 난데없이 싱글톤라는 주제가 등장하게 되었다. 싱글톤이 클래스로더와 관련이 있을까? 사실 아주 베이직하게 구현되고 있는 싱글톤 방식은 클래스로더의 개념을 사용하지는 않는다.

```java
public class Singleton {

    private static final Singleton INSTANCE = new Singleton();
    
    private Singleton() { }
    
    public static Singleton getInstance() {
        return this.INSTANCE;
    }
} 
```

위의 코드는 아주 기본적으로 사용되고 있는 싱글톤 방식이다. 해당 싱글톤은 `Early Initialization Singleton`이라는 특징을 가지고 있다. 한마디로 Java 파일이 실행되면 static 변수들은 
Java 파일이 실행됨과 함께 초기화가 진행되는데 이 때 early initialization 방식의 싱글톤도 Java 파일이 실행될 때 함께 초기화된다는 것이다.

싱글톤 클래스 개수가 적다면 Java 파일을 실행하는데 오래걸리지는 않겠지만, 만약 초기화해야하는 싱글톤 클래스가 많다면? 그럼 당연히 Java 파일을 실행하는데 필요한 시간이 길어질 것이다. (모든 싱글톤 클래스 파일을 로드하여 
초기화를 진행해야하기 때문에..!) 그래서 싱글톤 클래스에 대한 여러가지 방안들이 등장하기 시작한다.

```java
public class Singleton {
	
    private volatile static Singleton INSTANCE;
    
    private Singleton() { }
    
    public static Singleton getInstance() {
        if (INSTANCE == null) {
            synchronized (Singleton.class) {
                if (INSTANCE == null) {
                    INSTANCE = new Singleton();
                }
            }
        }
        
        return INSTANCE;
    }
}
```

해당 코드는 double checked locking 싱글톤 방식이다. 위의 코드는 Java 파일이 실행될 때 static에는 아직 초기화할 값이 있지 않기 때문에 INSTANCE가 런타임시 불릴 때 
클래스가 로드되어 인스턴스가 생성된다는 장점을 가지고 있다. 하지만 이 코드는 자칫 에러를 발생할 수 있는 여지를 가지고 있는 코드이다.

Java에서는 JVM, 컴파일러, 프로세서가 명령어를 재정렬할 수 있다. (Reordering) JVM의 특징 중 하나는 명령의 의미가 동일하게 유지된다면 성능상의 이유로 명령을 재정렬할 수 있다. 
그것을 방지해주는 것이 volatile 예약어이긴 하다. 그렇지만 volatile은 Java 5 버전 이하에서는 지원하지 않기 때문에 안티패턴으로 불린다.

volatile이 없다면 해당 코드는 Singleton 객체의 생성이 완료되기 전에 INSTANCE 참조가 업데이트될 수 있다. 이 때 다른 스레드가 INSTANCE를 읽게 되면 
아직 완전히 생성하지 않은 객체에 접근하게 되는 원리이다.

```java
public class Singleton {
	
	private Singleton() { }
    
    public static Singleton getInstance() {
		return LazyHolder.INSTANCE;
    }
	
	private static class LazyHolder {
		private static final Singleton INSTANCE = new Singleton();
    }
}
```

현재 가장 많이 사용되는 Singleton 방식이다. 이 싱글톤 방식이 바로 클래스로더의 특징을 사용해서 구현된 싱글톤 기법이다. 클래스는 해당 클래스가 호출되어 인스턴스를 만드는 코드가 
불리우기 전까지 로드되지 않는다. Java는 클래스를 동적 로드 방식으로 로드하기 때문이다.

해당 싱글톤은 Singleton 클래스로부터 getInstance 메소드가 호출될 때 LazyHolder 클래스가 로드됨과 함께 바로 초기화가 되는 방식이다. 따라서 JVM의 ClassLoader의 특징을 
이용하여 구현한 Lazy Initialization 싱글톤 기법이라고 할 수 있다.

### 2. jjwt library
이번 원티드 사전과제를 진행하면서 알게된 JWT의 jjwt 라이브러리이다. jjwt 라이브러리는 jjwt-api, jjwt-impl, jjwt-jackson 총 3개의 
라이브러리로 나뉘어져 있다. 각각 인터페이스, 구현체로 나뉘어져 있는 이 라이브러리는 얼핏 봤을 때는 굳이 나누어서 배포해야했을까? 라는 생각이 든다. 

jjwt github에 보면 jjwt-api는 implementation으로 라이브러리를 로드하고 나머지 jjwt-impl, jjwt-jackson은 runtime시에만 모듈을 로드하는 것을 볼 수 있다. 

jjwt-api에 선언되어 있는 API를 사용할 때면 jjwt-impl의 파일에 직접 접근하여 해당 클래스 파일을 로드하는 것을 볼 수 있다.

```java
    public static JwtParserBuilder parserBuilder() {
	    return Classes.newInstance("io.jsonwebtoken.impl.DefaultJwtParserBuilder");
	}
	--------
    public static <T> T newInstance(String fqcn) {
        return (T)newInstance(forName(fqcn));
    }
        --------
    public static <T> Class<T> forName(String fqcn) throws UnknownClassException {
    
        Class clazz = THREAD_CL_ACCESSOR.loadClass(fqcn);
    
        if (clazz == null) {
            clazz = CLASS_CL_ACCESSOR.loadClass(fqcn);
        }
    
        .....
    
        return clazz;
    }
        --------
    public Class loadClass(String fqcn) {
        Class clazz = null;
        ClassLoader cl = getClassLoader();
    
        if (cl != null) {
            try {
                clazz = cl.loadClass(fqcn);
            } catch (ClassNotFoundException e) {
                //Class couldn't be found by loader
            }
        }
		
        return clazz;
    }
```

해당 로직은 jjwt-api 라이브러리에 있는 API를 동작하게 하는 코드이다. 위에서부터 보면 jjwt-impl 라이브러리에 있는 구현체를 
실행시키기 위해 구현체가 있는 패키지 명부터 차례대로 호출되어 결국 loadClass 메소드에서 클래스로더에 의해 필요한 구현체 클래스가 로드되는 것을 
볼 수 있다.

이를 통해 jjwt 라이브러리가 가질 수 있는 이점을 생각해보았다.

1. 의존성을 분리한다.
- 각 모듈이 필요한 의존성만 포함하여 불필요한 의존성을 줄이고 라이브리러의 크기를 경량화시킬 수 있다.

2. 확장성
- 인터페이스와 구현을 분리함으로써, 다양한 환경과 요구 사항에 대응할 수 있다. (버전 호환이 가능하다.)

jjwt 라이브러리는 세 가지 주요 구성 요소로 나뉘며, 이는 의존성 관리, 확장성 향상, 경량화 등의 이점을 가져다 준다. 
이러한 설계 방향은 개[Gemfile.lock](..%2F..%2F..%2FGemfile.lock)발자가 자신의 프로젝트에 필요한 부분만 선택하여 사용할 수 있게 해준다. 
이런 유연성은 모던 라이브러리 설계에서 중요한 요소로 작용하며, jjwt가 많은 개발자들에게 선택받는 이유 중 하나라고 생각된다.

--------

## 🌱 정리
클래스로더에 대한 정의와 기술 자체에 대한 설명은 좋은 글들이 많다. ([Naver D2 - JVM Internal](https://d2.naver.com/helloworld/1230)) 
이번 글을 통해서는 클래스로더가 어떻게 쓰이는지에 대해 알아보고 활용하는 사례들을 스터디해보고 싶었다.

단순히 코드를 작성하는 것이 아닌 기술적인 특징을 파악하고 그것을 응용하여 설계하는 것 또한 좋은 엔지니어에게 필요한 
역량이라는 생각이 드는 스터디였다.


##### Reference
[Naver D2 - JVM Internal](https://d2.naver.com/helloworld/1230) <br />
[Double-Checked Locking with Singleton](https://www.baeldung.com/java-singleton-double-checked-locking)
