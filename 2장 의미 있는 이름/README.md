# 2장 의미 있는 이름

소프트웨어에서 `이름`은 어디에서나 쓰인다.

*변수, 함수, 클래스, 모듈, 패키지, 소스 파일, 디렉터리, jar 파일, war 파일, ear 파일*에도 이름을 붙인다.

이렇듯 이름을 많은 곳에서 사용하므로, 이름을 잘 짓기만 해도 여러모로 편한 점이 많아진다.

이 장에서는 `이름을 짓는 규칙`에 대해서 알아보겠다.

## 이름 짓는 규칙 - 1. 의도를 분명히 밝혀라

이름을 지을 때, 아래의 질문들에 답할 수 있어야 한다.
1. **변수(함수, 클래스)의 존재 이유는?**
2. **수행 기능은?**
3. **사용 방법은?**

***만약 따로 주석이 필요하다면 잘못된 것이다.***

```java
int d; //경과 시간을 의미(단위: 날짜)
```
* 이름 d 는 아무 의미도 드러나지 않는다. 경과 시간 또는 날짜라는 생각이 들지 않는다.
* *측정하려는 값*과 *단위*를 표현하려는 이름이 필요하다.

```java
int elapsedTimeInDays;
int daysSinceCreation;
int daysSinceModification;
int fileAgeInDays;
```

* 이런식으로 만들어야 한다.

의도가 분명한 이름을 사용하면 코드 이해와 변경이 쉬워진다.

```java
Bad
public List<int[]> getThem() {
    List<int[]> list1 = new ArrayList<int[]>();
    for(int[] x: theList)
        if(x[0] == 4)
            list1.add(x);
    return list1;        
}
```

* 위의 코드가 복잡하진 않지만 그렇다고 이해하기 쉬운 코드도 아니다.
* 위 코드의 문제는 단순성이 아닌 **`함축성`** 이다.
  1. theList 에 무엇이 들어있는가?
  2. theList 에서 0번째 값이 어째서 중요한가?
  3. 값 4는 무엇을 의미하는가?
  4. 함수가 반환하는 list1 을 어떻게 사용하는가?
   
```java
Good
public List<int[]> getFlaggedCells() {
    List<int[]> flaggedCells = new ArrayList<int[]>();
    for(int[] cell : gameBoard) 
        if(cell[STATUS_VALUE] == FLAGGED)
            flaggedCells.add(cell);
    return flaggedCells;
}
```

* 지뢰찾기 게임을 만드는 코드라고 가정해보자. 
  1. theList 가 게임판이라는 사실을 안다. theList -> gameBoard. 게임판에서 각 칸은 단순 배열을 의미한다.
  2. 배열에서 0번째 값은 칸 상태를 뜻한다.
  3. 값 4는 깃발이 꽂힌 상태를 가리킨다.
  4. 깃발이 꽃힌 list 를 반환하는 함수이다.

```java
Excellent
public List<Cell> getFlaggedCells() {
    List<Cell> flaggedCells = new ArrayList<Cell>();
    for(Cell cell : gameBoard) 
        if(cell.isFlagged())
            flaggedCells.add(cell);
    return flaggedCells;
}
```
좀 더 나아가서

* int 배열을 대신해서 칸을 간단한 Class로 만들어도 된다.
* FLAGGED라는 상수를 감추고, 대신에 isFlagged 라는 좀 더 명시적인 함수를 사용하자.

## 이름 짓는 규칙 - 2. 그릇된 정보를 피하라

1. 나름대로 널리 쓰이는 의미가 있는 단어를 다른 의미로 사용해서도 안된다.

ex) 직각삼각형의 빗변(hypotenuse)를 의미하는 hp가 변수명으로 적절해 보일지라도, hp는 유닉스 플랫폼이나 유닉스 변종을 뜻하기 때문에 변수 이름으로 적합하지 않다.

2. 여러 계정을 그룹으로 묶을 때, List와 같은 `컨테이너 유형`을 변수명에 사용하지 마라

