# week_7

### 목차

- 아이템 27: 변화로부터 코드를 보호하려면 추상화를 사용하라
- 아이템 28: API 안정성을 확인하라
- 아이템 29: 외부 API를 랩(wrap)해서 사용하라
- 아이템 30: 요소의 가시성을 최소화하라
- 아이템 31: 문서로 규약을 정의하라
- 아이템 32: 추상화 규약을 지켜라

---

<br>
<br>

### 아이템 27: 변화로부터 코드를 보호하려면 추상화를 사용하라

## 추상화 방식 [1] 상수

```kotlin
const val MAX_THREADS = 10
```

- 이름을 붙일 수 있다.
- 나중에 값을 쉽게 변경 가능하다.

## 추상화 방식 [2] 함수

```kotlin
fun Context.showMessage(
    message: String,
    duration: MessageLength = MessageLEngth.LONG
) {
    val toastDuration = when(duration) {
        SHORT -> Length.LENGTH_SHORT
        LONG -> Length.LENGTH_LONG
    }
    Toast.makeText(this, message, toastDuration).show()
}

enum class MessageLength { SHORT, LONG }
```

- 함수는 상태가 없고, 시그니처를 변경하면 프로그램 전체에 큰 영향을 준다는 단점이 있다.

## 추상화 방식 [3] 클래스

```kotlin
class MessageDisplay(val context: Context) {
    fun show(
        message: String,
        duration: MessageLength = MessageLEngth.LONG
    ) {
        val toastDuration = when(duration) {
            SHORT -> Length.LENGTH_SHORT
            LONG -> Length.LENGTH_LONG
        }
        Toast.makeText(context, message, toastDuration).show()
    }
}
enum class MessageLength { SHORT, LONG }

// 사용
val messageDisplay = MessageDisplay(context)
messageDisplay.show("Message")
```

- 상태를 가질 수 있고, 많은 함수를 가질 수 있어 강력하다.
- 의존성 주입 프레임워크를 이용하면 클래스 생성을 위임할 수도 있다.
- mock 객체를 활용해서 테스트할 수 있다.

## 추상화 방식 [4] 인터페이스

- 코틀린은 거의 모든 것을 인터페이스로 표현한다.
  - ex) `listOf`은 `List` 인터페이스를 리턴한다.
- 인터페이스 뒤에 객체를 숨겨 구현을 추상화하고 사용자가 추상화된 것에만 의존하도록 만들어서 결합을 줄인다.
- 테스트 시 모킹보다 간단하게 인터페이스 페이킹을 사용할 수 있다. 

## 추상화의 문제

- 추상화는 거의 무한하게 할 수 있다.
- 그러나 너무 많은 것을 숨기면 결과를 이해하는 것 자체가 어려워진다.
- 추상화를 이해하려면 예제(단위 테스트, 문서)를 잘 살펴보자.

## 결론

- 추상화가 너무 많지도 적지도 않게 균형을 잘 유지하자.
- 팀의 크기/경험, 프로젝트의 크기, feature set, 도메인 지식 등에 따라서 추상화 정도를 조절하자.

---

<br>
<br>

### 아이템 28: API 안정성을 확인하라

### 안정적이고 최대한 표준적인 API를 사용하자.

- 라이브러리의 작은 변경이 이를 활용하는 다른 코드에 영향을 미친다.
- 사용자가 새로운 API를 배워야 한다.

### API가 불안정하다면, 명확하게 알려주자.

- 시멘틱 버저닝(Semantic Versioning)
  - MAJOR : 호환되지 않는 수준의 변경
  - MINOR : 이전 변경과 호환되는 기능 추가
  - PATCH : 간단한 버그 수정
- 어노테이션 이용하기
  - `@Experimental`: 안정적이지 않음
  - `@Depreated`: 더 이상 사용되지 않음 (단, 사용자가 적응할 시간을 충분히 주자.)
    - `ReplaceWith`: 직접적인 대안 제시

메이저 버전이 0인경우(0.y.z) 초기 개발 전용버전을 의미한다. 따라서 언제든지 변경될 수 있고, 안정적이지 않다는 의미

### 질문
- 시맨틱 버저닝.. 정말 잘 지키고 있는가?

---

<br>
<br>

### 아이템 29: 외부 API를 랩(wrap)해서 사용하라

API를 설계한 개발자가 안전하지 않다고 하거나, 우리가 그것을 제대로 신뢰할 수 없다면 해당 API는 불안정한 것.
어쩔 수 없이 이런 API를 활용해야 한다면, 최대한 로직과 직접 결합시지키 않는 것이 좋다.

