# Week 6

### Items

- 아이템 23: 타입 파라미터의 섀도잉을 피하라
- 아이템 24: 제네렉 타입과 variance 한정자를 활용하라
- 아이템 25: 공통 모듈을 추출해서 여러 플랫폼에서 재사용하라
- 아이템 26: 함수 내부의 추상화 레벨을 통일하라

<hr>

### 아이템 23: 타입 파라미터의 섀도잉을 피하라 

```kotlin
    class Forest(val name: String){
    	fun addTree(name: String){
        	// ...
        }
    }
```

위 스니펫처럼 클래스의 프로퍼티와 메소드의 매개변수의 이름이 ``name``으로 같은 것을 확인할 수 있습니다. 이럴 경우에는 메소드 내에서 지역변수가 아닌 클래스의 프로퍼티로 인식을 하게 되는데 이를 ``섀도잉(shadowing)``이라고 부른다.
<br>

```kotlin
    interface Tree
    class Birch: Tree
    class Spruce: Tree
    
    class Forest<T: Tree>{
    	fun <T: Tree> addTree(tree: T){
        	// ..
        }
    }
    
    // main 사용
    val forest = Forest<Birch>()
    forest.addTree(Birch())
    forest.addTree(Spruce())
```

이런 섀도잉 현상은 제너릭 클래스의 타입 파라미터와 제너릭 함수의 타입 파라미터 사이에서도 발생한다. 섀도잉으로 인해 다양한 문제들이 발생하고 심각한 문제로 야기될 수 있다. 
아래 사용 코드를 봐서는 Forest의 클래스의 타입과 addTree의 타입이 다를 것이라고 유추하기는 어렵다. 또한 다른 클래스들을 더하는 것을 의도하지도 않았을 것이다.
<br>

```kotlin
    class Forest<T: Tree>{
    	fun addTree(tree: T){
        	// ..
        }
    }
    
    // main 사용
    val forest = Forest<Birch>()
    forest.addTree(Birch())
    forest.addTree(Spruce())	// ERROR, type mismatch
```

따라서 addTree도 동일한 타입을 사용할 수 있도록 맞추어야 합니다. 만약 독립적인 매개변수 타입이 필요할 경우에는 필히 다른 타입 파라미터를 사용하는 것을 권장한다.

**새도잉을 피하자**
- 다른 명칭으로 진행한다.

### 아이템 24: 제네렉 타입과 variance 한정자를 활용하라

```kotlin
class Box<T>
```

- 위의 코드에서 타입 파라미터 T는 variance 한정자(out or in)가 없으므로, 기본적으로 invariant(불공변성, 무공변성)이다.
- invariant라는 것은 제네릭 타입으로 만들어지는 타입들이 서로 관련성이 없다는 의미이다. 예를 들어, Box<Int>, Box<Number>, Box<Any>는 어떠한 관계도 갖지 않는다.

<br>

**제네릭 타입으로 만들어지는 타입들도 서로 관련성을 갖게 하려면 variance 한정자(out or in)를 사용하자**

- out은 타입 파라미터를 covariant(공변성)로 만든다.

> A가 B의 서브타입일 때, Box<A>도 Box<B>의 서브타입이다.

```kotlin
class Box<out T>

fun main(args: Array<String>) {
  val box1: Box<Number> = Box<Int>() // OK!!
  val box2: Box<Int> = Box<Number>() // Compile Error!!
}
```

- in은 타입 파라미터를 contravariant(반공변성)로 만든다

> A가 B의 서브타입일 때, Box<B>가 Box<A>의 서브타입이다.

```kotlin
class Box<in T>

fun main(args: Array<String>) {
  val box1: Box<Int> = Box<Number>() // OK!!
  val box2: Box<Number> = Box<Int>() // Compile Error!!

}
```



<br>

**함수 타입**

- 함수 타입은 파라미터 유형과 리턴 타입에 따라서 관계가 달라진다.

```kotlin
val intToDouble: (Int) -> Number = { it.toDouble() }
val numberAsText: (Number) -> Any = { it.toShort() }
val identity: (Number) -> Number = { it }
val numberToInt: (Number) -> Int = { it.toInt() }
val numberHash: (Any) -> Number = { it.hashCode() }

printProcessedNumber(intToDouble)
printProcessedNumber(numberAsText)
printProcessedNumber(identity)
printProcessedNumber(numberToInt)
printProcessedNumber(numberHash)

fun printProcessedNumber(transition: (Int) -> Any) {
  println(transition(42))
}
```

- 코틀린 함수의 타입의 모든 파라미터 타입은 contravariant(반공변성)이다.
- 코틀린 함수의 타입의 모든 리턴 타입은 covariant(공변성)이다.
- 함수 타입을 사용할 때는 자동으로 variance 한정자가 사용된다.

> (Tin, Tin2) -> Tout

> effective kotlin p138 ~ p139 그림 참고

- 자바의 배열은 covariant(공변성)이다. 이런 특성 때문에 다음과 같은 문제가 발생한다.

```java
// java code
Integer[] numbers = {1, 2, 3, 4};
Object[] objs = numbers;
object[2] = "B"; // Runtime에 ArrayStoreException이 발생
```

- 코틀린은 위와 같은 결함을 해결하기 위해 Array를 invariant(무공변성)으로 만들었다. 따라서 Array<Int>를 Array<Any>등으로 변경할 수 없다.



<br>

**variance(out or in) 한정자 사용시 주의사항**

- effective java를 보면 PECS (producer extends consumer super) 공식을 볼 수 있다. 이는 generic이 존재하는 언어에 모두 적용할 수 있다.

> out (extends), in (super)

