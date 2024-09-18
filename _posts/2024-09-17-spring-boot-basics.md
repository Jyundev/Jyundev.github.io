---
layout: single
title: "[Spring] 웹 서버와 스프링 프레임워크 "
categories: Backend
tag: [Spring]
toc: true
author_profile: false
sidebar:
  nav: "counts"
published: true
---
EJB는 1990년대 후반에 등장했다. 그 목적은 자바 기반의 대규모 엔터프라이즈 애플리케이션을 쉽게 개발하고 배포할 수 있는 표준을 제공하는 것이었지만 여러 문제들이 있었다. 

EJB의 구조는 매우 엄격하고 복잡해서, 단순한 비즈니스 로직을 작성하더라도 많은 코드를 작성해야 했고 이는 개발자들의 생산성을 저하시키고 유지보수를 어렵게 했다.

이로 인해 **"EJB 지옥(EJB Hell)"**이라는 말이 생겨났다. 이는 EJB를 사용하면서 느껴지는 복잡함과 비효율성을 풍자한 표현으로, 많은 개발자들이 어려움을 겪었다.

## 스프링 프레임워크와 스프링 부트


<div style="display: flex; justify-content: center;">
     <img src="{{site.url}}\images\2024-09-17-spring-boot-basics\springboot.png" alt="Alt text" style="width: 80%; height: 80%; margin: 20px">
</div>

스프링 프레임워크는 자바 엔터프라이즈 애플리케이션 개발을 단순화하는 목적으로 2003년에 처음 출시되었다. 스프링 프레임워크는 객체 지향 프로그래밍에서 가장 중요한 개념 중 하나인 **의존성 주입(DI, Dependency Injection)**과 **제어의 역전(IOC, Inversion of Control)**을 도입하여 개발자가 더 쉽게 코드 구조를 관리할 수 있도록 했다.

하지만 스프링을 사용하려면 복잡한 XML 설정과 초기화 코드를 많이 작성해야 했기 때문에, 진입 장벽이 높고 개발이 번거로웠다.

스프링 프레임워크의 복잡한 설정 문제를 해결하고, 빠른 애플리케이션 개발을 지원하기 위해 **스프링 부트(Spring Boot)**가 2014년에 처음 공개되었다.


### 스프링 부트 핵심기능

<div style="display: flex; justify-content: center;">
     <img src="{{site.url}}\images\2024-09-17-spring-boot-basics\javaee_spring.png" alt="Alt text" style="width: 80%; height: 80%; margin: 20px">
</div>

스프링 부트의 주요 특징은 다음과 같다. 

- WAS(내장 웹 서버): Tomcat 등의 웹 서버를 내장해 별도의 설치 없이 애플리케이션을 실행 가능.
- 라이브러리 관리: 스타터 종속성을 통해 쉽게 의존성을 추가하고, 스프링과 외부 라이브러리의 버전을 자동 관리.
- 자동 구성: 스프링 및 외부 라이브러리의 빈을 자동으로 등록해 초기 설정을 간소화.
- 외부 설정: 환경(개발, 배포 서버 등)에 따라 설정을 공통화하고 외부화하여 손쉽게 관리.
- 프로덕션 준비: 매트릭스와 상태 확인을 위한 모니터링 기능 제공.


### 스프링 vs 스프링 부트

스프링 부트는 스프링 프레임워크를 쉽게 사용하기 위한 도구이며, 본질은 스프링 프레임워크에 있다 

<div style="display: flex; justify-content: center;">
     <img src="{{site.url}}\images\2024-09-17-spring-boot-basics\spring_springboot.gif" alt="Alt text" style="width: 70%; height: 70%; margin: 20px">
</div>


### 스프링 부트를 배워야 하는 이유
1. 스프링 부트는 편리하지만 많은 기능을 자동화하므로, 작동 원리를 이해해야 문제 발생 시 해결 가능하다
2. 스프링 부트의 원리를 알면 문제를 더 효과적으로 파악하고 해결할 수 있다.
3. 다양한 편리한 기능(외부 설정, 모니터링 관리)을 통해 개발 시간을 단축시킬 수 있다.

<br>



## 웹서버와 스프링 부트 


### 전통적인 방식 - 웹 애플리케이션 서버 (was)

과거에는 자바로 웹 애플리케이션을 개발할 때 다음과 같은 과정을 거쳤다

1. 서버에 WAS(웹 애플리케이션 서버), 예를 들어 Tomcat을 설치.
2. 서블릿 스펙에 맞춰 코드를 작성.
3. 애플리케이션을 WAR(WEB Archive) 파일로 빌드.
4. 이 WAR 파일을 WAS에 배포하고 실행.

