# Week 2

### 목차

- 아이템 4: inferred 타입으로 리턴하지 말라
- 아이템 5: 예외를 활용해 코드에 제한을 걸어라
- 아이템 6: 사용자 정의 오류보다는 표준 오류를 사용하라
- 아이템 7: 결과 부족이 발생할 경우 null과 Failure를 사용하라
- 아이템 8: 적절하게 null을 처리하라
- 아이템 9: use를 사용하여 리소스를 닫아라

### 아이템 4: inferred 타입으로 리턴하지 말라

inferred type : 변수나 표현식의 타입을 컴파일러가 자동으로 추론하는 타입

- 코틀린 타입 추론을 사용할 때는 몇 가지 위험한 부분이 있는데, 이러한 위험한 부분을 피하려면 할당 시에 inferred 타입은 정확하게 오른쪽에 있는 피연산에 맞게 설정해야함
- 절대 수퍼 클래스 또는 인터페이스로는 설정되지 않음

**문제가 발생**
```kotlin
open class Animal
class Zebra: Animal()

fun main() {
    var animal = Zebra()
    animal = Animal()
}
```


**명시적으로 지정**
```kotlin
open class Animal
class Zebra: Animal()

fun main() {
    var animal: Animal = Zebra()
    animal = Animal()
}
```

그러나 직접 라이브러리를 조작할 수 없는 경우에는 이런 문제를 간단하게 해결할 수 없고 이러한 경우에서 inferred 타입을 노출하면 위험한 일이 발생할 수 있음
예시

```kotlin
val DEFAULT_CAR: Car = Flat126P()

interface CarFactory {
    fun produce() = DEFAULT_CAR 
}
```
- 위와 같이 CarFactory 인터페이스에서 DEFAULT_CAR를 리턴하는 메소드가 있다고 가정
- DEFAULT_CAR 는 Car로 명시적으로 지정되어 있어서 메소드 리턴 타입을 제거
- 그러나 이후 다른 사람이 코드를 보다가 DEFAULT_CAR 는 타입 추론에 의해 자동으로 타입이 지정될 것이므로 명시적으로 지정한 코드를 제거 하게 된다면 CarFactory는 Flat126P 이외의 자동차를 생산할 수 없게 됨
- 만약 인터페이스를 우리가 직접 만들었다면 문제를 쉽게 찾을 수 있지만 외부 API라면 쉽게 해결할 수 없기 때문에 리턴 타입을 명시적으로 지정해주는게 좋음

**정리**
- 타입을 확실하게 지정해야 하는 경우에는 명시적으로 타입을 지정
- 안전을 위해 외부 API를 만들 때는 반드시 타입을 지정하고 이렇게 지정한 타입을 특별한 이유와 확실한 확인 없이 제거하지 않는 것이 좋음
- inferred 타입은 프로젝트가 진전될 때, 제헌이 너무 많아지거나 예측하지 못하는 결과를 낼 수 있음


### 아이템 5: 예외를 활용해 코드에 제한을 걸어라

- 확실하게 어떤 형태로 동작해야 하는 코드가 있다면 예외를 활용해 제한을 걸어주는 것이 좋은데 코틀린에서는 코드의 동작에 제한을 걸 때 다음과 같은 방법을 사용할 수 있음


- require 블록 : 아규먼트 제한
- check 블록 : 상태와 관련된 동작 제한
- assert 블록 : 어떤 것이 true인지 확인 가능 그러나 테스트 모드에서만 동작
- return 도는 throw와 함께 활용하는 Elvis 연산자

