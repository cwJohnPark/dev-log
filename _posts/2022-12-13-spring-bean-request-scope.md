---
layout: post
title: "스프링 빈의 웹 스코프와 사용 예제"
date: 2022-12-13
category: Spring
tags: [spring, java]
---

# 스프링 빈의 웹 스코프와 사용 예제

웹 스코프란?

- 웹 환경에 최적화된 빈 스코프
- 웹 환경에서만 동작하므로 spring-mvc 의존성이 필요함
- 웹 스코프는 해당 스코프의 종료시점까지 관리하므로 빈 생명주기의 종료 메소드(`@PreDestroy`)가 호출됨

웹 스코프의 종류는?

- reuqest: HTTP 요청이 들어오고 반환될 떄 까지 유지되는 스코프, HTTP 요청과 빈 인스턴스 1:1 관계다.
- session: **HTTP Session**과 동일한 생명주기를 가지는 스코프
- application: **ServletContext**와 동일한 생명주기를 갖는 스코프
- websocket: **웹 소켓 세션**과 동일한 생명주기의 스코프다.

## Request 스코프 예제

스프링 웹 환경 의존성을 추가해야 한다.

`org.springframework.boot:spring-boot-start-web`

> 🤔 스프링부트는 웹 의존성이 없을 때, 다른 애플리케이션 컨텍스트가 다를까?
> 
> `AnnotationConfigApplicationContext`를 기반으로 컨테이너를 구동함
>  
> 반면, 웹 라이브러리가 존재할 경우, `AnnotationConfigServletWebServerApplicationContext` 를 기반으로 애플리케이션이 구동됨


- 예제 요구사항: 동시에 여러 HTTP 요청이 들어올 때 어떤 요청이 남긴 로그인지 구분이 어렵다. 어떤 요청이 로그를 남겼는지 확인하도록 수정하자.

- 예를 들어, 다음과 같은 로그가 남아야 한다.

```java
...
[d06b992f..] request scope bean create
[d06b992f..][/log-demo] controller test
...
[d06b992f..] request scope bean close
```

**포맷: [UUID][request-path]{message}**

(UUID는 HTTP요청을 구분하는 용도로 사용한다. )

```java
@Component
@Scope("request")
public class MyLogger {
  private String uuid;
  private String requestURL;

  /// setter
  public void log(String message) {
    log.info("[{}][{}] {}", uuid, requestUrl, message);
  }

  @PostConstruct
  public void init() {
    uuid = UUID.randomUUID().toString();
    log.info("[{}] request scope bean create: {}", uuid, this);
  }

  @PreDestroy
  public void close() {
    log.info("[{}] request scope bean close: {}", uuid, this);
  }
}

```

- `requestURL` 필드는 빈 생성시점에는 알 수 없으므로 외부에서 setter를 호출해야 한다.

```java
@RestController
@RequiredArgsConstructor
public class LogDemoController {
   // private final MyLogger myLogger; 의존성 주입 오류발생함
   private final ObjectProvider<MyLogger> myLoggerProvider;

   @GetMapping("/log-demo")
   public String logDemo(HttpServletRequest req) {
      MyLogger myLogger = myLoggerProvider.getObject();
      myLogger.setUrl(req.getRequestURL().toString());
      myLogger.log("controller test");
      ...
   }
}
```

🚨 주의사항

- 싱글톤 빈에 request 스코프 빈을 그대로 주입하면 안된다.
- 왜냐하면, 컨테이너 생성 후 의존성 주입시점에 request 스코프 빈은 존재하지 않기 때문이다.

---

[스프링 핵심 원리 - 기본편, 김영한](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8#curriculum)
