# 아이템 40. `equals`의 규약을 지켜라

- 모든 클래스의 슈퍼클래스인 `Any`에 구현되어 있어 모든 객체에서 사용할 수 있다.
- 다만 연산자를 사용해서 다른 타입의 두 객체를 비교하는 것은 허용 되지 않는다.
- 같은 타입이나 비교 대상이 상속관계를 갖는 경우는 비교할 수 있다.

### 동등성

- 구조적 동등성 `structural equality`
  - equals 메서드와 이를 기반으로 만들어진 == 연산자로 확인하는 동등성
- 레퍼런스 동등성 `referential equality`
  - 두 피연산자가 같은 객체를 가리키면 true를 리턴, === 연산자로 확인하는 동등성

### `equals`가 필요한 이유

- 두 인스턴스가 동일한지 비교하지만 두 객체가 기본 생성자의 프로퍼티가 같다면 같은 객체로 보는 케이스가 있을 수 있다.
- `data` 클래스로 정의하면 자동으로 이와 같은 동등성으로 동작한다.

### `equals` 를 직접 구현해야 하는 경우

- 기본적으로 제공되는 동작과 다른 동작을 해야 한느 경우
- 일부 프로퍼티만으로 비교해야 하는 경우
- `data` 한정자를 붙이는 것을 원하지 않거나 비교해야 하는 프로퍼티가 기본 생성자에 없는 경우

### `equals` 의 규약 

- 1.4.31 기준

1. 반사적 동작

   - `null`이 아닌 값이라면 `x.equals(x)`는 `y.equals(x)`와 같은 결과를 출력 해야 한다.

   ```kotlin
   class Time (
   	val millisArg: Long = -1,
   	val isNow: Boolean = false,
   ) {
   	val milllis: Long get () =
   		if (isNow ) System.currentTimeMillis()
   		else millis Arg
   
   	override fun equals(other: Any?): Boolean = 
   		other is Time ** millis == other.millis
   }
   
   val now = Time(isNow = true )
   now == now // 때로는 true, 때로는 false
   
   List(100000) { now }.all { it == now }
   // 대부분 false
   
   val now1 = Time(isNow = true)
   val now2 = Time(isNow = true)
   
   assertEquals(now1, now2) // 때로는 true, 때로는 false
   ```

   - 이처럼 규약을 위반하면 컬렉션 내부의 해당 객체가 포함되어 있어도 `contains` 메서드 등으로 포함되어 있는지 확인 불가한 문제가 있다.

2. 대칭적 동작

   - `x`, `y`가 널이 아닌 값이라면 `x.equals(y)`, `y.equals(x)`와 같은 결과를 출력해야 한다.
   - 일반적으로 다른 타입과 동등성을 확인하려고 할 때, 이런 동작이 위반

   ```kotlin
   class Complex (
   
   ) {
   	override fun equals() {
   		if (other is double) {
   
   		}
   		return other is Complex && 
   			real == other.real &&
   				imaginary == other.imaginary
   	}
   }
   
   Complex(1.0, 0.0).equals(1.0) // true
   1.0.equals(Complex(1.0, 0.0)) // false
   ```

   - 대칭적이지 못하다는 것은 `contains` 메서드와 단위 테스트 등에서 예측하지 못한 동작이 발생할 수 있다는 것이다.

   ```kotlin
   val list = listOf<Any>(Complex(1.0, 0.0))
   
   list.contains(1.0) // false
   ```

3. 연속적 동작

   - `x`, `y`, `z`가 널이 아닌 값이고 `x.equals(y)`, `y.equals(z)`가 `true`라면, `x.equals(z)`도 `true`여야 한다.
   - `Double`은 `Complex`와 비교할 수 없으므로 아규먼트의 순서에 따라서 결과가 달라짐
   - 현재 `Date`, `DateTime`이 상속 관계 이므로 같은 객체끼리만 비교하게 만드는 방법은 좋지 않은 선택지이다.
     - 이렇게 구현하면 리스코프 치환 원칙을 위반하기 때문이다.
     - 따라서 처음부터 상속 대신 컴포지션을 사용하고 두 객체를 아예 비교하지 못하게 만드는 것이 좋다.

   ```kotlin
   val o1 = DateTime(Date(1992, 10, 20), 12, 30 ,0)
   val o2 = DateTime(Date(1992, 10, 20))
   val o3 = DateTime(Date(1992, 10, 20), 14, 45, 30)
   
   o1 == o2 // true
   o2 == o3 // true
   o1 == o3 // false
   
   ```

4. 일관적 동작

   - `x`,`y`가 널이 아니라면 `x.equals(y)`는 여러번 실행하더라도 항상 같은 결과를 리턴 해야 한다.
   - `immutable` 객체라면 결과가 언제나 같아요 한다.
   - `equals`는 반드시 비교 대상이 되는 두 객체에만 의존하는 순수 함수 여여한다.
   - 대표적으로 `java.net.Url.equals()`, `Time` 클래스는 이러한 원칙을 위반한다.

5. 널과 관련된 동작

   - `x`가 널이 아니라면, `x.equals(null)`은 항상 `false`여야 한다.
   - `null`은 유일한 객체이기 때문에 절대 `null`이 아닌데 `null`과 같을 수 없다.

6. 빠르게 동작(?)

   - 공식 문서에는 없지만 매우 빠르게 동작해야 한다.
   - 몇초 걸리는 것은 일반적으로 예측하지 못하는 동작이다.

### URL과 관련된 `equals` 문제