ex) accountList -> accountGroup, bunchOfAccounts, Accounts

3. 서로 흡사한 이름을 사용하지 않도록 주의한다. 

ex) 한 모듈에서 XYZControllerForEfficientHandlingOfStrings, 조금 떨어진 다른 모듈에서 XYZControllerForEfficientStoragOfStrings

4. 유사한 개념은 유사한 표기법을 사용한다. 여기서 중요한건 `일관성`이다. 일관성을 떨어트리지 말자.

## 이름 짓는 규칙 - 3. 의미 있게 구분하라

연속된 숫자를 덧붙이거나 불용어(noise word)를 추가하는 방식은 안된다.

ex) 연속된 숫자

```java
Bad
public static void copyChars(char a1[], char a2[]) {}
-->
Good 
public static void copyChars(char source[], char destination[]) {}
```

* `함수 인수 이름`으로 source와 destination을 사용한다면 코드 읽기에 훨씬 편하다.

ex) 불용어(noise word)

`불용어`란? **없어도 의미 전달에 영향이 없는 단어**

* 접두어 a, an, the 와 같은 접두어는 의미가 분명히 다를 때만 사용한다.
  
  ex) `모든 지역 변수`는 a 접두어, `모든 함수 인수`는 the 접두어

* 불용어를 사용하여 중복을 발생시키지 말자.
  
  ex) Product 라는 Class 가 있을 때, ProductInfo, ProductData 와 같이 `개념이 중복`되는 이름을 사용하지 않는다. 변수 moneyAmount, money / customerInfo, customer / accountData, account / theMessage, message

## 이름 짓는 규칙 - 4. 발음하기 쉬운 이름을 사용하라

발음하기 어려운 이름은 서로 의사소통하기에도 어렵다. 사람들은 단어에 능숙하므로, `발음하기 쉬운 실제로 존재하는 단어`를 사용하자

```java
Bad
private Date genymdhms; //generate date, year, month, day, hour, minute, second
-->
Good
private Date generationTimestamp;
```

## 이름 짓는 규칙 - 5. 검색하기 쉬운 이름을 사용하라

이름 길이는 범위(Scope) 크기에 비례해야 한다. 
* 간단한 메서드에서 로컬 변수만 한 문자를 사용한다.

변수 
* 문자 하나만을 변수 이름에 사용하면 검색하기 어렵다.

상수
* 상수값을 의미를 의미 있는 변수로 정의하면 검색이 쉽다. 

```java
Bad
for (int j = 0; j < 34; ++j)
    s += (t[j] * 4) / 5;

-->

Good
int realDaysPerIdelDay = 4;
const int WORK_DAYS_PER_WEEK = 5;
int sum = 0;
for(int j = 0; j < NUMBER_OF_TASKS; ++j) {
    int realTaskDays = taskEstimates[j] * realDaysPerIdelDay;
    int realTaskWeeks = (realTaskDays / WORK_DAYS_PER_WEEK);
    sum += realTaskWeeks;
} 
```

## 이름 짓는 규칙 - 6. 인코딩을 피하라

유형이나 범위 정보까지 인코딩에 넣지 않아도 인코딩할 정보는 아주 많다. 과도한 인코딩을 피하자

1. `헝가리안 표기법 지양`
   * 예전 프로그래밍에서는 변수명 앞에 data type 을 붙이는 헝가리안 표기법을 사용했지만, 요즘 나오는 프로그래밍 언어에서는 컴파일러가 타입을 강제하고 억제하기 때문에 이제는 헝가리안 표기법이나 기타 인코딩 방식이 오히려 해가 된다!

```java
PhoneNumber phneString; // 타입이 바뀌어도 이름은 바뀌지 않는다!
```

2. `멤버 변수 접두어 지양`
   * 멤버 변수 접두어에 `m_` 이라는 접두어를 붙일 필요가 **없다.** 
   * 클래스와 함수는 접두어가 필요 없을 정도로 작게 만들어야 하기 때문에 
   * 애초에 멤버 변수를 다른 색깔로 표시해주는 IDE를 사용해야 마땅하다.
   * 심지어 사람들은 코드를 읽을 때 접두어(접미어)를 무시하고 읽고 지나갈 때가 더 많다
