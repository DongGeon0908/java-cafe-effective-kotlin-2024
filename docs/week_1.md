# Week 1

### Chapter

- 아이템 1: 가변성을 제한하라
- 아이템 2: 변수의 스코프를 최소화하라
- 아이템 3: 최대한 플랫폼 타입을 사용하지 말라



### Item.1 가변성을 제한하라

- 코틀린은 모듈로 프로그램을 설계하는데 모듈은 클래스, 객체, 함수, 타입 별칭, 톱레벨 프로퍼티 등 다양한 요소로 구성
- 이러한 요소 중에 일부는 상태를 가질 수 있는데, 프로퍼티 **var를 사용하거나 mutable 객체**를 사용하면 상태를 가질 수 있음
  - **read-write-property**
- 코틀린은 프로퍼티나 객체에 아래와 같이 상태를 가질 수 있다

```kotlin
var a = 10
var list: MutableList<Int> = mutableListOf()
```

- 이렇게 요소가 상태를 갖는 경우, 해당 요소의 동작은 **사용 방법뿐만 아니라 히스토리에도 의존**하게 됨



**상태를 가지는 클래스 예시**

```kotlin
class BankAccount {
    var balance = 0.0
        private set

    fun deposit(depositAmount: Double) {
        balance += depositAmount
    }

    fun withdraw(withdrawAmount: Double) {
        if (balance < withdrawAmount) {
            throw InsufficientFunds()
        }
        balance -= withdrawAmount
    }
}

class InsufficientFunds: Exception()

fun main() {
    val account = BankAccount()
    println(account.balance) // 0.0
    account.deposit(100.0)
    println(account.balance) // 100.0
    account.withdraw(50.0)
    println(account.balance) // 50.0
}
```

- 상태를 갖게 하는 것은 양나르이 검인데, 시간의 변화에 따라서 변하는 요소를 표현하는 것은 유요하지만, 상태를 적절하게 관리하는 것은 생각보다 어려운 문제



**상태를 가지는 경우 발생할 수 있는 문제점**

- 프로그램을 이해하고 디버그하기 힘들어짐
  - 클래스 이해도 어렵고 그에 따라 수정하기도 힘듬
- 가변성이 있으면 코드의 실행을 추론하기 어려워짐
  - 시점에 따라 값이 달라질 수 있으므로, 현재 어떤 값을 갖고 있는지에 따라 실행을 예측할 수 있음
- 멀티스레드 프로그램일 때는 적절한 동기화가 필요
  - 변경이 발생하는 부분은 충돌 문제가 항상 있음
- 테스트하기 어려움
  - 모든 상태를 테스트 해야하므로 변경이 많으면 더 많은 조함을 테스트 해야함
- 상태 변경이 일어날때 관련된 다른 부분에 알려줘야 하는 경우가 있음
  - 정렬되어 있는 리스트에 가변 요소가 추가되면 전체 다시 정렬을 해야함



**대규모 팀에수 이러한 문제로 많은 이슈가 발생**

- 어떤 값이 상태를 물고 있는지 모르고 있던, A 개발자.. 해당 값에 비즈니스로직을 추가하자.. 의도치 않은 여러 이슈가 발생하는데..



**멀티스레드 환경에서의 상태 프로퍼티의 문제점**

```kotlin
// 멀티 스레드
private fun multiThread() {
    var num = 0
    for (i in 1..1000) {
        thread {
            Thread.sleep(10)
            num += 1
        }
    }
    Thread.sleep(5000)
    println(num) // 1000이 아닐 확률이 매우 높음
}

// 코루틴
private suspend fun coroutine() {
    var num = 0
    coroutineScope {
        for (i in 1..1000) {
            launch {
                delay(10)
                num += 1
            }
        }
    }
    println(num)
}

suspend fun main() {
    multiThread()
    multiThread()
    coroutine()
    coroutine()
}
```

