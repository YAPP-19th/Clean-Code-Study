## 4장. 주석


제일 좋은 것은 주석 없이도 읽는 사람으로부터 이해하기 쉬운 코드를 짜는 것. 차선책은 잘 달린 주석으로 인해 코드의 이해도를 높히는 것. 최악은 의도에 맞지않는, 거짓말을 하는 주석을 달아놓는 것이다. 

이 책에서는 대체로 주석은 시간이 지날수록 거짓말을 한다는 부정적인 입장을 가지고 있다. 여러사람들의 손을 거치면서 코드가 변하게 되는 과정을 겪을텐데 이 때 주석은 챙겨지지 않는 경우가 대다수다. 처음에는 그럴싸한 주석일지라도 나중에는 코드를 설명하는 주석은 커녕 무의미한 줄낭비가 된다는 것이다. 그래서 주석은 매우 엄격하게 관리되어야하고 그 정확성이 높아야 한다는 의미이다. 

차라리 주석을 잘 적어야한다는 생각보다는 주석을 최대한 줄이고 정확한 코드를 제공해야한다는 마음가짐을 가지고 있어야한다. 주석을 추가하는 행동은 코드의 품질이 나쁘다는 것을 내포하고 있고, 잘 적어놓은 설명으로 코드를 포장해 그럴싸하게 보이기 위한 위장수단이다. 

그럼에도 불구하고, 주석이 필요한 경우가 존재할 수 있는데 이 경우 어떻게 적는 것이 `좋은 주석` 인가에 대해 적어보고자 한다.
<br>

**1. 정보를 제공하는 주석**

``` kotlin 
// kk:mm:ss EEE, MMM dd, yyyy 형식
val timeMatcher = Pattern.compile(
"\\d*:\\d*:\\d* \\w*, \\w* \\d*, \\d*")
```

시각과 날짜를 표현하는 정규표현식이다. 이 때, 어떤식으로 보여지는지 미리 주석으로 적어놓는다면 정확한 정보를 제공할 수 있다.
<br>

**2. 의도를 설명하는 주석**

``` kotlin
// ex1
// 스레드를 대량으로 생성하는 방법으로, 어떻게든 경쟁 조건을 만드려고 시도한다.
for (i in 0 .. 25000){
  val widgetBuilderThread = WidgetBuilderThread(widgetBuilder, text, parent, failFlag)
  val thread = Thread(widgetBuilderThread)
  thread.start()
}
…
// ex2
val b = PathParser.parse(“Page B”)
val a = PathParser.parse(“PageA”)
assertTrue(a.compareTo(a) == 0) // a == a
assertTrue(a.compareTo(b) != 0) // a != b
```

이 주석은 이 코드블럭이 왜 존재하는지에 대한 이유와 의도가 분명히 드러난다. 
<br>

**3. 결과를 경고하는 주석**

``` kotlin
// 여유시간이 없다면 실행하지 마세요.
fun testWIthReaalyBigFile() {
  fun writeLinesToFile(100000L)
  
  response.body = testFile
  response.readyToSend = this
  val responseString = output.toString()

  }
```

해당 주석은 이 함수를 사용하는 프로그래머에게 경고하는 목적으로 사용되었다. 
<br>

**4. TODO 주석**

``` kotlin
// Todo 현재 필요하지 않다. 
fun makeVersion(): VersionInfo? { return null }
```

앞으로의 할일을 주석으로 남겨두는 것도 좋은 방법이다. 하지만 어떤 용도로 사용하던, 시스템에 나쁜 코드를 남겨놓는 핑계가 되어서는 안된다.
<br>

**5. 중요성을 강조하는 주석**

``` kotlin
// 문자열 시작 시 공백이 있으면, 다른 문자열로 인식되기 때문에 trim() 은 정말 중요하다.
val listItemContent = match.group(3).trim()
…
```
<br>

그렇다면, 나쁜 주석이란 것은 무엇일까?


**1. 같은 이야기를 중복하는 주석**

``` kotlin 
// ex1
// 프로세스 지연 값
protected val backgroundProcessorDelay = -1

// 컴포넌트를 지원하기 위한 생명주기 이벤트
protected lifecycle = LifecycleSupport(this)

// 컴포넌트를 위한 컨테이너 이벤트 리스너
protected listener = ArrayList()

…

// ex2
// this.close 가 true일 때 반환되는 유틸리티 함수이다. 
// 타임아웃에 도달하면 예외를 던진다.
@Synchronized fun waitForClose(timeoutMillis: Long) {
    if(closed.not()) {
        wait(timeoutMillis)
        if(closed.not()) {
            throw Exception("MockResponseSender could not be closed!")
        }
    }
}
```

