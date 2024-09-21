---
layout: single
title: "[Spring] 스프링 부트와 내장 톰캣"
categories: Backend
tag: [Spring]
toc: true
author_profile: false
sidebar:
  nav: "counts"
published: true
---

과거에는 웹 애플리케이션을 개발하고 배포하기 위해 WAR(Web Application Archive) 방식이 표준이었다. 웹 애플리케이션 서버(WAS)와 애플리케이션 빌드 파일(WAR)을 분리해 운영하는 구조였지만 이 방식은 몇 가지 단점이 있다.

## WAR 배포 방식의 단점
1. WAS 설치의 번거로움
- 톰캣(Tomcat)과 같은 웹 애플리케이션 서버(WAS)를 별도로 설치해야 한다.

2. 복잡한 개발 환경 설정
- 단순한 자바 애플리케이션은 main() 메서드를 실행하기만 하면 되지만, 웹 애플리케이션은 WAS와 연동하기 위한 복잡한 설정이 필요하다.

3. 복잡한 배포 과정
- 애플리케이션을 배포하려면 먼저 WAR 파일을 빌드하고, 이를 WAS에 전달해 배포해야 한다.

4. 톰캣 버전 관리의 어려움
- 톰캣 버전을 변경할 경우, 새로운 톰캣을 다시 설치하고 설정해야 한다.

## 내장 서버 

이러한 불편함을 해결하기 위해, 톰캣과 같은 웹 서버를 라이브러리로 내장하는 방법이 등장하였다.

<div style="display: flex; justify-content: center;">
     <img src="{{site.url}}\images\2024-09-21-spring-boot-tomcat\was_jar.png" alt="Alt text" style="width: 80%; height: 80%; margin: 30px">
</div>

- 왼쪽 그림 (외장 서버)
    - WAR(Web Application Archive) 파일로 패키징한 후, Tomcat과 같은 WAS에 배포하는 방식
    - Tomcat이 실행되면, 서버는 WAR 파일을 읽고 애플리케이션을 로드하여 클라이언트의 요청을 처리
- 오른쪽 그림 (내장 서버)
    - Tomcat과 같은 WAS를 애플리케이션과 함께 JAR(Java Archive) 파일에 포함시켜, 별도의 WAS 설정 없이 바로 실행하는 **내장 톰캣(Embedded Tomcat) 방식**
    - 개발자가 만든 JAR 파일을 실행하면 main() 메서드에서 내장 톰캣이 자동으로 구동되고, 애플리케이션은 WAS 없이도 동작


## 내장 톰캣 

### gradle 설정 

```groovy

dependencies {
    //스프링 MVC 추가
    implementation 'org.springframework:spring-webmvc:6.1.11'

    //내장 톰켓 추가
    implementation 'org.apache.tomcat.embed:tomcat-embed-core:10.1.25'
}

// 일반 Jar 생성
task buildJar(type: Jar) {
    manifest {
        attributes 'Main-Class': 'hello.embed.EmbedTomcatSpringMain'
    }
    with jar
}

// Fat Jar 생성
// Jar 안에 Jar를 포함하지 못하는 문제 해결
task buildFatJar(type: Jar) {
    manifest {
        attributes 'Main-Class': 'hello.embed.EmbedTomcatSpringMain'
    }
    duplicatesStrategy = DuplicatesStrategy.WARN
    // 라이브러리들을 build 안에 포함시킴
    from { configurations.runtimeClasspath.collect { it.isDirectory() ? it : zipTree(it) } }
    with jar
}


```

- **tomcat-embed-core** : 톰캣 라이브러리이다. 톰캣을 라이브러리로 포함해서 톰캣 서버를 자바 코드로
실행할 수 있다.

### 서블릿 등록 

```java
public class EmbedTomcatServletMain {

    public static void main(String[] args) throws LifecycleException {
        System.out.println("EmbedTomcatServletMain.main");
        //톰캣 설정
        Tomcat tomcat = new Tomcat();
        Connector connector = new Connector();
        connector.setPort(8080);
        tomcat.setConnector(connector);
        
        //서블릿 등록
        Context context = tomcat.addContext("", "/");
        
        // DocBase 경로 처리 및 디렉터리 생성
        File docBaseFile = new File(context.getDocBase());
        if (!docBaseFile.isAbsolute()) {
            docBaseFile = new File(((org.apache.catalina.Host)
                    context.getParent()).getAppBaseFile(), docBaseFile.getPath());
        }
        docBaseFile.mkdirs();

        // 서블릿 등록 및 매핑
        tomcat.addServlet("", "helloServlet", new HelloServlet());
        context.addServletMappingDecoded("/hello-servlet", "helloServlet");
        tomcat.start();

    }
}

```