- 멀티스레드를 활용해 프로퍼티를 수정하는 경우 충돌로 인해 일부 연산이 이루어지지 않음, 코루틴의 경우 더 적은 스레드가 관여되어 충돌문제가 줄어들기는 하나 문제가 사라지지 않음
- 즉 프로퍼티가 상태에서 멀티 스레드를 활용하게 된다면 충돌이 되지 않도록 적절하게 동기화를 구현해야함
  - 그러나 동기화를 잘 구현하는 것은 굉장히 어려운 일

```kotlin
private fun locking() {
    val lock = Any()
    var num = 0

    for (i in 1..1000) {
        thread {
            Thread.sleep(10)
            synchronized(lock) {
                num += 1
            }
        }
    }

    Thread.sleep(100)
    println(num)
}
```

- 이렇듯 가변성은 생각보다 단점이 많아 가변성을 완전하게 제한하는 언어도 등장했을 정도로 가변성은 프로그램에서 중요한 문제
  - ex) Haskell → 매우 어렵고 코드 작성이 까다로움
- 즉 프로그램을 구현할때에 가변성을 염두해두고 변경이 일어나야 하는 부분을 신중하고 확실하게 결정하고 사용해야함



### 코틀린에서 가변성 제한하기

- 코틀린은 가변성을 제한할 수 있게 설계되어져 있어서 immutable 객체를 만들거나 프로퍼티를 변경할 수 없게 막는 것이 굉장히 쉬움



**코틀린에서 대표적으로 가변성을 제한하는 방법**

- 읽기 전용 프로퍼티 (val)
- 가변 컬렉션과 읽기 전용 컬렉션 구분
- 데이터 클래스의 copy



**읽기 전용 프로퍼티 (val)**

- 읽기 전용 프로퍼티를 활용해 마치 값처럼 동작하며 일반적인 방법으로는 변하지 않음

```kotlin
val a = 10
a = 20 // 오류
```

- 읽기 전용 프로퍼티가 mutable한 객체를 담고 있다면, 내부적으로 변할 수 있음

```kotlin
val list = mutableListOf(1,2,3)
list.add(4)
```

- 읽기 전용 프로퍼티는 다른 프로퍼티를 활용하는 사용자 정의 게터로도 정의할 수 있습니다. 이렇게 var프로퍼티를 사용하는 val 프로퍼티는 var프로퍼티가 변할 때 변할 수 있습니다.

```kotlin
var name: String = "Marcin"
var surname: String = "Mkskala"
val fullName 
    get() = "$name $surname"
```

- 코틀린의 프로퍼티는 기본적으로 캡슐화되어 있고, 추가적으로 사용자 정의 접근자를 가질 수 있음
  - 이러한 특성으로 코틀린은 API를 변경하거나 정의할 때 유연성을 가짐
- var는 게터와 세터를 모두 제공하지만 val은 게터만 제공하므로 val을 var로 오버라이드 할 수 있음

```kotlin
interface Element {
    var active: Boolean
}

class ActualElement: Element {
    override var active: Boolean = false
}
```

- val은 읽기 전용 프로퍼티지만, 변경할 수 없음을 의미하지 않음
  - 만약 완전히 변경할 필요가 없다면, final 프로퍼티를 시용하는 것이 좋음
  - 게터 또는 델리게이트로 정의 가능 (delegate : 특정 기능을 다른 객체에 위임)
- val은 정의 옆에 상태가 바로 적히므로, 코드의 실행을 예측하는 것이 훨씬 간단하고 스마트 캐스트(smart cast)등의 추가적 인 기능을 활용할 수 있음

```kotlin
val name: String? = "Marton"
val surname: String = "Braun"

val fullName: String? 
    get() = name?.let { "$it $surname" }
val fullName2: String? = name?.let { "$it $surname" }

fun main() {
    if (fullName != null) {
        println(fullName.length) // 오류
    }
    
    if (fullName2 != null) {
        println(fullName2.length)
    }
}
```

- fullName은 게터로 정의했으므로 스마트 캐스트를 할 수 없음
  - 게터를 활용하므로, **값을 사용하는 시점의 name에 따라서 다른 결과가 나올 수 있기 때문**
  - 반면 fullName2는 지역 변수가 아닌 프로퍼티가 final 이고, 사용자 정의 게터를 갖지 않음