ex1, ex2는 해당 주석은 코드의 내용을 그대로 적은 것이다. 자칫 코드보다 주석을 읽는 시간이 더 오래걸릴 수 있다. 주석은 코드보다 더 많은 정보를 제공할 수 없기 때문에, 이런경우는 주석이 불필요하다.

함수나 변수로 표현할 수 있다면 주석을 달지 마라.

``` kotlin 
// 전역 목록 <smodoule> 에 속하는 모듈이 우리가 속한 하위시스템에 의존하는가?
if (smodule.getDependSubsystems().contains(subSysMod.getSubSystem())
```

이 코드에서 주석을 없애고 다시 표현하면 아래와 같다.

``` kotlin 
val moduleDependences = module.getDepencendSubsystems()
val ourSubSystem = subSysMod.getSubSystem()
if (moduleDependences.contains(ourSubSystem)
```

주석을 추가하지 않으면서 개선할 수 있다면, 그렇게하는 편이 좋다.
<br>

**2. 모호한 관계**

``` kotlin 
// 모든 픽셀을 담을 만큼 충분한 헤더로 시작한다 (여기에 필터 바이트를 더한다.)
// 그리고 헤더 정보를 위해 200 바이트를 더한다.
val pngByte = ArrayList<Int> ((width+1 * height*3) + 200)
```

여기서 필터 바이트란 무엇을 의미할까? +1 혹은 *3 과 관계가 있을까? 해당 주석으로 완전한 설명이 불가능해진다. 오히려 주석 자체가 다시 설명을 요구하는 상황이 되어버린다.

<br>

## 5장. 형식 맞추기

팀으로 일하게 된다면 일관성있는 코드를 유지하기 위해 규칙을 정하고 모두가 그 규칙을 따른다. 어떠한 형식이 읽기 더 좋은지 살펴보자.

<br>

**1. 신문기사처럼 작성하라**

독자는 신문기사를 위에서 아래로 읽는다. 좋은 신문기사는 최상단에 요약하는 표제가 나오고, 독자는 이 표제를 보고 기사를 읽을지 말지 결정한다. 기사의 첫 문단에는 전체 기사를 요약하고 아래로 내려갈수록 세세한 그림(날짜, 이름, 발언, 주장 등등)을 보여주고 있다.

신문기사를 보면 한 면을 채우는 기사는 거의 없다. 무작위로 뒤섞어진 긴 기사를 하나 싣는다면 아무도 읽지 않을 것이다. 코드도 그렇게 구성해야한다. 클래스는 짧은 코드라인수를 가지고 있는 것이 가독성이 좋으며, 소스코드의 첫 부분은 고차원의 개념이, 아래로 내려갈수록 그 의도를 세세하게 설명하는 구조를 이루어야 한다. 
<br>

**2. 세로 밀집도**

* 우리는 줄바꿈을 이용해 새로운 개념을 시작한다는 의미를 줄 수 있다.
* 서로 밀접한 코드 행은 세로로 가까이 두어야 한다. 두 개념이 다른 파일에 속한다면 규칙이 통하지 않지만, 타당한 근거가 없다면 밀접한 개념은 한 파일에 속해야한다. 이것이 바로 protected 키워드를 피해야하는 이유이다. 밀접한 개념은 세로거리의 연관성을 표현한다. 연관성이 깊은 개념들이 떨어져있다면, 코드를 읽는 사람이 여러 파일을 찾게 될 가능성이 크다.
* 변수는 사용하는 곳에서 최대한 가까운 곳에 정의한다.
* 인스턴스 변수는 클래스의 맨 처음에 선언한다.
* 함수가 다른 함수를 호출한다면, 두 함수는 세로로 가까이 배치한다.
<br>

**3. 가로 밀집도**

* 가로로는 공백을 사용해 밀접한 개념과 느슨한 개념을 표현한다.
``` kotlin
fun measureLine(line: String) {
  lineCount ++
  val lineSize = line.length
  totalChars += lineSize
}
```
할당 연산자를 강조하기 위해 앞뒤에 공백을 주었다. 하지만 함수 이름과 이어지는 괄호 사이에는 공백이 없다. 함수와 인수는 인접하기 때문이다. 공백을 넣게되면 한 개념이 아니라 별개로 보이게 된다.