- 네트워크 상태에 따라서 결과가 달라진다.
  - 일반적인 상황에서 두 주소가 같은 IP 주소를 나타내므로 `true`를 출력한다.
  - 하지만 인터넷 연결이 끊겨 있으면 `false`를 출력한다.
- 네트워크 설정에 따라서도 결과가 달라질 수 있다.
  - 주어진 호스트의 IP 주소는 시간과 네트워크 상황에 따라 다르다.

### `equals` 구현하기

- 특별한 이유가 없는 이상, 직접 `equals`를 구현하는 것은 좋지 않다.
- 기본적으로 제공하는 것을 그대로 쓰거나 데이터 클래스로 만들어서 사용하는 것이 좋다.
- 직접 구현해야한다면 앞서 살펴본 반사적, 대칭적, 연속적, 일관적 동작을 하는지 꼭 확인해야 한다.
- 그리고 이러한 클래스는 `final`로 만드는 것이 좋다.
- 만약 상속을 한다면 서브 클래스에서 `equals`가 동작하는 방식을 변경하면 안된다.
  - 상속을 지원하면서도 완벽한 사용자 정의 `equals` 함수를 만드는 것은 매우 어렵다.

<br>
<br>
<hr>
<br>
<br>

# 아이템 41. `hashCode`의 규약을 지켜라

- 수많은 컬렉션과 알고리즘에 사용되는 자료구조인 해시 테이블을 구축할 때 사용된다.
- 데이터베이스, 인터넷 프로토콜, 여러 언어의 표준 라이브러리 컬렉션(`LinkedHashSet`, `LinkedHashMap`)에서 사용됨
- 코틀린에서 해시 코드를 만들 때 `hashCode` 함수를 사용함

### 해시 테이블

- 선형적인 자료구조의 탐색 시간의 한계를 해결하기 위해 필요
- 해시 함수는 각각의 요소에 특정한 값을 할당하고 이를 기반으로 요소를 다른 버킷에 넣는다.
- 해시 함수가 좋기 위한 조건
  - 빠르다
  - 충돌이 적다


### 가변성과 관련된 문제

- 요소가 추가될 때만 해시 코드를 계산한다.
- 요소가 변경되어도 해시 코드는 계산되지 않으며, 버킷 재배치도 이루어지지 않는다.
- 그래서 기본적인 `LinkedHashSet`와 `LinkedHashMap`의 키는 한 번 추가한 요소를 변경할 수 없다.
- 세트와 맵의 키로 `mutable` 요소를 사용하면 안되고, 사용하더라도 요소를 변경해서는 안된다.

```kotlin
data class FullName(
	var name: String,
	var surname: String,
)

val person = FullName("minseong", "kim")
val s = mutableSetOf<FullName>()
s.add(person)
person.surname = "fitz"

print(person) // FullName(name = minseong, surname = kim)
print(person in s) // false
print(s.first() == person) // true
```

### `hashCode` 규약

- `1.3.11`기준
- 어떤 객체를 변경하지 않았다면 `hashCode`는 여러번 호출해도 그 결과가 항상 같아야 한다.
  - 일관성 유지를 위해
- equals 메서드의 실행 결과로 두 객체가 같다고 나온다면 `hashCode` 메서드의 호출 결과도 같다고 나와야 한다.
  - `hashCode`, `equals`는 일관성 있게 동작해야 함
  - 즉 같은 요소는 반드시 같은 해시 코드를 가져야 한다.
  - 그렇지 않으면 컬렉션 내부에 요소가 들어 있는지 제대로 확인하지 못하는 문제가 발생한다. (해시 코드로 비교해서 판단하기 때문)
- 만약에 `hashCode`가 항상 같은 숫자를 리턴한다면?
  - 해시 테이블을 사용할 필요가 없어진다.

### `hashCode` 구현하기

- 일반적으로 `data` 한정자를 붙이면 코틀린이 알아서 `equals`와 `hashCode`를 정의해 주므로 이를 직접 정의할 일은 거의 없다.
- 다른 `equals`를 따로 정의했다면 반드시 `hashCode`도 함께 정의해 줘야 한다.
- `equals`로 같은 요소라고 판정되는 요소는 `hashCode`가 반드시 같은 값을 리턴해야 한다.
- `hashCode`를 구현할 때 유용한 함수로는 코틀린/JVM의 `Objects.hashCode`가 있고 이 함수는 해시를 계산해준다.
- 코틀린 `stdlib`에는 `hashCode`를 직접 구현할 일이 거의 없어 이러한 함수가 따로 없다. 따라서 다른 플랫폼에서는 직접 구현 해야 한다.
- `hashCode`를 구현할 때 가장 중요한 규칙은 언제나 `equals`와 일관된 결과가 나와야 하기 때문에 같은 객체라면 언제나 같은 값을 리턴하게 만들어줘야 한다.

<br>
<br>
<hr>
<br>
<br>

# 아이템 42. `compareTo`의 규약을 지켜라

### `compareTo`란

  - 한 객체와 다른 객체의 순서를 비교해주는 메서드
    - 같으면 0 리턴
    - 아규먼트보다 작으면 음수 리턴
    - 아규먼트보다 크면 양수 리턴
  - Comparable<T>에 정의되어 있고 이 인터페이스를 구현하고 있거나 compareTo 연산자 메서드를 가지고 있으면 **해당 객체가 어떤 순서를 가지고 있어 비교할 수 있다는 의미임**