**가변 컬렉션과 읽기 전용 컬렉션 구분하기**

- 코틀린은 읽기 전용 컬렉션과 읽고 쓸 수 있는 컬렉션으로 구분됨
  - 읽기 전용 : Iterable, Collection, Set, List 인터페이스
  - 읽고 쓸 수 있는 : MutableInterable, MutableCollection, MutableSet, MutableList 인터페이스
- 읽기 전용 컬렉션이 내부의 값을 변경할 수 없다는 의미는 아니고 대부분의 경우에는 변경할 수 있지만 읽기 전용 인터페이스가 이를 지원하지 않으므로 변경할 수 없음
  - Iterable<T>.map 과 Iterable<T>.filter 함수는 ArrayList를 리턴한다



**Iterable map**

```kotlin
inline fun<T, R> Iterable<T>.map(
    transformation: (T) -> R
): List<R> {
    val list = ArrayList<R>()
    for (elem in this){
        list.add(transformation(elem))
    }
    return list
}
```

- 이러한 컬렉션을 진자로 불변하게 만들지 않고, 읽기 전용으로 설계한 것이 중요한 부분
- 코틀린은 내부적으로 immutable하지 않은 컬렉션을 외부적으로 immutable하게 보이게 만들어서 얻어지는 안정성



**다운캐스팅 문제**

```kotlin
val list = listOf(1, 2, 3)

if (list is MutableList) {
	list.add(4)
}

// 만약 변경이 필요하다면 아래와 같이 새로운 mutable list를 만들어서 리턴
val list = listOf(1,2,3)

val mutableList = list.toMutableList()
mutableList.add(4)
```

- 리스트를 읽기 전용으로 리턴하면, 이를 읽기 전용으로만 사용해야하는데, 이는 단순한 규약의 문제인데 다운 캐스팅은 이러한 계약을 위반하고 추상화를 무시하는 행위이기 때문에 이러한 코드는 안전하지 않고 예측하지 못한 결과를 초래함
- 위 결과는 플랫폼에 따라 다르며 JVM에서 listOf는 자바의 List 인터페이스을 구현한 ArrayList 인스턴스를 리턴 그러나 1년 뒤에 이것이 어떻게 동작하는지 보장할 수 없음
- 즉 읽기 전용 컬렉션을 mutable로 다운 캐스팅을 하는 것은 굉장히 좋지 않고 **만약 변경이 필요하다면 copy를 통해 새로운 mutable 컬렉션을 만들어서 활용**하는 것이 좋음



**데이터 클래스의 Copy**

- Immutable 객체의 장점

  - 한 번 정의된 상태가 유지되므로 코드 이해가 쉬움
  - 공유했을 때도 충돌이 따로 이루어지지 않으므로, 병렬 처리를 안전하게 할 수 있음
  - 객체에 대한 참조는 변경되지 않으므로 쉽게 캐시가 가능
  - 방어적 복사본을 만들 필요가 없음
  - 다른 객체를 만들 때 활용하기 좋고 실행을 더 쉽게 예측가능
  - set, map 키로 사용할 수 있는데 mutable 객체는 이러한 것으로 사용할 수 없다.
    - 세트와 맵인 내부적으로 해시 테이블을 사용하고 해시 테이블은 처음 요소를 넣을때 요소의 값을 기반으로 버킷을 결정하기 때문에 요소의 값이 수정이 되면 해시 테이블 내부에서 요소를 찾을 수 없게되기 때문

  ```kotlin
  val names: SortedSet<FullName> = TreeSet()
  val person = FullName("AAA", "AAA")
  names.add(person)
  names.add(FullName("Jordan", "Hansen"))
  names.add(FullName("David", "Blanc"))
  println(names)
  println(person in names) // true
  
  person.name = "ZZZ"
  println(names)
  println(person in names) // false
  ```

- immutable 객체는 위와 같은 장점을 가지고 있으나 객체를 변경할 수 없다는 단점이 있는데 만약 수정하고자 하면 새로운 객체를 만들어 내는 메서드를 가져야 함