관련 code
```kotlin
/*
 * Copyright 2010-2018 JetBrains s.r.o. and Kotlin Programming Language contributors.
 * Use of this source code is governed by the Apache 2.0 license that can be found in the license/LICENSE.txt file.
 */

@file:kotlin.jvm.JvmMultifileClass
@file:kotlin.jvm.JvmName("PreconditionsKt")

package kotlin

import kotlin.contracts.contract

/**
 * Throws an [IllegalArgumentException] if the [value] is false.
 *
 * @sample samples.misc.Preconditions.failRequireWithLazyMessage
 */
@kotlin.internal.InlineOnly
public inline fun require(value: Boolean): Unit {
    contract {
        returns() implies value
    }
    require(value) { "Failed requirement." }
}

/**
 * Throws an [IllegalArgumentException] with the result of calling [lazyMessage] if the [value] is false.
 *
 * @sample samples.misc.Preconditions.failRequireWithLazyMessage
 */
@kotlin.internal.InlineOnly
public inline fun require(value: Boolean, lazyMessage: () -> Any): Unit {
    contract {
        returns() implies value
    }
    if (!value) {
        val message = lazyMessage()
        throw IllegalArgumentException(message.toString())
    }
}

/**
 * Throws an [IllegalArgumentException] if the [value] is null. Otherwise returns the not null value.
 */
@kotlin.internal.InlineOnly
public inline fun <T : Any> requireNotNull(value: T?): T {
    contract {
        returns() implies (value != null)
    }
    return requireNotNull(value) { "Required value was null." }
}

/**
 * Throws an [IllegalArgumentException] with the result of calling [lazyMessage] if the [value] is null. Otherwise
 * returns the not null value.
 *
 * @sample samples.misc.Preconditions.failRequireNotNullWithLazyMessage
 */
@kotlin.internal.InlineOnly
public inline fun <T : Any> requireNotNull(value: T?, lazyMessage: () -> Any): T {
    contract {
        returns() implies (value != null)
    }

    if (value == null) {
        val message = lazyMessage()
        throw IllegalArgumentException(message.toString())
    } else {
        return value
    }
}

/**
 * Throws an [IllegalStateException] if the [value] is false.
 *
 * @sample samples.misc.Preconditions.failCheckWithLazyMessage
 */
@kotlin.internal.InlineOnly
public inline fun check(value: Boolean): Unit {
    contract {
        returns() implies value
    }
    check(value) { "Check failed." }
}

/**
 * Throws an [IllegalStateException] with the result of calling [lazyMessage] if the [value] is false.
 *
 * @sample samples.misc.Preconditions.failCheckWithLazyMessage
 */
@kotlin.internal.InlineOnly
public inline fun check(value: Boolean, lazyMessage: () -> Any): Unit {
    contract {
        returns() implies value
    }
    if (!value) {
        val message = lazyMessage()
        throw IllegalStateException(message.toString())
    }
}

/**
 * Throws an [IllegalStateException] if the [value] is null. Otherwise
 * returns the not null value.
 *
 * @sample samples.misc.Preconditions.failCheckWithLazyMessage
 */
@kotlin.internal.InlineOnly
public inline fun <T : Any> checkNotNull(value: T?): T {
    contract {
        returns() implies (value != null)
    }
    return checkNotNull(value) { "Required value was null." }
}

/**
 * Throws an [IllegalStateException] with the result of calling [lazyMessage]  if the [value] is null. Otherwise
 * returns the not null value.
 *
 * @sample samples.misc.Preconditions.failCheckWithLazyMessage
 */
@kotlin.internal.InlineOnly
public inline fun <T : Any> checkNotNull(value: T?, lazyMessage: () -> Any): T {
    contract {
        returns() implies (value != null)
    }

    if (value == null) {
        val message = lazyMessage()
        throw IllegalStateException(message.toString())
    } else {
        return value
    }
}


/**
 * Throws an [IllegalStateException] with the given [message].
 *
 * @sample samples.misc.Preconditions.failWithError
 */
@kotlin.internal.InlineOnly
public inline fun error(message: Any): Nothing = throw IllegalStateException(message.toString())
```


```kotlin
class Stack<T>(
    var collection: List<T>
) {
    private val size = 3
    private val isOpen = false

    fun pop(num: Int = 1): List<T> {
        require(num <= size) {
            "Cannot remove more elements than current size"
        }
        check(isOpen) { "Cannot pop from closed stack" }
        val ret = collection.take(num)
        collection = collection.drop(num)
        assert(ret.size == num)
        return ret
    }
}
```
위 코드의 장점