### `compareTo`는 다음과 같이 동작해야 한다.

  - 비대칭적 동작
    - a ≥ b 이고, b ≥a 라면, a == b여야 한다.
      - 즉 비교와 동등성 비교에 어떠한 관계가 있어야 하고 서로 일관성 있어야 한다.
  - 연속적 동작
    - a ≥ b, b ≥ c, a ≥ c 여야 한다.
    - 이러한 동작을 하지 못하면, 요소 정렬이 무한 루프에 빠질 수 있음
  - 코넥스적 동작
    - 두 요소는 어떠한 확실한 관계를 갖고 있어야 한다.
    - 즉, a ≥ b 또는 b ≥ a 중에 적어도 하나는 항상 true 여야 한다.
    - 두 요소 사이에 관계가 없으면 퀵 정렬과 삽입 정렬 등의 고전적인 정렬 알고리즘을 사용할 수 없는 문제가 있다.

### `compareTo`를 따로 정리해야할까?

  - 일반적으로 어떤 프로퍼티 하나를 기반으로 순서를 지정하는 것으로 충분하기 때문에 따로 정의해야 하는 상황은 거의 없다.

  - `sortedBy`를 통해 원하는 프로퍼티로 컬렉션 정렬을 할 수 있다.

  - `sortedWith`를 통해 여러 프로퍼티를 기반으로 정렬을 할 수 있다.

  - 만약 비교에 대한 절대적인 기준이 없다면 아예 비교하지 못하게 만드는 것도 좋다.

  - 문자열은 알파벳과 숫자 등의 순서가 있는데 이는 내부적으로 `Comparable<String>` 를 구현하고 있기 때문이다.
        

    ```kotlin
      // 단 이렇게 하지 말자. 가독성 떨어진다.
      print("Kotlin" > "Java") // true
        
      // 참고) "Kotlin".compareTo("Java") > 0 으로 바뀐다.
    ```

  - 측정 단위, 날짜, 시간과 같이 자연스러운 순서를 갖는 객체들은 `comparator` 를 사용하는 것이 좋다.
        

    ```kotlin
      class User(val name: String, val surname: String) {
        
      companion object  {
        val DISPLAY_ORDER = compareBy(User::surname, User::name)
        }
      } 
    
      val sorted = names.sortedWith(User.DISPLAY_ORDER)
    ```

### `compareTo` 구현하기

  - `compareValues` 톱레벨 함수를 쓰면 `compareTo` 를 구현할 때 유용하다.

    ```kotlin
      class User {
        val name: String,
        val surname: String
      } : Comparable<User> {
        override fun compareTo(other: User): Int =
          compareValues(surname, other.surname)
      }
    ```

  - `compareValuesBy`를 통해 더 많은 값을 비교하거나 `selector`를 활용해서 비교할 수 있다.

    ```kotlin
      class User {
        val name: String,
        val surname: String
      } : Comparable<User> {
        override fun compareTo(other: User): Int =
          compareValuesBy(this, other { it.surname }, { it.name })
      }
    ```

<br>
<br>
<hr>
<br>
<br>


# 아이템 43. API의 필수적이지 않는 부분을 확장 함수로 추출하라

### 멤버 함수와 확장 함수의 차이

1. 확장 함수는 읽어 들여야 한다.

  - 확장은 우리가 직접 멤버 변수를 추가할 수 없는 경우 데이터와 행위를 분리하도록 설계된 프로젝트에서 사용된다.
  - 필드가 있는 프로퍼티는 클래스에 있어야 하지만 확장 메서드는 클래스의 `public` API만 활용하면 어디에 위치해도 상관없다.
  - `import` 해서 사용하기 때문에 같은 타입에 같은 이름으로 여러개 만들 수도 있어 여러 라이브러리에서 여러 메서드를 받을 수도 있고 충돌이 발생하지 않는다는 장점이 있음.
  - 하지만 같은 이름인데 다른 동작을 한다는 점에서 리스크가 있으니, 리스크가 있다면 그냥 멤버 함수로 만드는게 나을 수도 있다.

2. 확장 함수는 `virtual`이 아니다.

  - 즉 서브 클래스에서 오버라이드 할 수 없다.
  - 확장 함수는 컴파일 시점에 정적으로 선택되기 때문에 확장 함수는 가상 멤버 함수와 다르게 동작한다.
  - **상속을 목적으로 설계된 요소는 확장 함수로 만들면 안된다.**

3. 확장 함수는 클래스 위가 아니라 타입 위에 만들어짐

  - 그래서 아래와 같이 `nullable` 또는 구체적인 제너릭 타입에도 확장함수를 정의할 수 있음
        

  ```kotlin
    // nullble 타입에 확장함수 정의 가능
    inline fun CharSequence?.isNullOrBlank(): Boolean {
      contract {
        returns(false) implies (this@isNullOrBlank != null)
      }

      return this == null || this.isBlank()
    }
        
    // 구체적인 제너릭 타입에 확장함수 정의 가능
    public fun Iterable<Int>.sum(): Int {
      var sum :Int = 0
      for (element in this) {
        sum += element
      }
      return sum
      }
  ```

4. 확장 함수는 클래스 레퍼런스에 나오지 않음

  - 그래서 확장 함수는 어노테이션 프로세서가 따로 처리하지 않는다.
  - 따라서 필수적이지 않는 요소를 확장 함수를 추출하면 어노테이션 프로세서로부터 숨겨진다.
  - 이는 확장 함수가 클래스 내부에 있는 것이 아니기 때문이다.

### 정리

  - 확장함수는 더 많은 자유와 유연성을 준다
  - 확장 함수는 상속 어노테이션 처리 등을 지원하지 않고 클래스 내부에 없으므로 약간 혼동을 줄 수 있다.
  - **API의 필수적인 부분은 멤버로 두는 것이 좋지만 필수적이지 않은 부분은 확장 함수로 만드는 것이 좋다.**

