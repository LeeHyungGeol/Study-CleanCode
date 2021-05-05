# 9장 단위 테스트

애자일과 TDD(Test Driven Development) 덕분에 단위 테스트를 자동화하는 프로그래머들이 점점 많아졌으며 더 늘어나는 추세이며 이는 아주 눈부신 성장이다.

하지만, 테스트를 추가하는데만 급급 하다보니 정작 좀 더 미요하고 중요한 사실을 놓쳐버렸다.

이 장에서는 **테스트 케이스를 잘 작성하는 방법**에 대해 설명해주고 있다.

## TDD 법칙 3가지

`TDD`가 *실제 코드*를 짜기 전에 **단위 테스트**부터 짜라고 요구한다는 사실은 누구나 알고 있지만, 그 규칙은 극히 일부에 불과하다.

1. `실패하는 단위 테스트를 작성할 때까지 실제 코드르 작성하지 않는다.`
2. `컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성한다.`
3. `현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성한다.`

## 깨끗한 테스트 코드 유지하기

이 책에서는 사례로 ***지저분해도 빨리*** 테스트 코드를 작성한 팀의 사례를 보여주며 ***`깨끗한 테스트 코드를 유지하는 것의 중요성`***을 보여준다.

* '지저분해도 빨리' 작성한 테스트 코드는 결국 오히려 테스트를 안하느니만 못한 결과를 내놓는다.
* 테스트 코드는 **유연성, 유지보수성, 재사용성**을 제공한다.
* 테스트 케이스가 있으면 잠정적인 버그 및 결함이 없을 것이라는 확신 하에 코드 변경이 쉬워진다.

따라서, **`테스트 코드는 실제 코드 못지 않게 중요하다.`** 깨끗한 테스트 코드만이 성공한다.

## 깨끗한 테스트 코드

깨끗한 테스트 코드를 만드려면 어떻게 해야 하는가?

* -> **가독성, 가독성, 가독성!!!!**

그렇다면 테스트 코드에 가독성을 높이려면 어떻게 해야하는가?

* -> 명료성, 단순성, 풍부한 표현력이 필요하다.

```java
BAD 
// 리팩토링 전
// 전체적으로 독자를 고려하지 않은 코드이다. 테스트와 상관없는 코드들을 독자들이 이해해야 전체 코드를 이해할 수 있다.
public void testGetPageHieratchyAsXmlDoesntContainSymbolicLinks() throws Exception {
    // 테스트와 무관한 코드, 테스트 코드의 의도만 흐린다.
	WikiPage pageOne = crawler.addPage(root, PathParser.parse("PageOne"));
	crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
	crawler.addPage(root, PathParser.parse("PageTwo"));

	PageData data = pageOne.getData();
	WikiPageProperties properties = data.getProperties();
	WikiPageProperty symLinks = properties.set(SymbolicPage.PROPERTY_NAME);
	symLinks.set("SymPage", "PageTwo");
	pageOne.commit(data);

    // 이 코드 역시 테스트 코드와 무관한 잡음에 불과하다.
    // resource와 인수에서 요청 URL을 만드는 어설픈 코드도 보인다.
	request.setResource("root");
	request.addInput("type", "pages");
	Responder responder = new SerializedPageResponder();
	SimpleResponse response =
		(SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
	String xml = response.getContent();

	assertEquals("text/xml", response.getContentType());
	assertSubString("<name>PageOne</name>", xml);
	assertSubString("<name>PageTwo</name>", xml);
	assertSubString("<name>ChildOne</name>", xml);
	assertNotSubString("SymPage", xml);
}

GOOD
// 리팩토링 후
// BUILD-OPERATE-CHECK 패턴
public void testSymbolicLinksAreNotInXmlPageHierarchy() throws Exception {
    // 1. 테스트 자료를 만든다.
	WikiPage page = makePage("PageOne");
	makePages("PageOne.ChildOne", "PageTwo");

    // 2. 테스트 자료를 조작한다.
	addLinkTo(page, "PageTwo", "SymPage");

	submitRequest("root", "type:pages");

    // 3. 조작한 결과가 올바른지 확인한다.
	assertResponseIsXML();
	assertResponseContains(
		"<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>");
	assertResponseDoesNotContain("SymPage");
}
```

리팩토링 전
* 전체적으로 독자를 고려하지 않은 코드이다. 테스트와 상관없는 코드들을 독자들이 이해해야 전체 코드를 이해할 수 있다.
* 테스트와 무관한 코드가 많다. 테스트 코드의 의도만 흐린다.
* resource와 인수에서 요청 URL을 만드는 어설픈 코드도 보인다.

