# 10장 클래스

## 클래스 체계

### 클래스를 정의하는 표준 자바 관례

* 순차적으로 내려가는 추상화 단계
  1. 변수 목록
     - `public static` 정적 공개 상수
     - `private static` 정적 비공개 상수
     - `private` 비공개 인스턴스 변수 (공개 변수가 필요한 경우는 거의 없다.)
  2. 함수 목록
     - 생성자
     - 호출하는 함수 (public)
     - 호출되는 함수 (private)

### 캡슐화

* 변수와 유틸리티 함수는 가능한 `private`로 하는 것이 좋다.
  * 변수나 유틸리티 함수를 `protected`로 선언하거나 패키지 전체로 공개해 테스트 코드에 접근을 허용하게 할 수 있다.
* 그러나, `캡슐화`를 풀어주는 결정은 언제나 최호의 수단이다. 

## 클래스는 작아야 한다!

앞서 보았던 `함수` 장에서와 마찬가지로 `클래스`를 만들 때 가장 중요한 규칙은 **`작게`** 만드는 것이다.

그렇다면 얼마나 작아야 하는가?

클래스가 맡은 **`책임`** 을 센다.
- 함수에서는 물리적인 행 수로 크기를 측정했었다. - 20줄 이내 (3 ~ 5 줄이 최고)

EX) 충분히 작을까?

```java
public class SuperDashboard extends JFrame implements MetaDataUser {
    public Component getLastFocusedComponent()
    public void setLastFocused(Component lastFocused)
    public int getMajorVersionNumber()
    public int getMinorVersionNumber()
    public int getBuildNumber() 
}
```

- 클래스가 맡은 **책임을 최소하하라!**
- **클래스 이름**은 해당 **클래스 책임을 기술**해야 한다.
  - 만약 간결한 이름이 떠오르지 않는다면 클래스 책임이 너무 많다는 것이다.
  - 함수에서 처럼 불용어(Processor, Manager, Super) 등과 같은 모호한 단어는 클래스가 너무 많은 책임을 떠안았다는 증거다.
- 클래스 설명은 if, and, or, but을 사용하지 않고서 25단어 내외로 가능해야 한다.
- 메서드 수가 작다고 작은 함수가 아니다.

### 단일 책임 원칙 (SRP: Single Responsibility Principle)

`단일 책임 원칙 (SRP)` : ***클래스나 모듈을 변경할 이유는 단 한가지여야 한다는 원칙이다.***

`책임` : **변경할 이유**

***만능 클래스를 단일 책임 클래스 여러개로 나누는 것이 바람직하다.***

- 소프트웨어를 *돌아가게* 만드는 활동과 소프트웨어를 *깨끗하게* 만드는 활동은 완전히 다르다.
- 우리는 처음에 시스템을 만들 때 소프트웨어를 *돌아가게* 만드는 활동에 집중을 한다. 바람직한 행동이다. 하지만, 여기서 대부분이 *돌아가게만* 만들고 *깨끗하고 체계적인 소프트웨어*를 만드는 다음 관심사로 넘어가지 않고 끝내버린다.

깨끗하고 체계적인 소프트웨어를 만드려면
* 큰 클래스가 아니라 작은 클래스 여럿으로 이루어진 시스템이 더 바람직하다.
* 작은 클래스는 책임이 하나여야 하며(변경할 이유가 하나), 다른 작은 클래스와 협력해 시스템에 필요한 동작을 수행한다.

### 응집도(Cohesion)

* 클래스는 **인스턴스 변수 수가 작아야 한다.** 
* 각 **클래스 메서드**는 **인스턴스 변수를 하나 이상 사용**해야 한다. 
* 일반적으로 메서드가 인스턴스 변수를 많이 사용할수록 메서드와 클래스의 응집도는 더욱 높아진다.