```java
Bad
public class Part {
    private String m_dsc; // 설명 문자열
    void setName(String name) {
        m_dsc = name;
    }
}

-->
Good
public class Part {
    private String description;
    void setDescription(String description) {
        this.description = description;
    }
}
```

3. 인터페이스 클래스와 구현 클래스 -> 인코딩 필요

Abstract Factory 와 같은 인터페이스 클래스(Interface Class)와 그것을 구현하는 구체 클래스(Concrete Class)가 있을 때
* 인터페이스 클래스보다 `구체 클래스`에 인코딩을 하는 것이 좋다!

```java
Bad
public interface IShapeFactory
public class ShapeFactory implements IShapeFactory
```
--> 
```java
Good
public interface ShapeFactory
public class ShapeFactoryImpl implements ShapeFactory
public class ShapeFactoryImpl implements ShapeFactory
```

## 이름 짓는 규칙 - 7. 자신의 기억력을 자랑하지 마라

1. `문자 하나`만 변수명을 짓지 마라
   * loop 에서 반복 횟수를 세는 변수 i, j, k 까지는 봐준다. (l은 안된다!)
   * 나머지는 대부분 적절하지 못하다. 

2. 나만 이해하는 변수명이 아닌 우리 모두가 이해하는 변수명을 짓자. -> `명료함`이 최고다!!

## 이름 짓는 규칙 - 8. 클래스 이름

1. 클래스 이름과 객체 이름은 `명사`나 `명사구`가 적절하다
* ex) Customer, WikiPage, Account, Address, AddressParser

2. 불용어(noise word) 사용은 피한다. 동사도 사용하지 않는다.
* ex) Manager, Processor, Data, Info

## 이름 짓는 규칙 - 9. 메서드 이름

1. 메서드 이름은 `동사`나 `동사구`가 적합하다.
* ex) postPayment, deletePage, save 등등

2. 접근자(Accessor), 변경자(Mutator), 조건자(Predicate)는 `javabean 표준`에 따라 `값 앞에 get, set, is`를 붙인다.
```java
//get
string name = employee.getName();
//set
cutomer.setName("Lee");
//is
if(paycheck.isPosted())
```
3. 생성자를 오버로드(Overload) 할 경우, `정적 팩토리 메서드(static method)`를 사용하고, 해당 `생성자를 private`로 선언
```java
Bad
Complex fulcrumPoint = new Complex(23.0);
-->
Good
Complex fulcrumPoint = Complex.FromRealNumber(23.0);
```

## 이름 짓는 규칙 - 10. 기발한 이름을 피하라

특정 문화에서만 사용하는 농담은 피하라. 의도를 분명하고 명확하게 표현하자.


## 이름 짓는 규칙 - 11. 한 개념에 한 단어를 사용하라

추상적인 `개념 하나`에 `단어 하나`만을 사용하라 
* fetch, retrieve, get -> 중에 하나만을 사용
* controller, manager, driver -> 중에 하나만을 사용

개념 하나에 단어 하나 만을 사용하되 `일관성 있는 어휘` 사용하자
* add 라는 메서드는 기존 값 2개를 더해서 새로운 값을 더하는 메서드이었다.
* 집합에 값 하나를 추가하는 메서드를 작성할 때도 add 를 사용해서는 안된다! insert, append 가 맞다.

## 이름 짓는 규칙 - 12. 해법 영역(기술)에서 가져온 이름을 사용하라

모든 이름을 문제 영역(domain)에서 찾지 않는다. 전산 용어, 알고리즘 이름, 패턴 이름, 수학 용어 등을 사용해도 좋다.

ex) JobQueue, SingletonInstance 등   

## 이름 짓는 규칙 - 13. 문제 영역(domain)에서 가져온 이름을 사용하라

