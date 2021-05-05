# 7장 오류 처리

깨끗한 코드와 오류 처리는 확실히 연관성이 있다.

상당수 코드 기반은 전적으로 오류 처리 코드에 좌우된다.

여기저기서 흩어진 오류 처리 코드로 인해 프로그램 논리를 하악하기가 거의 불가능해진다는 의미다.

오류 처리 코드로 인해 프로그램 논리를 리해하기 어려워진다면 깨끗한 코드가 아니다.

***`비즈니스 논리와 오류 처리를 잘 분리해낸 코드를 작성해야 한다.`***

## 오류 코드보다 예외를 사용하라

```java
// 오류 플래그를 설정하거나 호출자에게 오류 코드를 반환하는 코드
BAD
public class DeviceController {
	//호출자 코드
	public void sendShutDown() {
		Devicehandle handle = gethandle(DEV1);
		//디바이스 상태 점검
		if (handle != DeviceHandle.INVALID) {
			//레코드 필드 디자이스 상태 저장
			retrieveDeviceRecord(handle);
			//디바이스가 일시정지 상태가 아니라면 종료.
			if (record.getStatus() != DEVICE_SUSPENDED) {
				pauseDevice(handle);
				clearDeviceWorkQueue(handle);
				closeDevice(handle);
			} else {
				logger.log("Device suspended. Unable to shut down");
			}
		} else {
				logger.log("Invalid handle for: " + DEV1.toString());
		}
	}
}
```

* 오류 코드를 사용하면, 호출자 코드가 복잡해진다.
* 함수를 호출한 즉시 오류를 확인해야 한다. 이러한 단계는 잊어버리기 쉽다.

```java
// 오류를 발견하면 예외를 던지는 코드
GOOD
public class DeviceController {
    //호출자 코드
	public void sendShutDown() {
		try {
			tryToShutDown();
		} catch (DeviceShutDownError e) {
			logger.log(e);
		}
	}
	
	private void tryToShutDown() throws DeviceShutDownError {
		DeviceHandle handle = gethandle(DEV1);
		DeviceRecord record = retrieveDeviceRecord(handle);
	
		pauseDevice(handle);
		clearDeviceWorkQueue(handle);
		closeDevice(handle);
	}

	private DeviceHandle gethandle(DeviceID id){
		...
		throw new DeviceShutDownError("Invalid handle for: " + id.toString());
	}
}
```

* 디바이스를 종료하는 알고리즘과 오류를 처리하는 알고리즘을 분리 -> 코드 품질 및 이해도 UP

## Try-Catch-Finally 문부터 작성하라

* try 블록은 transaction 과 비슷하다.
* try 블록에서 무슨 일이 생기든지 `catch 블록`은 프로그램 상태를 **일관성 있게 유지해야 한다.**
* 따라서, 예외가 발생하는 코드를 짤 때는 `Try-Catch-Finally 문`으로 짜는 것이 좋다.
* => try 블록에서 무슨 일이 일어나든지 **호출자가 기대한 상태를 정의하기 쉬워진다.**

```java
// 파일이 없으면 예외를 던지는지 알아보는 단위 테스트
@Test(expected = StorageException.class)
public void retrieveSectionShouldThrowOnInvalidFileName() {
	sectionStore.retrieveSection("invalid - file");
}

// 단위 테스트에 맞춰 구현한 코드
public List<RecordedGrip> retrieveSection(String sectionName) {
    try {
        FileInputStream stream = new FileInputStream(sectionName);
        ... 	// 나머지 추가 로직 
	     
	    stream.close();
    } catch (FileInputException e) {
        throw new StorageException("retrieval error", e);
    }
    return new ArrayList<RecordedGrip>();
}
```

* **try-catch 구조**로 범위를 정하였다. -> `TDD`를 이용하여 나머지 로직을 추가한다.
  * 먼저 강제로 예외를 일으키는 테스트 케이스를 작성한다
  * 테스트를 통과하는 코드를 그 이후에 작성한다,
    * -> 자연스럽게 ***try 블록의 트랜잭션 범위부터 구현***하므로, 범위 내에서 transaction의 본질을 유지하기 쉬워진다.  

## 미확인(Unchecked) 예외를 사용하라

||Checked Exception|UnChecked Exception|
|---|---|---|
|확인 단계|컴파일 시점|런타임 시점|
|처리 여부|반드시 처리|명시적인 처리를 강제하지 않음|
|트랜잭션 처리|roll-back X|roll-back O|
|예시|Exception의 상속받는 하위 클래스 중 **Runtime Exception을 제외한 모든 예외** => IOException, SQLException|**Runtime Exception 하위 예외** => NullPointerException, IllegalArgumentException, IndexOutOfBoundException, SystemException|

확인된 예외(Checked Exception)은 **OCP(Open Closed Principle)를 위반한다.**

* 메서드에서 확인된 예외를 던졌는데 catch 블록이 3단계 위에 있다면 그 사이의 메서드 모두가 선언부에 해당 예외를 정의해야 한다.
* ***하위 단계에서 코드를 변경하면 상위 단계 메서드 선언부를 전부 고쳐야 한다는 의미*** 