→ 외부 API를 wrap해서 사용하자. 이를 사용하면 다음과 같은 자유와 안전성을 얻을 수 있음.

## API 랩의 장점

- 문제가 있다면 래퍼 만 수정. API 변경에 쉽게 대응
- 프로젝트 스타일에 맞게 적용 가능
- 라이브러리 오류에 대한 능동적인 대처

## API 랩의 단점

- 래퍼를 매번 정의
- 새로운 사람이 작업할때, 래퍼의 내용을 모두 이해해야 함
- 내부에서 사용하는 래퍼들이므로 외부의 도움을 받을 수 없다.

> 현실은 래퍼를 하더라도, 외부 API 변경 시 새로 작업

---

### 아이템 30: 요소의 가시성을 최소화하라

클래스의 상태를 나타내는 프로퍼티를 외부에서 변경할 수 있다면, 클래스는 자신의 상태를 보장할 수 없다.

해당 클래스가 만족하는 규약이 있으므로, 이를 모르는 다른 사람들은 클래스의 상태를 마음대로 변경할 수 있음.

→ 클래스의 불변성이 무너질 가능성이 있다.

```kotlin
class Counter {
    var elementsAdded: Int = 0
        private set

    fun add(num: Int): Int {
        elementsAdded++
        return elementsAdded
    }
}
```

- 일반적으로 코틀린은 접근자의 가시성을 제한하여 모든 프로퍼티를 캡슐화하는 것이 좋다.
  - 외부에선 함수를 통해서 값을 핸들링
- 서로 의존하는 프로퍼티가 있을 땐 객채 상태를 보호하는 것이 더 중요하다.
  - 프로퍼티가 노출될 경우 예상하지 못한 변경에 의해 예외가 발생
- 가시성이 제한될수록 클래스의 변경을 쉽게 추적할 수 있고, 프로퍼티의 상태를 더 쉽게 이해할 수 있다.
  - 상태 변경은 병렬 프로그래밍에서 문제가 발생한다. 많은 것을 제한할수록 병렬 프로그래밍을 할 때 안전해짐.

### 가시성 한정자 사용하기

- public(디폴트) : 어디에서나 볼 수 있음
- private : 클래스 내부에서만 볼 수 있음
- protected : 클래스와 서브 클래스 내부에서만 볼 수 있음
  - 프로퍼티에만 적용 클래스에선X
- internal : 모듈 내부에서만 볼 수 있음

top level 요소에는 세가지 가시성 한정자를 사용할 수 있다.

- public(디폴트) : 어디에서나 볼 수 있음
- private : 같은 파일 내부에서만 볼 수 있음
- internal : 모듈 내부에서만 볼 수 있음

```kotlin
모듈이 다른 모듈에 의해 사용될 가능성이 있다면, internal을 사용할 것.
```

### 정리

가시성은 최대한 제한적인 것이 좋다

- 최대한 제한이 되어 있어야 변경하기 쉽다.
- 클래스의 상태를 나타내는 프로퍼티가 노출되어 있다면, 클래스가 자신의 상태를 책임질 수 없다.
- 가시성이 제한되면 API의 변경을 쉽게 추적할 수 있다.

## 가시성 한정자 사용하기

- public
- private
- protected
- internal

---

<br>
<br>

### 아이템 31: 문서로 규약을 정의하라

- 함수로 모든 개발자들이 사용하는 기능을 만들었다해도 문서화가 제대로 되어 있지 않다면 명확하게 어떠한 기능을 하는지 알아보기 힘듦.
  - 함수와, 클래스가 이름만으로 예측할 수 없는 세부사항에 대해서
- 함수가 무엇을 하는지 명확하게 설명하고 싶다면 `KDoc` 주석을 붙여 주는 것이 좋다. → 밑에서 더 설명.

### 규약

어떤 행위(ex 클래스, 함수)를 구현하기 전 지켜야하는 일종의 약속

### 규약 정의하기

다양한 방법이 있지만, 대표적인 몇 가지를 정리해 보자.

- 이름 : 함수나 클래스의 이름으로 동작 예측이 가능하도록 설계
- 주석과 문서 : 모든 규약을 적을 수 있는 방법
- 타입 : 해당 객체에 대해서 많은 것을 알려줌

### 주석을 써야할까?

- 함수 이름과 파라미터만으로 정확하게 표현되는 요소에는 따로 주석을 넣지 않는 것이 좋다.
- 주석을 다는 것보다 함수로 추출하는 것이 훨씬 좋다.

```kotlin
fun update() {
    updateUser()
    updateBooks()
}
```

