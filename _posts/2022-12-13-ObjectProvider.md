---
layout: post
title: "ObjectProvider를 사용한 싱글톤 빈의 프로토타입 빈 참조"
date: 2022-12-13
category: Spring
tags: [spring, java]
---
# ObjectProvider를 사용한 싱글톤 빈의 프로토타입 빈 참조

## 방법1: ApplicationContext 직접 참조

```java
@Test
void test() {
	AnnotationConfigApplicationContext(ClientBean.class, PrototypeBean.class);
	
	ClientBean clientBean1 = ac.getBean(ClientBean.class);
	clientBean1.callPrototypeBeanMethod();
	ClientBean clientBean2 = ac.getBean(ClientBean.class);
	clientBean2.callPrototypeBeanMethod();
}

static class ClientBean {
  @Autowired
  private ApplicationContext ac;
  public int callPrototypeBeanMethod() {
    PrototypeBean prottypeBean = ac.getBean(PrototypeBean.class);
    // 프로토타입 빈 메서드 호출..
  }
}
```

위 방법의 문제점은 ApplicationContext를 직접적으로 사용해야 한다는 것이다.

💡 의존성 주입이 아니라 직접 의존관계를 찾는 것을 DL(Dependency Lookup) 이라고 한다.

### 방법2 ObjectProvider<T> 사용

ObjectProvider는 스프링의 ApplicationContext 를 간접적으로 사용하는 것과 비슷하다.

```java
@Scope("singleton")
static class ClientBean {
  @Autowired
  ObjectProvider<PrototypeBean> provider;

  public int callPrototypeBeanMethod() {
     PrototypeBean bean = provider.getObject();
     bean.call();
     ...
  }
}
```

🤔 ObjectProvider는 런타임에 안전할까? 
즉, 스프링 컨테이너 생성 시점에 등록되지 않은 빈을 ObjectProvider로 사용하는 것을 방지해줄까?

- 정답
    
    😥 아쉽게도 런타임에 안전하지 못하다.
    
    ```java
    @SpringBootTest(classes = ObjectProviderTest.MyBean.class)
    class ObjectProviderTest {
    
    	@Autowired MyBean myBean;
    
    	@Test
    	void name() {
    		int result = myBean.getAndIncrement();
    		assertThat(result).isEqualTo(0);
    
    		assertThatThrownBy( () -> myBean.callNotBeanType())
    			.isInstanceOf(NullPointerException.class);
    	}
    
    	interface NotBeanType {
    		void youCannotCallThis();
    	}
    
    	@Component
    	static class MyBean {
    		private ObjectProvider<NotBeanType> notBeanType;
    		private int value;
    
    		public int getAndIncrement() {
    			return value++;
    		}
    		public void callNotBeanType() {
    			NotBeanType notBeanType = this.notBeanType.getObject();
    			notBeanType.youCannotCallThis();
    		}
    	}
    }
    ```
    

### ObjectFactory vs ObjectProvider

ObjectFactory<interface>

- 예전 1.0.2 에 만들어짐
- `T getObject()` 메서드 하나만을 제공

ObjectProvider<interface>

- ObjectFactory를 상속
- Iterable을 상속함
- `T getIfAvailble(Supplier<T> defaultSupplier)` 등의 안전성을 더한 편의 메서드도 제공함

🤔 ObjectProvider(혹은 ObjectFactory) 는 어떻게 빈을 가져올까? ApplicationContext를 직접 사용할까?

- 정답
    
    
    BeanFactory(`ApplicationContext` 의 부모타입) 를 참조하여 `getBean()` 메서드를 호출한다.
    
    ![image](https://user-images.githubusercontent.com/37852769/207189489-324057b9-1ecc-4123-85f8-6b31c6951e30.png)
    

🤔 스프링 내부에서 언제 사용할까?

- 정답
    
    `DefaultListableBeanFactory` 의 `resolveDependency()` 메서드 내부에서 Provider를 사용하고 있다.
    
    ![image](https://user-images.githubusercontent.com/37852769/207189647-e790b39b-c76d-44d1-8b97-eedb6fbe6498.png)
    

## 방법3: JSR-330 Provider

`javax.inject.Provider` 라는 JSR-330자바 표준을 사용하는 방법이 있음

```
interface Provider<T> {
  T get();
}
```

### 어떤 방법이 더 좋을까?

🥊 편의 메서드를 더한 ObjectProvider vs 스프링에 의존하지 않은 javax Provider

실무에서는 대부분 싱글톤 빈으로 웹 애플리케이션의 도메인은 해결할 수 있다.

### Provider는 언제 사용하는게 좋을까?

- 같은 타입의 인스턴스를 여러개 생성하고 싶을 때
- 늦은 초기화로 인스턴스 생성
- 순환 참조 피해가기
- 스코프를 추상화하여 하위타입의 스코프의 인스턴스를 lookup 할 수 있음

> 실무에서는 주로 어떤것이 많이 사용될까?</br>
  실무에서는 스프링이 제공하는 Provider를 사용한다.</br>
  스프링이 사실상의 자바 웹 프레임워크의 표준(de facto)이다. </br>
  JSR 은 자바 표준이지만, 별도 라이브러리 의존이 필요하다.</br>
---
[스프링 핵심 원리 - 기본편, <김영한>](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8#curriculum)