리팩토링 후
* ***`BUILD-OPERATE-CHECK 패턴`*** 사용(위와 같은 테스트 구조에 적합하다)
   1. 테스트 자료를 만든다.
   2. 테스트 자료를 조작한다.
   3. 조작한 결과가 올바른지 확인한다.

잡다하고 세세한 코드들을 진짜 필요한 자료 유형과 함수로 묶어 처리했으므로, 코드를 읽는 독자는 테스트 코드가 수행하는 기능을 빠르게 정확하게 알아챌 수 있다.

### 도메인에 특화된 테스트 언어

위에서의 리팩토링 후 코드는 ***`도메인에 특화된 언어(DSL(Domain Specified Language))`***로 테스트 코드를 구현하는 기법을 보여준다.

* API 위에다 함수와 유틸리티를 구현 한 후, 그 함수와 유틸리티를 사용하므로 테스트 코드를 짜기도 읽기도 쉬워진다.
* 이렇게 구현한 함수와 유틸리티는 테스트 코드에서 사용하는 **특수한 API**가 된다.
* 즉, 테스트 코드를 도와주는 **테스트 특화 언어**가 되는 것이다.

### 이중 표준

테스트 API에 적용하는 표준은 실제 코드에 대한 표준과 **당연히 다르다.**
* 단순, 간결, 표현력이 풍부해야 하지만, ***실제 코드만큼 효율적일 필요는 없다.***
* 실제 환경이 아니라 테스트 환경에서 돌아가는 코드이고, 그 각각의 환경의 요구사항은 판이하게 다르다.
  
```java
// 임베디드 시스템에서 사용하는 테스트 코드에 필요한 getState() 메서드
public String getState() {
  String state = "";
  state += heater ? "H" : "h"; 
  state += blower ? "B" : "b"; 
  state += cooler ? "C" : "c"; 
  state += hiTempAlarm ? "H" : "h"; 
  state += loTempAlarm ? "L" : "l"; 
  return state;
}
```

효율을 높이려면 StringBuffer가 더 적합하다. 하지만, StringBuffer는 보기에 흉하다는 이유로 저자는 테스트 코드에 사용하지 않았다. 심지어, 이 코드를 사용하는 애플리케이션은 실시간 임베디드 시스템이다. 컴퓨터 자원과 메모리가 제한적일 가능성이 높다. ***하지만, 테스트 환경은 자원이 제한적일 가능성이 낮다. 이것이 이중 표준의 본질이다.***

* 실제 환경에서는 절대로 안되지만 테스트 환경에서는 전혀 문제 없는 방식이다. 
* **대개 메모리나 CPU 효율과 관련있는 경우다.** 코드의 깨끗함과는 무관하다.

## 테스트 당 assert 하나

```java
public void testGetPageHierarchyAsXml() throws Exception {
    //given 
	givenPages("PageOne", "PageOne.ChildOne", "PageTwo");
    //when
	whenRequestIsIssued("root", "type:pages");
    //then
	thenResponseShouldBeXML(); 
}

public void testGetPageHierarchyHasRightTags() throws Exception { 
	//given
    givenPages("PageOne", "PageOne.ChildOne", "PageTwo");
    //when
	whenRequestIsIssued("root", "type:pages");
    //then
	thenResponseShouldContain(
		"<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
	); 
}
```

JUnit으로 테스트 코드를 짤 때마다 메서드 하나당 assert문을 하나만 하자는 것이다.
* **`Given-When-Then 구조`**를 사용
* 가독성이 더욱 높아진다. 
* **하지만, 그만큼 중복성이 높아진다.**

따라서, 무조건적으로 테스트 하나당 assert문을 하나로 줄이는 것이 아닌 **assert문 개수를 최대한 줄이려고 노력하자.**

### 해결책은 Test당 개념 하나