응집도가 높다 : 클래스에 속한 메서드와 변수가 서로 의존하며 논리적인 단위로 묶인다는 의미 -> ***함수를 작게, 매개변수 목록을 짧게***

`함수를 작게`, `매개변수 목록을 짧게` 라는 전략을 따르다 보면
- 몇몇 메서드에서만 사용하는 인스턴스 변수들이 있다.
- 이것들을 따로 새로운 클래스로 만들어주는 것이 바람직하다.
- **응집도가 높아지도록 변수와 메소드를 적절히 분리해 새로운 클래스로 쪼개야 한다.**

### 응집도를 유지하면 작은 클래스 여럿이 나온다.

***큰 함수를 작은 함수 여럿으로 나누기만 해도 클래스 수가 많아진다.***

EX)

변수가 많은 큰 함수 하나가 있다. 큰 함수를 작은 함수로 쪼개려 하는데, 작은 함수는 변수 4개가 필요하다. 이때 매겨변수로 변수 4개를 넘기는 것이 아닌, 4개의 변수를 인스턴스 변수로 승격시켜준다. 그러면 작은 함수는 매개변수가 필요없게 되고 **그만큼 쪼개기 쉬워진다.**

하지만, 이렇게 하면 클래스는 인스턴스 변수 목록이 많아져서 응집도를 잃게 된다. 그러나, 몇몇 함수가 몇몇 변수만 사용하므로 다른 클래스로 쪼개도 된다!

**`클래스가 응집력을 잃는다면 쪼개라!`**

## 변경하기 쉬운 클래스

대부분의 시스템들은 지속적인 변경이 가해진다. 

* 변경할 때마다 시스템이 의도대로 동작하지 않을 위험이 따른다.
* 깨끗한 시스템은 클래스를 체계적으로 정리해 변경에 수반되는 위험을 줄인다.

EX) 주어진 메타 자료로 적절한 SQL 문자열을 만드는 Sql 클래스, 아래 클래스를 보고 무슨 생각이 드는가? 잘 만들어진 클래스인가?

```java
BAD

public class Sql {
    public Sql(String table, Column[] columns)
    public String create()
    public String insert(Object[] fields)
    public String selectAll()
    public String findByKey(String keyColumn, String keyValue)
    public String select(Column column, String pattern)
    public String select(Criteria criteria)
    public String preparedInsert()
    private String columnList(Column[] columns)
    private String valuesList(Object[] fields, final Column[] columns) 
    private String selectWithCriteria(String criteria)
    private String placeholderList(Column[] columns)
}
```

- 아직 미완성이어서 update 문과 같은 일부 Sql 기능을 지원하지 않는다. 
- 새로운 SQl 문을 지원하려면 반드시 Sql 클래스에 손대야 한다. 

`변경할 이유(책임)이 2가지` 이므로 **SRP를 위반**한다.

EX) Sql 클래스를 Refactoring

```java
abstract public class Sql {
	public Sql(String table, Column[] columns) 
	abstract public String generate();
}
public class CreateSql extends Sql {
	public CreateSql(String table, Column[] columns) 
	@Override 
  	public String generate()
}

public class SelectSql extends Sql {
	public SelectSql(String table, Column[] columns) 
	@Override 
  	public String generate()
}

public class InsertSql extends Sql {
	public InsertSql(String table, Column[] columns, Object[] fields) 
	@Override 
 	public String generate()
	private String valuesList(Object[] fields, final Column[] columns)
}

public class SelectWithCriteriaSql extends Sql { 
	public SelectWithCriteriaSql(String table, Column[] columns, Criteria criteria) 
	@Override 
  	public String generate()
}

public class SelectWithMatchSql extends Sql { 
	public SelectWithMatchSql(String table, Column[] columns, Column column, String pattern) 
	@Override 
	public String generate()
}

public class FindByKeySql extends Sql {
  	public findByKeySql(String table, Column[] columns, String keyColumn, String keyValue) 
	@Override 
  	public String generate()
}

public class PreparedInsertSql extends Sql {
	public PreparedInsertSql(String table, Column[] columns) 
	@Override 
  	public String generate() 
	private String placeholderList(Column[] columns)
}

public class Where {
	public Where(String criteria) 
  	public String generate()
}

public class ColumnList {
	public ColumnList(Column[] columns) 
  	public String generate()
}
```