#### Tomcat 설정
- 내장 톰캣을 생성하고, 톰캣이 제공하는 커넥터를 사용해서 8080 포트에 연결한다.


#### 서블릿 등록
- HelloServlet이라는 서블릿 클래스를 Tomcat 서버에 **"helloServlet"**이라는 이름으로 등록한다.
- /hello-servlet 경로로 들어오는 HTTP 요청을 helloServlet으로 매핑한다.

#### Tomcat 서버 시작
-  Tomcat 서버를 시작하여, 클라이언트 요청을 처리할 준비를 완료한다.

#### 실행 
- http://localhost:8080/hello-servlet 에 접속하면 `hello servlet!` 이 표시된다.

내장 톰캣을 사용한 덕분에, IDE에서 별도의 복잡한 톰캣 설정 없이 main() 메서드만 실행하면 톰캣 서버까지 매우 편리하게 실행할 수 있었다. 

### 스프링 

내장 톰캣에 스프링까지 연동해보자.

```java

public class EmbedTomcatSpringMain {

    public static void main(String[] args) throws LifecycleException {
        System.out.println("EmbedTomcatSpringMain.main");
        //톰캣 설정
        Tomcat tomcat = new Tomcat();
        Connector connector = new Connector();
        connector.setPort(8080);
        tomcat.setConnector(connector);

        // 스프링 컨테이너 설정
        AnnotationConfigWebApplicationContext appContext = new AnnotationConfigWebApplicationContext();
        appContext.register(HelloConfig.class);

        //스프링 MVC 디스패처 서블릿 생성, 스프링 컨테이너 연결
        DispatcherServlet dispatcherServlet = new DispatcherServlet(appContext);

        // 디스패처 서블릿 등록
        Context context =  tomcat.addContext("", "/");
        tomcat.addServlet("", "dispatcher", dispatcherServlet);
        context.addServletMappingDecoded("/", "dispatcher");

        tomcat.start();

    }
}

```

#### Tomcat 설정
- 내장 톰캣을 생성하고, 톰캣이 제공하는 커넥터를 사용해서 8080 포트에 연결한다.

#### 서블릿 등록
- tomcat.addServlet() 을 통해서 서블릿을 등록한다.
- context.addServletMappingDecoded() 을 통해서 등록한 서블릿의 경로를 매핑한다.
-  루트 경로(/)로 들어오는 모든 요청을 "dispatcher" 서블릿으로 매핑한다.

#### Tomcat 서버 시작
- Tomcat은 포트 8080에서 HTTP 요청을 대기하며, 요청이 들어오면 Spring의 DispatcherServlet을 통해 애플리케이션의 로직을 실행한다.

<br>

내장 톰캣 방식 코드를 보면 서블릿 컨테이너 초기화와 거의 동일한 흐름으로 내장 톰캣을 설정하고 실행하는 것을 알 수 있다. 하지만 실행 주체가 다르다는 점이 핵심적인 차이이다. 

서블릿 컨테이너 방식은 서블릿 컨테이너가 자체적으로 서버를 시작하고, 애플리케이션을 초기화하지만 내장 톰캣 방식은 개발자가 main() 메서드를 호출하여 톰캣 서버를 직접 제어하고, 실행한다.


### 빌드와 배포 
애플리케이션에 내장 톰캣을 라이브러리로 포함할 뗴 어떻게 빌드하고 배포하는지 알아보자. 

자바의 main() 메서드를 실행하기 위해서는 **jar 형식**으로 빌드해야 한다. 그리고 jar 안에는 META-INF/MANIFEST.MF 파일에 실행할 main() 메서드의 클래스를 지정해주어야 한다.

Gradle의 도움을 받으면 이 과정을 쉽게 진행할 수 있다.

#### gradle 설정 

```groovy

// 일반 Jar 생성
task buildJar(type: Jar) {
    manifest {
        attributes 'Main-Class': 'hello.embed.EmbedTomcatSpringMain
    }'
    with jar
}



```
buildJar 태스크에서 attributes를 사용하여 Main-Class 를 설정했다. 

