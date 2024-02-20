# Week_4

### 목차

- 아이템 14: 변수 타입이 명확하게 보이지 않는 경우 확실하게 지정하라
- 아이템 15: 리시버를 명시적으로 참조하라
- 아이템 16: 프로퍼티는 동작이 아니라 상태를 나타내야 한다
- 아이템 17: 이름 있는 아규먼트를 사용하라
- 아이템 18: 코딩 컨벤션 지켜라

<br>
<hr>
<br>

### 아이템 14: 변수 타입이 명확하게 보이지 않는 경우 확실하게 지정하라

코틀린은 컴파일 시점에 타입이 결정된다. 코틀린은 수준 높은 타입 추론 시스템을 갖추고 있다.

```kotlin
val num = 10 // Int
val name = "Marcin" // String
val ids = listOf(12, 112, 554, 997) // List<Int>
```

유형이 명확할 때 코드가 짧아져 가독성이 크게 향상된다.

하지만 유형이 명확하지 않을 때 좋지 않다.

```kotlin
val data = getSomeData() // 타입을 숨김
```

이런 행위는 가독성이 떨어진다. 깃허브 같은 환경에서는 코드 정의로 쉽게 이동하기 어렵다.

따라서 유형이 명확하지 않다면 아래와 같이 타입을 지정해준다.

```kotlin
val data: UserData = getSomeData()
```

이는 아이템3, 아이템4 에서 다루었듯 가독성 향상 이외에 안전을 위한 행위다.

**타입을 무조건 지정하는 것이 아닌 상황에 맞게 사용하면 된다.**

<br >
<hr >
<br >

### 아이템 15: 리시버를 명시적으로 참조하라

코틀린에서 리시버를 사용하는 경우가 있다.ex.this

```kotlin
Class User : Person (){
    private var beersDrunk: Int = 0

    fun drinknBeers(num: Int) {
        this.beersDrunk += num
    }
}
```

확장 메소드에서 this를 명시적으로 참조할 수 있다.아래는 리시버를 명시적으로 표시하지 않은 퀵소트와 명시적으로 표시한 퀵소트이다.

```kotlin

// 명시적으로 표시하지 않은 퀵소트
fun <T : Comparable<T>> List<T>.quickSort(): List<T> {
    if (size < 2) return this
    val pivot = first()
    val (smaller, bigger) = drop(1)
        .partition { it < pivot }
    return smaller.quickSort() + pivot + bigger.quickSort()
}

// 명시적으로 표시한 퀵소트
fun <T : Comparable<T>> List<T>.quickSort(): List<T> {
    if (size < 2) return this
    val pivot = this.first()
    val (smaller, bigger) = this.drop(1)
        .partition { it < pivot }
    return smaller.quickSort() + pivot + bigger.quickSort()
}

```

두 함수의 사용 결과는 차이가 없다 . this를 생략하면 코드가 간결해져 읽기 좋아진다.여러 개의 리시버
스코프 내부에 아래와 같이 둘 이상의 리시버가 있는 경우, 리시버를 명시적으로 나타내면 좋다 .

```kotlin

class Node(val name: String) {
    fun makeChild(childName: String) =
// 클래스의 this.name 인지 새로 생성한 자식 node.name 인지 알 수 없다.
        create("$name.$childName")
            .apply { print("Created ${name}") }

    fun create(name: String): Node? = Node(name)

}

fun main() {
    val node = Node("parent")
    node.makeChild("child") // created parent
}

```

위 코드의 결과는 일반적으로 Created parent . child 라고 예상하지만, 실제로는 Created parent이다.이해하기 쉽게 앞에 리시버를 붙여보자.

```kotlin

class Node(val name: String) {
    fun makeChild(childName: String) =
        create("$name.$childName")
            .apply { print("Created ${this.name}") } // compile error

    fun create(name: String): Node? = Node(name)

}

fun main() {
    val node = Node("parent")
    node.makeChild("child") // created parent
}

```