```kotlin
class User(
    val name: String,
    val surname: String,
) {
    fun withSurname(surname: String) = User(name, surname)
}
var user = User("Nathan", "Hong")
user = user.withSurname("Jo")
print(user)
```

- 다만 모든 프로퍼티를 대상으로 이런 함수를 하나하나 만드는 것은 굉장히 귀찮은 일이기 때문에 data 한정사를 사용해 copy 메소드를 활용하면 기본 생성자 프로퍼티가 같은 새로은 객체를 만들어낼 수 있음

```kotlin
data class(
    val name: String,
    val surname: String,
)
var user = User("Nathan", "Hong")
user = user.copy(usrname = "Jo")
print(user)
```



### 다른 종류의 변경 가능 지점

- 변경할 수 있는 리스트는 아래 두 가지 방식으로 만들 수 있음

```kotlin
val list1: MutableList<Int> = mutableListOf()
var list2: List<Int> = listOf()

list1.add(1)
list2 = list2 + 1

list1 += 1 // list1.plusAssign(1)
list2 += 1 // list2 = list2.plus(1)
```

- 위 두 가지 방식 비교

  - list1 : mutable 컬렉션 사용

    - 구체적인 리스트 구현 내부에 변경 가능 지점이 있음
    - 멀티스레디 처리가 이루어지는 경우 내부적으로 적절한 동기화가 되어 있는지 확실하게 알 수 없음

  - list2 : immutable 컬렉션 사용 & 프로퍼티 자체가 변경 가능

    - 객체 변경을 제어하기 더 쉬움
    - 사용자 정의 세터 혹은 이를 사용하는 델리케이트를 활용해 변경을 추적할 수 있음

    ```
    var list = listOf<Int>()
    for(i in 1..1000) {
        thread {
            list = list + i
        }
    }
    Thread.sleep(1000)
    println(list.size)
    
    var names by Delegates.observable(listOf<String>()) {_, old, new ->
        println("Names changed from $old to $new")
    }
    
    names += "Fabio"
    names += "Bill"
    ```

  - mutable 컬렉션도 관찰할 수 있게 만들려면 추가적 구현이 필요하기 때문에 mutable 프로퍼티에 읽기 전용 컬렉션을 넣어 사용하는게 쉬움

- 최악의 방식

  - mutable 컬렉션과 mutable 프로퍼티 동시에 사용
    - 프로퍼티, 컬렉션 모두 변경 가능 지점

  ```kotlin
  var list3 = mutableListOf<Int>()
  ```

- 위 두 가지 방식 모두 상태를 제어 및 이해하는 비용이 들어가므로 가변성을 제한하는 것이 가장 좋음



### 변경 가능 지점 노출하지 말기

- 상태를 나타내는 mutable 객체를 외부에 노출하는 것은 굉장히 위험

```kotlin
data class User(
    val name: String
)

class UserRepository {
    private val storedUsers: MutableMap<Int, String> = mutableMapOf()
    
    fun loadAll(): MutableMap<Int, String> {
        return storedUsers
    }
}

val userRepository = UserRepository()
val storedUsers = userRepository.loadAll()
storedUsers[4] = "Kiraill"
```

- 위와 같은 코드는 돌발적인 수정이 일어나면 위험할 수 있으므로 아래 두 가지 방식 중 하나로 적절하게 처리해야함
  - 리턴되는 객체를 복제
    - user.copy()
  - 읽기 전용 슈퍼 타입으로 업캐스팅하여 가변성을 제어
    - fun loadAll(): Map<Int, String>

### 정리

- 가변성을 제한한 immutable 객체를 사용하는 것이 좋은 이유에 대한 내용이고 코틀린이 가변 지점을 제어하기 위해 사용되는 다양한 도구를 소개
  - var 보다는 val을 사용하는 것이 좋음
  - mutable 프로퍼티보다는 immutable 프로퍼티를 사용하는 것이 좋음
  - mutable 객체와 클래스보다는 immutable 객체와 클래스를 사용하는 것이 좋음
  - 변경이 필요한 대상을 만들어야 한다면, immutable 데이터 클래스로 만들고 copy를 활용하는 것이 좋음
  - 컬렉션에 상태를 저장해야 한다변, mutable 컬렉션보다는 읽기 전용 컬렉션을 사용하는 것이 좋음
  - 변이 지점을 적절하게 설계하고, 불필요한 변이 지점은 만들지 않는 것이 좋음
  - mutable 객체를 외부에 노출하지 않는 것이 좋음