<br>
<br>
<hr>
<br>
<br>

# 아이템 44. 멤버 확장 함수의 사용을 피하라

* 어떤 클래스에 대한 확장함수를 정의할 때 이를 멤버로 추가하는 것은 좋지 않음
* 확장함수는 첫번째 아규먼트로 리시버를 받는 단순한 일반 함수로 컴파일 됨

~~~kotlin
  fun String.isPhoneNumber(): Boolean = 
	  length == 7 && all { it.isDigit() } // 컴파일 전

  fun String.isPhoneNumber(): Boolean = 
	  '$this'.length == 7 && '$this'.all { it.isDigit() } // 컴파일 후
~~~

### 왜 사용을 피해야할까?

1. 가시성을 제한하지 못함 (아래와 같이 확장 함수를 사용하기 어렵게 함)
2. 레퍼런스를 지원하지 않음

~~~kotlin
val ref = String::isPhoneNumber
val str = "1234567890"
val boundedRef = str::isPhoneNumber

val refX = PhoneBookIncorrect::isPhoneNumber // 컴파일 에러
val book = PhoneBookIncorrect() 
val bookRefX = book::isPhoneNumber // 컴파일 에러 
~~~


3. 암묵적 접근을 할 때 두 리시버 중에 어떤 리시버가 선택될지 혼동됨

~~~kotlin
class A {
	val a = 10
}

class B {
	val a = 20
	val b = 30

	fun A.test() = a + b // 40일까? 50일까?
}

fun main() {    
	B().apply { println(A().test()) } // 정답 : 40
}
~~~

4. 확장 함수가 외부에 있는 다른 클래스를 리시버로 받을 때 해당 함수가 어떤 동작을 하는지 명확하지 않음

~~~kotlin
class A {
	var state = 10
}

class B {
	var state = 20

	fun A.update() = state + 10  // A와 B중에서 어떤 것을 업데이트 할까요?
}

fun main() { 
	B().apply { println(A().update()) } // 정답 : 20 (A를 업데이트 한다)
}
~~~

5. 경험이 적은 개발자에게 낯설 수 있음

### 정리

* 멤버 확장 함수를 사용하는 것이 의미있는 경우는사용해도 되지만, 단점을 인지하고 사용하지 않는 것이 좋다.
* 가시성을 제한하려면 멤버로 만들지 말고 가시성과 관련된 한정자를 사용하자.

<br>
<br>
<hr>
<br>
<br>

## 45. 불필요한 객체 생성을 피하라

상황에 따라 객체 생성 비용은 클 수 있다. 

<br>

### **객체 생성 비용은 항상 클까?**

어떤 객체를 wrap시 크게 3가지 비용

- 객체는 더 많은 용량을 차지함.
  - 현대의 64비트 JDK 에서 객체는 8바이트의 배수, + 앞부분은 12바이트 헤더.
  - int는 4바이트지만 Integer는 16바이트 + 레퍼런스 8바이트
- 캡슐화된 객체는 함수 호출 비용이 발생. 비용이 크진 않지만 티끌모아태산이다.
- 객체 생성시 비용 발생
  - 매모리 영역에 할당
  - 이에 대한 레퍼런스 생성 등의 비용이 발생

<br>

### 불필요한 객체를 제거할 방법

**객체 선언**

- 객체를 재사용한다. (싱글톤)

```kotlin
///////// AS-IS /////////
sealed class LinkedList<T>

class Node<T>( 
	val head: T, 
	val tail: LinkedList<T> 
): LinkedList<T>()

class Empty<T>: LinkedList<T>() // <---- 문제

// 사용
val list1: LinkedList<Int> = Node(1, Node(2, Node, Empty()) 
val list2: LinkedList<Int> = Node(3, Node(4, Node, Empty()) // Empty() 인스턴스를 매번 만들어야 함

///////// TO-BE /////////
....
object Empty : LinkedList<Nothing>() 

// 사용
val list1: LinkedList<Int> = Node(1, Node(2, Empty))
val list2: LinkedList<Int> = Node(2, Node(2, Empty)) // 하나의 Empty 인스턴스를 재활용함
```

<br>

**캐시를 활용하는 팩토리 함수**

- 팩토리 메서드를 사용하면 캐시를 가질 수 있다.

- 쓰레드풀, 커넥션풀 등 객체생성이 무겁고 동시에 여러 mutable 객체를 사용하는 경우 적합

- 모든순수 함수는 캐싱을 활용할 수 있다. (메모이제이션)

  예) 피보나치 수 구하는 함수

  - 한번 계산된 값은 캐싱해서 재사용한다.

  ```kotlin
  private val FIB_CACHE = mutableMapOf<String, Connection>()
  
  fun fib(n: Int): BigInteger = FIB_CACHE.getOrPut(n) {
  		if (n <=1) BigInteger.ONE else fib(n - 1) + fib(n - 2)
  }
  ```

- (큰) 단점은 메모리 사용량이 늘어난다.

  - 메모리문제로 크래시가 생기면 메모리 해제.

- SoftReference 사용하면 메모리 필요할 때만 GC가 자동으로 메모리를 해제 해준다.

  - WeakReference 는 객체를 사용하지 않으면 곧바로 제거한다.

- 캐시는 메모리와 성능의 트레이드 오프

<br>

**무거운 객체를 외부 스코프로 보내기**