기존의 코드에서 this가 붙었다 . 이런 경우 this의 타입이 Node ? 여서 컴파일 에러가 발생한다.그러면 this?.name 을 사용하면 되지않나 라고 할 수 있다 . 이는 apply의 잘못된
사용이다.일반적으로 nullable 값을 처리할 때 also 또는 let을 사용하는 것이 더 좋다 . class Node(val name: String) {
fun makeChild(childName: String) {
create("$name.$childName").also { print("Created ${it?.name}") }
}

```kotlin

fun create(name: String) = Node(name)
}

```

**DSL 마커**
코틀린 DSL을 사용할 때 여러 리시버를 가진 요소들이 중첩되더라도, 리시버를 명시적으로 붙이지 않는다 . DSL은 리시버를 생략하도록 설계되었기 때문에 .. 그렇지만, DSL 외부의 함수를 사용하는 것이 위험한
경우가 있다 .

```kotlin

table {
    tr {
        td { +"Column1" }
        td { +"Column2" }
    }
    tr {
        td { +"Value1" }
        td { +"Value2" }
    }
}

```

이런 잘못된 사용을 막으려면, 암묵적으로 외부 리시버를 사용하는 것을 막는 DslMarker라는 메타 어노테이션을 사용해야한다 .

```kotlin

@DslMarker
annotation class HtmlDsl

fun table(f: TableDsl.() -> Unit) { /*...*/
}

@HtmlDsl
class TableDsl { /*...*/ }

```

이렇게 하면 암묵적으로 외부 리시버를 사용하는 것이 금지된다 .

```kotlin

table {
    tr {
        td { +"Column1" }
        td { +"Column2" }
    }
    tr { // 컴파일 오류
        td { +"Value1" }
        td { +"Value2" }
    }
}

// 사용하고 싶으면
table {
    tr {
        td { +"Column1" }
        td { +"Column2" }
    }
    this@table.tr {
        td { +"Value1" }
        td { +"Value2" }
    }
}

```

**정리 * *
-짧게 적을 수 있따는 이유만으로 리시버를 제거하지 말아야 한다. -> 혼란야기
-여러 개의 리시버가 있는 상황 등에서는 리시버를 명시적으로 적어 주는 것이 좋다 . -> 가독성 향상
-DSL 외부 스코프에 있는 리시버를 명시적으로 적게 강제하고 싶다면, DslMarker 메타 어노테이션을 사용한다 .

<br >
<hr >
<br >

### 아이템 16: 프로퍼티는 동작이 아니라 상태를 나타내야 한다

코틀린의 프로퍼티는 자바의 필드와 비슷해 보인다 . 하지만 서로 완전 다른 개념이다 .

```kotlin

// kotlin property
var name: String? = null

// java field
String name = null;

```

둘 다 데이터를 저장한다는 점은 같지만, 프로퍼티에는 더 많은 기능이 있다.일단 기본적으로 프로퍼티는 사용자 정의 getter와 setter를 가질 수 있다 .

```kotlin

// 기본 getter와 setter
class Person(var name: String)

fun main() {
    val person = Person("ABC")
    val name = person.name // person.getName()
    person.name = "EFG" // person.setName()
}

// custom getter setter
class Person(name: String) {
    var name: String? = null
        get() = field?.toUpperCase()
        set(value) {
            if (!value.isNullOrBlank()) {
                field = value
            }
        }
}

```

위 코드에서 프로퍼티의 데이터를 저장해 두는 backing field에 대한 레퍼런스인 field를 확인할 수 있다 . 배킹 필드는 세터와 게터의 디폴트 구현에 사용되므로, 따로 만들지 않아도 디폴트로
생성된다.val을 사용해 읽기 전용 프로퍼티를 만들 때는 field가 생성되지 않는다 .

프로퍼티를 통한 getter, setter접근은 필드와 다를 수 있다.

```kotlin

var date: Date
    get() = Date(millis) // 매번 Date 객체를 생성합니다.
    set(value) {
// 데이터를 millis라는 별도의 프로퍼티로 옮기고 Date 프로퍼티에 저장하지 않는다.
        millis = value.time
    }

```

프로퍼티는 필드가 필요 없다 . 오히려 프로퍼티는 개념적으로 접근자(val의 경우 getter, var의 경우 getter 와 setter) 를 나타낸다.따라서 코틀린은 인터페이스에도 프로퍼티를 정의할 수 있다.

```kotlin

interface Person {
    val name: String // 프로퍼티
}
open class SuperComputer {
    open val theAnswer: Long = 42
}

class AppleComputer : SuperComputer() {
    override val theAnswer: Long = 1_800_275_2273
}

```

이러한 이유로 코틀린은 프로퍼티를 위임(property delegation) 을 할 수 있다.프로퍼티 위임은 아이템21에서 자세하게 설명된다.

```kotlin

//property delegation
val db: Database by lazy { connectToDb }

```

프로퍼티는 본질적으로 함수이므로, 확장 프로퍼티를 만들 수 있다.

```kotlin

val Context.preferences: SharedPreferences
    get() = PreferenceManager
        .getDefaultSharedPreferences(this)

val Context.inflater: LayoutInflater
    get() = getSystemService(
        Context.LAYOUT_INFLATER_SERVICE
    ) as LayoutInflater

val Context.notificationManager: NotificationManager
    get() = getSystemService(Context.NOTIFICATION_SERVICE)
            as NotificationManager

```

위에서 확인할 수 있는 것 처럼 프로퍼티는 필드가 아니라 접근자를 나타낸다.프로퍼티를 함수 대신 사용할 수도 있지만, 그렇다고 완전히 대체해 사용하는 것은 좋지 않다.

```kotlin

// 프로퍼티를 함수처럼 사용할 수 있지만 완전히 대체하지 마라
val Tree<Int>.sum: Int
    get() = when (this) {
        is Leaf -> value
        is Node -> left.sum + right.sum
    }

// 함수로 구현해라
fun Tree<Int>.sum(): Int = when (this) {
    is Leaf -> value
    is Node -> left.sum() + right.sum()
}

```

**프로퍼티에 알고리즘 동작이 들어가는 것은 좋지 않다.프로퍼티 대신 함수를 사용하는 것이 좋은 경우**

-연산 비용이 높거나 복잡도가 O(1) 보다 큰 경우
-비즈니스 로직을 포함하는 경우
-결정적이지 않은 경우
-변환의 경우
-게터에서 프로퍼티의 상태 변경이 일어나야 하는 경우
-상태를 추출 / 설정할 때는 프로퍼티를 사용해야 한다 . 특별한 이유가 없다면 함수를 사용하면 안된다.

```kotlin

// 부적절
class UserIncorrect {
    private var name: String = ""

    fun getName() = name

    fun setName(name: String) {
        this.name = name
    }

}

// 적절
class UserCorrect {
    var name: String = ""
}

```

프로퍼티는 상태를 나타내고, 함수는 행동을 나타낸다.

<br >
<hr >
<br >

### 아이템 17: 이름 있는 아규먼트를 사용하라

        코드에서 아규먼트의 의미가 명확하지 않은 경우가 있 예를 살펴보자.

```kotlin

// 무슨 의미인지 알기 어렵다.
val text = (1..10).joinToString("|")

// 변수를 사용해 의미를 명확하게 하는 법
val separator = "|"
val text = (1..10).joinToString(separator = separator)

```

-이름 있는 아규먼트는 언제 사용해야 할까 ?
-이름 있는 아규먼트를 사용하면 코드가 길어지지만, 두가지 장점을 얻을 수 있다.이름을 기반으로 값이 무엇을 의미하는지
파라미터 입력 순서와 상관없이 안전

```kotlin

sleep(100) // 100ms 인지 100s 인지 명확하지 않음

sleep(timeMillis = 100) // sleep(Millis(100)), sleep(100.ms)

```

타입은 이러한 정보를 전달하는 굉장히 좋은 방법이다.만약 성능에 영향을 줄 것 같아 걱정이 된다면, 추 후 아이템 46 에 나오는 인라인 클래스를 사용하면 된다.이름 있는 아규먼트를 사용하면 파라미터의 순서를 잘못
입력하는 등의 문제가 발생하지 않는다 . 이런 경우 더욱 사용하자.

**디폴트 아규먼트**
같은 타입의 파라미터가 많은 경우
함수 타입의 파라미터가 있는 경우(마지막 경우 제외)

**디폴트 아규먼트**
프로퍼티가 디폴트 아규먼트를 가질 경우, 항상 이름을 붙여 사용하는 것이 좋다 .

**같은 타입의 파라미터가 많은 경우**

- 파라미터가 모두 다른 타입이라면, 위치를 잘못 입력하면 오류가 발생해 쉽게 문제를 발견할 수 있다 .
- 하지만, 같은 타입이라면 잘못 입력할 경우 오류를 찾기 어렵다 . 이름 있는 아규먼트를 사용하자.

```kotlin

// 아래와 같은 함수가 있다면 이름 있는 아규먼트를 사용하자
fun sendEmail(to: String, message: String) { /*...*/
}

sendEmail(to = "contact@kt.academy", message = "Hello,...")

```

**함수 타입 파라미터**
함수 타입 파라미터는 조금 특별하게 다뤄야한다 . 일반적으로 함수타입 파라미터는 마지막 위치에 배치하고 이름을 붙여 사용하면 훨씬 이해하기 쉽다.

```kotlin

val view = linerLayout {
    text("Click below")
    button(onClick = {/*...*/ }) {
/*...*/
    }
}

```

자바에선 람다 표현식과 주석을 활용했다면, 코틀린에서는 이름 있는 아규먼트를 사용해 의미를 명확하게 할 수 있다 .

```kotlin

observable.getUser()
    .subscribeBy(
        onNext = { user: List<User> -> /*...*/ },
        onError = { throwable: Thorwable -> /*...*/ },
        onCompleted = { /*...*/ }
    )

```

**정리**
이름 있는 아규먼트의 장점

- 디폴트 값 생략
- 코드 가독성, 안정성 향상
- 이름있는 아규먼트 활용의 예외는 마지막 파라미터가 DSL처럼 특별한 의미를 가질 경우이다.

<br>
<hr>
<br>

### 아이템 18: 코딩 컨벤션 지켜라

코틀린은 굉장히 잘 정리된 코딩 컨벤션을 가지고 있다. 어떤 개발 언어든 코딩 컨벤션은 최대한 지켜주는 것이 좋다. 코딩 컨벤션을 지켜야하는 이유.

- 어떤 프로젝트를 접해도 쉽게 이해할 수 있따.
- 다른 개발자도 코드의 작동 방식을 쉽게 추측할 수 있다.
- 코드를 병합하고, 한 프로젝트의 코드 일부를 다른 코드로 이동하는 것이 쉽다.
- 많은 파라미터를 갖고 잇는 클래스나 함수는 한 줄 씩 작성한다.

```kotlin
class Person(
    val id: Int = 0,
    val name: String = "",
    val surname: String = "",
) : Human(id, name) { /*...*/ }
```

위 코드와 아래 코드는 완전히 다르다 .

```kotlin

class Person(
    val id: Int = 0,
    val name: String = "",
    val surname: String
) : Human(id, name) {
// 본문               
}

```

**이 코드는 두 가지 측면에서 문제가 된다.**

- 모든 클래스의 아규먼트가 클래스 이름에 따라 다른 크기의 들여쓰기를 갖는다.
    - 이런 형태로 작성하면, 클래스 이름을 변경할 때 모든 기본 생성자 파라미터의 들여쓰기를 조정
- 클래스가 차지하는 공간의 너비가 너무 크다.

**코딩 컨벤션을 지키는 데 도움이 되는 것**

- IntelliJ 포매터
- Setting ➡ Editor ➡ Code Style ➡ Kotlin 클릭
- 우측 상단 Set from... ➡ 원하는 스타일 가이드를 지정
- Style issues File ➡ [File is not formatted according to project settings] 체크
- 정적 분석 툴이나 IntelliJ 플러그인
- ktlint - 공식 가이드에 기반한 코드 스타일과 컨벤션을 검사한다.
- detekt - 다양한 옵션을 제공하며 컨벤션과 함께 코드 품질을 검사한다.

**질문**

- 동료 개발자가, 코딩 컨벤션을 지키지 않은 경우, 어떻게 대처하시나?
- 코딩 컨벤션을 맞추다보니, 기존 코드의 변경점이 나로 변하는 경우 어떻게 하나?

<br>
<hr>
<br>