- 함수들의 규약 설명과, 최소한의 설명 등을 할 때 주석은 굉장히 유용하다.
  - ex) `listOf` 함수의 설명
    - “read-only 그리고, 직렬화할 수 있는 List를 리턴”

### KDoc 형식

- 주석으로 함수를 문서화할 때 사용되는 공식적인 형식.
- 모든 주석은 `/**` 로 시작해서 `*/` 로 끝난다.
- 모든 줄은 일반적으로 `*` 로 시작한다.
- 첫 번째 부분은 요소에 대한 요약 설명이다.
- 두 번째 부분은 상세 설명이다.
- 이어지는 줄은 모두 태그로 시작한다.
  - 추가적인 설명을 위해 사용

```kotlin
/**
 * 트리 자료 구조(immutable) -> [요약 설명]
 * 
 * 1개 이상의 요소를 갖는 트리를 나타내는 클래스입니다.(immutale) -> [상세 설명]
 * 트리의 노드는 요소를 가질 수 있으며,
 * 또한 왼쪽과 오른쪽에 서브 트리를 가질 수 있습니다.
 * 
 * @param T 트리가 갖는 요소의 타입을 지정합니다. -> [태그]
 * @property value 트리의 현재 노드에 할당할 값을 의미합니다.
 * 
 */
```

모든 것을 설명할 필요는 없다. 짧으면서 명확하지 않은 부분을 자세하게 설명하는 문서가 좋은 문서이다.

### 타입 시스템과 예측

- 리스코프 치환 원칙 : 클래스가 어떤 동작을 할 것이라 예측되면, 그 서브클래스도 이를 보장해야 한다.
  - S가 T의 서브타입이라면, 변경이 없어도 T타입 객체를 S타입 객체로 대체할 수 있어야함.
- 사용자가 클래스의 동작을 확실하게 예측할 수 있게 하려면 인터페이스를 활용하여 규약을 잘 정의해야함.
- 설명과 규약은 인터페이스를 더욱 유용하게 만듬.

### 정리

- 외부 API를 구현할 때는 규약을 잘 정의해야 한다.
- 규약은 사용자가 객체를 사용하는 방법을 쉽게 이해하는 등 요소를 쉽게 예측할 수 있게 해준다.
- 규약은 어떻게 동작하고, 앞으로 어떻게 동작할지를 사용자에게 전달해 준다.

### 주요 태그

- @param <name> 함수 파라미터 또는 클래스, 프로퍼티, 함수 타입 파라미터를 문서화
- @return 함수의 리턴 값을 문서화
- @throws 메서드 내부에서 발생할 수 있는 예외에 대한 문서화
- @see 특정한 클래스 또는 메서드에 대한 링크 추가. 문서 내용 중간에서 링크가 필요하다면 `[]`로 대체 가능

---

### 아이템 32: 추상화 규약을 지켜라

- 규약은 개발자들의 단순한 합의이다.
- 한쪽에서 규약을 위반할 수도 있다. → 규약 위반 발생

리플렉션을 활용하면, 우리가 원하는 것을 열고 사용할 수 있다.

→ ex) 클래스 외부에서 private 함수 호출

```kotlin
import kotlin.reflect.full.declaredMemberFunctions
import kotlin.reflect.jvm.isAccessible

class Employee {
    private val id: Int = 2
    override fun toString(): String = "User(id = $id)"

    private fun privateFunction() = println("Private function called")
}

fun callPrivateFunction(employee: Employee) {
    employee::class.declaredMemberFunctions
        .first { it.name == "privateFunction" }
        .apply { isAccessible = true }
        .call(employee)
}

fun changeEmployeeId(employee: Employee, newId: Int) {
    employee::class.java.getDeclaredField("id")
        .apply { isAccessible = true }
        .set(employee, newId)
}

fun main() {
    val employee = Employee()
    callPrivateFunction(employee)
    changeEmployeeId(employee, 1)
    print(employee)
}
// Private function called
// User(id = 1)
```

무언가를 할 수 있다는 것이 그것을 해도 괜찮다는 의미가 아님.

### 상속된 규약

- 모든 클래스는 `equals` 와 `hashCode` 메서드를 가진 `Any` 클래스를 상속받는다.
- 이러한 메서드는 우리가 반드시 지켜야하는 규약을 가지고 있다.

규약을 지키지 않는 경우, 객체가 제대로 동작하지 않을 수 있다.

- `equals` 가 구현이 안되어 중복을 허용하는 경우
- `hashCode` 가 제대로 구현되지 않아 Hash함수를 사용하는 경우
- 등

### 정리

- 프로그램을 안정적으로 유지하자면, 규약을 지키자.
- 규약을 깰 수 밖에 없다면, 문서화를 잘하자.

---