* 때로는 간단한 if, while문을 들여쓰기 없이 한줄로 쓰려고 하는 경우가 발생하는데, 이보다 들여쓰기를 제대로 표현한 코드가 읽기 더 쉽다.

## 6장. 객체와 자료구조

변수를 private 으로 정의하는데는 이유가 있다. 개발자가 변수 타입이나 구현을 맘대로 바꾸고 싶은 욕구가 들게 되는데, 이 때 변수에 의존하지 않게 만들기 위해서다. 그렇다면 get/set 함수는 왜 public 으로 노출하는 것일까?
<br>

**1. 자료 추상화**

Point interface 와 Point class 의 차이를 보자.

``` kotlin
class Point {
  val x: Double
  val y: Double
}
```

```kotlin
interface Point {
  fun getX(): Double
  fun getY(): Double
  fun setCartesian(x: Double, y: Double)
  fun getR(): Double
  fun getTheta(): Double
  fun setPolar(r: Double, theta: Double)
}
```

Point class 는 구현을 외부로 노출하고, Point interface 는 구현을 완전히 숨긴다. interface 의 경우에는 코드를 보면서도 점들이 어떤 좌표계를 표준으로 두고 사용하는지 알 수 없다. 그럼에도 불구하고 자료구조를 명백하게 표현하고 있다. Point class 의 경우에는 **개별적으로 좌표를 하나씩 설정**할 수 있도록 두었다면 interface 의 경우에는 **두 좌표는 개별적으로 읽고, 설정은 한꺼번에 진행**해야한다. 

변수 사이에 함수라는 계층을 넣는다고 구현이 저절로 감추어지지 않는다. get/set 함수를 다룬다고해서 클래스가 완성되는 것이 아니다. 그보다 더 추상 인터페이스를 제공해 사용자가 구현을 모른 채 핵심을 조작할 수 있어야 진정한 의미의 클래스가 완성되는 것이다. 

``` kotlin
interface Vehicle {
  fun getFuelTankCapacityInGellons(): Double //1
  fun getGallonsOfGasoline(): Double //2
  
  fun getPercentFuelRemaining(): Double //3
}
```
위 세가지의 함수를 담고있는 Vehicle 인터페이스가 있다고 가정해보자. 1, 2번의 경우는 자동차의 연료 상태를 구체적인 수치로 알려주고 있지만 3번의 경우는 백분율이라는 추상적인 개념으로 알려주고 있다. 즉 1, 2번의 의미는 Vehicle 을 구현하게 될 클래스에서 연료 상태를 나타내는 변수를 그대로 반환(get)하는 역할만 수행하기 때문에 추상화가 이뤄지지 않는다. 

<br>

**2. 자료/객체 비대칭**

1, 2, 3번의 함수들은 객체와 자료구조 사이에 벌어진 차이를 보여준다. 객체는 추상화 뒤로 자료를 숨긴 채 **자료를 다루는 함수만 공개**한다. 자료구조는 자료를 그대로 공개하며 별다른 **함수를 제공하지는 않는다.** 두 개념이 정반대를 이루고 있다.

``` kotlin
interface Shape
data class Square(
  val topLeft: Point,
  val side: Double
): Shape

data class Rectangle(
  val topLeft: Point,
  val height: Double,
  val width: Double
): Shape

data class Circle(
  val center: Point,
  val radius: Double
): Shape

class Geometry {
    companion object {
        const val PI = 3.141592f
    }

    fun area(shape: Shape): Double {
        try {
            if (shape is Square) {
                return shape.side * shape.side
            } else if (shape is Rectangle) {
                return shape.height * shape.width
            } else if (shape is Circle) {
                return PI * shape.radius * shape.radius
        } catch (e: Exception) {
            // throw
        }
    }
}
```
위 예제에서는 Geometry 클래스가 세가지의 도형클래스를 다루고 있다. 각 도형 클래스는 자료구조 형태이며  아무 메서드도 구현되어있지 않고, 도형이 동작하는 방식은 Geometry 에서 구현되고있다. 만약 Geometry 클래스에서 둘레 길이를 구하는 함수를 추가하고 싶은 경우가 생길 때는 Geometry 클래스에만 추가하면 된다. 하지만 새로운 도형을 추가해야할 때는 Geometry 클래스에 속한 모든 함수를 고쳐야한다. 이런 형태를 절차적인 코드라고 하는데 절차적인 코드는 자료구조 형태는 기존 형태를 유지하기 때문에 함수의 추가는 간편하지만, 새로운 클래스(자료구조)를 추가하는 것은 어렵다. 