- 제한을 걸면 문서를 읽지 않은 개발자도 문제 확인 가능
- 문제가 있을 경우에 함수가 예상하지 못한 동작을 하지 않고 throw함
- 예상하지 못한 동작을 하는 것보단 throw 하는 것이 훨씬 안정적이고 관리하기 좋음
- 코드가 어느 정도 자체적으로 검사가 됨
- 스마트 캐스트 기능을 활용할 수 있게 되므로, 캐스트를 적게할 수 있음

**아규먼트**
- 함수 정의할 때 타입 시스템을 활용해서 아규먼트에 제한을 거는 코드를 많이 사용

```kotlin
fun factorial(n: Int): Long {
    require(n >= 0)
    return if (n <= 1) 1 else factorial(n - 1) * n
}

fun findClusters(points: List<Point>): List<Cluster> {
    require(points.isNotEmpty())
    //...
    return points.map { Cluster() }
}

fun sendEmail(user: User, message: String) {
    requireNotNull(user.email)
    require(isValildEmail(user.email))
}

fun isValildEmail(email: String): Boolean {
    // ... 
    return false
}
```

- 이와 같은 형태의 입력 유효성 검사 코드는 함수의 가장 앞부분에 배치되므로 읽는 사람도 확인하기 쉬움
- require 함수는 조건을 만족시키지 못할 때 무조건적으로 IllegalArgumentException 을 발생시키므로 제한을 무시할 수 없음
- 추가로 람다를 활용해 메시지 정의할 수 있음

**상태**
- 어떤 구체적인 조건을 만족할 때만 함수를 사용할 수 있게 해야할 때가 있는데 이런 경우 check 함수를 통해 상태에 제한을 검

```kotlin
fun speak(text: String) {
    check(isInitialized)
    // ...
}
```

- check 함수는 require과 비슷하지만 지정된 예측을 만족하지 못할 때 IllegalStateException 을 throw함
- 일반적으로 함수 전체에 대한 어떤 에측이 있을대는 require 블록 뒤에 배치

**Assert 계열 함수 사용**
```kotlin
fun pop(num: Int = 1): List<T> {
    assert(ret.size == num)
    return ret
}
```

- Assert 계열의 함수는 코드를 자체 점검
- 특정 상황이 아닌 모든 상황에 대한 테스트 할 수 있음
- 실행 시점에 정확하게 어떻게 되는지 확인할 수 있음
- 코틀린/jvm에서만 활성화되며, -ea JVM 옵션을 활성해야 확인 가능
  
**nullability와 스마트 캐스팅**

- 코틀린에서 require와 check 블록으로 어떤 조건을 확인해서 true가 나왔다면 해당 조건 이후로도 true일 것이라고 가정해 이를 활용해 타입 비교를 했을시 스마트 캐스트가 작동
뿐만 아니라 nullability 목적으로 오른쪽에 throw 또는 return을 두고 Elvis 연산자를 활용하는 경우도 많음

```kotlin
fun changeDress(person: Person) {
   require(person.outfit is Dress)
   val dress: Dress = person.outfit
   //...
}
```

```kotlin
fun sendEmail(person: Person, text: String) {
    val email: String = person.email ?: return
}
```

**정리**
- 코틀린에서 코드에 제한을 걸때의 장점
- 제한을 훨씬 더 쉽게 확인할 수 있음
- 애플리케이션을 더 안정적으로 지킬 수 있음
- 코드를 잘못 쓰는 상황을 막을 수 있음
- 스마트 캐스팅 활용할 수 있음
- 코틀린에서 제공하는 제한 매커니즘
- require 블록 : 아규먼트와 관련된 에측을 정의할 때 사용하는 범용적인 방법
- check 블록 : 상태와 관련된 에측을 정의할 때 사용하는 범용적인 방법
- assert 블록 : 테스트 모드에서 테스트할 때 사용하는 범용적인 방법
- return과 throw 와 함께 Elvis 연산자 사용

