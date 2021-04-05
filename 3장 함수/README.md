# 3장 함수 - `함수 잘 만드는 법`

어떤 프로그램이든 가장 기본적인 단위가 `함수`다.

함수를 읽기 쉽고 이해하기 쉽게 만드려면 어떻게 해야 할까? 

의도를 분명히 표현하는 함수를 어떻게 구현할 수 있을까?

함수에 어떤 속성을 부여해야 처음 읽는 사람이 프로그램 내부를 직관적으로 파악할 수 있을까?

이번 장에서는 이름 그대로 **`함수를 잘 만드는 법`** 을 소개한다.

## 함수 잘 만드는 법 - 1. 작게 만들어라!

함수를 만들 때 최대한으로 **`작게`** 만들자!

```java
HtmlUtil.java

public static String renderPageWithSetupsAndTeardowns(PageData pageData, boolean isSuite) throws Exception {
	boolean isTestPage = pageData.hasAttribute("Test"); 
	if (isTestPage) {
		WikiPage testPage = pageData.getWikiPage(); 
		StringBuffer newPageContent = new StringBuffer(); 
		includeSetupPages(testPage, newPageContent, isSuite); 
		newPageContent.append(pageData.getContent()); 
		includeTeardownPages(testPage, newPageContent, isSuite); 
		pageData.setContent(newPageContent.toString());
	}
	return pageData.getHtml(); 
} 
```

* 위의 코드도 길다. 더 짧게 만들자!

```java
public static String renderPageWithSetupsAndTeardowns(PageData pageData, boolean isSuite) throws Exception { 
   if (isTestPage(pageData)) 
   	includeSetupAndTeardownPages(pageData, isSuite); 
   return pageData.getHtml();
}
```

**블록과 들여쓰기**

* if문/else 문/while 문 등에 들어가는 블록은 `한 줄`이어야 한다. 보통 거기서 함수를 호출한다.
* 중첩구조가 생길 만큼 함수가 커져서는 안된다.
* 함수에서 들여쓰기 수준은 1단이나 2단을 넘어서는 안된다.



## 함수  잘 만드는 법 - 2. 한 가지만 해라


**함수는 한 가지를 해야 한다. 그 한 가지를 잘 해야 한다. 그 한 가지만을 해야 한다.**

**지정된 함수 이름 아래에서 추상화 수준이 하나인 단계만 수행한다면 그 함수는 한가지 작업만 한다.**

* TO 문단으로 기술해보자. (~하기 위해서)

ex) TO renderPageWithSetupsAndTeardowns -> 페이지가 테스트 페이지인지 확인한 후 테스트 페이지라면 설정 페이지와 해제 페이지를 넣는다. 테스트 페이지든 아니든 페이지를 HTML로 렌더링한다.

* 단순히 다른 표현이 아니라 의미 있는 이름으로 다른 함수를 추출할 수 있다면 그 함수는 여러 작업을 하는 셈이다.

함수 내 섹션

* 한 함수에서 여러 섹션으로 나뉜다면 여러 작업을 하는 함수라는 뜻이다.


## 함수 잘 만드는 법 - 3. 함수 당 추상화 수준은 하나로

**함수가 확실히 *한 가지* 작업만 하려면 함수 내 모든 문장의 추상화 수준이 동일해야 한다.**

함수의 추상화 수준 - 특정 표현이 1. 근본 개념인지 2. 세부 사항인지

**위에서 아래로 코드 읽기: `내려가기` 규칙**

* 한 함수 다음에는 추상화 수준이 한단계 낮은 함수가 온다.
* -> 위에서 아래로 프로그램을 읽으면 함수 추상화 수준이 한 단계씩 낮아진다.
* --> 일련의 TO 문단을 읽듯이 프로그램이 읽혀야 한다는 의미.

## 함수 잘 만드는 법 - 4. Switch 문

Switch 문, if/else가 여럿이 이어지는 문 은 작게 만들기 어렵다. 본질적으로 N가지 일을 처리한다.

```java
public Money calculatePay(Employee e) throws InvalidEmployeeType {
    switch (e.type) { 
		case COMMISSIONED:
			return calculateCommissionedPay(e); 
		case HOURLY:
			return calculateHourlyPay(e); 
		case SALARIED:
			return calculateSalariedPay(e); 
		default:
			throw new InvalidEmployeeType(e.type); 
	}
}
```

* 각 Switch 문을 저차원 클래스에 숨기고 절대로 반복하지 않는다. -> `다형성(Polymorohism)`을 사용
* -> Switch 문을 `추상 팩토리(Abstract Factory)`에 숨겨서 `다형적 객체를 생성하는 코드` 안에서만 switch를 사용하도록 한다.

* 추상 팩토리 패턴(Abstract Factory Pattern) : 상세화된 서브클래스를 정의하지 않고도 서로 관련성이 있거나 독립적인 여러 객체의 군을 생성하기 위한 인터페이스를 제공합니다.

```java
public abstract class Employee {
	public abstract boolean isPayday();
	public abstract Money calculatePay();
	public abstract void deliverPay(Money pay);
}
-----------------
public interface EmployeeFactory {
	public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType; 
}
-----------------
public class EmployeeFactoryImpl implements EmployeeFactory {
	public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
		switch (r.type) {
			case COMMISSIONED:
				return new CommissionedEmployee(r) ;
			case HOURLY:
				return new HourlyEmployee(r);
			case SALARIED:
				return new SalariedEmploye(r);
			default:
				throw new InvalidEmployeeType(r.type);
		} 
	}
}
```

* 하지만 Switch 문을 불가피하게 써야 하는 상황이 있으므로, 상황에 따라서는 사용할 수도 있다.

## 함수 잘 만드는 법 - 5. 서술적인 이름을 사용하라

일관적이고 서술적인 함수 이름을 사용하여 함수가 수행하는 기능을 그대로 짐작 가능하게 만들어야 한다. 이름이 길어도 괜찮다.

ex) includeSetUpandTearDownPages, includeSetUpPages, includeSuiteSetUpPage

## 함수 인수

함수 인수는 적을 수록 좋다!

* 최선은 입력 인수가 없는 경우(0개 (무항)), 차선은 입력 인수가 1개(단항)뿐인 경우이다.
* 출력인수는 함수의 반환 값이 아닌 입력 인수로 결과를 받는 경우라면, 이해하기 어려우므로 왠만하면 쓰지 않는 것이 좋다.

### 많이 쓰는 단항 형식