- 컬렉션 처리에서 무거운 연산을 내부에서 외부 스코프로 보낸다.

  ```kotlin
  // as-is : max 연산이 element 수 만큼 수행됌
  fun <T: Comparable<T>> Iterable<T>.countMax(): Int =
  		count { it == this.max() }
  
  // to-be : max 연산은 한번만 하면 됌
  fun <T: Comparable<T>> Iterable<T>.countMax(): Int =
  		val max = this.max()
  		return count { it == max }
  }
  ```

<br>

**지연 초기화**

- 하나의 클래스에서 무거운 인스턴스들을 같이 생성해야하는 경우

  - 해당 객체 생성이 굉장히 무거워 짐

  ```kotlin
  class A {
  	val b = B() // 엄청 무거움
  	val c = C() // 엄청 무거움
  	val d = D() // 엄청 무거움
  }
  
  // => A 클래스를 생성하는 일이 엄----청 무거움
  ```

- 지연초기화하면 사용할때 만들어지므로 객체 생성 가져워짐

  ```kotlin
  class A {
  	val b by lazy { B() } // 엄청 무거움
  	val c by lazy { C() } // 엄청 무거움
  	val d by lazy { D() } // 엄청 무거움
  }
  
  // A 클래스 생성해도 b,c,d 는 아직 생성하지 않아도 됌
  // => A 객체 생성 과정이 가벼워졌다.
  ```

- 단점

  - 메서드 호출이 빨라야하는 경우 처음 호출하면 객체 생성하느라 첫 호출의 응답이 오래 걸릴수도.
  - → 백엔드 어플리케이션에 안좋음, 성능 테스트도 어려움

<br>

**기본 자료형 사용하기**

- 기본 자료형을 wrap 한 자료형이 사용되는 경우는 다음과 
  - nullable 타입 연산시
  - 타입을 제네릭으로 사용할 때
    - 코틀린 : Int, Int?, List<Int>
    - 자바 : int, Integer, List<Integer>
- 숫자 연산의 경우 기본 자료형과 wrap 한 자료형의 성능 차이는 크지 않다.
  - 굉장히 큰 컬렉션을 처리할 때 차이를 확인 가능.
- 자료형을 일괄 변경하면 코드를 읽기 힘들어질 수 있다.
  - 그래서 코드와 라이브러리 성능이 굉장히 중요한 경우만 최적화를 하는게 좋다.
  - 그게 아니라면 큰 의미가 없는 최적화다.

<br>
<br>
<hr>
<br>
<br>


## 46 - 함수 타입 파라미터를 갖는 함수에 inline 한정자를 붙여라

inline 한정자를 사용하면 함수본문으로 점프 → 함수 호출위치로 다시 점프 과정이 일어나지 않는다.

<br>

### inline 한정자의 장점

1. 타입 아규먼트에 reified 한정자를 붙여 사용 가능
2. 함수 타입 파라미터를 가진 함수가 훨씬 빠르게 동작
3. 비지역적 리턴을 사용할 수 있음

<br>

**타입 아규먼트를 reified로 사용할 수 있다**

- JVM 바이트코드에는 제네릭이 존재하지 않는다.

  - 자바는 컴파일하면 제네릭 타입 관련 정보가 제거된다.
  - List<Int> → List
  - 제네릭 타입과 관련된 정보 사용 불가능

- 함수를 inline 으로 만들면 이러한 제한을 무시 가능

  - 함수 호출이 본문으로 대체되므로
  - reified 한정자를 지정하면 타입 파라미터를 사용한 부분이 타입 아규먼트로 대체됌

  ```kotlin
  inline fun <reified T> printTypeName() { 
  		print(T::class.simpleName)
  }
  
  // 사용
  printTypeName<Int>() // Int
  printTypeName<Char>() // Char
  printTypeName<String>() // String
  
  // 컴파일하면 실제로는
  print(Int::class.simpleName)
  print(Char::class.simpleName)
  print(String::class.simpleName) 
  ```

<br>

**함수 타입 파라미터를 가진 함수가 훨씬 빠르게 동작한다**

- inline 한정자가 붙으면 조금 더 빠르다.

  - 함수 호출과 리턴을 위한 점프 과정과 백스택을 추적하는 과정이 없어서.
  - 그래서 표준 라이브러리의 간단한 함수들에 다 inline 붙어 있음

- 하지만 함수 파라미터를 가지지 않는 함수는 큰 성능차이 X

  - 함수를 객체로 조작할때 발생하는 문제를 이해해야 함

  - 코틀린/JS 에서 함수를 일급객체로 처리해서 간단하게 변환

  - 코틀린/JVM 에서는 JVM 익명 클래스 또는 일반 클래스를 기반으로 함수를 객체로 만듦

    ```kotlin
    // 컴파일 대상
    var lambda: ()->Unit = {}
    
    // 위의 람다 표현식을 컴파일해서 나온 익명 클래스
    Function0<Unit> lambda = new Function0<Unit>() {
    		public Unit invoke() { ... }
    }
    
    // 또는 별도의 파일에 정의되어 있는 일반 클래스
    public class Test$lambda implements Function0<Unit> {
    		public Unit invoke() { ... }
    }
    
    // 사용
    Function0 lambda = new Test$lambda()
    ```

    - JVM 에서 아규먼트가 없는 함수 타입은 Function0 타입으로 변환 됌
    - ()→Unit 은 Function0<Unit>
    - (Int)→Int 는 Function1<Int, Int>
    - (Int, Int)→Int 는 Function2<Int, Int, Int>
    - ⇒ 함수 타입은 단순한 인터페이스다.