``` kotlin
data class Square(
    val topLeft: Double,
    val side: Double
) : Shape {
    override fun area(): Double {
        return side * side
    }
}

data class Rectangle(
    val topLeft: Point,
    val height: Double,
    val width: Double
) : Shape {
    override fun area(): Double {
        return height * width
    }
}

interface Shape {
    fun area(): Double
}
```
반면에 위 예제에서는 각 클래스가 area() 라는 함수의 구현체를 가지고 있어서, Geometry 클래스를 가질 필요가 없다. 새 도형을 추가해도 기존 함수에 영향을 미치지 않는다는 의미이다. 하지만, 새 함수를 추가하고 싶은 경우에는 모든 클래스를 수정해야하는 상황이 발생한다. 이런 형태를 객체지향 코드라고 하는데, 객체지향 코드는 기존 함수를 변경하지 않으면서 새 클래스 추가는 쉽지만 새로운 함수의 추가는 어렵다. 
<br>

**3. 디미터 법칙**
디미터 법칙은 다른 객체가 어떠한 자료를 갖고 있는지 속사정을 몰라야 한다는 것을 의미이며, "클래스 C의 메소드f 는 다음과 같은 객체의 메서드만 호출해야한다" 라고 주장한다
* 클래스 C
* f가 생성한 객체
* f인수로 넘어온 객체
* C 인스턴스 변수에 저장된 객체

이 의미에서 아래의 outputDir 은 디미터 법칙을 어기고 있는 코드이다.
``` kotlin
val outputDir = context.getOptions().getScratchDir().getAbsoultePath()
```
context 뒤에 호출되는 메서드들이 반환하는 객체를 이용해 다음 메서드를 호출하고 있는 형태이기 때문이다. 이 방식은 여러 객차가 한줄로 이어진 기차처럼 보여서 기차충돌이라고도 불리게 되는데, 조잡해보일 수 있기 때문에 피하는 것이 좋다. 

객체 지향 프로그래밍에서 가장 중요한 것은 "객체가 어떤 데이터를 가지고 있는가?"가 아니라, **"객체가 어떤 메세지를 주고 받는가?"** 이다. 때문에 위 코드는 외부 클래스에게 어떠한 데이터를 가지고 있는지를 노출하는 형태이므로, 좋은 프로그래밍이 아니다. (context가 options 를 포함하고, options가 scratchDir을 포함하고 scratchDir이 absoultePath를 포함하고 있는 것을 알려주고 있음)

하지만, 위 코드가 자료구조라면 이야기가 달라진다. 자료구조는 내부 구조를 노출하는 형태이기 때문에 디미터 법칙이 적용되지 않는다. 그렇기에 아래와 같이 코드가 작성되어있다면 디미터 법칙을 거론할 필요가 없어지게 된다.
``` kotlin
val outputDir = context.options.scratchDir.absoultePath
```

만약 아래와 같은 코드가 있다고 가정해보자. 
``` kotlin
// as is 
context.getAbsolutePathOfScrathDirectoryOptions()
```
context에게 무엇인가를 하라고 요구하는 코드가 되어버린다. 이 코드 또한 어떠한 데이터를 가지고 있다는 것을 암시하기 때문에 지양해야하는 코드이다. 그렇다면 어떻게 개선하면 좋을까? 

이건 절대경로를 얻으라는 get 함수인데, 첫번째로 절대경로를 얻는 이유를 알아보면 좋을 것 같다. 만약 절대경로를 얻으려는 이유가 임시 파일을 생성하기 위해서라면 context에게는 임시파일을 생성하라는 지시를 내리면 내가 원하는 목적도 수행할 것이고, 데이터를 감출수도 있게 될 것이다. 디미터 법칙을 지키면서 원하는 바를 이뤄낸 것이다. 
``` kotlin
// to be
context.createScratchFileStream(fileName)
```


<br>


우리가 새로운 시스템을 구현하게 될 때, 자료구조 혹은 객체 중 상황에 맞게 사용하되 그 의도에 맞게 설계하는 것이 중요하다. 자료구조와 객체는 서로 상반되는 각자의 장점과 단점을 가지고 있는데, 설계를 하다보면 나도 모르게 양쪽의 장점만을 채택한 절반은 객체, 절반은 자료구조인 형태를 만들게되는 경우가 존재하게 될 것이다. 사실 이것은 양쪽 구조에서 단점만을 가져온 잡종 구조일 뿐이며, 어중간한 설계에 불가하니 더욱 주의하자.