### 아이템 6: 사용자 정의 오류보다는 표준 오류를 사용하라

- require, check, assert 함수를 사용한다면, 대부분의 코틀린 오류를 처리할 수 있음
- 하지만 이외에도 예측하지 못한 상황을 나태내야 하는 경우가 있음

```kotlin
inline fun <reified T> String.readObject(): T {
    // ...
    val result: Int
    if (incorrectSign) {
        throw JsonParsingException()
    }
    
    return result
}
```
- 위와 같이 표준 라이브러리에서 나타내는 적절한 오류가 없는 경우 사용자 정의 오류를 사용하는 경우도 있지만 가급적 직접 오류를 정의하기보단 표준 라이브러리의 오류를 사용하는 것이 좋음
- 잘만들어진 규약을 재사용하면 다른 사람들이 API를 더 쉽게 배우고 이해할 수 잇음

**일반적으로 사용되는 예외**

- IllegalArgumentException, IllegalStateException
- IndexOutOfBoundsException
- ConcurrentModificationException : 동시 수정을 금지했는데 발생했을 경우
- UnsupportedOperationException
- NoSuchElementException


### 아이템 7: 결과 부족이 발생할 경우 null과 Failure를 사용하라

**함수가 원하는 경우를 만들 수 없는 경우**
- 서버로 데이터를 읽어 들이려고 했는데, 인터넷 연결 문제로 읽어 들이지 못한경우
- 조건에 맞는 첫 번째 요소를 찾으려 했는데, 조건에 맞는 요소가 없는 경우
- 텍스트를 파싱해서 객체를 만들려고 했는데, 텍스트의 형식이 맞지 않는 경우
- null or throw를 발생한다. 정보를 전달해야하는 상황에는 throw를 리턴하고, 추가 정보를 리턴해야하는 경우에는 sealed. 클래스를 사용한다.

**정보를 전달해야 하는 경우 예외를 사용하면 안되는 이유**
- 전파되는 과정 추적이 어려움
- 코틀린은 unchecked 예외만 존재하므로 예외처리에 자유도가 있음
- try-catch 블록 내부에 코드를 배치하면 컴파일러 최적화 제한된다
- 예측이 가능한 오류 범위일 경우에는 null 과 Failure를 사용하고 어려운 예외적인 범위의 오류는 예외를 throw한다.

```kotlin
val random = Random()

fun someMethod(): Result<String> {
    val num = random.nextInt(5)
    return if (num == 1) {
        Failure(IllegalStateException())
    } else {
        Success("직업 성공")
    }
}
```

```kotlin
sealed class Result<out T>
class Success<out T>(val result: T) : Result<T>()
class Failure(val throwable: Throwable) : Result<Nothing>()
```
이렇게 Result와 같은 결과 봉투 클래스를 활용해 리턴하면 클라이언트 측에서 when 절을 활용해 결과를 처리하기 용이해지고, try-catch블록보다 효율적임

만약 null을 리턴하더라도 Elvis 연산자를 활용해 널 안정성 기능을 활용하여 처리가능
→ null과 sealed의 차이는 추가정보 전달 필요여부로 정한다.

**본인 생각 정리**

- 함수를 만들 때, exception 대신, nullable 혹은 Fail 객체로 wrap
- runCatching에서도 해당 방식 사용함


### 아이템 8: 적절하게 null을 처리하라

**null을 안전하게 처리하기**
- 안전호출
  - printer?.print()
- 스마트 캐스팅
  - if(printer != null) printer.print()
- 엘비스 연산자
```kotlin
  printer?.name ?: "Unnamed"
  printer?.name ?: return
  printer?.name ?: throw Error("error")
```

**프로그래밍 기법**
- 방어적 프로그래밍 : 모든 가능성을 올바른 방식으로 처리
- 공격적 프로그래밍 : 예상하지 못한 상황이 발생했을 때, 이러한 문제를 개발자에게 알려서 수정하게 만듬