- 위의 공식을 지키지 않으면 어떤 문제가 발생하는지 살펴보자.

```kotlin
open class Dog
class Puppy: Dog()
class Hound: Dog()

class Box<out T> {

  private var value: T? = null

  // pseudo code
  fun set(value: T) {
    this.value = value
  }

  // pseudo code
  fun get(): T = value ?: error("value not set")
}

val puppyBox = Box<Puppy>()
val dogBox: Box<Dog> = puppyBox
dogBox.set(Hound()) // Puppy를 위한 공간이므로 문제 발생

val dogHouse = Box<Dog>()
val box: Box<Any> = dogHouse
box.set("Some String") // Dog를 위한 공간이므로 문제 발생
box.set(42) // Dog를 위한 공간이므로 문제 발생
```

캐슽팅 후에 실질적인 객체가 그대로 유지되고, 타이핑 시스템에서만 다르게 처리되기 때문에 위의 코드는 안전하지 않다.
Int를 설정하려고 하는데, 해당 위치는 Dog만을 위한 자리이다. 만약 이것이 가능하다면, 오류가 발생할 것이다.
코틀린은 public in 한정자 위치(private인 경우에는 컴파일에러가 발생하지 않음)에 covariant 타입 파라미터(out 한정자)가 오는 것을 금지하여 이러한 상황을 막는다.

<br>



**PECS 공식을 적용하면 public in 한정자는 consumer에 해당하기 때문에 set을 하는(값을 소비) 위치에만 사용이 가능하다. 반면, public out 한정자는 producer에 해당하기 때문에 get을 하는(값을 생산) 위치에만 사용이 가능하다.**

```kotlin
class Box<out T>

  var value: T? = null // 컴파일 에러, set하는 위치

  // pseudo code
  fun set(value: T) {
    this.value = value // 컴파일 에러, set하는 위치
  }

  // pseudo code
  fun get(): T = value ?: error("value not set") // OK, get하는 위치
```

```kotlin
class Box<in T>

  var value: T? = null // 컴파일 에러, get하는 위치

  // pseudo code
  fun set(value: T) {
    this.value = value // OK, set하는 위치
  }

  // pseudo code
  fun get(): T = value ?: error("value not set") // 컴파일 에러, get하는 위치
```

- 가시성을 private으로 제한하면, 오류가 발생하지 않는다. 객체 내부에서는 업캐스트 객체에 covariant(out 한정자)를 사용할 수 없기 때문이다. (in도 같은 이유로 적용됨)



<br>

**variance 한정자의 위치**

- 선언 부분에 사용

```kotlin
class Box<out T>(val value: T)
val box1: Box<String> = Box("Box")
val box2: Box<Any> = box1
```

- 사용(활용)하는 부분에 사용

```kotlin
class Box<T>(val value: T)
val box1: Box<String> = Box("Box")
val box2: Box<out Any> = box1
```

- 모든 인스턴스에 variance 한정자를 적용하면 안 되고, 특정 인스턴스에만 적용해야 할 때 위와 같이 사용한다.


### 아이템 25: 공통 모듈을 추출해서 여러 플랫폼에서 재사용하라

- kotlin은 멀티 플랫폼을 지원하기 때문에 코틀린 코드를 서로 다른 플랫폼(웹 백엔드 & 프런트엔드, Android & IOS 등등)에서 공유가 가능하다.
- 공통 로직을 모듈화해서 서로 다른 플랫폼에서 사용할 수 있다.

(kotlin은 다른 언어, 다른 플랫폼으로의 변화가 자유롭기 때문에, 공통 모듈을 통해 코드를 공유할 수 있다!)

### 아이템 26: 함수 내부의 추상화 레벨을 통일하라

**추상화 레벨**

- 추상화 레벨은 구체적인 동작, 프로세서, 입출력과 가까울수록 낮은 레벨이라고 한다.
- 높은 레벨일수록 걱정해야 하는 세부 내용은 적어지지만, 제어력(control)을 잃는다.
  - ex) 자바는 가비지 컬렉터가 자동으로 메모리 관리를 해주므로 메모리 사용을 최적화하기 어렵다.

**추상화 레벨 통일**

- 코드도 추상화를 계층처럼 만들어서 사용할 수 있다.
- 그 도구가 바로 `함수`이고, 함수도 높은 레벨과 낮은 레벨을 구분해서 사용해야 한다 = `추상화 레벨 통일(Single Level of Abstraction, SLA)` 
- 예시

```kotlin
class CoffeeMachine {
    fun makeCoffee() {
        // 복잡한 로직
    }
}
```

```kotlin
class CoffeeMachine {
    fun makeCoffee() {
        boilWater()
        brewCoffee()
        pourCoffee()
        pourMilk()
    }
}
```

- 위 예시에서 간단한 추상화를 추출함으로써 더 많은 이점을 갖게 된다.
  - 재사용과 테스트가 쉬워진다
  - 가독성이 높아진다
  - 이해가 쉬워진다 (필요한 경우에만 낮은 레벨의 코드를 살펴보면 된다)
- 함수는 작아야하며 최소한의 책임만 가져야 한다.
  - 함수가 복잡하면 일부분을 추출해서 추상화는 것이 좋다.

**프로그램 아키텍처의 추상 레벨**
- 추상화를 구분하면 서브 시스템의 세부사항을 숨김으로써 상호 운영성과 플랫폼 독립성을 얻을 수 있다.
- ex) 모듈 시스템에서 입출력 모듈(낮은 레벨 모듈)과 비즈니스 로직(높은 레벨 모듈)을 잘 분리한다.

**궁금한점**
실무에서는 어떤 방식을 차용하는지?

