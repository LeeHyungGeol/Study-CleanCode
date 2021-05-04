# 8장 경계

프로그래머는 외부 코드(패키지, 오픈 소스, 다른 팀이 제공하는 컴포넌트 등)를 우리 코드에 깔끔하게 통합해야만 한다.

이 장에서는 그러한 소프트웨어 경계를 깔끔하게 처리하는 기법과 기교에 대해서 살펴본다.

## 외부 코드 사용하기

인터페이스 제공자(서버)와 인터페이스 사용자 사이에는 원하는 방향성에 있어서 차이가 있다.

* 인터페이스(패키지, 프레임워크) 제공자(서버) : 범용성, 적용성을 최대한 넓히려 한다.
* 사용자(클라이언트) : 자신의 요구에 맞는 인터페이스를 원한다.

또한, 코드는 결합도는 낮추고 응집도는 높혀야 한다.

java.util.Map을 보자. 프로그램에서 Map을 여기저기 넘긴다고 가정해보자. 넘기는 쪽에서는 아무도 Map 내용을 삭제하지 않을거라고 생각할 수 있지만, 실제로는 Map 사용자라면 누구나 Map.clear() 메서드를 사용가능하다. 또한, 설계시 Map에 특정 객체 유형만을 저장한다고 설계했다고 가정하자. 그렇지만 Map은 객체 유형을 제한하지 않는다. 

```java
import java.util.Map;

Map sensors = new HashMap();
Sensor s = (Sensor)sensors.get(sensorId);
```

* Object를 올바른 유형으로 변환할 책임은 Map을 사용하는 클라이언트에 있다. 
* 대신 제네릭스(Generics)를 사용하면 코드 가독성은 높아진다.

```java
// 제네릭스(Generics) 사용
Map<String, Sensor> sensors = new HashMap<Sensor>;
Sensor s = sensors.get(sensorId);
```

* 위 방법도 Map<String, Sensor>가 사용자에게 필요하지 않는 기능까지 제공한다는 문제를 해결하진 못한다.
* Map 인터페이스가 변할 경우, 수정할 코드도 상당히 많아진다.

```java
public class Sensors {
    private Map sensors = new HashMap();
    
    public Sensor getById(String id) {
        return (Sensor) sensors.get(id);
    }
}
```

* 경계 인터페이스인 Map을 Sensors class 안으로 숨긴다.
* Map 인터페이스가 변하더라도 나머지 프로그램에 영향을 미치지 않는다.
* Sensors class는 프로그램에 필요한 인터페이스만 제공한다.
* 코드는 이해하기 쉽지만 오용하기는 어렵다.

Map 클래스를 사용할 때마다 캡슐화를 하라는 것이 아니다!!! Map(혹은 유사한 경계 인터페이스)를 여기저기 넘기지 말라는 소리다. Map과 같은 경계 인터페이스를 이용할 때는 이를 이용하는 클래스나 클래스 계열 밖으로 노출되지 않도록 주의한다. 

## 경계 살피고 익히기

외부 코드를 익히기는 어렵다. 외부 코드를 사용하기도 어렵다. 

* -> 우리 코드를 작성해 외부 코드를 호출하는 대신 먼저 간단한 **Test Case를 작성해 외부 코드를 익혀보자.**
* 이것이 바로 `학습 테스트`이다.

학습 테스트는 프로그램에서 사용하려는 방식대로 외부 API를 호출하며, 통제된 환경에서 API를 제대로 이해하는지를 확인하는 셈이다.

### log4j 익히기 - 학습테스트 예시

상황) 로깅 기능을 직접 구현하는 대신 Apache의 log4j 패키지를 사용하려고 가정한다. 

1. 첫번째 테스트 케이스 작성
```java
@Test
public void testLogCreate(){
	Logger logger = Logger.getLogger("MyLogger");
	logger.info("hello");
}
```

* Appender 오류 발생

2. 두번째 테스트 케이스 작성
   * 문서를 읽어본 후, ConsoleAdapter라는 class가 있다는 사실을 확인