- 함수 본문을 객체로 wrap 하면 코드의 속도는 더 느려진다. 
  그래서 다음 예제의 첫번째가 더 빠르다.

    ```kotlin
  - // 각각의 함수를 1억번씩 실행시킨 결과
  // 저자의 컴퓨터에서 평균 189ms
  inline fun repeat(times: Int, action: (Int) → Unit) {
  		for (intdex in 0 until times) {
  				action(index)
  		}
  }
  
  // 저자의 컴퓨터에서 평균 477ms
  fun repeat(times: Int, action: (IInt) → Unit) {
  		for (intdex in 0 until times) {
  				action(index)
  		}
  }
    ```

    - 첫번째 함수는 숫자로 반복을 돌고 빈 함수를 호출
    - 두번째 함수는 반복을 돌면서 객체를 호출하고, 이 객체가 빈 함수를 호출

- 더 중요한 차이는 함수 리터럴 내부에서 지역 변수를 캡처할 때 확인 가능하다.

  - 캡처된 값은 객체로 래핑해야하고, 사용할 때 마다 객체를 통한 작업이 일어남

  ```kotlin
  var l = 1L
  nolineRepeat(100_000_000) { 
  		l += it 
  }
  // 인라인이 아닌 람다 표현식에서는 지역 변수 l을 직접 사용 불가능. 
  
  // 컴파일 과정중 아래처럼 레퍼런스 객체로 래핑된다.
  val a = Ref.LongRef()
  a.element = 1L
  noinlineRepeat(100_000_000) {
  		a.element = a.element + it
  }
  ```

  - 저자의 1억번 반복 결과
    - 인라인 - 30ms
    - 인라인X - 274ms
    - 지역 변수가 래핑되는 문제가 누적된 큰 차이

- 함수 타입 파라미터를 활용해 유틸리티 함수를 만들 때는 그냥 인라인을 붙여준다고 생각

<br>

**비지역적 리턴(non-local return)을 사용할 수 있다**

- 다음 함수는 일반 반복문 처럼 사용가능

  ```kotlin
  fun repeatNoinline(times: Int, actionL (Int) -> Unit) {
  		for (index in 0 until times) {
  			action(index)
  		}
  }
  
  // 사용
  repeatNoninline(10) { print(it) }
  ```

  - 일반적인 반복문과 차이 : 함수 내부에서 return 불가능

- inline 을 붙이면 가능.  (함수 내부에 return 이 박히기 때문)

<br>

### inline 한정자의 비용

inline 은 유용하지만, 모든 곳에서 사용은 불가능 하다.

- 인라인 함수는 재귀적으로 동작할 수 없음.

  - 사용하면 무한하게 대체되는 문제가 발생
  - 인텔리제이가 오류로 잡아주지 못해서 위험함

  ```kotlin
  inline fun a() { b() }
  inline fun b() { c() }
  inline fun c() { a() }
  // a -> b -> c -> a -> ... 무한반복
  ```

- 더 많은 가시성 제한을 가진 요소 사용 불가능

  - public 인라인 함수 내부에서 private, internal 가시성을 가진 함수/프로퍼티 사용 불가능
  - 그래서 인라인 함수는 구현을 숨길 수 없어서 클래스에 거의 사용되지 않는다.

- 코드의 크기가 쉽게 커진다.

  - 서로 호출하는 인라인 함수가 많아지면 기하급수적으로 증가할 수 있다.

  ```kotlin
  inline fun printThree() { print(3) }
  
  inline fun threePrintThree() {
  		printThree()
  		printThree()
  		printThree()
  }
  
  inline fun threeThreePrintThree() {
  		threePrintThree() 
  		threePrintThree() 
  		threePrintThree() 
  }
  ```

  ```kotlin
  // 컴파일 결과
  inline fun printThree() { print(3) }
  
  inline fun threePrintThree() {
  		print(3)
  		print(3)
  		print(3)
  }
  
  inline fun threeThreePrintThree() {
  		print(3)
  		print(3)
  		print(3)
  		print(3)
  		print(3)
  		print(3)
  		print(3)
  		print(3)
  		print(3)
  }
  ```

<br>

### crossinline과 noinline

함수를 인라인으로 만들고 싶지만, 일부 함수 타입 파라미터는 inline 으로 받고 싶지 않은 경우 사용하는 한정자

- **crossinline**
  - 아규먼트로 인라인 함수는 받지만, 비지역적 리턴을 하는 함수는 받을수 없게 함
  - 인라인으로 들지 않은 다른 람다와 조합해서 사용할 때 문제가 발생하는 경우 활용
- **noinline**
  - 아규먼트로 인라인 함수를 받을 수 없게 만듦
  - 인라인 함수가 아닌 함수를 아규먼트로 사용하고 싶을 때 활용

```kotlin
inline fun requestNewToken (
		hasToken: Boolean,
		crossline onRefresh: ()->Unit,
		noinline onGenerate: ()->Unit
) {
		if (hasToken) {
				httpCall("get-token", onGenerate)
				// 인라인이 아닌 함수를 아규먼트로 함수에 전달하려면 no inline을 사용
		} else {
				httpCall("get-token") { 
						onRefresh() 
						// Non-local 리턴이 허용되지 않는 컨텍스트에서 
						// inline 함수를 사용하고 싶다면 crossinline 사용 
						onGenerate()
				}
		}
}

fun httpCall(url: String, callback: ()→Unit) { 
				/*..*/
}
```

<br>
<br>
<hr>
<br>
<br>

### 인라인 클래스의 사용을 고려하라

#### 인라인 클래스의 기능 및 사용법