- attributes 'Main-Class': 'hello.embed.EmbedTomcatSpringMain': JAR 파일이 실행될 때 Gradle이 이 클래스의 main() 메서드를 호출한다.

- 빌드 과정: ./gradlew clean buildJar 명령어를 이용해 JAR 파일을 빌드하면 build/libs 위치에 JAR 파일이 생성된다.

- JAR 파일 실행: 이제 java -jar embed-0.0.1-SNAPSHOT.jar 명령어로 JAR 파일을 실행

하지만 실행 결과를 보면 기대했던 내장 톰캣 서버가 실행되지 않고 오류가 발생하는 것을 확인할 수 있습니다. 그 이유는 **JAR 파일은 다른 JAR 파일을 포함할 수 없기 때문이다.**

특히, 스프링 부트와 내장 톰캣과 같은 라이브러리를 포함해야 하는데, 이들 라이브러리가 현재 JAR 파일 안에 포함되어 있지 않아서 문제가 발생한다.

### FatJar

대안으로는 Fat Jar 또는 Uber Jar라는 방법이 있다. JAR 파일 안에는 다른 JAR 파일을 포함할 수 없지만, 클래스 파일은 얼마든지 포함할 수 있다는 점을 이용했다.

1. 라이브러리 JAR 풀기: 사용되는 라이브러리 JAR 파일을 압축 해제하면 내부에 있는 클래스 파일들이 나온다.

2. 클래스 파일 포함: 이렇게 추출한 클래스 파일들을 새로 생성하는 JAR 파일에 포함한다.

3. 뚱뚱한(Fat) JAR 생성: 여러 라이브러리에서 나오는 클래스 파일이 포함되기 때문에, 최종적으로 생성된 JAR 파일은 상당히 큰 용량을 가지게 된다. 그래서 이러한 JAR 파일을 Fat Jar라고 부른다.


```groovy

// Fat Jar 생성
// Jar 안에 Jar를 포함하지 못하는 문제 해결
task buildFatJar(type: Jar) {
    manifest {
        attributes 'Main-Class': 'hello.embed.EmbedTomcatSpringMain'
    }
    duplicatesStrategy = DuplicatesStrategy.WARN
    // 라이브러리들을 build 안에 포함시킴
    from { configurations.runtimeClasspath.collect { it.isDirectory() ? it : zipTree(it) } }
    with jar
}

```

- 빌드 과정: ./gradlew clean buildFatJar 명령어를 이용해 JAR 파일을 빌드하면 build/libs 위치에 JAR 파일이 생성된다.

- JAR 파일 실행: 이제 java -jar embed-0.0.1-SNAPSHOT.jar 명령어로 JAR 파일을 실행

정상 동작하는 것을 확인할 수 있다. Fat Jar은 WAR를 외부 서버에 배포하는 방식의 단점을 해결했다.


#### Fat Jar의 장점 - WAR 단점 해결
- 별도의 WAS 설치 필요
    - 문제: 톰캣 같은 WAS를 별도로 설치해야한다.
    - 해결: WAS를 별도로 설치할 필요 없이, 톰캣과 같은 WAS가 라이브러리로 JAR 내부에 포함되어 있다.

- 복잡한 개발 환경 설정
    - 문제: 단순한 Java 애플리케이션이라면 별도의 설정 없이 main() 메서드만 실행하면 되지만, 웹 애플리케이션은 WAS와의 연동을 위한 복잡한 설정이 필요하다.
    - 해결: IDE에 복잡한 WAS 설정이 필요 없으며, 단순히 main() 메서드만 실행하면 된다.

- 복잡한 배포 과정
    - 문제: WAR 파일을 만들고, 이를 WAS에 전달하여 배포해야 하는 과정이 복잡하다.
    - 해결: 배포 과정이 단순해져, JAR 파일을 만들고 원하는 위치에서 실행하기만 하면 된다.

- 톰캣 버전 업데이트 시 번거로움
    - 문제: 톰캣의 버전을 업데이트하려면 톰캣을 다시 설치해야 한다.
    - 해결: Gradle에서 내장 톰캣 라이브러리의 버전만 변경하고 빌드 후 실행하면 된다.


<br>

**Fat Jar는 완벽해 보이지만 몇가지 단점을 여전히 포함하고 있다.**

