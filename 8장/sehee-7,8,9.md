## 7장. 오류처리
깨끗한 코드와 오류처리에는 연관성이 존재한다. 여기저기 흩어진 오류처리 코드 때문에 실제 코드를 파악하기 어려운 경우도 존재하기 때문이다. 이 장에서는 오류처리를 깔끔하게 처리하는 방법에 대해서 소개한다.
<br>

** 1. 오류코드보다 예외를 사용해라 **

```kotlin
// AS IS
class DeviceController {
    fun sendShutDown() {
        val handle = Handle(DEV1)

        // 디바이스 상태 점검 
        if (handle != DeviceHandle.INVALID) {
            retrieveDeviceRecord(handle)

            // 디바이스가 일시정지 상태가 아니라면 종료한다.
            if (recode.status != DEVICE_SUSPENDED) {
                pauseDevice(handle)
                clearDeviceWorkQueue(handle)
                closeDevice(handle)
            } else {
                Log.e("Device Suspended. Unable to shut down")
            }
        } else {
            Log.e("Invalid handle for: ${DEV1.toString()}")
        }
    }
}
```
AS IS는 오류코드를 사용해 에러를 핸들링하는 예시이다. 논리와 오류처리코드가 뒤섞이게 되어 읽기가 힘들어진다. 코드를 읽는 사람은 함수를 호출한 즉시 오류를 확인하게되고, 아래로 내려가면서 오류 검증 단계를 잊어버리기 때문이다.

``` kotlin
// TO BE
class DeviceController {
    fun sendShutDown() {
        try {
            tryToShutDown()
        } catch (e: DeviceShutDownError) {
            Log.e(e)
        }
    }

    private fun tryToShutDown() {
        val handle = getHandle(DEV1)
        retrieveDeviceRecord(handle)

        pauseDevice(handle)
        clearDeviceWorkQueue(handle)
        closeDevice(handle)
    }

    private fun getHandle(id: DeviceId) {
        ...
        throw DeviceShutDownError("Invalid handle for: ${id.toString()}")
    }
}
```

TO BE로 개선하게 되었을 때 논리와 오류처리코드가 분리되기 때문에 읽기가 더 쉬워진다. 
<br>

** 2. Try-Catch-Finally 문부터 작성해라 **

예외처리는 프로그램 안에 범위를 정의한다. try 블록은 어떤면에서 트랜잭션과 비슷한데, try 블록에서는 무슨일이 생겨도 catch 블록에서는 프로그램 상태를 일관성있게 유지해야한다. 예외가 발생할 여지가 있는 코드는 try-catch 문으로 작성하는 것이 좋은데, try 블록에서 무슨 일이 생기든지 호출자가 기대하는 상태를 정의하기가 쉬워지기 때문이다.

테스트코드를 작성할 때도, 먼저 예외를 일으키는 테스트케이스를 작성한 후 테스트를 통과하게 코드를 작성하는 방식을 권한다. 그러면 자연스럽게 try 블록의 트랜젝션 범위부터 구현하게 되므로 범위 내에서 트랜젝션 본질을 유지하기 쉬워진다. 
<br>

** 3. 미확인 예외를 사용해라 **

예전에는 메소드를 선언할 때, 메소드가 반환할 예외를 모두 열거했었다. 하지만 이렇게 확인된 예외를 나열하는 것은 객체지향 원칙 중 OCP 에 위반하게 된다. 예시로, 대부분의 시스템에서는 최상위함수가 아래함수를 호출하고, 아래함수가 그 아래함수를 호출하는 연쇄과정을 거치면서 호출하는 함수의 수가 증가하게 된다. 이 때, 최하위함수를 변경해 새로운 오류를 던진다고 가정한다면 최상위함수까지 연쇄적인 수정이 일어나게되어 캡슐화를 깨버리게된다. 이 점을 유의해서, 확인된 예외와 미확인 예외를 적절하게 사용하는 것이 중요하다. 
<br>

** 4. 호출자를 고려해 예외 클래스를 정의해라 **

오류를 분류하는 방법은 정말 다양한데, 이 책에서는 오류가 발생 유형으로 분류하고 있다. 

