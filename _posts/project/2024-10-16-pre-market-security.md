---
layout: post
title: "[Spring] Spring Security 동작 프로세스"
subtitle: "Spring Security가 동작하는 프로세스를 설명합니다."
categories: devlog
tags: project security
---

> Spring Security가 동작하는 프로세스를 설명합니다.

<!--more-->

#### 목차

- [여는 글](#-여는-글)
- [Spring Security란?](#-spring-security란)
  - [Spring Security의 동작원리](#-spring-security의-동작-원리)
  - [Spring Security가 동작하는 과정](#-spring-security가-동작하는-과정)
    - [1. Servlet 및 Spring Beans 초기화](#1-servlet-및-spring-beans-초기화)
    - [1.1 SecurityBuilder](#11-securitybuilder)
    - [1.2 SecurityConfigurer](#12-securityconfigurer)
    - [2. Security Beans 초기화](#2-security-beans-초기화)
    - [2.1 HttpSecurityConfiguration](#21-httpsecurityconfiguration)
    - [2.2 SpringBootWebSecurityConfiguration](#22-springbootwebsecurityconfiguration)
    - [2.3 WebSecurityConfiguration](#23-websecurityconfiguration)
    - [3. DelegatingFilterProxy에서 Http Request Intercept -> SpringSecurityFilterChain에 요청 처리 위임](#3-delegatingfilterproxy에서-http-request-intercept---springsecurityfilterchain에-요청-처리-위임)
- [닫는 글](#-닫는-글)

---

## 🌱 여는 글

많은 사용자를 보유한 서비스에서 보안은 필수적인 요소입니다. 안전한 서비스를 제공하려면 체계적이고 견고한 보안 체계를 갖추는 것이 필수적입니다. 
하지만 모든 보안 기능을 직접 구현하는 것은 복잡도를 증가시키고, 시스템의 안정성을 저해할 수 있는 위험이 존재합니다.

이러한 이유로, Pre Market 프로젝트에서는 Spring Security를 활용하여 인증과 인가를 구현하고, CSRF 보호, 세션 관리, 암호화, 그리고 역할 기반 접근 제어 등의 보안 기능을 효율적으로 적용했습니다. 
Spring Security는 검증된 보안 프레임워크로, 이를 통해 안전하고 효율적인 인증 인가 시스템을 구축함으로써 서비스의 신뢰성을 강화하고 유지보수성을 향상시켰습니다.

---

## 🌱 Spring Security란?

> Spring Security is a framework that provides authentication, authorization, and protection against common attacks. - [Spring Security](https://docs.spring.io/spring-security/reference/index.html)

Spring Security는 인증과 인가를 효율적으로 처리하고, CSRF 공격 방지, 세션 고정 공격 방지, 암호화와 같은 다양한 보안 기능을 제공하여 애플리케이션을 보호하는 프레임워크입니다.

Pre Market 프로젝트에서는 회원 가입 시 비밀번호 암호화, 로그인, 로그아웃, 사용자 인가 처리와 같은, 사용자를 보유한 서비스에 필수적인 기능들을 지원합니다.
이러한 보안 기능을 모두 직접 구현하려면 시간과 리소스가 많이 소모될 뿐 아니라, 본격적인 도메인 개발에 착수하기도 전에 큰 어려움에 직면할 수 있습니다.

Spring Security와 같은 프레임워크를 활용하면, 이러한 보안 기능을 간단하고 안정적으로 구현할 수 있어, 더 효율적이고 신뢰성 있는 서비스를 구축할 수 있습니다.


### 🥕 Spring Security의 동작 원리

Spring Security는 <strong>필터 기반</strong>으로 동작하는 보안 프레임워크입니다. Spring Security는 기본적으로 서버에 접근하는 모든 요청을 사전에 Intercept하여 
여러 필터들을 검증한 후 요청이 유효하다고 판단하면 다음 필터나 최종 Servlet(ex. DispatcherServlet)으로 전달합니다.

<img src="https://i.ibb.co/5KYKQMD/security2.jpg" alt="security2" border="0">

Spring Security는 기본적으로 Servlet 컨테이너 안의 Filter로 동작하기 때문에 Spring의 DI 특징을 다른 방법으로 지원받습니다. 바로 DelegatingFilterProxy입니다. 
서버가 실행되면 DelegatingFilterProxyRegistrationBean 클래스는 Spring의 ApplicationContext를 통해 SpringSecurityFilterChain이라는 Bean을 참조하도록 설정됩니다.

... 

##### 💡 Spring Security의 Filter들이 초기화되는 시점에는 이미 Servlet Container와 Spring DI Container는 초기화가 완료된 상태입니다.

```java
public ConfigurableApplicationContext run(String... args) {
	//..........
    try {
		//...........
        context = this.createApplicationContext();
        context.setApplicationStartup(this.applicationStartup);
        this.prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
        this.refreshContext(context);
        //......
    } catch (Throwable e) {
      // ...
    }
}
```

SpringApplication.run() 메서드는 초기에 먼저 `context = this.createApplicationContext()과 this.refreshContext()`를 통해 Servlet들을 초기화시킵니다. 이 때, Servlet은 DispatcherServlet을 포함한 Spring Container 또한 
함께 초기화시킵니다.

###### 🔗 [Servlet의 동작 원리]()

...

### 🥕 Spring Security가 동작하는 과정

#### 1. Servlet 및 Spring Beans 초기화

Spring 서버가 실행될 때, **Servlet이 초기화되는 과정**에서 **Spring MVC**의 `DispatcherServlet`이 서블릿으로 등록되고 초기화됩니다. 이와 동시에, **Spring ApplicationContext**가 초기화되면서 **Spring Security** 프레임워크에서 정의된 보안 관련 Bean들이 **ApplicationContext**에 등록되고 초기화됩니다.
따라서 Spring Security의 Bean들은 **ApplicationContext**가 초기화될 때 함께 초기화되며, 이후 `DelegatingFilterProxy`를 통해 Servlet 환경과 연동되어 필터로 작동합니다.

Spring Security를 동작시키는 핵심 클래스들은 다음과 같습니다.

...

#### 1.1 SecurityBuilder

SecurityBuilder는 Spring Security에서 각종 웹보안을 구성하는 빈 객체와 설정 클래스들을 생성 및 관리하는 역할을 합니다. 구현체로는 WebSecurity, HttpSecurity가 있습니다.

```java
public interface SecurityBuilder<O> {
    O build() throws Exception;
}
```

build 메서드를 호출하면 HttpSecurity 안에서 구성되는 `SecurityConfigurer` 등의 필터들이 SecurityFilterChain Bean으로 생성됩니다.

대표적인 구현체로는 WebSecurity, HttpSecurity가 있습니다.

#### 1.2 SecurityConfigurer
 
SecurityConfigurer는 Http 요청과 관련된 보안 처리를 담당하는 구성 요소들(ex. 필터, 인증 및 인가)을 생성하고 설정하는 역할을 합니다. SecurityBuilder가 이를 참조하고 있으며, 
SecurityBuilder의 구현체에서 메서드 체이닝과 같은 방법을 통해 다양한 SecurityConfigurer을 적용하여 보안 설정을 구성할 수 있습니다.

```java
public interface SecurityConfigurer<O, B extends SecurityBuilder<O>> {
    void init(B builder) throws Exception;
    
    void configure(B builder) throws Exception;
}
```

init 메서드는 SecurityConfigurer가 초기화 단계에서 SecurityBuilder에 필요한 기본 설정이나 필수 컴포넌트를 등록할 수 있도록 하는 메서드입니다. 이 메서드는 보안 설정의 초기 상태를 구성하고, SecurityBuilder가 이후 보안 구성을 적용할 때 사용할 수 있는 기반을 마련합니다.

configure 메서드는 init 메서드에서 초기화된 설정을 기반으로 SecurityBuilder에 구체적인 보안 설정을 적용하는 메서드입니다. 이 메서드에서는 필터, 인증 매니저, 인가 규칙 등을 구체적으로 구성하여 SecurityBuilder의 구현체(ex. HttpSecurity)에 적용합니다.

대표적인 구현체로는 CsrfConfigurer, FormLoginConfigurer 등이 있습니다.

##### 💡 SecurityBuilder와 SecurityConfigurer의 관계
<img src="https://i.ibb.co/qn63fy7/security3.jpg" alt="security3" border="0">

---

#### 2. Security Beans 초기화

#### 2.1 HttpSecurityConfiguration

Security Beans가 초기화될 때, `HttpSecurityConfiguration`클래스에서 `HttpSecurity` 인스턴스가 생성되고 **ApplicationContext**에 초기화됩니다. 

```java
@Bean({"org.springframework.security.config.annotation.web.configuration.HttpSecurityConfiguration.httpSecurity"})
@Scope("prototype")
HttpSecurity httpSecurity() throws Exception {
    //....
    HttpSecurity http = new HttpSecurity(this.objectPostProcessor, authenticationBuilder, this.createSharedObjects());
    http.csrf(Customizer.withDefaults()).addFilter(webAsyncManagerIntegrationFilter).exceptionHandling(Customizer.withDefaults()).headers(Customizer.withDefaults()).sessionManagement(Customizer.withDefaults()).securityContext(Customizer.withDefaults()).requestCache(Customizer.withDefaults()).anonymous(Customizer.withDefaults()).servletApi(Customizer.withDefaults()).apply(new DefaultLoginPageConfigurer());
    //.....
    return http;
}

//-------------------

public HttpSecurity csrf(Customizer<CsrfConfigurer<HttpSecurity>> csrfCustomizer) throws Exception {
	ApplicationContext context = this.getContext();
	csrfCustomizer.customize((CsrfConfigurer)this.getOrApply(new CsrfConfigurer(context)));
	return this;
}

```

위의 과정을 통해 **HttpSecurity**가 SecurityBuilder의 구현체로서 초기화되고, SecurityConfigurer의 구현체인 여러 필터들을 구성합니다. 
하지만 **HttpSecurity**는 Bean으로 등록되지 않으며, 보안 구성을 빌드하는 데 사용되는 객체입니다. 최종적으로 필터 체인이 구성된 후 springSecurityFilterChain이 필터로 등록됩니다.

#### 2.2 SpringBootWebSecurityConfiguration

해당 클래스는 2.1에서 Bean으로 등록된 HttpSecurity Bean을 필요로 합니다.

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnDefaultWebSecurity
static class SecurityFilterChainConfiguration {
    SecurityFilterChainConfiguration() {
    }

    @Bean
    @Order(2147483642)
    SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests((requests) -> {
            ((AuthorizeHttpRequestsConfigurer.AuthorizedUrl)requests.anyRequest()).authenticated();
        });
        http.formLogin(Customizer.withDefaults());
        http.httpBasic(Customizer.withDefaults());
        return (SecurityFilterChain)http.build();
    }
}
```

`defaultSecurityFilterChain` 메서드는 `@Bean`으로 등록된 **HttpSecurity** 인스턴스를 주입받아 보안 설정을 추가로 구성합니다. 
이 메서드는 주입받은 **HttpSecurity** 인스턴스에 여러 **SecurityConfigurer**를 적용하여 HTTP 요청에 대한 보안 규칙(예: 인증 요구, 폼 로그인, HTTP 기본 인증)을 설정합니다.

`http.build()` 메서드는 이 설정을 기반으로 **SecurityFilterChain** 인스턴스를 생성하며, 이 **SecurityFilterChain** 인스턴스가 ApplicationContext에 Bean으로 등록됩니다.

결과적으로, `defaultSecurityFilterChain` 메서드는 구성된 `SecurityFilterChain`을 반환하며, 이 필터 체인은 애플리케이션의 보안을 관리하는 핵심 필터 체인으로 사용됩니다.

#### 2.3 WebSecurityConfiguration

**WebSecurityConfiguration**는 **WebSecurity**를 생성하고 초기화합니다. WebSecurity는 2.2에서 만들어진 SecurityFilterChain을 참조하여 보안 구성을 관리하며, 
SecurityFilterChain을 SecurityBuilder의 구현체로서 저장하고, 이를 바탕으로 전체 보안 필터 체인을 구성합니다. 

WebSecurity 또한 SecurityBuilder의 구현체인데, `build()`를 실행하게 되면 이전 단계에서 생성된 하나 이상의 SecurityFilterChain 인스턴스들이 FilterChainProxy에 저장됩니다.

```java
protected Filter performBuild() throws Exception {
    //.......
    FilterChainProxy filterChainProxy = new FilterChainProxy(securityFilterChains);
    if (this.httpFirewall != null) {
        filterChainProxy.setFirewall(this.httpFirewall);
    }
    
    if (this.requestRejectedHandler != null) {
        filterChainProxy.setRequestRejectedHandler(this.requestRejectedHandler);
    } else if (!this.observationRegistry.isNoop()) {
        CompositeRequestRejectedHandler requestRejectedHandler = new CompositeRequestRejectedHandler(new RequestRejectedHandler[]{new ObservationMarkingRequestRejectedHandler(this.observationRegistry), new HttpStatusRequestRejectedHandler()});
        filterChainProxy.setRequestRejectedHandler(requestRejectedHandler);
    }
    
    filterChainProxy.setFilterChainDecorator(this.getFilterChainDecorator());
    filterChainProxy.afterPropertiesSet();
    Filter result = filterChainProxy;
    if (this.debugEnabled) {
        this.logger.warn(
            "\n\n********************************************************************\n**********        Security debugging is enabled.       *************\n**********    This may include sensitive information.  *************\n**********      Do not use in a production system!     *************\n********************************************************************\n\n");
        result = new DebugFilter(filterChainProxy);
    }
}
```

위의 코드를 통해 WebSecurity가 build 메서드를 실행하면, **FilterChainProxy**가 생성되고, 이전 단계에서 만들어진 하나 이상의 **SecurityFilterChain**이 FilterChainProxy에 등록됩니다. FilterChainProxy가 Filter로 반환되어 
Servlet Filter Chain에서 동작할 수 있도록 구성됩니다.

```java
@Bean(
    name = {"springSecurityFilterChain"}
)
public Filter springSecurityFilterChain() throws Exception {
    boolean hasFilterChain = !this.securityFilterChains.isEmpty(); // 1
    if (!hasFilterChain) {
        this.webSecurity.addSecurityFilterChainBuilder(() -> {
            this.httpSecurity.authorizeHttpRequests((authorize) -> {
                ((AuthorizeHttpRequestsConfigurer.AuthorizedUrl)authorize.anyRequest()).authenticated();
            });
            this.httpSecurity.formLogin(Customizer.withDefaults());
            this.httpSecurity.httpBasic(Customizer.withDefaults());
            return (SecurityFilterChain)this.httpSecurity.build();
        });
    }

    Iterator var2 = this.securityFilterChains.iterator(); // 2

    while(var2.hasNext()) {
        SecurityFilterChain securityFilterChain = (SecurityFilterChain)var2.next();
        this.webSecurity.addSecurityFilterChainBuilder(() -> {
            return securityFilterChain;
        });
    }

    var2 = this.webSecurityCustomizers.iterator();

    while(var2.hasNext()) {
        WebSecurityCustomizer customizer = (WebSecurityCustomizer)var2.next();
        customizer.customize(this.webSecurity);
    }

    return (Filter)this.webSecurity.build();
}
```

최종적으로 WebSecurityConfiguration에서 DispatcherServlet 안에서 생성되는 Spring Security Beans들이 구성됩니다. (1) hasFilterChain이 false라면 httpSecurity의 Configurer를 설정하고, 
이를 기반으로 보안 필터 체인을 구성합니다.

(2)에서는 이전에 생성된 다수의 SecurityFilterChain이 WebSecurity에 의해 추가로 등록되는 과정을 보여줍니다.

마무리로 webSecurity가 build() 메서드를 실행하게 되면, springSecurityFilterChain이란 이름의 Bean이 Filter 타입의 객체 인스턴스로 ApplicationContext에 등록됩니다.

<img src="https://i.ibb.co/Ms73sxX/security4.jpg" alt="security4" border="0">

#### 3. DelegatingFilterProxy에서 Http Request Intercept -> SpringSecurityFilterChain에 요청 처리 위임

```java
@Bean
@ConditionalOnBean(
    name = {"springSecurityFilterChain"}
)
public DelegatingFilterProxyRegistrationBean securityFilterChainRegistration(SecurityProperties securityProperties) {
    DelegatingFilterProxyRegistrationBean registration = new DelegatingFilterProxyRegistrationBean("springSecurityFilterChain", new ServletRegistrationBean[0]);
    registration.setOrder(securityProperties.getFilter().getOrder());
    registration.setDispatcherTypes(this.getDispatcherTypes(securityProperties));
    return registration;
}
```

서버가 실행되면 SecurityFilterAutoConfiguration 클래스에서 securityFilterChainRegistration 메서드를 통해 DelegatingFilterProxyRegistrationBean 인스턴스가 ApplicationContext에 등록됩니다.
이 시점은 DispatcherServlet이 Servlet으로 초기화되는 시점과 동시에 이루어집니다. (this.refreshContext(context); 라인에서 모두 처리)

Servlet 컨테이너에서 필터로 동작하는 Spring Security가 Spring Bean을 사용할 수 있는 이유는 **Spring Boot**가 `ServletContextInitializer`를 사용하여 **ServletContext**와 **Spring의 ApplicationContext**를 통합하기 때문입니다.
이 과정에서 `DelegatingFilterProxy`가 **ServletContext**에 필터로 등록되고, 이를 통해 **Spring Security**의 필터 체인(`springSecurityFilterChain`)이 **ApplicationContext**에서 Bean으로 관리되며 필터로 작동하게 됩니다.

이로 인해 Servlet 컨테이너는 **Spring Security**의 Bean들을 **ApplicationContext**에서 찾아 적용할 수 있게 됩니다.

---

## 🌱 닫는 글

지금까지 Spring Security가 어떻게 동작하는지에 관해 자세하게 다뤄보았습니다.

결론적으로 개발자가 인증/인가를 구현하기 위해 Spring Security를 사용할 때, Filter를 기반으로 동작한다는 것을 기억해야합니다. 
Spring Security는 이미 체계적으로 구축되어 있기 때문에, 개발자는 SecurityFilterChain을 반환하는 설정을 수정하고, 원하는 Configurer를 추가하는 것만으로도 필요한 보안 체계를 쉽게 확보할 수 있게 되었습니다.