1. **포함된 라이브러리 확인 어려움**
    - 문제: 어떤 라이브러리가 포함되어 있는지 확인하기 어렵다. 모든 클래스가 풀려 있기 때문에, 사용 중인 라이브러리를 추적하기가 힘들다.

2. **파일명 중복 문제**
    - 문제: 클래스나 리소스의 이름이 같은 경우, 하나를 포기해야 하는 문제가 발생한다. 이는 심각한 문제를 초래할 수 있다.

    - 예시: **META-INF/services/jakarta.servlet.ServletContainerInitializer** 파일이 여러 라이브러리(JAR)에 존재할 수 있다. 예를 들어, A 라이브러리와 B 라이브러리 모두 이 파일을 사용하여 서블릿 컨테이너 초기화를 시도한다고 가정해보자

    - Fat Jar를 생성할 때, 이 파일명이 같으므로 A와 B 라이브러리가 가진 파일 중 하나만 선택된다. 결과적으로 나머지 하나는 포함되지 않으며, 이로 인해 정상적인 동작이 이루어지지 않을 수 있다.
.

### My Boot Class 

지금까지 진행한 내장 톰캣 실행, 스프링 컨테이너 생성, 디스패처 서블릿 등록의 모든 과정을 편리하게 처리해주는 부트 클래스를 만들어보자. 부트는 이름 그대로 시작을 편하게 처리해주는 것을 뜻한다.

### MySpringApplication

```java
package hello.boot;

public class MySpringApplication {
    public static void run(Class configClass, String[] args) {
        System.out.println("MySpringApplication.run args = "+ List.of(args));

        //톰캣 설정
        Tomcat tomcat = new Tomcat();
        Connector connector = new Connector();
        connector.setPort(8080);
        tomcat.setConnector(connector);

        // 스프링 컨테이너 설정
        AnnotationConfigWebApplicationContext appContext = new AnnotationConfigWebApplicationContext();
        appContext.register(configClass);

        //스프링 MVC 디스패처 서블릿 생성, 스프링 컨테이너 연결
        DispatcherServlet dispatcherServlet = new DispatcherServlet(appContext);

        // 디스패처 서블릿 등록
        Context context =  tomcat.addContext("", "/");
        tomcat.addServlet("", "dispatcher", dispatcherServlet);
        context.addServletMappingDecoded("/", "dispatcher");

        try {
            tomcat.start();
        } catch (LifecycleException e) {
            throw new RuntimeException(e);
        }


    }
}


```

기존 코드를 모아서 편리하게 사용할 수 있는 클래스를 만들었다. MySpringApplication.run() 을 실행하면 바로 작동한다.

- configClass : 스프링 설정을 파라미터로 전달받는다.
- args : main(args) 를 전달 받아서 사용한다. 참고로 예제에서는 단순히 해당 값을 출력한다.
- tomcat.start() 에서 발생하는 예외는 잡아서 런타임 예외로 변경했다.


### @MySpringBootApplication

@MySpringBootApplication은 Spring Boot 애플리케이션의 주요 진입점으로 사용될 수 있는 커스텀 어노테이션이다. 

```java
package hello.boot;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@ComponentScan
public @interface MySpringBootApplication {
}


```
#### 어노테이션 설명

- @Target(ElementType.TYPE):
    - 이 어노테이션이 적용될 수 있는 대상을 정의한다. 
    - ElementType.TYPE은 클래스, 인터페이스(열거형 포함), 또는 애너테이션 자체에 적용될 수 있음을 의미하며, @MySpringBootApplication은 클래스 레벨에서 사용할 수 있다.

- @Retention(RetentionPolicy.RUNTIME)
    - 이 어노테이션의 유지 정책을 정의한다. 
    - RetentionPolicy.RUNTIME은 이 어노테이션이 런타임 동안 유지됨을 의미한다.

- @Documented
    - 이 어노테이션이 문서화되어야 한다는 것을 나타낸다. 
    - Javadoc와 같은 문서 생성 도구에서 이 어노테이션에 대한 설명이 포함된다.

- **@ComponentScan**
    - 스프링에서 제공하는 어노테이션으로, 스프링 컨테이너가 지정된 패키지(기본적으로 어노테이션이 적용된 클래스의 패키지) 내의 컴포넌트(예: @Component, @Service, @Repository, @Controller 등)를 검색하고 자동으로 등록할 수 있도록 한다.
    - **어노테이션이 적용된 클래스가 위치한 패키지와 그 하위 패키지에서 스프링 빈을 자동으로 찾아서 등록한다.**


