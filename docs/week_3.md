# Week 3

### 목차

- 아이템 10: 단위 테스트를 만들어라
- 아이템 11: 가독성을 목표로 설계하라
- 아이템 12: 연산자 오버로드를 할 때는 의미에 맞게 사용하라
- 아이템 13: Unit?을 리턴하지 말라

### 아이템 10: 단위 테스트를 만들어라

- 단위 테스트로 확인하는 내용
  - 일반적인 유스 케이스
  - 일반적인 오류 케이스와 잠재적인 문제: 제대로 동작하지 않을 거라고 예상되는 부분, 과거 문제가 발생했던 부분
  - 에지 케이스와 잘못된 아규먼트
- 단위 테스트의 장점
  - 코드에 대한 신뢰감
  - 리펙토링
  - 생산성 향상
- 단위 테스트의 단점
  - 시간소요 - 장기적으로는 시간 단축
  - 테스트를 작성할 수 있게 코드를 조정해야함 - 이러한 변경은 좋은 아키텍처로 이끌어줌
  - 좋은 단위 테스트를 만드는 작업이 어려움

### 아이템 11: 가독성을 목표로 설계하라

**인식 부하를 줄이자...**

```
if (person != null && person.isAdult) {
            view.showPerson(person)
        } else {
            view.showError()
 }
person?.takeIf { it.isAdult }
            ?.let(view::showPerson)
            ?: view.showError()
```kotlin

- 일반적인 관용구 사용은 이해하기 쉽다
- 코틀린에서 자주 사용되는 관용구 takeIf,let,엘비스 연산자는 코틀린 개발자는 익숙하지만, 숙련된 개발자만을 위한 코드는 좋은 코드가 아니다
  - takeIf
  - public inline fun <T> T.takeIf(predicate: (T) -> Boolean): T? = if (predicate(this)) this else null
  - let
  - inline fun <T, R> T.let(block: (T) -> R): R { return block(this) }
- 첫번째 예제는 두번째 예제보다 수정이 용이하다
  - 함수 참조를 사용할 수 없으므로 람다식 수정 필요
- let의 리턴값이 null이면 showError가 호출된다. 첫번째 예제와 다르게 동작할 수 있음

**극단적이 되지 않기**
- let
  - 연산을 아규먼트 처리 후로 이동시킬 때
  - print(students.filter{}.joinToString{}) -> students.filter{}.joinToString{}.let(::print)
  - 데코레이터를 사용해서 객체를 랩할 때
  - var obj = FileInputStream("/file.gz") .let(::BufferedInputStream) .let(::ZipInputStream) .let(::ObjectInputStream) .readObject() as SomeObject

**컨벤션**
- 사람에 따라서 가독성에 대한 관점이 다르다는 것을 알아봤다. 이를 이해하고 기억해야하는 몇 가지 규칙이 있다.

```
val abe = "A" { "B" } and "C"
print(abc) // ABC
operator fun String.invoke(f: ()->String): String = this + f()


infix fun String.and(s: String) = this + s
```kotlin

- 연산자는 의미에 맞게 사용해야 한다. invoke를 이러한 형태로 사용하면 안된다.
- '람다를 마지막 아규먼트로 사용한다'라는 컨벤션을 여기에 적용하면, 코드가 복잡하다. invoke 연산자와 함께 이러한 컨벤션을 적용하는 것은 신중해야 한다.
- 현재 코드에서 and라는 함수 이름이 실제 함수 내부에서 이루어지는 처리와 맞지 않다
- 문자열을 결합하는 기능은 이미 언어에 내장되어 있다. 이미 있는 것을 다시 만들 필요없다.

### 아이템 12: 연산자 오버로드를 할 때는 의미에 맞게 사용하라
- 연산자 오버로딩은 강력한 기능이지만, 책임이 따른다.

```
fun Int.factorial(): Int = (1..this).product()
fun Iterable<Int>.product(): Int = fold(1) { acc, i -> acc * i }

operator fun Int.not() = factorial()
```kotlin

- not은 논리 연산에 사용해야지, 팩토리얼 연산에 사용하면 안된다.
- 코틀린에서 각 연산자의 의미는 항상 같게 유지해야 된다.

**분명하지 않은 경우**
infix를 활용한 확장 함수를 사용하자

```
infix fun Int.timesRepeated(operation: ()->Unit)={
      repeat(this) { operation() }
}

3 timeRepeated { print("Hello")}
```kotlin

- 톱 레벨 함수를 사용하자
  - 최상위 함수, 클래스로 묶지 않은..

**규칙을 무시해도 되는 경우**
- DSL를 설계할 때는 연산자 오버로딩 규칙을 무시해도 된다. [js dsl](https://kotlinlang.org/docs/typesafe-html-dsl.html)
```
body{
  div{
+"Some text"
}
}
```

### 아이템 13: Unit?을 리턴하지 말라
```
// Boolean 으로 사용하는 경우
fun keyIsCorrect(key: String) : Boolean = // …
if(!keyIsCoreect(key) return

// 같은 코드를 Unit?으로 사용하는 경우
fun verifyKey(key:String): Unit? = //…
verifyKey(key) ?: return
```kotlin
Unit? 으로 Boolean을 표현하는 것은 오해의 소지가 있으며, 예측하기 어려운 오류를 만들 수 있다.

위 예제처럼 같은 역할로 사용이 가능하지만, 올바르지 않은 용례이기 때문에 Boolean을 사용하자.

---


### Infix
- 표기법은 연산자를 어디에 두냐에 따라 전위(prefix), 후위(postfix), 그리고 중위(infix)로 나뉘게 됩니다. 즉, infix라는 것은 두개의 값 사이에 중간에 특정한 표현식을 넣는다는 의미입니다.
- 코틀린에서는 이를 접목 시켜서 특별한 함수식을 만들어낼 수 있습니다. infix function이란 호출자인 점(.)과 파라미터 괄호()를 생략하고 함수명 만으로 호출할 수 있는 코틀린에서 제공해주는 표현식입니다.
- 아래 예제는 코틀린의 mockk라는 테스트 프레임워크에서 나오는 표현식입니다. 아래의 2개의 표현식은 완전동일합니다. 2번째 라인의 방식이 infix function을 이용한 방식입니다. 훨씬 가독성 측면에서 좋은것을 확인할 수 있습니다.

```
every { service.get() }.returns(5) // 일반 함수 호출 방법
every { service.get() } returns 5 // infix 함수 호출 방법
```kotlin

**예제**
- 이번에는 infix function을 만드는 방법에 대해서 알아보도록 하겠습니다. 먼저 커스텀하게 만들기 전에 코틀린에서는 기본적으로 사용할 수 있는 infix function을 살펴보도록 하겠습니다.

```
val pairs = mapOf(
    Pair(1, "a"),
    Pair(2, "b"),
    Pair(3, "c"),
)

print("pair[1] = ${pairs[1]}") // "a"

val pairs = mapOf(
    1 to "a",
    2 to "b",
    3 to "c"
)

print("pair[1] = ${pairs[1]}") // "a"

public infix fun <A, B> A.to(that: B): Pair<A, B> = Pair(this, that)
```kotlin

**infix function을 만들기 위한 제약 조건**

- 맴버 함수 또는 확장 함수
- 단일 파라미터
- 단일 파라미터로 여러개의 값을 받을 수 없고 기본값 설정 불가

```
private infix fun Int.add(value: Int): Int = this + value

val result: Int = 1 add 2

print("result = $result") // "result"
```kotlin