개발자는 WAS와 애플리케이션을 별도로 관리하고, 애플리케이션 업데이트 시 다시 빌드하고 배포하는 번거로운 과정을 반복해야 했다.

### 최근 방식 - 스프링 부트
스프링 부트를 사용하면 WAS와 애플리케이션이 결합된 형태로 동작한다.

1. 스프링 부트는 내장 Tomcat과 같은 WAS를 포함하고 있다.
2. 개발자는 애플리케이션 코드를 작성하고 JAR(Java Archive) 파일로 빌드한다.
3. 이 JAR 파일을 실행하면 애플리케이션과 WAS가 함께 실행된다.

JAR 파일을 실행하는 것만으로 애플리케이션과 WAS가 동시에 동작하며, 별도의 WAS 설치나 설정이 필요 없다.

쉽게 이야기해서 개발자는 main() 메서드만 실행하면 되고, WAS 설치나 IDE 같은 개발 환경에서 WAS와 연동하는 복잡한 일은 수행하지 않아도 된다.



#### JAR (Java Archive)

JAR는 자바 클래스 파일과 리소스를 하나로 묶은 압축 파일이다. 직접 실행하려면 main() 메서드가 필요하며, MANIFEST.MF 파일에 실행할 메인 클래스를 지정해야 한다.

JAR 파일은 애플리케이션 실행 파일로 사용할 수 있을 뿐만 아니라 라이브러리로도 활용할 수 있습니다. 간단한 압축 파일로, 여러 자바 클래스와 리소스를 포함하며, 애플리케이션을 배포하거나 다른 프로젝트에서 의존성으로 사용할 수 있다.

#### WAR (Web Application Archive)
WAR 파일은 **웹 애플리케이션 서버(WAS)**에 배포할 때 사용하는 파일이다. JAR 파일이 JVM에서 직접 실행되는 반면, WAR 파일은 WAS에서 실행된다. WAR 파일은 특정한 디렉토리 구조를 가져야하며 주요 디렉토리는 다음과 같다.

- WEB-INF: 애플리케이션의 설정 파일과 관련 리소스를 포함.
  - classes: 자바 클래스 파일들이 위치.
  - lib: 애플리케이션에 필요한 외부 라이브러리(JAR 파일들)가 위치.
  - web.xml: 애플리케이션 설정 정보(서블릿, 필터 등)를 정의하는 XML 파일.
- index.html : 정적 리소스


## 서블릿 컨테이너

<div style="display: flex; justify-content: center;">
     <img src="{{site.url}}\images\2024-09-17-spring-boot-basics\was_spring.png" alt="Alt text" style="width: 100%; height: 100%; margin: 30px">
</div>


**서블릿(Servlet)**은 자바 기반의 웹 애플리케이션에서 클라이언트의 요청(주로 HTTP 요청)을 처리하고, 그 결과를 동적으로 생성하여 응답하는 자바 프로그램이다.

**서블릿 컨테이너(Servlet Container)**는 **웹 애플리케이션 서버(WAS)**의 일종으로, 자바 서블릿을 실행하고 관리하는 역할을 담당하는 컴포넌트이다. 서블릿 컨테이너는 클라이언트로부터 들어오는 HTTP 요청을 처리하고, 서블릿을 통해 동적인 웹 페이지를 생성하거나 서버 로직을 실행한 뒤, 그 결과를 클라이언트로 반환하는 작업을 수행한다.

**Apache Tomcat**은 가장 널리 사용되는 서블릿 컨테이너 중 하나로, 자바 웹 애플리케이션을 실행할 수 있다.


스프링 프레임워크의 작동 원리를 이해하기 위해 서블릿 컨테이너의 초기화 기능을 알아보고 이어서 이 초기화 기능을 활용해 스프링 만들고 연결해보자.


<br>
<br>

----
Reference

- <a href = 'https://devrant.com/rants/1867059/spring-vs-spring-boot'>Spring vs Spring Boot</a>
- <a href = 'https://nintriva.com/blog/java-ee-spring-boot-comparison//'>Java EE vs. Spring Boot</a>
- <a href = 'https://se.ewi.tudelft.nl/desosa2019/chapters/spring-boot/'>Spring Boot - production-grade Spring-based Applications that you can “just run”</a>
- <a href = 'https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-%ED%95%B5%EC%8B%AC%EC%9B%90%EB%A6%AC-%ED%99%9C%EC%9A%A9/dashboard'>스프링 부트 - 핵심 원리와 활용</a>