- public 메서드(공개 인터페이스)를 각각 Sql 클래스에서 파생하는 클래스로 만들었다.
- private 메서드는 각각에 해당하는 파생 클래스로 옮겼다.
- 모든 파생 클래스가 공통으로 사용하는 private 메서드는 Where, ColumnList라는 두 유틸리티 클래스에 넣었다.
- 개인적인 생각으로 *TEMPLATE METHOD 패턴*을 사용한 것 같다.

장점:
 
* 클래스가 다 분리 되었기 때문에 단순해졌고 코드 이해도도 높아졌으며, 함수 하나를 수정했을 때 다른 함수에 전혀 영향을 미치지 않는다.
* 테스트 관점에서도 모든 논리를 구석구석 테스트 하기도 쉬워졌다.
* 새로운 기능을 추가할 때 새로운 클래스를 추가하면 된다.

***결과적으로 SRP와 OCP를 지원하게 되었다.***

### 변경으로부터 격리

요구사항은 변하기 마련이다. 따라서, 코드도 변하기 마련이다. 

객체 지향 프로그래밍 입문에서 우리는 구체적인 클래스(concrete class)와 추상 클래스(abstract class)가 있다고 배웠다. 구체적인 클래스는 상세한 구현(코드)를 포함하며, 추상 클래스는 개념만 포함한다고 배웠다.

상세한 구현에 의존하는 클라이언트 클래스는 구현이 바뀌면 위험해진다. 그래서 우리는 **인터페이스와 추상 클래스를 사용해** 구현에 미치는 영향을 최소화한다.

EX) 외부 API를 사용할 때

***상세한 구현에 의존하는 코드는 테스트하기 어렵다!***

Portfolio 클래스는 외부 API인 TokyoStockExchange를 사용해 portfolio 값을 계산한다. 따라서 Test Code는 시세 변화에 영향을 받게 되고, 이러한 5분 마다 바뀌는 API로 Test code를 짜기란 쉽지 않다. 

* -> **외부 API를 사용하는 클래스는 외부 API를 `인터페이스`로 한번 감싸자!**

```java
// 1. 외부 API인 TokyoStockExchange 대신 StockExchange interface 구현
public interface StockExchange {
  Money currentPrice(String symbol);
}

// 2. StockExchange interface를 구현하는 TokyoStockExchange 클래스를 구현한다. 

// 3. portfolio 생성자를 수정해 interface StockExchange 참조자를 인수로 받는다.
public class Portfolio {
  private StockExchange stockExchange;
  public Portfolio(StockExchange stockExchange) {
    this.stockExchange = stockExchange;
  }
}
```

TokyoStockExchange 클래스를 흉내내는 테스트용 클래스를 만들 수 있다. 

```java
public class PortfolioTest {
	private FixedStockExchangeStub exchange;
	private Portfolio portfolio;

  //테스트용 클래스는 StockExchage 인터페이스를 구현하며 고정된 주가를 반환한다. 테스트에서 주식 5개를 구입하면 테스트용 클래스는 주가로 언제나 100불을 반환한다. 
  //일부러, 테스트용 클래스는 단순히 미리 정해놓은 표 값만을 참조하도록 한 것이다.
	@Before
	protected void setUp() throws Exception {
		exchange = new FixedStockExchangeStub();
		exchange.fix("MSFT", 100);
		portfolio = new Portfolio(exchange);
	}

	@Test
	public void GivenFiveMSFTTotalShouldBe500() throws Exception {
		portfolio.add(5, "MSFT");
		Assert.assertEquals(500, portfolio.value());
	}
}
```

