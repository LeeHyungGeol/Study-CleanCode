# 의미 있는 이름

소프트웨어에서 `이름`은 어디에서나 쓰인다.

*변수, 함수, 클래스, 모듈, 패키지, 소스 파일, 디렉터리, jar 파일, war 파일, ear 파일*에도 이름을 붙인다.

이렇듯 이름을 많은 곳에서 사용하므로, 이름을 잘 짓기만 해도 여러모로 편한 점이 많아진다.

이 장에서는 `이름을 짓는 규칙`에 대해서 알아보겠다.

## 1. 의도를 분명히 밝혀라

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
* Flagged라는 상수를 감추고, 대신에 isFlagged 라는 좀 더 명시적인 함수를 사용하자

## 2. 그릇된 정보를 피하라

1. 나름대로 널리 쓰이는 의미가 있는 단어를 다른 의미로 사용해서도 안된다.

ex) 직각삼각형의 빗변(hypotenuse)를 의미하는 hp가 변수명으로 적절해 보일지라도, hp는 유닉스 플랫폼이나 유닉스 변종을 뜻하기 때문에 변수 이름으로 적합하지 않다.

2. 여러 개정을 그룹으로 묶을 때, List와 같은 `컨테이너 유형`을 변수명에 사용하지 마라

ex) accountList -> accountGroup, bunchOfAccounts, Accounts

3. 서로 흡사한 이름을 사용하지 않도록 주의한다. 

ex) 한 모듈에서 XYZControllerForEfficientHandlingOfStrings, 조금 떨어진 다른 모듈에서 XYZControllerForEfficientStoragefStrings

4. 유사한 개념은 유사한 표기법을 사용한다. 여기서 중요한건 `일관성`이다. 일관성을 떨어트리지 말자.

## 3. 의미 있게 구분하라

1. 연속된 숫자를 덧붙이거나 불용어(noise word)를 추가하는 방식은 안된다.

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
  
  ex) 모든 지역 변수는 a 접두어, 모든 함수 인수는 the 접두어

* 불용어를 사용하여 중복을 발생시키지 말자.
  
  ex) Product 라는 Class 가 있을 때, ProductInfo, ProductData 와 같이 `개념이 중복`되는 이름을 사용하지 않는다. 변수 moneyAmount, money / customerInfo, customer / accountData, account / theMessage, message

## 4. 발음하기 쉬운 이름을 사용하라

발음하기 어려운 이름은 서로 의사소통하기에도 어렵다. 사람들은 단어에 능숙하므로, `발음하기 쉬운 실제로 존재하는 단어`를 사용하자

```java
Bad
private Date genymdhms; //generate date, year, month, day, hour, minute, second
-->
Good
private Date generationTimestamp;
```

## 5. 검색하기 쉬운 이름을 사용하라

이름 길이는 범위(Scope) 크기에 비례해야 한다. 
* 간단한 메서드에서 로컬 변수만 한 문자를 사용한다.

* 변수 
  * 문자 하나만을 변수 이름에 사용하면 검색하기 어렵다.

* 상수
  * 상수값을 의미를 의미 있는 변수로 정의하면 검색이 쉽다. 

```java
Bad
for (int j = 0; j < 34; ++j>)
    s += (t[j]*4)/5;

-->

Good
int realDaysPerIdelDay =4;
const int WORK_DAYS_PER_WEEK =5;
int sum = 0;
for(int j=0;j<NUMBER_OF_TASKS; ++j) {
    int realTaskDays = taskEstimates[j] * realDaysPerIdelDay;
    int realTaskWeeks = (realTaskDays / WORK_DAYS_PER_WEEK);
    sum += realTaskWeeks;
} 
```