``` kotlin
// AS IS
try {
    port.open()   
} catch(e: DeviceResponseException) {
    reportPortError(e)
} catch(e: ATM1212UnlockedException) {
    reportPortError(e)
} catch(e: GMXError) {
    reportPortError(e)
}
```
위 예제는 외부 라이브러리를 호출할 때 발생할 예외를 디바이스 실패, 네트워크 실패, 프로그래밍 오류로 분류하고 있다. 하지만 이 경우에는 예외가 발생할 때 대응하는 방식이 `reportPortError` 로 동일한데, 이 때는 호출하는 라이브러리의 API를 감싸면서 예외유형 하나를 반환하면 쉽게 개선할 수 있다.


``` kotlin
// TO BE
try {
    port.open()  
// 위의 예외케이스들을 PortDeviceFailure 로 Wrapping 해서 사용하는 형태
} catch(e: PortDeviceFailure) {
    reportPortError(e)
} finally {
    ....
}
```
외부 API를 중간에 Wrapping 하게되면 외부 라이브러리와 우리 프로그램 사이의 의존성이 크게 줄어들고, 추후에 다른 라이브러리로 변경하게 되더라도 비용이 적게 들 수 있다. 또한, 외부 API 가 설계한 방식에 발목이 잡히지 않게 된다.
<br>

** 5. 정상 흐름을 정의하라 **

때로는 중단이 적합하지 못할 때가 존재한다.
``` kotlin
    try {
        val expenses = expenseReportDAO.getMeals(employee.id)
        mealTotal += expenses.getTotal()
    } catch(e: MealExpensesNotFound) {
        mealTotal += getMealPerDiem()
    }
```
위 예시는 비용 청구 어플리케이션에서 총계를 계산하는 코드이다. 위에서 식비를 비용으로 청구했다면, 직원이 청구한 식비`expenses.getTotal()` 에서 총계`mealTotal`를 더하고, 식비를 비용으로 청구하지 않은 경우`MealExpensesNotFound` 에는 기본 식비`getMealPerDiem()` 를 총계에 더하는 로직이다.

하지만 이 경우 예외처리가 논리를 따라가기 어렵게 만든다. 읽어보면 "식비를 비용으로 청구하지 않은 경우"가 오류로 인식되지 않기 때문이다. 

``` kotlin
val expenses = expenseReportDAO.getMeals(employee.id)
mealTotal += expenses.getTotal()

class PerDiemMealExpenses: MealExpenses {
    fun getTotal() {
        //기본값으로 기본 식비를 반환한다.
    }
}
```
이 경우에는 expenseReportDAO 를 약간 수정해, 청구한 식비가 없는 경우 기본식비를 반환할 수 있도록 개선하는 것이 좋다.
<br>

**6. null을 반환하지 마라**
null을 반환하는 코드는 호출자에게 문제를 떠넘긴다. null 확인을 빼먹게 된다면 어플리케이션이 동작하지 않을지도 모른다. 무조건 null을 반환하는 것보다 그나마 비슷한 목적을 가질 수 있는 empty()를 활용해보는 것이 좋을 수 있다.

``` kotlin
val employees = getEmployees()
for (i in employees.indices) {
    totalPay += i.getPay()
}

fun getEmployees(): List<Employees> {
    if ( /** 직원이 없다면 */ ) return emptyList()
}
```
<br>

**7. null을 전달하지 마라**
null을 반환하는 방식도 좋지 않지만, 전달하는 방식은 더더욱 나쁘다. 정상적인 인수로 null을 반환하는 경우가 아니라면 최대한 피하는 것이 좋다. 
<br>
<br>

## 8장. 경계

이 장에서는 외부 코드와 우리의 코드 사이의 경계를 깔끔하게 처리하는 방법에 대해서 소개한다.

대표적으로 `java.util.Map` 를 오픈소스 중 하나의 경계 인터페이스 예시로 들고 있다. `Map` 을 만든 공급자는 clear(), containesKey(), get(), equals() 등등 많은 기능을 소비자가 편하게 사용할 수 있도록 제공한다. 공급자 입장에서는 최대한 많은 환경에서 오픈소스를 사용할 수 있게 적용성을 넓히게 만드는데, 이는 기능성과 유연성에 있어 유용하지만 그만큼 위험성도 크게 증가하게 된다.