- 하나의 값을 보유하는 객체도 inline으로 만들 수 있음 (kotlin 1.3부터 도입된 기능)
- 기본 생성자 프로퍼티가 하나인 클래스 앞에 inline을 붙이면 해당 객체를 사용하는 위치가 모두 해당 프로퍼티로 교체됨.

```kotlin
inline class Name(private val value: String)

// 컴파일 전 코드
val name: Name = Name("Marcin")

// 컴파일 후 코드
val name: String = "Marcin"
```

- inline 클래스의 메서드는 모두 정적 메서드로 만들어짐.

```kotlin
inline class Name(private val value: String) {

  fun greet() {
    print("Hello, I am $value")
  }
}

// 컴파일 전 코드
val name: Name = Name("Marcin")
name.greet()

// 컴파일 후 코드
val name: String = "Marcin"
Name.`greet-impl`(name)
```

#### 인라인 클래스 활용

- 다른 자료형을 래핑해서 새로운 자료형을 만들 때 많이 사용
  - 측정 단위 표현 (Millis, Minutes, KB, MB...)
  - 타입 오용으로 발생하는 문제를 방지

```kotlin
@Entity(tableName = "grades")
class Grades(
  @ColumnInfo(name = "studentId")
  val studentId: Int,
  @ColumnInfo(name = "teacherId")
  val teacherId: Int,
  @ColumnInfo(name = "schoolId")
  val schoolId: Int
  ...
)

// => inline을 활용하여 각각의 Id를 타입으로 제한 가능
inline class StudentId(val id: Int)
inline class TeacherId(val id: Int)
inline class SchoolId(val id: Int)

@Entity(tableName = "grades")
class Grades(
  @ColumnInfo(name = "studentId")
  val studentId: StudentId,
  @ColumnInfo(name = "teacherId")
  val teacherId: TeacherId,
  @ColumnInfo(name = "schoolId")
  val schoolId: SchoolId
  ...
)
```

#### 인라인 클래스이지만 인라인으로 작동되지 않는 경우

- inline class가 인터페이스를 구현하게 될 경우 inline 기능이 동작하지 않음
  - 인터페이스를 통해서 타입을 나타내려면, 객체를 래핑해서 사용해야하기 때문
  - 인터페이스를 구현하는 인라인 클래스는 아무 의미 없음.

#### typealias

- 특정 가시성(alias 대상과 같거나 좁은 접근제어자 선언 가능)에서 타입에 새로운 이름을 붙여줄 수 있음.

```kotlin
typealias ClickListener = (view: View, event: Event) -> Unit

class View {
  fun addClickListener(listener: ClickListener) {}
  fun removeClickListener(listener: ClickListener) {}
  ...
}
```

<br>
<br>
<hr>
<br>
<br>

### 더 이상 사용하지 않는 객체의 레퍼런스를 제거하라

- JVM의 GC가 메모리 관리를 해준다고 메모리 관리를 완전히 무시한다면, 메모리 누수가 발생하여, 상황에 따라 OOME가 발생할 수 있음.
- 객체에 대한 참조를 companion으로 유지하면 GC가 해당 객체에 대한 메모리 해제를 할 수 없음.
  - 의존 관계를 정적으로 저장하지 말고, 다른 방법을 활용하는 것이 좋음
  - 객체에 대한 레퍼런스를 다른 곳에 저장할 때는 메모리 누수가 발생할 가능성을 언제나 신경써야함.

```kotlin
class MainActivity : Activity() {

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    ...

    // this에 대한 레퍼런스 누수 발생
    logError = { Log.e(this::class.simpleName, it.message)}
  }

  companion object {
    var logError: ((Throwable) -> Unit?)? = null
  }
}
```

- 객체를 더 이상 사용하지 않을 때, 그 레퍼런스에 null을 설정해주면 GC가 GC cycle에 더 이상 참조되지 않는 객체를 정리.

```kotlin
class Stack {

  private var elements: Array<Any?> = arrayOfNulls(DEFAULT_INITIAL_CAPACITY)
  private var size = 0

  fun push(e: Any) {
    ensureCapacity()
    elements[size++] = e
  }

  fun pop:(): Any? {
    if (size == 0) {
      throw EmptyStackException()
    }
    return elements[--size]
  }

  private fun ensureCapacity() {
    if (elements.size == size) {
      elements = elements.copyOf(2 * size + 1)
    }
  }

  companion object {
    private const val DEFAULT_INITIAL_CAPACITY = 16
  }
}
```

- 위 코드의 문제점은 pop을 할 때 size만 감소시키기만 하고, 배열의 요소를 해제하는 부분이 없다는 것이다.

### SoftReference & WeakReference & StrongReference

- SoftReference: 어떤 객체를 SoftReference로 연결하면, 메모리가 부족한 경우에만 gc가 해당 객체를 수거해감.
- WeakReference: 어떤 객체를 WeakReference로 연결하면, GC cycle에 해당 객체를 반드시 수거해감
- StrongReference: new로 객체를 생성하여 연결하는 일반적인 경우 => 해당 레퍼런스가 계속 유지된다면 gc가 수거해가지 않음.

<br>
<br>
<hr>
<br>
<br>

## 아이템 49. 하나 이상의 처리 단계를 가진 경우에는 시퀀스를 사용하라

# 컬렉션

- `Iterator`는 처리 함수마다 연산이 이루어져 List가 만들어진다. 

# 시퀀스

- `Sequence`는 lazy 처리 되므로 `toList`, `count` 등 최종 연산이 이루어질 때 계산이 수행된다.

### 장점