```java
public void testAddMonths() {
    //(5월처럼) 31일로 끝나는 달의 마지막 날짜가 주어지는 경우
	SerialDate d1 = SerialDate.createInstance(31, 5, 2004);
	
	// (6월처럼) 30일로 끝나는 한 달을 더하면 날짜는 30일이 되어야지 31일이 되어서는 안된다.
	SerialDate d2 = SerialDate.addMonths(1, d1); 
	assertEquals(30, d2.getDayOfMonth()); 
	assertEquals(6, d2.getMonth()); 
	assertEquals(2004, d2.getYYYY());

	// 두 달을 더하면 그리고 두 번째 달이 31일로 끝나면 날짜는 31일이 되어야 한다.
	SerialDate d3 = SerialDate.addMonths(2, d1); 
	assertEquals(31, d3.getDayOfMonth()); 
	assertEquals(7, d3.getMonth()); 
	assertEquals(2004, d3.getYYYY());

	// 31일로 끝나는 한 달을 더하면 날짜는 30일이 되어야지 31일이 되어서는 안된다.
	SerialDate d4 = SerialDate.addMonths(1, SerialDate.addMonths(1, d1)); 
	assertEquals(30, d4.getDayOfMonth());
	assertEquals(7, d4.getMonth());
	assertEquals(2004, d4.getYYYY());
}   
```

이것저것 잡다한 개념을 연속적으로 테스트하는 긴 함수는 피한다.
* 메서드에 assert문이 여러개 있다는 사실이 문제가 아니다.
* 한 테스트에서 여러개의 개념을 한번에 테스트 하는 것이 문제이다.

가장 좋은 규칙은 **`개념 당 assert문 수를 최소한으로 줄여라`**, **`테스트 함수 하나는 개념 하나만 테스트하라`** 이다.

## F.I.R.S.T.

* **`Fast(빠르게)`** : 테스트는 빨리 돌아야 한다. 테스트가 느리면 자주 돌릴 엄두가 안난다. 자주 돌리지 못하면 초반에 문제를 찾아내지 못하게 되고, 곧 코드 품질이 망가지게 된다.
* **`Independent(독립적으로)`** : 각 테스트는 서로 의존하면 안된다. 한 테스트가 다음 테스트가 실행될 환경을 준비해서는 안된다. 각 테스트는 독립적으로 어떤 순서로 실행해도 괜찮아야 한다.
* **`Repeatable(반복가능하게)`** : 테스트는 어떤 환경에서도 반복 가능해야 한다. 실제 환경, QA 환경, 버스를 타고 집에 가는 길에 사용하는 노트북 환경(네트워크에 연결되지 않은)`**에서도 실행할 수 있어야 한다.  
* **`Self-Validating(자가 검증하는)`** : 테스트는 부울(Boolean) 값으로 결과를 내야 한다. 성공 아니면 실패다.
* **`Timely(적시에)`** : 테스트는 적시에 작성해야 한다. 단위 테스트는 실제 코드를 구현하기 직전에 구현해야 한다. 실제 코드를 구현한 다음에 테스트 코드를 만들면 실제 코드가 테스트하기 어렵다는 사실을 발견할지도 모른다. 

## 결론

깨끗한 테스트 코드는 실제 코드만큼이나 오히려 실제 코드보다 더 중요할 수 있다. 실제 코드의 **유연성, 재사용성, 유지보수성**을 보존하고 강화하기 때문이다. **테스트 API**를 구현해 **도메인 특화 언어(DSL)**를 만들어 테스트 코드를 지속적으로 깨끗하게 관리하고 표현력을 높이자.


## BUILD-OPERATE-CHECK 패턴

* Build(Given) : Input 데이터를 생성

* Operate(When) : Build 단계에서 생성한 데이터로 실제 코드를 생성 

* Check(Then) : Operate 단계의 결과를 확인

given-when-then 을 관례적으로 주석으로 사용한다.

EX)
```java
// Unit Test(단위 테스트)
class MemberServiceTest { //Ctrl + Shift + F10 - 이전에 수행했던 테스트 그대로 다시 수행

    MemberService memberService;

    MemoryMemberRepository memberRepository;

    @BeforeEach
    public void BeforeEach() {
        memberRepository = new MemoryMemberRepository();
        memberService = new MemberService(memberRepository);
    }

    @AfterEach
    public void afterEach() { //test 는 서로가 독립적이어야 한다. // 서로 순서와 상관이 없어야 하고 의존성이 없어야 한다.
        //test 가 하나 끝났을 때, data 를 clear 해주기
        //일종의 callback 메소드 // 다른 메소드의 동작이 하나 끝날 때마다 작동한다.
        memberRepository.clearStore();
    }

    @Test
    void 회원가입() { //테스트는 과감하게 한글로 바꿔서 사용 가능// 바로 바로 직관적으로 알아듣기 위해서
        //given  // Test Case 작성 방법 //권장한다. // given-when-then
        Member member = new Member();
        member.setName("Spring");
        //when
        Long saveId = memberService.join(member);
        //then
        Member findMember = memberService.findOne(saveId).get();
        assertThat(member.getName()).isEqualTo(findMember.getName());
    }