>
>예시1) 우리 프로그램(소비자)에서 `프로그래머1`은 Map을 다른 곳에서 넘기는 `함수1`를 만들었다고 가정하자. 함수1은 Map을 넘기는데에만 집중했고, 프로그래머1은 다른 곳에서 Map 요소를 지울 것이라고는 생각하지 못할 것이다. 지운다는 가정을 하지 않았으니까. 하지만 `프로그래머2`가 `함수2`를 만들어 clear()를 사용해 Map 요소를 지웠다.
>
>예시2) 두번째로 프로그래머1과 2가 우리 프로그램에서 Map에 특정 객체만을 담기로 약속했다고 하자. 하지만 Map은 객체의 유형을 제한하지 않기 때문에, 나중에 들어오는 프로그래머3이 이 사실을 모르고 다른 객체 유형을 넣었다.

이렇게 소비자는 Map을 제어하는 많은 권한을 가지고 있지만, 사실은 자신에 환경에 딱 맞는 제한된 인터페이스만을 제공받기를 바란다. (예제1처럼 Map 요소에 대한 접근을 강하게 제어한다던지, 예시2처럼 특정 객체만을 넣을 수 있도록 제한한다던지...)

만약 우리가 `Map<String, Sensor>` 형태로 객체를 생성했고, 이 객체를 여기저기서 넘긴다고 가정하자. 이 때 Map 인터페이스가 변경된다면 어떻게 될까? 수정할 코드가 여기저기 증가하게 될 것이다. 기존 시스템에 Map을 여러군데에 사용했기 때문에 발생한 문제이다. 

이 때, 경계선을 나누어 제한하는것이 좋다.

``` kotlin
class Sensors {
  private val sensors = HashMap<String, Sensor>()
  
  fun getById(id: String) {
     return sensor.get(id)
     }
}
```
위 예시는 Map을 Sensor 안으로 숨겨 경계선을 나눈 예시이다. Map 인터페이스가 바뀌어도, Sensor 클래스 내부에서만 영향을 받고, 나머지 프로그램에는 영향을 미치지 않는다. 그래서 코드를 이해하기 쉬우면서 외부에서 오용하기는 어려운 구조가 된다. 이렇게, 경계 인터페이스를 사용하는 경우에는 이를 이용하는 클래스나 클래스 계열 밖으로 노출되지 않도록 하는 것이 중요하다.

<br>

** 아직 존재하지 않는 코드를 사용하기 **