EX)
```java
public void catchFileInputException() {
    try { 
        a();
    } catch (FileInputException e) {
        throw new StorageException("retrieval error", e);
    }
}

public void a() throws FileInputException { 
    ab(); 
}

public void ab() throws FileInputException {
    abc();
}

public void abc() throws FileInputException {
    FileInputStream stream = new FileInputStream(sectionName);
    stream.close();
}
```


## 예외에 의미를 제공하라

```java
public List<RecordedGrip> retrieveSection(String sectionName) {
    try {
        FileInputStream stream = new FileInputStream(sectionName);
        ... 	// 나머지 추가 로직 
	    stream.close();
    } catch (FileInputException e) {
        throw new StorageException("retrieval error", e);
    }
    return new ArrayList<RecordedGrip>();
}
```

* 예외를 던질 때는 전후 상황을 충분히 덧붙인다. -> 오류가 발생한 원인과 위치를 찾기가 쉬워진다.
* 오류 메시지에 정보를 담아 예외와 함께 던진다. 실패한 연산 이름과 실패 유형도 언급한다.
* 로깅 기능을 사용한다면 catch 블록에서 오류를 기록한다.

## Wrapper 클래스를 이용하라

**외부 API를 이용할 때**는 `감싸기(Wrapping) 기법`이 최선이다.

* 외부 API를 감싸면 외부 라이브러리와 프로그램 사이에서 의존성이 크게 줄어든다. 
* 나중에 다른 API로 갈아타기에도 용이하다.
* 감싸기 클래스(Wrapper class)에서 외부 API를 호출하는 대신 Test Code를 넣어줌으로써 프로그램을 test 하기에도 용이하다.
* 특정 업체가 API를 설계한 방식에 발목 잡히지 않는다.

```java
BAD

ACMEPort port = new ACMEPort(12);

try {
    port.open();
} catch (DeviceResponseException e) {
    reportPortError(e);
    logger.log("Device response exception", e);
} catch (ATM1212UnlockedException e) {
    reportPortError(e);
    logger.log("Unlock exception", e);
} catch (GMXError e) {
    reportPortError(e);
    logger.log("Device response exception");
} finally {
    ...
}

GOOD

LocalPort port = new LocalPort(12);
try {
    port.open();
} catch (PortDeviceFailure e) {
    reportError(e);
    logger.log(e.getMessage(), e);
} finally {
    ...
}

//Wrapper class
public class LocalPort {
    private ACMEPort innerPort;

    public LocalPort(int portNumber) {
        innerPort = new ACMEPort(portNumber);
    }

    public void open() {
        try {
            innerPort.open();
        } catch (DeviceResponseException e) {
            throw new PortDeviceFailure(e);
        } catch (ATM1212UnlockedException e) {
            throw new PortDeviceFailure(e);
        } catch (GMXError e) {
            throw new PortDeviceFailure(e);
        }
    }
    ...
 }
```

## 정상 흐름을 정의하라

Wrapper class를 이용하여 코드를 분리하고 예외 처리를 하면, 비즈니스 논리와 오류 처리가 잘 분리된 깔끔한 코드가 나온다.

그러나, 예외 처리도 가독성을 낮추기 때문에 코드 중간에 예외 처리를 피하자.

=> 특수 사례 패턴(Special Case Pattern)을 이용

* `특수 사례 패턴(Special Case Pattern)` : **반환할 값이 없거나 null 일때, 예외를 던지는 것이 아닌 기본 값을 반환하여 특수사례를 처리한다.**
  
  * 클라이언트 코드가 예외적인 상황을 처리할 필요가 없어진다.

```java
BAD
try {
    MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
    m_total += expenses.getTotal();
 } catch(MealExpencesNotFound e) {
    m_total += getMealPerDiem();
 }

GOOD
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();

 public class PerDiemMealExpenses implements MealExpenses {
     private static final int DEFAULT_EXPENSES = 100;
     public int getTotal() {

        // 기본값으로 기본 식비를 반환한다.
        return DEFAULT_EXPENSES;
    }    
}
```

## null을 반환하지 마라

메서드가 null을 반환하면 해당 메서드를 사용하는 클라이언트 코드는 null을 처리하는 코드를 항상 추가해야 한다. 그 과정에서 null 처리 코드를 하나라도 빼먹는다면 버그가 발생할 것이고, 애플리케이션을 점점 통제하기가 힘들어질 것이다.

* `Wrapper class 메서드` 혹은 `Special Case 객체`를 이용하여 해결할 수 있다.

```java
BAD
List<Employee> employees = getEmployees();
if(employees != null) {
	for(Employee e : employees) {
		totalPay += e.getPay();
	}
}

GOOD
List<Employee> employees = getEmployees();

for(Employee e : employees) {
	totalPay += e.getPay();
}

public List<Employee> getEmployees() {
	if (..직원이 없다면..)
		return Collections.emptyList();
}
```

## null을 전달하지 마라

* null을 리턴하는 것도 나쁘지만 null을 메서드로 넘기는 것은 더 나쁘다.
* null을 메서드의 파라미터로 넣어야 하는 API를 사용하는 경우가 아니면 null을 메서드로 넘기지 마라.
* 일반적으로 대다수의 프로그래밍 언어들은 파라미터로 들어온 null에 대해 적절한 방법을 제공하지 못한다.
* 가장 이성적인 해법은 null을 파라미터로 받지 못하게 하는 것이다.