    @Test
    public void 중복_회원_예외() { //test 는 정상 플로우도 중요하지만
        // 더 중요한 것은 예외에 대한 처리가 제대로 되는지 테스트 하는 것이다.
        //given
        Member member1 = new Member();
        member1.setName("Spring");

        Member member2 = new Member();
        member2.setName("Spring");
        //when
        memberService.join(member1);
        IllegalStateException e = assertThrows(IllegalStateException.class,
                () -> memberService.join(member2));//예외가 발생해야 한다.

        //then
        assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");
    }
```

출처: https://alkhwa-113.tistory.com/entry/단위-테스트 [기(술) 블로그]

## 도메인 특화 언어-DSL(Domain Specified Language)

`DSL` : 특정 도메인에 특화된 언어. 특정 영역을 타겟하고 있는 언어를 말한다. 특정 영역의 문제 해결에는 그 영역에 맞는 특화된 도구를 사용하자라는 것이다.

EX) SQL : DB의 데이터를 참조하기 위해 날리는 query는 말 그대로 "DB에 데이터를 참조하기 위한 목적"으로만 사용되며 SQL로 웹 애플리케이션 서버를 만드는 것은 절대 불가능 하다.

EX2) JAVA :  SQL을 만들어 낼 수도 있고 (사실상 SQL은 특정한 문법을 가진 문자열이기 때문이다), 웹 애플리케이션 서버를 만들 수도 있고, 그 외 원하는 모든 것을 만들어 낼 수 있다. 단지 다른 분야에선 다른 언어가 더 좋을 뿐이지 가능은 할 것이다.



* 우리 주변에 있는 DSL
  - java 
    - ANT, Maven, struts-config.xml, Seasar2 S2DAO, HQL(Hibernate Query Language), JMock
  - Ruby
    - Rails Validations, Rails ActiveRecord, Rake, RSpec, Capistrano, Cucumber
  - 기타
    - SQL, CSS, Regular Expression(정규식), Make, graphviz

참고: https://lannstark.tistory.com/13 , https://unabated.tistory.com/entry/DSLDomain-Specific-Language-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0

## TEMPLATE-METHOD 패턴

- 알고리즘의 구조를 메소드에 정의하고, 하위 클래스에서 알고리즘 구조의 변경없이 알고리즘을 재정의 하는 패턴

- **상위 클래스**에 **공통 로직을 수행하는 템플릿 메서드**와 하위 클래스에 **오버라이딩을 강제하는 추상 메서드** 또는 **선택적 오버라이딩이 가능한 Hook 메소드**를 두는 패턴

* 전체적으로는 동일하면서 부분적으로는 다른 구문으로 구성된 메서드의 ***코드 중복을 최소화 할 때 유용하다.*** 

* 다른 관점에서 보면 동일한 기능을 상위 클래스에서 정의하면서, 확장/변화가 필요한 부분만 서브 클래스에서 구현할 수 있도록 한다. 

EX 1)
```java
public abstract class CaffeineBeverageWithHook {
    void final prepareRecipe() { // 템플릿 메서드(Template Method)
        boilWater();
        brew();
        pourInCup();
        if ( customerWantsCondiments() ) {
            addcondiments();
        }
    }

    abstract void brew(); // 추상 메서드(Abstract Method)

    abstract void addcondiments(); // 추상 메서드(Abstract Method)

    void boilWater() {
        System.out.println("물 끓이는 중");
    }

    void pourInCup() {
        System.out.println("컵에 따르는 중");
    }
    
    // 후크 메서드(Hook Method)
    boolean customerWantsCondiments() {    //이 메소드는 서브클래스에서 필요에 따라 오버라이드 할수 있는 메소드이므로 후크(Hook)이다.
    return true;                                   
    }
}

public class CoffeeWithHook extends CaffeineBeverageWithHook {
    @Override
    void brew() {
        System.out.println("필터를 통해 커피를 우려내는 중");
        }

    @Override
    public void addCondiments() {
        System.out.println("설탕과 우유를 추가하는 중");
    }
    
    @Override
    public boolean customerWantsCondiments() {
        String answer = getUserInput();

        if( answer.toLowerCase().startWith("y")) {
            return true;
        } else { 
            return false;
        }
    }
    
    private String getUserInput() {
        // 입력받는 로직
    }
}
```

참고: https://jusungpark.tistory.com/24 [정리정리정리], https://www.crocus.co.kr/1531 [Crocus]