```java
@Test
public void testLogCreate(){
	Logger logger = Logger.getLogger("MyLogger");
	ConsoleAppender appender = new ConsoleAppender();
	logger.addAppender(appender);
	logger.info("hello");
}
```

* Appender에 출력 스트림이 없다는 사실을 발견

3. 세번째 테스트 케이스 작성
   * 구글링 후, 다시 시도
```java
@Test
public void testLogCreate(){
	Logger logger = Logger.getLogger("MyLogger");
	logger.removeAllAppenders();
	logger.addAppender(new ConsoleAppender(
				new PatternLayout("%p %t %m%n"),
				ConsoleAppender.SYSTEM_OUT));
	logger.info("hello");
}
```

* 정상 작동 확인
* 근데 여기서 문서와 구글링 후, log4j의 동작 방식에 대해 많이 이해하게 되었으며, 이것을 바탕으로 간단한 단위 테스트를 좀 더 추가해볼 수 있다.

```java
public class LogTest {
     private Logger logger;

     @Before
     public void initialize() {
         logger = Logger.getLogger("logger");
         logger.removeAllAppenders();
         Logger.getRootLogger().removeAllAppenders();
     }

     @Test
     public void basicLogger() {
         BasicConfigurator.configure();
         logger.info("basicLogger");
     }

     @Test
     public void addAppenderWithStream() {
         logger.addAppender(new ConsoleAppender(
             new PatternLayout("%p %t %m%n"),
             ConsoleAppender.SYSTEM_OUT));
         logger.info("addAppenderWithStream");
     }

     @Test
     public void addAppenderWithoutStream() {
         logger.addAppender(new ConsoleAppender(
             new PatternLayout("%p %t %m%n")));
         logger.info("addAppenderWithoutStream");
     }
 }
```

* 학습 테스트를 통해 얻은 지식을 바탕으로 log4j를 래핑(Wrappping)하는 독자적인 로거 클래스를 만들 수 있다.

### 학습테스트는 공짜 이상이다.

`학습 테스트`는 드는 비용은 없지만 필요한 지식만 확보할 수 있는 손쉬운 방법이다.

* 외부 패키지에 대한 **이해도**를 높여준다.
* 외부 패키지가 예상대로 도는지 검증한다.
* 외부 패키지의 새 버전이 나온다면 학습 테스트를 돌려 호환성을 확인할 수 있고, 새 버전으로 이전하기 쉬워진다.

## 아직 존재하지 않는 코드를 사용하기

경계의 또 다른 유형 : 아는 코드와 모르는 코드 사이의 경계

* **`우리가 원하는 코드를 interface로 작성하기`**
* **`ADAPTER 패턴 활용하기`**

예시 상황) 송신기(Transmitter) 라는 하위 시스템이 필요한데, 아직 개발되지 않은 상태에서 나머지 코드를 구현해보자.

* 우리가 송신기 모듈에서 원하는 기능 : 지정한 주파수(frequency)를 이용해 이 스트림(stream)에서 들어오는 자료를 아날로그 신호로 전송하라.

<img width="700" alt="송신기 예측하기" src="https://user-images.githubusercontent.com/56071088/116956942-7bcb5e80-acd1-11eb-8a5a-e62c6da21d1b.png">
 
* Transmitter interface를 정의, transmit 메서드 정의 : 우리가 원하는 인터페이스를 정의함으로써 인터페이스를 전적으로 통제
* Transmitter API 에서 Communication Controller를 깔끔하게 분리
* 추후에 추가되는 Transmitter API는 Transmitter Adapter를 정의해 ADAPTER Pattern을 사용하였다.
* Adapter 패턴으로 API 사용을 캡슐화해 API가 바뀔 때 수정할 코드를 한 곳으로 모았다.
* FakeTransmitter를 이용하여 Communication Controller 클래스를 test도 가능하다.

## 깨끗한 경계