### MySpringBootMain 

실제 애플리케이션을 실행하는 코드이다. @MySpringBootApplication 어노테이션을 통해 스프링의 기능을 자동으로 활성화한다.

```java
package hello;

import hello.boot.MySpringApplication;
import hello.boot.MySpringBootApplication;

@MySpringBootApplication
public class MySpringBootMain {
    public static void main(String[] args) {
        System.out.println("MySpringBootMain.main");
        MySpringApplication.run(MySpringBootMain.class, args);
    }
}

```

MySpringBootMain 클래스는 hello 패키지에 위치하고 있다. 이 위치가 중요한 이유는 @MySpringBootApplication 애노테이션이 컴포넌트 스캔 기능을 포함하고 있기 때문이다. 컴포넌트 스캔의 기본 동작은 해당 애노테이션이 붙은 **클래스의 현재 패키지와 그 하위 패키지를 스캔**하여 스프링 빈으로 등록하는 것이다. 따라서 HelloController 클래스도 자동으로 스프링 빈에 등록된다(hello.spring.HelloController)


**MySpringApplication.run(MySpringBootMain.class, args);** 이 한 줄로 애플리케이션을 실행할 수 있다. 

- 내장 톰캣 실행: 웹 서버를 자동으로 시작
- 스프링 컨테이너 생성: 필요한 스프링 빈들을 생성하고 관리
- 디스패처 서블릿 설정: HTTP 요청을 처리하는 중앙 관리자를 설정
- 컴포넌트 스캔: 지정된 패키지에서 스프링 빈을 검색하고 등록


## 스프링 부트
이제 위 과정을 자동으로 해주는 스프링 부트를 활용하여 프로젝트를 만들어 보자

### gradle 설정

**spring-boot-starter-web** 라이브러리를 추가한다. 

``` groovy

plugins {
	id 'java'
	id 'org.springframework.boot' version '3.3.4'
	id 'io.spring.dependency-management' version '1.1.6'
}

group = 'hello'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '17'

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

```

### BootApplication

```java

@SpringBootApplication
public class BootApplication {

	public static void main(String[] args) {
		SpringApplication.run(BootApplication.class, args);
	}

}

```

이 단순해 보이는 코드 한줄 안에서는 수 많은 일들이 발생하지만 핵심은 2가지다. 

1. 스프링 컨테이너를 생성한다. 
2. WAS(내장 톰캣)를 생성한다.

### 스프링 부트 내부에서 스프링 컨테이너를 생성하는 코드


```java
class ServletWebServerApplicationContextFactory implements
ApplicationContextFactory {
 ...
 private ConfigurableApplicationContext createContext() {
 if (!AotDetector.useGeneratedArtifacts()) {
 return new AnnotationConfigServletWebServerApplicationContext();
 }
 return new ServletWebServerApplicationContext();
}
```

new AnnotationConfigServletWebServerApplicationContext() 이 부분이 바로 스프링 부트가 생성하는 스프링 컨테이너이다. 이름 그대로 애노테이션 기반 설정이 가능하고, 서블릿 웹 서버를 지원하는 스프링 컨테이너이다.


### 스프링 부트 내부에서 내장 톰캣을 생성하는 코드

org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory
```java
@Override
public WebServer getWebServer(ServletContextInitializer... initializers) {
 ...
Tomcat tomcat = new Tomcat();
 ...
Connector connector = new Connector(this.protocol);
 ...
return getTomcatWebServer(tomcat);
}

```

스프링 부트도 우리가 앞서 내장 톰캣에서 진행했던 것과 동일한 방식으로 스프링 컨테이너를 만들고, 내장 톰캣을 생성하고 그 둘을 연결하는 과정을 진행한다.


<br>

### 빌드와 배포 

내장 톰캣이 포함된 스프링 부트를 직접 빌드해보자.

- 빌드 과정: ./gradlew clean build 명령어를 이용해 JAR 파일을 빌드하면 build/libs 위치에 JAR 파일이 생성된다.`

- JAR 파일 실행: 이제 java -jar embed-0.0.1-SNAPSHOT.jar 명령어로 JAR 파일을 실행

스프링 부트 애플리케이션이 실행되고, 내장 톰캣이 8080 포트로 실행된 것을 확인할 수 있다.

```
java -jar boot-0.0.1-SNAPSHOT.jar

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/

 :: Spring Boot ::                (v3.3.4)

