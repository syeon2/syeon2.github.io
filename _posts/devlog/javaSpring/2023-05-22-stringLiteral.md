---
layout: post
title: "String literal과 new String (feat. VisualVM)"
subtitle: "String literal과 new String를 비교해보자"
categories: devlog
tags: java_spring JVM Memory
---

> 모니터링 프로그램을 사용해보고 싶어 비교해보는 String Literal과 new String의 비교

<!---more--->

## 📚 왜 굳이 String literal과 new String(생성자)을 비교해볼까?
Spring의 JDBC 환경에서 개발하던 중 Repository 컴포넌트에서 DB와 커넥션을 위해서 JdbcTemplate을 사용해야 했고, 쿼리문을 실행하기 위해서는 SQL문을
직접 작성해서 JdbcTemplate에 주입해주어야 했다.
그렇다보니 각 Repository의 메서드마다 `String sql = "insert into TABLE(attribute) values (?...);"` 식으로 실행할 쿼리문을 넣어주어야 했다.
<br />
String 하나쯤이야~ 라고 생각할 수 있겠지만 Spring 서버 어플리케이션은 Servlet을 통해 각 클라이언트의 요청마다 스레드가 할당되어 요청을 수행한다. 만일 
String을 생성할 때마다 객체로 저장되어(Java에서 String은 객체이다.) Heap 메모리에 저장되면 많은 클라이언트들이 요청을 수행했을 때 OOME(OutOfMemroyException)이 
발생하지 않을까? 생각했다.
<br />
당연히 Java에서는 String만의 메모리 관리법이 있기 때문에 지금 우리가 Java 어플리케이션을 잘 사용하고 있겠지만, 모니터링 프로그램(VisualVM, Profiler) 등도 익숙해질 겸
String Literal일 때와 new String일 때 어디에 데이터가 저장되는지 알아보고자 한다.

---

## 📚 예상 시나리오
본래 String literal로 생성하는 데이터는 PermGen 영역에 저장되었다. PermGen은 Heap 영역에 속하지는 않지만 메모리 MaxSize가 고정되어 있어 String 데이터가 많아지면 OOME가 종종? 발생하였다...고 한다.
Java 7 이후부터 PermGen에 저장되었던 String literal 데이터는 Heap 영역에 저장되었고, Heap의 특성인 GC가 가능하도록 변경되었다. <a href="https://www.baeldung.com/java-string-pool">관련 링크 [Baeldung]</a>
<br />
(여기서 조심해야할 부분은 String literal을 보관하는 String Constant Pool은 Java7에 Heap 영역으로 옮겨졌고, PermGen이 Metaspace로 변경된 것은 Java8에 이루어졌다. 햇갈리지 않기!)
<br />
<br />
위의 정보들을 토대로 예상되는 결론은
- 하나의 String literal 데이터를 변수에 할당하는 반복문을 N번 호출하게 된다면 데이터는 String literal 데이터 하나를 저장하는 미세한 변동만 보여질 것이다.
- 각각 다른 String literal 데이터 100개를 N번 호출하면 100개를 Heap에 저장하는 것 이외의 변동은 없을 것이다.
- 같은 String 문자열을 new String으로 생성시에는 Heap에 차지하게 되는 객체의 수가 생성된 객체만큼 늘어날 것이다.

---

## 📚 모니터링 결과 (VisualVM)

- 사용한 환경 : Java11, Spring Web(API 호출을 위해)
- Tool : VisualVM, PostMan(String literal 생성하는 API N번 호출)
- 공통으로 적용된 값
  - 반복문으로 변수에 100번씩 할당
  - Postman으로 해당 로직을 총 N * M(10000000)이 되도록 호출 (Servlet의 영향으로 각 요청마다 스레드가 생겨나는 이슈 생각하기)

### 🌱 같은 String literal을 생성하는 로직을 100 * N 번 호출한 결과
```
public void literal() {
    for (int i = 0; i < 100; i++) {
        String sql = "insert into table(attribute) values (?..);";
    }
}
```
<img src="https://wkblog-images.s3.ap-northeast-2.amazonaws.com/devlog/javaSpring/2023_05_22_stringLiteral1.png" width="600" />

CPU가 사용된 주황색 라인 지점에서 메서드가 실행되었다. 동일한 지점에서 Heap 메모리는 아주 미세하게 값이 증가하였는데, 이는 Servlet의 Thread 생성 비용으로 추정됨 (확인 필요)

### 🌱 서로 다른 100개의 String literal을 생성하는 로직을 N번 호출한 결과
```
public void literal2() {
    for (int i = 0; i < 100; i++) {
        String sql = "insert into table(attribute) values (?..);" + " " + i;
    }
}
```
<img src="https://wkblog-images.s3.ap-northeast-2.amazonaws.com/devlog/javaSpring/2023_05_22_stringLiteral100.png" width="600" />

하나의 String literal만 생성하는 것보다 약간 증가폭이 늘었지만 여전히 비슷한 결과를 보여주고 있다.

### 🌱 new String()(생성자)를 사용한 로직을 N번 호출한 결과
```
public void newString() {
    for (int i = 0; i < 100; i++) {
        String sql = new String("insert into table(attribute) values (?..);");
    }
}
```
<img src="https://wkblog-images.s3.ap-northeast-2.amazonaws.com/devlog/javaSpring/2023_05_22_newString.png" width="600" />

String literal로 생성했던 결과들과 확연하게 차이가 보인다. 즉 생성자로 생성하는 String 값 모두 새롭게 생성되고 일정 메모리 이상 올라가면 GC가 발생하는 것을 확인할 수 있다.

### 🌱 서로 다른 10000개의 String literal 생성하는 로직을 N번 호출한 결과
```
public void literal3() {
    for (int i = 0; i < 10000; i++) {
        String sql = "insert into table(attribute) values (?..);" + " " + i;
    }
}
```
<img src="https://wkblog-images.s3.ap-northeast-2.amazonaws.com/devlog/javaSpring/2023_05_22_stringLiteral10000.png" width="600" />

Java7 이후부터 String literal이 Heap영역으로 옮겨졌기 때문에 위와같이 String literal도 GC가 발생한다.

---

### 📚 마무리
String 데이터가 많으면 많아질 수록 literal를 사용하는 것이 매우 효율적이라는 것을 알 수 있다. 하나의 문자열을 String Constant Pool에 저장하고 GC가 일어나기 전까지 계속 캐싱된 문자열을 
사용할 수 있기 때문에 메모리 효율을 높일 수 있다. 생성자를 통해 생성하는 String은 생성할 때마다 새로운 Heap 메모리를 차지하기 때문에 메모리 효율도 않좋을 뿐더러 GC가 더욱 자주 발생된다. 이는 GC를 동작시키는 
컴퓨팅 리소스를 불필요하게 소비하기 때문에 literal을 통한 문자열 생성을 적극 권장한다.
<br />
<br />
이와는 별개로 Spring의 Repository에 SQL문을 작성하는 것은 다른 접근 방법이 필요할 것 같다. 가령 클라이언트로(스레드)로부터 동시에 메소드가 호출된다면 SQL문을 literal로 생성하는 것이 이중으로 발생할 수 있어서 이 또한 
효율이 좋지 않을 수 있을 것 같다.
<br />
이러한 이유로 SQL문을 관리하는 enum을 만들어 SQL문을 상수화하고(enum은 상수로써 런타임 시 바로 Method Area에 초기화되는 특징을 가지고 있다.) enum 클래스로부터 SQL문을 주입받는 것도 괜찮은 선택인 것 같다.