경계에서는 흥미로운 일이 많이 벌어진다. 변경이 대표적인 예다. 소프트웨어 설계가 우수하다면 변경하는데 많은 투자와 재작업이 필요하지 않다. 경계에 위치하는 코드는 깔끔히 분리한다. 또한 기대치를 정의하는 테스트 케이스도 작성한다. 이쪽 코드에서 외부 패키지를 세세하게 알아야 할 필요가 없다. 통제가 불가능한 외부 패키지에 의존하는 대신 통제가 가능한 우리 코드에 의존하는 편이 훨씬 좋다. **자칫하면 오히려 외부 코드에 휘둘리고 만다.**

**외부 패키지를 호출하는 코드를 가능한 줄여서 경계를 관리하자.**

* Map에서 봤듯이, Wrapper class를 활용하여 캡슐화 한다.
* ADAPTER 패턴을 활용하여 우리가 원하는 인터페이스를 패키지가 제공하는 인터페이스로 변환하자.

## ADAPTER 패턴

* ***한 클래스의 인터페이스를 클라이언트에서 사용하고자 하는 인터페이스로 변환하는 패턴***
* 호환성이 없는 인터페이스 때문에 함께 동작할 수 없는 클래스들이 함께 작동하도록 해준다.

![adapter-pattern-1](https://user-images.githubusercontent.com/56071088/116960576-c94cc900-acdb-11eb-9be2-76b0b9a6286e.png)

`Client`

써드파티 라이브러리나 외부시스템을 사용하려는 쪽이다.

`Adaptee`

써드파티 라이브러리나 외부시스템을 의미한다.

`Target Interface`

Adapter 가 구현(implements) 하는 인터페이스이다. **클라이언트는 Target Interface 를 통해 Adaptee 인 써드파티 라이브러리를 사용하게 된다.**

`Adapter`

**Client 와 Adaptee 중간에서 호환성이 없는 둘을 연결시켜주는 역할을 담당한다.** Target Interface 를 구현하며, 클라이언트는 Target Interface 를 통해 어댑터에 요청을 보낸다. 어댑터는 클라이언트의 요청을 Adaptee 가 이해할 수 있는 방법으로 전달하고, 처리는 Adaptee 에서 이루어진다.

![adpter pattern 사용 예제](https://user-images.githubusercontent.com/56071088/116960706-2e082380-acdc-11eb-9d86-5ecfdae92c2b.png)

|사진 1|사진 2|code|
|---|---|---|
|Client|WebClient|AdapterDemo|
|Target Interface|WebRequester|WebRequester|
|Adapter|WebAdapter|WebAdapter|
|Adaptee|WebService|FancyRequester|

예시 code)
```java
// Adapter
public class WebClient {
    private WebRequester webRequester;

    public WebClient(WebRequester webRequester) {
        this.webRequester = webRequester;
    }

    public void doWork() {
        webRequester.requestHandler();
    }
}

// Target interface
public interface WebRequester {
    void requestHandler();
}

public class OldWebRequester implements WebRequester {
    @Override
    public void requestHandler() {
        System.out.println("OldWebRequester is working");
    }
}

// Adaptee
public class FancyRequester {
    public void fancyRequestHandler() {
        System.out.println("Yay! fancyRequestHandler is called!");
    }
}

public class WebAdapter implements WebRequester {
    private FancyRequester fancyRequester;

    public WebAdapter(FancyRequester fancyRequester) {
        this.fancyRequester = fancyRequester;
    }

    @Override
    public void requestHandler() {
        fancyRequester.fancyRequestHandler();
    }
}

// Client
public class AdapterDemo {
    public static void main(String[] args) {
        WebAdapter adapter = new WebAdapter(new FancyRequester());
        WebClient client = new WebClient(adapter);
        client.doWork();
    }
}
```

### 어댑터 패턴 정리
* Adaptee 를 감싸고, Target Interface 만을 클라이언트에게 드러낸다.
* Target Interface 를 구현하여 클라이언트가 예상하는 인터페이스가 되도록 Adaptee 의 인터페이스를 간접적으로 변경한다.
* Adaptee 가 기대하는 방식으로 클라이언트의 요청을 간접적으로 변경한다.
* 호환되지 않는 우리의 인터페이스와 Adaptee 를 함께 사용할 수 있다.


출처 : https://yaboong.github.io/design-pattern/2018/10/15/adapter-pattern/