적절한 기술 이름(프로그래머 용어)이 없는 경우, 문제 영역(domain)에서 이름을 가져온다.

## 이름 짓는 규칙 - 14. 의미 있는 맥락을 추가하라

스스로 의미가 분명한 이름은 많지 않다.

* `클래스, 함수, namespace 등`으로 이름을 감싸서 맥락(Context)를 부여한다.
* 모든 방법이 실패하면 `마지막 수단`으로 `접두어`를 붙인다.

ex1)

```java
Bad //주소를 의미하는 변수들이지만 어느 메서드가 state 하나만 사용한다면 주소의 일부라는 것을 빨리 알아채지 못한다.
private String firstName, lastName, street, housenNumber, city, state, zipcode;

-->

Good // addr 이라는 접두어를 추가하자.
private String addrFirstName, addrLastName, addrStreet...;

-->
Excellent // 접두어를 붙이기 보다는 Address class 를 만들자.
public class Address {
    private String firstName;
    private String lastName;
    private String street;
    ...
}
```

ex2)

```java
Bad
private void printGuessStatistics(char candidate, int count) {
    String number;
    String verb;
    String pluralModifier;
    
    if (count == 0) {
        number = "no";
        verb = "are";
        pluralModifier = "s";
    } else if (count == 1) {
        number = "1";
        verb = "is";
        pluralModifier = "";
    } else {
        number = Integer.toString(count);
        verb = "are";
        pluralModifier = "s";
    }
    
    String guessMessage = String.format(
        "There %s %s %s", verb, number, candidate, pluralModifier
    );

    print(guessMessage);
}
```

* 일단 변수에는 좀 더 의미있는 맥락이 필요해보이지는 않는다.
* 함수 이름(printGuessStatistics)이 맥락을 제공하며, 알고리즘이 나머지 맥락을 제공한다.
* 하지만, 함수를 끝까지 읽고 나서야 number, verb, pluralModifier 라는 변수 3개가 GuessStatistics message에 어떻게 사용된다는 것을 알 수 있다.
* 불행하게도, **독자**가 **맥락을 유추**해야만 하는 것이다.
* 메서드만 훑어서는 세 변수의 의미가 불분명하다.


```java
public class GuessStatisticsMessage {   
    private String number;
    private String verb;
    private String pluralModifier;

    public String make(char candidate, int count) {
        createPluralDependentMessageParts(count);
        return String.format(
            "There %s %s %s", verb, number, candidate, pluralModifier   
        );
    }

    private void createPluralDependentMessageParts(int count) {
        if (count == 0) {
            thereAreNoLetters();
        } else if(count == 1) {
            thereIsOneLetter();
        } else {
            thereAreManyLetters(count);
        }
    }

    private void thereAreManyLetters(int count) {
        number = Integer.toString(count);
        verb = "are";
        pluralModifier = "s";
    }

    private void thereIsOneLetter() {
        number = "1";
        verb = "is";
        pluralModifier = "";
    }

    private void thereAreNoLetters() {
        number = "no";
        verb = "are";
        pluralModifier = "s";
    }
}
```

* GuessStatisticsMessage 라는 class 를 만들어서 세 변수의 `맥락`을 분명하게 밝혔다.
* 맥락을 개선하면 함수를 쪼개기도 쉬워지고, 알고리즘도 명확해진다.

## 이름 짓는 규칙 - 15. 불필요한 맥락을 없애라

모든 클래스를 아우르는 맥락을 이름에 넣지 말자.
* IDE 에서 검색하기에도 좋지 못하다.
* 일반적으로는 짧은 이름이 긴 이름보다 좋다.(단, 의미가 분명한 경우에 한해서다)

```JAVA
Bad
package com.gsd;
public class GSDAccountAddress
```
-->

```java
Good
package com.gsd;
public class Address
```

## 마지막으로

다른 개발자의 반대를 두려워 하지 말고 좋은 이름으로 바꾸는 연습을 `꾸준히` 하며 `코드 가독성`을 높이자!