### 질문

- 위에 나온 내용을 읽었을 때, 가변 객체보다는, 불변 객체를 사용해라! 라고 판단되어지는데...
  - 그럼 불변 객체만 사용했을 때 발생할 수 있는 문제들은?
  - 저의 경우에는 새롭게 객체를 계속 만들다 보니, gc로 바로 보내질 객체들이 너무 많이 생성되는 이슈가 발생









---





### Item.2 변수의 스코프를 최소화하라

- 상태를 정의할 때는 변수와 프로퍼티의 소코프를 최소화하는 것이 좋음
  - 프로퍼티보다는 지역 변수 사용
  - 최대한 좁은 스코프를 갖게 되는 변수 사용
- 요소의 스코프라는 것은 요소를 볼 수 있는 컴퓨터 프로그램 영역
  - 코틀린의 스코프는 기본적으로 중괄호로 만들어지며, 내부 스코프에서 외부 스코프에 있는 요소에만 접근할 수 있음

```kotlin
val a = 1
fun fizz() {
    val b = 2
    println(a + b)
}

val buzz = {
    val c = 3
    println(a + c)
}
```

변수 스코프를 제한하는 예

```kotlin
val users = listOf<User>()
var user: User

// 외부/내부 모두 사용할 수 있는 변수 사용
for (i in users.indices) {
    user = users[i]
    println("User at $i is $user")
}

// user 스코프를 내부로 제한
for (i in users.indices) {
    val user = users[i]
    println("User at $i is $user")
}

for ((i, user) in users.withIndex()){
    println("User at $i is $user")
}
```

- 스코프를 좁게 만드는 것이 좋은 가장 큰 이유는 프로그램을 추적하고 관리하기 쉽기 때문임
  - 스코프 범위가 너무 넓으면 다른 개발자에 의해 변수가 잘못 사용될 수 있음
- 추가로 변수 정의시에도 초기화하는 것이 좋은데 코틀린은 if, when, try-catch, Elvis 표현식 등을 활용하면 최대한 변수를 정의할때 초기할 수 있음
- 여러 프로퍼티를 한꺼번에 설정해야 하는 경우, 구조분해 선언(destructuring declaration), 구조분해 할당

```kotlin
// bad
val user: User

if (hasValue) {
   user = getValue()
} else {
   user = User()
}

// good
val user: User = if (hasValue) {
    getValue()
} else {
    User()
}

// 구조분해 선언

// bad
fun updateWeather(degrees: Int) {
    val description: String
    val color: String
    if (degrees < 5) {
        description = "cold"
        color = "BLUE"
    } else if (degrees < 23) {
        description = "mild"
        color = "YELLOW"
    } else {
        description = "hot"
        color = "RED"
    }
}

// good
fun updateWeather(degrees: Int) {
    val (description, color) = when {
        degrees < 5 -> "color" to "BLUE"
        degrees < 23 -> "mild" to "YELLOW"
        else -> "hot" to "RED"
    }
}
```



### Kotlin의 Capture

- 람다식이나 익명 함수에서 외부의 변수나 파라미터를 참조할때, 해당 변수나 파라미터 값을 복사하여 사용하는 것을 의미



### 넓은 스코프 범위의 문제 : 캡처링

- 에라토스테네스의 체 (소수를 구현하느 알고리즘)를 통해 캡처링 이슈 파악
  1. 2부터 시작하는 숫자 리스트 만듬
  2. 첫번째 요소를 선택
  3. 남아 있는 숫자 중에서 2번에서 선택한 소수로 나눌 수 있는 모든 숫자를 제거
- 에라토스테네스의 체 구현

구현 예

