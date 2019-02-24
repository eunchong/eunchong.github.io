---
title:  "[디자인패턴] 의존성 주입 (Dependency Injection)"
search: true
categories: 
  - 디자인패턴
tags:
  - w3sdesign
  - 디자인패턴
  - design pattern
  - DI
  - Dependency Injection
classes: wide
---

* 이 포스팅은 w3sdesign의 [Dependency Injection](http://w3sdesign.com)을 정리한 것입니다.
* 내용에 대한 조언 및 의견은 작성자에게 큰 도움이 됩니다
* 모든 저작권과 권리는 w3sdesign에 있습니다. 


## 목적 (Intent)

의존성 주입(Dependency Injection) 디자인 패턴의 목적은 개체의 생성을 애플리케이션으로부터 분리하는 것이다. 다시 말해 의존성 주입은 애플리케이션에서 객체를 어떻게 생성하는지에 대한 책임을 분리하고 싶을 때 사용하게 된다.

## 문제 (problem)

코드를 디자인, 리팩토링, 테스트를 하다 보면, 다음과 같이 의존성 주입에 대한 필요성이 생길 수 있다.

- (디자인) 클래스에서 사용하는 외부 객체가 있을 때, 객체 생성에 대한 책임을 분리할 수 있을까?
- (디자인) 객체가 생성되는 방법을 분리된 설정 파일 또는 분리된 객체에 지정할 수 있을까?
- (리팩토링) 애플리케이션에 분산되어 있는 객체 생성을 중앙화하고 외부화할 수 있을까?
- (테스팅) 클래스가 사용하고 있는 외부 객체와 독립적으로 클래스를 유닛 테스트할 수 있을까?

## 해결방법 (Solution)

의존성 주입은 [제어권 역전(inversion of control)](https://en.wikipedia.org/wiki/Inversion_of_control)이라고도 불리는데, 아이디어의 핵심은 객체의 사용을 생성으로부터 분리하는 것이다. 인젝터는 객체를 생성하고 클래스에 주입한다. 클래스는 주입된 객체를 사용하고, 생성을 하지 않는다.

> 세 가지 step로 나눠서 보자. 이 단계를 통해 구현, 변경, 테스트, 재사용이 쉬운 심플한 코드가 될 수 있다.
> 1. 객체가 생성되는 방법을 분리된 설정 파일 또는 분리된 객체에 지정한다.
> 2. 인젝터가 객체를 삽입할 수 있도록 클래스는 객체를 받기 위한 생성자 또는 setter 메서드를 제공한다.
> 3. 클래스는 인터페이스를 통해 주입된 객체를 사용한다.

## 예제 코드 (Sample Code)

다음은 오픈 소스 Google Guice Injector를 사용하는 자바 코드이다.
{% highlight java %}
     1  package com.sample.di.basic;
     2  import com.google.inject.Guice;
     3  import com.google.inject.Injector;
     4  public class MyApp { 
     5      public static void main(String[] args) { 
     6          // Configuration1를 사용하는 인젝터 객체를 요청한다. (1 step)
     7          Injector injector = Guice.createInjector(new Configuration1());
     8          // 인젝터에게 Client 객체를 요청한다.
     9          Client client = injector.getInstance(Client.class);
    10          // client의 operation을 실행한다.
    11          System.out.println(client.operation());
    12      } 
    13  }
{% endhighlight %}
Client : Accepting objects from the injector.  
Hello World from ServiceA1 and ServiceB1!
{: .notice}
{% highlight java %}
     1  package com.sample.di.basic;
     2  import com.google.inject.Inject;
     3  public class Client { 
     4      private ServiceA serviceA;
     5      private ServiceB serviceB;
     6      
     7      @Inject    // 생성자를 통한 인젝션 방법을 사용한다. (2 step)
     8      public Client(ServiceA serviceA, ServiceB serviceB) { 
     9          System.out.println("Client : Accepting objects from the injector.");
    10          this.serviceA = serviceA;
    11          this.serviceB = serviceB;
    12      } 
    13      public String operation() { 
    14          // 전달 받은 객체(serviceA, serviceB)를 다음과 같이 사용할 수 있다. (3 step)
    15          return "Hello World from " + serviceA.getName() + " and " 
    16                  + serviceB.getName() + "!";
    17      } 
    18  }
    
     1  package com.sample.di.basic;
     2  public interface ServiceA { 
     3      String getName();
     4  } 
    
     1  package com.sample.di.basic;
     2  public class ServiceA1 implements ServiceA { 
     3      public String getName() { 
     4          return "ServiceA1";
     5      } ;
     6  } 
    
     1  package com.sample.di.basic;
     2  public interface ServiceB { 
     3      String getName();
     4  } 
    
     1  package com.sample.di.basic;
     2  public class ServiceB1 implements ServiceB { 
     3      public String getName() { 
     4          return "ServiceB1";
     5      } ;
     6  } 
    
     1  package com.sample.di.basic;
     2  import com.google.inject.*;
     3  public class Configuration1 extends AbstractModule { 
     4      @Override
     5      protected void configure() { 
     6          // 인터페이스에 인스턴스를 바인딩 한다.
     7          bind(ServiceA.class).to(ServiceA1.class);
     8          bind(ServiceB.class).to(ServiceB1.class);
     9      } 
    10  }
{% endhighlight %}
위의 예제 코드는 먼저(1 step)로 분리된 설정 파일을 통해 객체를 지정했다. 다음으로(2 step) Client는 생성자를 통해 ServiceA, B를 주입받았다. 그리고(3 step) Client의 operation은 주입된 서비스를 사용한다. 
{: .notice}
{% highlight java %}
     1  package com.sample.di.basic;
     2  import com.google.inject.Guice;
     3  import com.google.inject.Injector;
     4  import junit.framework.TestCase;
     5  public class ClientTest extends TestCase { 
     6      // ConfigurationMock를 사용하는 인젝터 객체를 요청한다. (1 step)
     7      Injector injector = Guice.createInjector(new ConfigurationMock());
     8      // 인젝터에게 Client 객체를 요청한다. 
     9      Client client = injector.getInstance(Client.class);
    10      
    11      public void testOperation() { 
    12          assertEquals("Hello World from ServiceAMock and ServiceBMock!", 
    13                  client.operation());        
    14      } 
    15      // More tests ...
    16  } 
    
     1  package com.sample.di.basic;
     2  public class ServiceAMock implements ServiceA { 
     3      public String getName() { 
     4          return "ServiceAMock";
     5      } 
     6  } 
    
     1  package com.sample.di.basic;
     2  public class ServiceBMock implements ServiceB { 
     3      public String getName() { 
     4          return "ServiceBMock";
     5      } 
     6  } 
    
     1  package com.sample.di.basic;
     2  import com.google.inject.*;
     3  public class ConfigurationMock extends AbstractModule { 
     4      @Override
     5      protected void configure() { 
     6          // 인터페이스에 인스턴스를 바인딩 한다.
     7          bind(ServiceA.class).to(ServiceAMock.class);
     8          bind(ServiceB.class).to(ServiceBMock.class);
     9      } 
    10  }
{% endhighlight %}
다음은 UnitTest를 위해 ClientTest 클래스를 만들었다. 1 step을 보면, 앞에서의 `new Configuration1()`가 `new ConfigurationMock()`로 변경된 것을 볼 수 있다. ConfigurationMock 설정을 통해 Injector는 Test 용 Mock Service를 생성했다. Client의 2, 3 step은 전혀 변경 없이 MockServiceA, B가 Client 클래스에 주입(inject) 되는 것을 볼 수 있다. 이를 통해 Client 클래스의 코드를 변경 없이, 의존되는 service를 외부 설정에서 변경하여 unit test를 수행할 수 있게 된다. 
{: .notice}

## 결론 (Conclusion)

의존성 주입은 클래스에서 사용하는 서비스 객체의 생성에 대한 책임을 외부로 분리한다. 앞으로 서비스 객체를 생성하고 관리하는 방법이 고민된다면, mock Service를 활용해 unit test 하는 구조를 만들고 싶다면 의존성 주입 패턴을 활용할 수 있을 것이다.

## 향후 (Future Work)

예제 코드에서 구글의 Guice를 활용하여 쉽고 간단하게 의존성 주입을 할 수 있었다. 향후 기회가 된다면 구글의 [Guice](https://github.com/google/guice)이 내부적으로 어떻게 객체를 관리([binding](https://github.com/google/guice/wiki/Bindings), [scope](https://github.com/google/guice/wiki/Scopes), [injection](https://github.com/google/guice/wiki/Injections)) 하는지 자세히 알아보고자 한다.