```

### 스프링 부트 실행 가능 Jar

JAR를 푼 결과를 보면 Fat Jar가 아니라 처음보는 새로운 구조로 만들어져 있다. 

Fat Jar는 하나의 Jar 파일에 라이브러리의 클래스와 리소스를 모두 포함했다. 그래서 실행에 필요한 모든 내용을 하나의 JAR로 만들어서 배포하는 것이 가능했다. 하지만 Fat Jar는 파일명 중복 문제 등 여러 문제를 가지고 있었다.

####  META-INF/MANIFEST.MF 파일

META-INF/MANIFEST.MF

```
Manifest-Version: 1.0
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: hello.boot.BootApplication
Spring-Boot-Version: 3.3.4
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Spring-Boot-Classpath-Index: BOOT-INF/classpath.idx
Spring-Boot-Layers-Index: BOOT-INF/layers.idx
Build-Jdk-Spec: 17
```

- **Main-Class** : 우리가 기대한 main() 이 있는 hello.boot.BootApplication 이 아니라 JarLauncher 라 는 전혀 다른 클래스를 실행하고 있다.
- **JarLauncher** 는 스프링 부트가 빌드시에 넣어준다. org/springframework/boot/loader/JarLauncher 에 실제로 포함되어 있다.

JAR 파일 내에 포함된 다른 JAR 파일들을 읽어들이는 기능이 필요하며, 스프링 부트는 이러한 기능을 제공하기 위해 JarLauncher를 사용한다.

#### JarLauncher의 기능

- JAR 파일 구조 처리
    스프링 부트는 JAR 파일 내에 JAR 파일을 포함할 수 없지만, JarLauncher는 내부 JAR 파일을 적절하게 읽어들이고 클래스 정보를 처리하는 기능을 수행한다.

- 클래스 로딩
    JarLauncher는 지정된 Start-Class에 따라 애플리케이션의 main() 메서드를 호출하기 전에 필요한 설정을 처리한다. 

- 애플리케이션 시작
    모든 준비 작업이 완 료된 후, JarLauncher는 Start-Class에 지정된 클래스의 main() 메서드를 호출하여 실제 애플리케이션을 실행한다.


### 스프링 부트의 실행 가능한 JAR
WAR (Web Application Archive) 구조는 웹 애플리케이션을 패키징하기 위한 형식으로, 일반적으로 WEB-INF라는 폴더를 포함한다. 이 폴더 안에는 사용자 클래스와 라이브러리(JAR 파일)가 위치한다.

스프링 부트의 실행 가능한 JAR도 이 구조를 따르며, 내부 폴더는 **BOOT-INF**로 이름 붙여졌다. JarLauncher는 이 폴더에 포함된 클래스와 라이브러리를 읽어들이는 역할을 한다.



#### 실행 과정

1. java -jar xxx.jar
2. MANIFEST.MF 인식
3. JarLauncher.main() 실행
    - JarLauncher의 main() 메서드가 호출된다. 이 단계에서 스프링 부트 애플리케이션의 실행 준비가 시작된다.
4. BOOT-INF/classes/ 인식
5. BOOT-INF/lib/ 인식
    - 의존성 라이브러리(JAR 파일)가 포함된 BOOT-INF/lib/ 폴더가 인식
6. BootApplication.main() 실행
    - 최종적으로 BootApplication 클래스의 main() 메서드가 실행되며, 여기에서 스프링 애플리케이션이 시작된다.


## 정리 
스프링 부트를 이용하면 내장 톰캣과 스프링 컨테이너 설정을 **SpringApplication.run(BootApplication.class, args);** 한줄로 해결 할 수 있다. 

또한, 스프링 부트는 Fat Jar의 문제점을 해결하기 위해 JarLauncher를 사용하여 BOOT-INF/classes와 BOOT-INF/lib에 있는 JAR 파일들을 읽어들인다. 



<br>
<br>

Reference
----
- <a href = 'https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-%ED%95%B5%EC%8B%AC%EC%9B%90%EB%A6%AC-%ED%99%9C%EC%9A%A9/dashboard'>스프링 부트 - 핵심 원리와 활용</a>