```kotlin
var numbers = (2..100).toList()
val primes = mutableListOf<Int>()

while (numbers.isNotEmpty()) {
    val prime = numbers.first()
    primes.add(prime)
    numbers = numbers.filter { it % prime != 0 }
}
println(primes)
```

시퀀스를 활용한 구현 예

```kotlin
val primes: Sequence<Int> = sequence {
    var numbers = generateSequence(2) { it + 1 }

    while (true) {
        val prime = numbers.first()
        yield(prime)
        numbers = numbers.drop(1)
            .filter {
                it % prime != 0
            }
    }
}
println(primes.take(10).toList()) // [2, 3, 5, 7. 11, 13, 17, 19, 23, 29]
```

**잘못 활용한 예**

```kotlin
val primes: Sequence<Int> = sequence {
    var numbers = generateSequence(2) { it + 1 }
    var prime: Int
    while (true) {
        prime = numbers.first()
        yield(prime)
        numbers = numbers.drop(1)
            .filter {
                it % prime != 0
            }
    }
}
println(primes.take(10).toList()) // [2, 3, 5, 6, 7, 8, 9, 10, 11, 12]
```

- 위 결과가 잘못 나온 이유
  - 필터링은 시퀀스를 사용하기 때문에 나중에 실행되는데 (시퀀스의 지연처리), 모든 스텝에서 점점 필터가 체이닝 되는데 위 코드에서는 항상 변경 가능한 prime 을 참조하게 됨
  - 따라서 항상 가장 마지막의 prime 값으로만 필터링 된 것
- **이러한 잠재적인 캡처 문제등이 발생할 수 있으므로 가변성을 피하고 스코프 범위를 좁게 만들어야 함**

|      | code                                        | prime |
| ---- | ------------------------------------------- | ----- |
| 2    | numbers.first()                             | 0     |
|      | generate 2 (hit)                            |       |
|      | prime 2                                     | 2     |
|      | yield 2                                     |       |
|      | numbers.drop.filter()                       |       |
| 3    | numbers.first()                             | 2     |
|      | generate 2                                  |       |
|      | numbers.drop[2].filter (drop)               |       |
|      | generate 3                                  |       |
|      | numbers.drop.filter { 3 2 } → hit           |       |
|      | → prime 3                                   | 3     |
|      | yield 3                                     |       |
|      | numbers.drop.filter.drop.filter             |       |
| 5    | numbers.first()                             | 3     |
|      | generate 2                                  |       |
|      | numbers.drop[2].filter.drop.filter (drop)   |       |
|      | generate 3                                  |       |
|      | numbers.filter[3 3].drop.filter (pass)      |       |
|      | generate 4                                  |       |
|      | numbers.drop.filter[4 3].drop.filter (hit)  |       |
|      | numbers.drop.filter.drop[4].filter (drop)   |       |
|      | generate 5                                  |       |
|      | numbers.drop.filter.drop.filter[5 3] (hit)  |       |
|      | → prime 5                                   | 5     |
|      | yield 5                                     |       |
|      | numbers.drop.filter.drop.filter.drop.filter |       |

###  

### **정리**

- 변수의 스코프는 좁게 만들어서 활용하는 것이 좋고 var보다는 val을 사용하는 것이 좋음
- 람다에서 변수를 캡처한다는 것을 꼭 기억해야함



---





### Item.3 최대한 플랫폼 타입을 사용하지 마라

- 코틀린은 null-safety 매커니즘으로 인해 NPE를 거의 찾아보기 힘듬
- null-safety 매커니즘이 없는 자바, C 등의 프로그래밍 언어와 코틀린을 연결해서 사용할 때는 NPE 예외가 발생할 수 있음

```kotlin
public class JavaTest{ 
    public String giveName() { ... }
}
```

- 위 자바 코드로 반환된 타입을 사용할때에 @Nullable 어노테이션이 붙어 있다면 nullable로 추정하고 String?으로 변경하면 되는데 만약 붙어 있지 않다면 자바에서 모든 것이 nullable일 수 있으므로 최대한 안전하게 접근하기 위해 nullable로 가정하고 접근해야 함



**제네릭 타입**