- 자연스러운 처리 순서를 유지한다.

  - 고전적인 반복문과 조건문을 활용해서 구현했을때, element-by-element order와 순서가 같으므로 시퀀스 처리가 더 자연스러운 순서라고 볼 수 있다.

  ```kotlin
  sequenceOf(1, 2, 3)
        .filter { print("F$it, "); it % 2 == 1 }
        .map { print("M$it, "); it * 2 }
        .forEach { print("E$it, ") }
  // F1, M1, E2, F2, F3, M3, E6,
  
  listOf(1, 2, 3)
        .filter { print("F$it, "); it % 2 == 1 }
        .map { print("M$it, "); it * 2 }
        .forEach { print("E$it, ") }
  // F1, F2, F3, M1, M3, E2, E6, 
  ```

- 최소한만 연산한다.

  - 시퀀스는 중간 연산을 적용하므로, (컬렉션 전체에 원하는 처리를 할 필요 없이) 필요한 앞의 요소에만 원하는 처리를 적용할 수 있다.

  ```kotlin
  (1..10).asSequence()
        .filter { print("F$it, "); it % 2 == 1 }
        .map { print("M$it, "); it * 2 }
        .find { it > 5 }
  // F1, M1, F2, F3, M3,
  
  (1..10)
        .filter { print("F$it, "); it % 2 == 1 }
        .map { print("M$it, "); it * 2 }
        .find { it > 5 }
  // F1, F2, F3, F4, F5, F6, F7, F8, F9, F10, M1, M3, M5, M7, M9,
  ```

- 무한 시퀀스 형태로 사용할 수 있다.

  - 무한 시퀀스는 `generateSequence`나 `sequence`와 중단함수를 이용해서 만들 수 있다.
  - 단, 무한 반복에 빠지지 않도록 종결 연산을 잘 사용하자.

- 각각의 단계에서 컬렉션을 만들어내지 않는다.
  - 메모리 측면에서도 좋다.

### 시퀀스가 빠르지 않은 경우

- 컬렉션 전체를 기반으로 처리해야 하는 연산은 시퀀스를 사용해도 빨라지지 않는다.

  - ex) `sorted` : 시퀀스를 List로 변환한 뒤 sort함
  - 무한시퀀스처럼 시퀀스 다음 요소를 lazy하게 구하는 시퀀스에 sorted를 적용하면 무한반복에 빠진다.

  ```kotlin
  generateSequence(10) {it + 1}.take(10).sorted().toList()
  generateSequence(10) {it + 1}.sorted().take(10).toList() // 무한반복
  ```

### 시퀀스를 쓰는 게 좋은 경우

- 무거운 객체나 규모가 큰 컬렉션을 여러 단계에 걸쳐서 처리할 때 시퀀스를 사용하라.

### 자바 스트림과의 비교

- 코틀린의 시퀀스가 더 많은 처리함수를 가지고, 사용하기 쉽다.
- 자바 스트림은 병렬 함수를 이용해 병렬 모드에서 실행가능 하다.
- 코틀린 시퀀스는 일반적인 모듈(코틀린/JVM, 코틀린/JS ...)에서 모두 사용 가능하지만, 자바 스트림은 코틀린/JVM(버전 8이상)에서만 동작한다.

<br>
<br>
<hr>
<br>
<br>

# 아이템 50. 컬렉션 처리 단계 수를 제한하라.

- 대부분의 컬렉션 처리 단계는 '전체 컬렉션에 대한 반복'과 '중간 컬렉션 생성'이라는 비용이 발생한다.
- 컬렉션의 단계를 줄일 수 있는 함수들이 있으니 사용해보자.

  - ex)

  ```kotlin
  // bad
  students
    .filter { it != null }
    .map { it!! }
  
  // good
  studnets
    .filterNotNull()
  ```

<br>
<br>
<hr>
<br>
<br>

## 아이템 51.성능이 중요한 부분에는 기본 자료형 배열을 사용하라

코틀린은 기본 자료형(primitive)을 선언할 수 없지만, 최적화를 위해 내부적으로 사용 할 수 있다.

- 가볍다 : 일반적인 객체와 다르게 추가적으로 포함되는 것들이 없기 때문
- 빠르다 : 값에 접근할 때 추가 비용이 들어가지 않는다.

코틀린은 기본 맵핑 컬렉션을 사용한다. 성능이 중요한 코드라면 IntArray, LongArray 등의 기본 자료형을 활용하는것이 좋다.
예로 1,000,000개의 정수 컬렉션을 비교하면

- IntArray : 400,000,016바이트

- List<Int> : 2,000,006,944바이트 활당

> 5배 차이 발생

 ## 결론

 - 일반적으로 Array 보다 List, Set 사용하는것이 좋다.
 - 하지만 기본 자료형의 컬렉션을 굉장히 많이 보유해야 하는 경우에는 성능과 메모리 사용량을 줄일 수 있는 Array 활용이 좋다.

<br>
<br>
<hr>
<br>
<br>

## 52.mutable 컬렉션 사용을 고려하라

immutable 컬렉션 보다 mutable 컬렉션이 좋은 점은 성능적인 측면에서 더 빠르다는 부분이다.
immutable 컬렉션에 요소 추가시 새로운 컬렉션을 만들고, 여기에 요소를 추가하기 때문에 새로운 객체 생성 비용이 발생한다.

## 정리하면

내부에서는 mutable 컬렉션을 사용하고, 외부에 제공할 경우에는 immutable을 처리하는것이 좋다.

```kotlin
class Sample {
   private val _list = mutableListOf<Int>()
   val list: List<Int> = _list
}
```