![](https://images.velog.io/images/jshme/post/59b0cb80-66a0-46ca-a480-eef6da735585/image.png)

무선 통신 시스템에 들어갈 소프트웨어 개발을 시작했다고 가정하자. 이 때, 해당 소프트웨어에는 송신기라는 하위 시스템이 있고 송신기 시스템을 개발하는 사람들이 따로 존재한다. 하지만 송신기 시스템을 책임진 사람들은 아직 인터페이스 정의를 시작하지 못한 상태이다. 이 때, 우리의 소프트웨어가 송신기 시스템에 접근하는 경계를 알아야 외부 소스로부터의 경계선을 뚜렷하게 구분할 수 있다. 이 경계에서는 대체로 어댑터 패턴을 이용해, 우리가 원하는 인터페이스로 변환하는 과정을 거친다. 우리가 바라는 인터페이스를 송신기 시스템에서 구현한다면 우리가 인터페이스를 전적으로 통제할 수 있다는 장점이 생기며, 가독성도 높아질 것이다.

<br>
<br>

## 9장. 단위테스트

** 1. TDD 법칙 세 가지**

1. 실패하는 단위테스트를 작성할 때까지 실제 코드를 작성하지 않는다.
2. 컴파일은 실패하지 않으면서, 실행이 실패하는 정도로만 단위테스트를 작성한다.
3. 현재 실패하는 파일을 통과할 정도로만 실제 파일을 작성한다.
<br>

** 2. 깨끗한 테스트 코드 유지하기**

실제 코드가 변한다면, 테스트 코드도 변해야한다. 테스트 코드가 지저분해질수록 실제 코드를 짜는 시간보다 테스트 케이스 추가 시간이 더 길어지게 되고, 실패하는 테스트 케이스가 더욱 증가하게 되어 결국은 테스트 코드를 짜는데 부담이 증가하게 될 것이다. 이 상황이 계속 지속된다면 결국 팀에서는 테스트 코드를 비난하게 되고, 결국은 해당 테스트 코드들을 폐기하게 될 것이다.

하지만 테스트 코드가 없다면, 개발자는 자기가 수정한 코드가 제대로 작동하는지 자신할 수 없다. 이 쪽에서 수정해도 다른쪽이 안전하다는 사실을 검증하지 못하게 되고, 결함율이 높아지기 시작할 것이다. 이 과정은 개발자가 변경을 주저하게 될 것이고, 변경을 하게되면 득보다 해가 많다는 사실에 더이상 코드를 정리하지 않을 것이다. 

테스트 코드를 처음부터 깨끗하게 짰더라면 테스트에 쏟아부은 노력이 허사로 돌아가지 않을 것이다. 이 책에서는 `테스트 코드가 실제 코드 못지않게 중요하다.`라는 것을 강조하고 있다. 

테스트는 유연성, 유지보수성, 재사용성을 제공한다. 테스트 코드를 깨끗하게 유지하지 않으면 결국은 잃어버리고, 실제 코드를 유연하게 만드는 버팀목도 사라지게된다. **테스트 케이스가 없다면 모든 변경이 잠정적인 버그**이다. 아키텍처가 아무리 유연하더라도, 테스트 케이스가 없는 코드는 개발자가 변경을 주저하게 만든다. 
<br>

** 3. 깨끗한 테스트 코드 **

깨끗한 테스트 코드를 만들기 위해서는 가독성이 제일 중요하다. 코드를 읽는 사람을 고려하면서, 테스트 케이스를 이해하기 쉽도록 구성해보자. 잡다하고 세세한 코드는 없애고 테스트 코드에 필요한 진짜 자료와 함수만을 사요해 코드가 수행하는 의도를 재빨리 이해할 수 있도록 만들어야 한다. 또한 테스트 코드는 간결하고 표현력이 풍부해야하지만, 실제 코드만큼 효율적인 필요는 없다. 말 그대로 테스트 환경에서만 돌아가는 코드이기 때문이다. 
<br>

** 4. 테스트 당 assert 하나 **

테스트 코드를 짤 때는 함수마다 assert 문을 하나만 사용하는 것이 좋다. 가혹한 규칙일 수 있지만, assert문이 단 하나인 함수는 결론이 하나이기 때문에 코드를 이해하기 쉽다. 
대신, 테스트를 분리함으로써 중복되는 코드가 많아질 수 있다. 하지만 배보다 배꼽이 크지 않은 이상 중복 코드는 감안하면서 assert 문을 하나씩 배치하는 것이 더욱 좋다. 
<br>

** 5. F.I.R.S.T **

깨끗한 테스트는 다섯가지 규칙을 따른다.

1. Fast
테스트는 빨라야 한다. 빨리 돌아야한다는 의미이다. 테스트가 느리면, 자주 돌릴 엄두를 못내고 자주 돌리지 않다보면 코드의 품질이 망가지기 시작한다.

2. Independent
각각의 테스트는 서로 의존하면 안된다. 한 테스트가 다음 테스트가 실행될 환경을 준비해서는 안된다. 테스트가 서로에 의존하게되면 후반 테스트가 찾아낼 결함이 숨겨지게된다.

3. Repeatable
테스트는 어떤 환경에서든 반복 가능해야한다. 테스트가 돌아가지 않는 환경이 하나라도 있다면 테스트가 실패한 이유를 둘러댈 변명이 생긴다.

4. Self-Validating
테스트는 Bool 값으로 결과를 내야한다. 무조건 성공 아니면 실패이다.

5. Timely
테스트는 적시에 작성해야한다. 단위테스트는 테스트하려는 실제 코드를 구현하기 직전에 구현한다. 실제 코드를 짜고난 뒤에 테스트코드를 만들면, 실제 코드가 테스트하기 어렵다는 사실을 발견할 것이다.

<br>
<br>