```kotlin
public class UserRepo {
    public List<User> getUsers() { ...}
}

val users: List<User> = UserRepo().users!!.filterNotNull()
```

- 코틀린이 디폴트로 모든 타입을 nullable로 다룬다면, 이를 사용할 때 이러한 리스트와 리스트 내부의 User 객체들이 널 아니라는 것을 알아야 함
- 그래서 코틀린은 자바 등의 다른 프로그래밍 언어에서 넘어온 타입들을 특수하게 다루고 이러한 타입을 플랫폼 타입이라고 부름
  - 플랫폼 타입 : 다른 프로그래밍 언어에서 전달되어서 nullable인지 아닌지 알 수 없는 타입

```kotlin
val repo = UserRepo()
val user1 = repo.user // user1의 타입 User!
val user2: User = repo.user // User
val user3: User? = repo     // User?

val users: List<User> = UserRepo().users
val users: List<List<User>> = UserRepo().groupedUsers
```

- 코틀린에서는 플랫폼 타입은 타입 뒤에 ! 기호를 붙여서 표기함
- 그러나 문제는 null이 아니라고 생각되는 것이 null일 가능성이 있으므로 여전히 위험하기 때문에 항상 주의를 기울여야 하고 설계자가 명시적으로 어노테이션으로 표기하거나 주석으로 달아두어야함



**어노테이션 지원 목록**

- JetBrains의  @Nullable @NotNull
- 안드로이드 @Nullable @NonNull
- JSR-305( @Nullable, @CheckForNull @Nonnull
- JavaX( @Nullable@CheckForNull@Nonnulljavax.annotation
- FindBugs( @Nullable, @CheckForNull, @PossiblyNull @NonNull
- ReactiveX( @Nullable @NonNull
- 이클립스 ( @Nullable @NonNull
- 롬복 ( @NonNull
- 또는 JSR 305의 @ParametersAreNonnullByDefault주석을 사용하여 기본적으로 모든 유형이 Notnull이어야 함을 Java에서 지정할 수 있습니다 .



**플랫폼 타입 사용의 문제점**

```kotlin
public class JavaClass {
    public String getValue() {
        return null;
    }
}

fun staredType() {
    val value: String = JavaClass().value
    println(value.length)
}

fun platformType() {
    val value = JavaClass().value
    println(value.length)
}
```

- 코틀린에서도 위와 같이 플랫폼 코드를 사용할 수 있으나 플랫폼 타입은 안전하지 않으므로 빨리 제거하는 것이 좋음
- 위 stratedType, platformType 모두 null을 리턴한다고 가정하지 않으면 NPE가 발생
  - **startedType**의 경우 자바에서 가져오는 값을 가져오는 위치에서 NPE 발생
    - null이 아니라고 예상했지만 null이 나와 가져오는 지점에서 발생
  - **platformType**은 값을 활용할때 NPE 발생
    - 플랫폼 타입으로 지정된 변수는 nullable일 수도 있고, 아닐 수도 있어서 실제로 활용하는 라인에서 발생

```kotlin
interface UserRepo {
    fun getUserName() = JavaClass().value
}

class UserRepoImp: UserRepo {
    override fun getUserName():String? {
        return null
    }
}

fun main() {
    val repo: UserRepo = UserRepoImp()
    val text: String = repo.getUserName()
    println("User name length is ${text.length}")
}
```

- 위 인터페이스 메소드의 inferred 타입이 플랫폼 타입이므로 누구나 nullable여부를 지정할 수 있는데 사용하는 쪽에서 nullable이 아니라고 받아들였다면 문제가 발생
- 즉 플랫폼 타입이 전파되는 일은 굉장히 위험하고 가급적 제거하는 것이 좋음



### 정리

- 다른 프로그래밍 언어에서 와서 nullable 여부를 알 수 없는 타입을 플랫폼 타입이라고 함
- 플랫폼 타입은 사용하는 코드말고도 활용하는 곳까지 영향을 줄 수 있으므로 가급적 제거하는 것이 좋음
- 혹은 사용하게 되었을때는 nullable 여부를 지정하는 어노테이션을 활용하는 것이 좋음

