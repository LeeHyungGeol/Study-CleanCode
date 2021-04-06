# 3장 함수 - `함수 잘 만드는 법`

어떤 프로그램이든 가장 기본적인 단위가 `함수`다.

함수를 읽기 쉽고 이해하기 쉽게 만드려면 어떻게 해야 할까? 

의도를 분명히 표현하는 함수를 어떻게 구현할 수 있을까?

함수에 어떤 속성을 부여해야 처음 읽는 사람이 프로그램 내부를 직관적으로 파악할 수 있을까?

이번 장에서는 이름 그대로 **`함수를 잘 만드는 법`** 을 소개한다.

- [3장 함수 - `함수 잘 만드는 법`](#3장-함수---함수-잘-만드는-법)
  - [함수 잘 만드는 법 - 1. 작게 만들어라!](#함수-잘-만드는-법---1-작게-만들어라)
  - [함수  잘 만드는 법 - 2. 한 가지만 해라](#함수--잘-만드는-법---2-한-가지만-해라)
  - [함수 잘 만드는 법 - 3. 함수 당 추상화 수준은 하나로](#함수-잘-만드는-법---3-함수-당-추상화-수준은-하나로)
  - [함수 잘 만드는 법 - 4. Switch 문](#함수-잘-만드는-법---4-switch-문)
  - [함수 잘 만드는 법 - 5. 서술적인 이름을 사용하라](#함수-잘-만드는-법---5-서술적인-이름을-사용하라)
  - [함수 잘 만드는 법 - 6. 함수 `인수`](#함수-잘-만드는-법---6-함수-인수)
    - [**많이 쓰는 단항 형식**](#많이-쓰는-단항-형식)
    - [**플래그 인수**](#플래그-인수)
    - [**이항 함수**](#이항-함수)
    - [**삼항 함수**](#삼항-함수)
    - [**인수 객체**](#인수-객체)
    - [**인수 목록**](#인수-목록)
    - [**동사와 키워드**](#동사와-키워드)
  - [함수 잘 만드는 법 - 7. 부수 효과를 일으키지 마라](#함수-잘-만드는-법---7-부수-효과를-일으키지-마라)
    - [출력 인수](#출력-인수)
  - [함수 잘 만드는 법 - 8. 명령과 조회를 분리하라](#함수-잘-만드는-법---8-명령과-조회를-분리하라)
  - [함수 잘 만드는 법 - 9. 오류 코드보다 예외를 사용하라](#함수-잘-만드는-법---9-오류-코드보다-예외를-사용하라)
  - [함수 잘 만드는 법 - 10. 반복하지 마라](#함수-잘-만드는-법---10-반복하지-마라)

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

## 함수 잘 만드는 법 - 6. 함수 `인수`

함수 **`인수`** 는 적을 수록 좋다!

* 최선은 입력 인수가 없는 경우(0개 (무항)), 차선은 입력 인수가 1개(단항)뿐인 경우이다.
* 출력인수는 함수의 반환 값이 아닌 입력 인수로 결과를 받는 경우라면, 이해하기 어려우므로 왠만하면 쓰지 않는 것이 좋다.

### **많이 쓰는 단항 형식**

1. **`인수`** 에 `질문`을 던지는 경우

ex) `boolean fileExists("MyFile");`

2. **`인수`** 를 뭔가로 `변환`해 `결과를 반환`하는 경우

ex) `InputStream fileOpne("MyFile");`

3. 이벤트 함수인 경우 : 이벤트 함수는 입력 인수만 있다. 출력 인수를 없다.

ex) `passwordAttemptsNTimes(int attempts);`

이외의 경우라면 단항 함수는 가급적 피한다.

### **플래그 인수**

플래그(flag) 인수는 추하다. 함수로 부울(boolean) 값을 넘기는 관례는 끔찍하다.

함수가 대놓고 **여러 가지**를 처리한다고 광고하는 셈이기 때문이다.

### **이항 함수**

인수가 2개인 함수는 인수가 1개인 함수보다 이해하기 어렵다.

ex) `writeField(name); vs writeField(outputStream, name);`
* 둘 다 의미는 명백하지만 전자가 더 쉽고 이해도 더 빨리 된다.

ex2) `assertEquals(expected, actual);`

Point class 와 같은 경우는 이항 항수가 적절한 경우이다. Point class 는 오히려 인자가 1개이면 더 놀란다.

ex) `Point p = new Point(1, 1);`

이항 함수가 무조건 나쁘다는 것은 아니다. 불가피한 경우도 물론 있다. 위헝의 소지가 있으므로 가능하면 단항 함수로 바꾸도록 하자.

### **삼항 함수**

인수가 3개인 함수는 당연히 이해하기 더 어렵다. 위험도 2배 이상으로 많아진다. 삼항 함수를 만들 때는 더 신중하게 고려하라.

ex) `assertEquals(message, expected, actual);` 첫 인수가 expected 일 것이라고 예상하고 주춤하게 된다.

### **인수 객체**

인수가 2, 3개 필요하다면 일부를 독자적인 클래스로 만들어서 하나의 인수로 만들자.

ex) `Circle makeCircle(double x, double y, double radius);`

-> `Circle makeCircle(Point center, double radius);`

* 위 예시에서 x, y를 묶었듯이 변수를 묶어 넘기려면 `이름`을 붙여야 하므로 결국은 `개념`을 표현하게 된다.

### **인수 목록**

인수 개수가 가변적인 함수도 필요하다. String.format 메서드

ex) `String.format("%s worked %.2f" hours.", name, hours);`

-> `public String format(String format, Object... args);`

사실상 `이항 함수`다. 이와 같은 형식으로 단항, 이항, 삼항이 있을 수 있지만 이를 넘어서는 인수는 문제가 있다.

### **동사와 키워드**

단항 함수는 **함수와 인수**가 **동사/명사 쌍**을 이루어야 한다.

ex) writeField(name);

함수 이름에 **키워드**를 추가하는 형식이다.

ex) `assertEquals(expected, actual);` -> `assertExpectedEqualsActual(expected, actual);`

## 함수 잘 만드는 법 - 7. 부수 효과를 일으키지 마라

부수 효과는 거짓말이다. 함수에서 한 가지를 하겠다고 약속하고선 **몰래** 다른 짓도 한다는 뜻이다. ***함수에서는 한 가지 일만 시키자!!***

```java
public class UserValidator {
	private Cryptographer cryptographer;
	public boolean checkPassword(String userName, String password) { 
		User user = UserGateway.findByName(userName);
		if (user != User.NULL) {
			String codedPhrase = user.getPhraseEncodedByPassword(); 
			String phrase = cryptographer.decrypt(codedPhrase, password); 
			if ("Valid Password".equals(phrase)) {
				Session.initialize();
				return true; 
			}
		}
		return false; 
	}
}
```

* checkPassword 함수 중 Session.initialize(); 는 함수명과는 맞지 않는 부수 효과이다.
* checkPasswordAndInitializeSession 이라는 이름이 훨씬 낫다. 하지만, 이 경우도 한 가지 일만 한다는 것에 위배된다.
### 출력 인수

일반적으로 프로그래머는 함수 인수를 입력 인수로 인지한다. 출력 인수는 피해야 한다. 

함수에서 상태를 변경해야 한다면, 함수가 속한 객체 상태를 변경하자.

## 함수 잘 만드는 법 - 8. 명령과 조회를 분리하라

함수가 하는 역할은 2가지이다. 한 함수에 한 가지 일만 해야한다. 둘 다 하면 안된다.

1. 뭔가를 수행한다. (객체 상태를 변경한다.) -> 명령
2. 뭔가에 답한다. (객체 정보를 반환한다.) -> 조회

ex) `public boolean set(String attribute, String value);`

-> `if(set("username, "uncleBob"))`

* 위 함수는 이름이 attribute인 속성을 찾아 값을 value로 설정한 후 성공하면 true, 실패하면 false를 반환한다. 
* 명령과 조회를 한 함수에서 했기 때문에 if(set()) 과 같은 괴상한 코드가 나온다.

```java
if (attributeExists("username")) {
    setAttribute("username", "uncleBob");
    ...
}
```

* 명령과 조회를 분리해서 혼란을 주지 말자!

## 함수 잘 만드는 법 - 9. 오류 코드보다 예외를 사용하라

명령 함수에서 오류 코드 사용의 문제점

1. 오류 코드를 반환하면 호출자는 오류 코드를 곧바로 처리해야 한다.
2. 의존성 자석(dependdency magnet) : 오류 코드의 등록/ 수정/ 삭제 가 일어날 경우 해당 오류 코드의 처리 코드도 처리해야 한다.

```java
public enum Error { //오류 코드를 enum 클래스로 정리
	OK,
	INVALID,
	NO_SUCH,
	LOCKED,
	OUT_OF_RESOURCES, 	
	WAITING_FOR_EVENT;
}
```


오류 처리도 `한 가지` 작업 이므로 오류 처리 함수는 try/catch 문을 사용하여 오류를 처리하는 함수는 오류 처리만 하도록 해야 한다.

```java 
Bad 
if (deletePage(page) == E_OK) {
	if (registry.deleteReference(page.name) == E_OK) {
		if (configKeys.deleteKey(page.name.makeKey()) == E_OK) {
			logger.log("page deleted");
		} else {
			logger.log("configKey not deleted");
		}
	} else {
		logger.log("deleteReference from registry failed"); 
	} 
} else {
	logger.log("delete failed"); return E_ERROR;
} 

--> 
//오류 코드 대신 예외를 적용하여 오류 처리 코드가 원래 코드에서 분리되므로 깔끔해진다.
Good
try {
	deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
} catch (Exception e) {
  	logger.log(e.getMessage());
}

-->

//정상 동작과 오류 처리 동작을 try/catch 블록에서 별도의 `함수`로 뽑아낸다.
Excellent 
public void delete(Page page) {
	try {
		deletePageAndAllReferences(page);
  	} catch (Exception e) {
  		logError(e);
  	}
}
//정상 동작
private void deletePageAndAllReferences(Page page) throws Exception { 
	deletePage(page);
	registry.deleteReference(page.name); 
	configKeys.deleteKey(page.name.makeKey());
}
//오류 처리 동작
private void logError(Exception e) { 
	logger.log(e.getMessage());
}
```

* 오류 코드 대신 예외를 적용하여 오류 처리 코드가 원래 코드에서 분리되므로 깔끔해진다.
* 정상 동작과 오류 처리 동작을 try/catch 블록에서 별도의 `함수`로 뽑아낸다.

## 함수 잘 만드는 법 - 10. 반복하지 마라 