**Not-null assertion(!!)과 관련된 문제**
- !!문은 nullability가 제대로 표현되지 않는 라이브러리를 사용할 때 정도에만 사용해야함
- 미래에 코드가 어떻게 변화할지 모르니 현재 null이 아니라고 !!문을 쓰지 않는게 좋음
- !! 연산자 사용은 최대한 피하자

**의심 없는 nullability 피하기**
- nullability를 안전하게 처리할 수 있는 함수 사용
- List getOrNull
- 어떤 값이 클래스 생성 이후 확실하게 설정된다는 보장이 있으면 lateinit 프로퍼티와 notqnull 델리게이트를 사용
- 빈 컬렉션 대신 null을 리턴하지 마세요
- nullable enum vs none enum -> none enum 정의되지 않았기 때문에, 사용하는 측에서 변경 가능, nullable enum 정말 값 없음

**lateinit 프로퍼티와 notNull 델리게이트**
- lateinit 한정자는 프로퍼티가 이후에 설정될 것임을 명시하는 한정자
- lateinit 선언이 되어있는데 초기화 안하고 사용시 예외발생
- 장점
  - 언팩 하지 않아도 된다
  - 이후에 어떤 의미를 나타내기 위해서 null을 사용하고 싶을 때, nullable로 만들 수 있음
  - 프로퍼티가 초기화된 이후에는 초기화되지 않은 상태로 돌아갈 수 없다.
- 단점
  - Int,Long,Double,Boolean과 같은 기본 타입에 사용 불가
  - Delegates로 대체

```kotlin
var text: String by Delegates.notNull()
private class NotNullVar<T : Any>() : ReadWriteProperty<Any?, T> {
    private var value: T? = null

    public override fun getValue(thisRef: Any?, property: KProperty<*>): T {
        return value ?: throw IllegalStateException("Property ${property.name} should be initialized before get.")
    }

    public override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        this.value = value
    }
}
```

- notNull을 위임하면 get에서 널 검사를 진행한다.

### 아이템 9: use를 사용하여 리소스를 닫아라

**Closeable 인터페이스를 구한혀는 리소스는 레퍼런스가 없어질 때 가비지 컬렉터가 처리한다.**
  - 자동 처리는 느리며 쉽게 처리되지 않아 close를 명시적으로 호출해주는 것이 좋다

**java의 try with resource 같은 키워드가 없으므로 try finally로 처리가능하다**
  - try finally로 처리할 시 try 예외 발생 후 finally 구문에서 예외가 발생하면 둘 중 하나의 예외만 전파된다
  - close 예외가 발생할 경우를 대비해 예외처리를 해주어야 한다.

**kotlin은 위 불편을 해소하기 위해 use 함수를 제공한다**
- 코틀린의 확장함수로 제공
- Closeable에 사용할 수 있다.
- Closeable.kt에 구현되어 있는 extension
```kotlin
@InlineOnly
public inline fun <T : Closeable?, R> T.use(block: (T) -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    var exception: Throwable? = null
    try {
        return block(this)
    } catch (e: Throwable) {
        exception = e
        throw e
    } finally {
        when {
            apiVersionIsAtLeast(1, 1, 0) -> this.closeFinally(exception)
            this == null -> {}
            exception == null -> close()
            else ->
                try {
                    close()
                } catch (closeException: Throwable) {
                    // cause.addSuppressed(closeException) // ignored here
                }
        }
    }
}
```

- 파일을 한줄씩 처리할 때 활용할 수 있는 useLines도 제공한다
- 메모리에 한줄씩만 유지된다
```kotlin
/**
 * Calls the [block] callback giving it a sequence of all the lines in this file and closes the reader once
 * the processing is complete.
 * @return the value returned by [block].
 */
public inline fun <T> Reader.useLines(block: (Sequence<String>) -> T): T =
    buffered().use { block(it.lineSequence()) }
```

**생각정리**
- Closeable과 같이 다른 동작이 가능하도록 기능을 제공하는 extension을 잘 활용해라! 라는게 이번장의 핵심인 것인가?