- 위와 같은 테스트가 가능할 정도로 시스템의 결합도를 낮추면 유연성과 재사용성도 더욱 높아진다.
- 결합도가 낮다는 소리는 각 시스템 요소가 다른 요소로부터 그리고 변경으로부터 잘 격리되었다는 소리다.
- 시스템 요소가 서로 잘 격리되어 있으면 각 요소를 이해하기 더 쉬워진다.

***이렇게 결합도를 낮추면 DIP(의존 관계 역전 원칙)를 따르는 클래스가 나온다.***

* 본질적으로 DIP는 클래스가 상세한 구현이 아니라 추상화에 의존해야 한다는 원칙이다.

## 유틸리티 클래스, 유틸리티 메서드

- 인스턴스 메서드와 인스터스 변수를 일절 제공하지 않고, 정적 메서드와 변수만을 제공하는 클래스를 뜻한다.
- 클래스 본래의 목적인 **데이터와 데이터 처리를 위한 로직의 캡슐화**를 실행하는 것이 아닌,
**비슷한 기능의 메서드와 상수를 모아서 캡슐화**한 것이 유틸리티 클래스이다.

정적 메소드를 참조 할 때 해당 정적 메소드에 대한 엄격한 종속성을 작성합니다. 때때로 이것은 괜찮습니다. 때로는 그렇지 않습니다. 소프트웨어 개발의 핵심 원칙은 재사용입니다. 다른 상황에서도 정적 함수를 호출하는 코드를 재사용 할 수 있습니까? 아니면 정적 함수가 상황에 따라 다릅니까?

프로그래밍의 또 다른 핵심 원칙은 의존성 주입입니다. 고객 포맷터가 필요한 Class A가 있다고 가정 해보겠습니다. 당신이 할 수있는 것은 interface CustomerFormatter를 만들 수 있다는 것입니다. Class A에서 생성자를 정의하십시오 : A(CustomerFormatter formatter). 이를 통해 다른 Class A 구현을 사용하면서 CustomerFormatter를 재사용 할 수 있습니다. 이것은 Class A의 재사용 성을 크게 향상시킵니다. 정적 함수를 사용하면 더 이상 불가능합니다.

일반적인 일반 작업 (예 : Math, min(), max() 또는 ceil()에 정의 된 함수)은 정적 함수 여야합니다. 그들은 일반적으로 사용되는 한 가지 일만합니다. 일반적으로 정적 메소드가 완벽하게 보장되는 경우의 예로 이러한 기능을 사용할 수 있습니다.
당신의 기능은 순수한 기능입니까? 이것은 고유한 입력에 대해 항상 정확히 동일한 출력을 제공하는 기능입니다. 동일한 함수에 대한 후속 호출은 입력이 동일하게 유지되면 항상 동일한 결과를 산출합니다. 이 경우 메서드를 정적으로 만드는 것이 좋습니다. 그렇지 않은 경우, 동일한 함수에 대한 후속 호출이 임의의 임의 상태 또는 이전에 수행 된 호출에 따라 다른 출력을 제공하는 경우 정적이 아닌 것으로 간주해야합니다. 어떤 상태에 의존하기 때문에 동일한 입력에 대해 다른 출력을 제공하는 정적 함수를 만드는 것은 일반적으로 반 패턴으로 간주됩니다.

참고 : https://morningcoding.tistory.com/entry/Java15-%ED%81%B4%EB%9E%98%EC%8A%A4-%EB%B3%80%EC%88%98-%ED%81%B4%EB%9E%98%EC%8A%A4-%EB%A9%94%EC%84%9C%EB%93%9C%EC%99%80-%EC%9C%A0%ED%8B%B8%EB%A6%AC%ED%8B%B0-%ED%81%B4%EB%9E%98%EC%8A%A4 , https://pythonq.com/so/java/1756148



