# Week 5

<hr>

## 목차

- 아이템 19: knowledge를 반복하지 말라
- 아이템 20: 일반적인 알고리즘을 반복해서 구현하지 말라
- 아이템 21: 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라
- 아이템 22: 일반적인 알고리즘을 구현할 때 제네릭을 사용하라

<br>
<hr>
<br>

### 아이템 19: knowledge를 반복하지 말라

> 프로젝트에서 이미 있던 코드를 복사해서 붙여넣고 있다면, 무언가가 잘못된 것이다

**knowldege**
- knowledge는 의도적인 정보를 나타내는 개념을 뜻하며 코드 혹은 데이터로 표현할 수 있다.
- 프로젝트를 진행할 때 정의한 모든 것이 knowledge이며 알고리즘의 작동 방식, UI의 형태, 원하는 결과 등 모든 의도적인 정보를 말한다.
- 코드, 설정, 템플릿 등으로 표현할 수 있다. 프로그램에서 중요한 knowledge를 크게 두 가지 뽑는다면, 다음과 같다.
  - 로직: 프로그램이 어떠한 식으로 동작하는지와 프로그램이 어떻게 보이는지
  - 공통 알고리즘: 원하는 동작을 하기 위한 알고리즘 둘의 큰 차이점은 시간에 따른 변화이다. 비즈니스 로직은 시간이 지나며 계속 변하지만, 공통 알고리즘은 크게 변하지 않는다.

**모든 것은 변화한다.**

> 프로그래밍에서 유일하게 유지되는 것은 변화한다는 속성이라는 말이 있다. 기술, 언어도 빠른 속도로 변화하고 프로젝트의 라이브러리, 아키텍처, 설계가 꽤 많이 변화한다. 이처럼 우리 프로젝트의 knowledge도 계속해서 변화한다.

- 회사가 사용자의 요구 또는 습관을 더 많이 알게 되었다.
- 디자인 표준이 변화했다.
- 플랫폼, 라이브러리, 도구 등이 변화해서 이에 대응해야 한다.

**모든 것은 지속해서 변화하고 이에 대비해, 적은 knowledge의 반복을 가져야한다. knowledge 반복은 프로젝트의 확장성을 막고, 쉽게 깨지게 만든다.**

**언제 코드를 반복해도 될까?**
> 추출을 통해 knowledge를 줄이면 안되는 상황이 존재한다. 얼핏보면 knowledge 반복처럼 보이지만, 실질적으로 다른 knowledge를 나타내므로 추출하면 안되는 부분은 반복을 줄이면 안된다. 예를 들면.. 어느 한 프로젝트에서 독립저거인 2개의 안드로이드 어플리케이션을 만들 고 있을 때, 빌드 도구 설정이 비슷할 것이므로, 이를 추출해 knowledge 반복을 줄일 수 있다고 생각할 수 있다. 하지만 > 두 어플리케이션은 독립적으로 구성 변경이 일부 필요할 수 있다. 이는 문제가 된다.. 위와 같이 신중하지 못한 추출은 변경을 더 어렵게 만든다. 또한 구성을 읽을 때도 더 어려울 수 있다. 두 코드가 같은 knowledge를 나타내는지, 다른 knowledge를 나타내는지는 아래 두 질문으로 알 수 있다.

1. 함께 변경될 가능성이 높은가
2. 따로 변경될 가능성이 높은가

**잘못된 코드 추출로 우리를 보호할 수 있는 규칙도 있다. 바로 단일 책임 원칙(Single Responsibility Principle, SRP) 이다.**

**단일 책임 원칙 (SOLID - 객체지향 설계 5원칙)**

> SRP(Single Responsibility Principle) : 단일 책임 원칙

**예시**
어떤 대학에서 Student라는 클래스를 갖는다. 이 클래스는 장학금과 관련된 부서와 인증과 관련된 부서에서 모두 사용된다. 두 부서에서는 Student라는 클래스에 다음과 같은 두 가지 프로퍼티를 추가했다.

- qualifiesForScholarship : 장학금 관련 부서에서 만든 프로퍼티로, 학생이 장학금을 받을 수 있는 포인트를 갖고 있는지 나타냄
- isPassinig : 인증 관련 부서에서 만든 프로퍼티로, 학생이 인증을 통과했는지를 나타냄

이 두 프로퍼티는 모두 학생의 이전 학기 성적을 기반으로 계산됨. 그래서 개발자는 두 프로퍼티를 한꺼번에 계산하는 calculatePointsFromPassedCourses 함수를 만들었다.

```kotlin
class Student {
	// ...
	fun isPassing(): Boolean =
			calculatePointsFromPassedCourses() > 15

	fun qualifiesForScholarship(): Boolean =
			calculatePointsFromPassedCourses() > 30

	private fun calculatePointsFromPassedCourses(): Int {
			...
	}
}
```

그런데 어느날 학부장이 "덜 중요한 과목은 장학금 포인트를 줄여달라" 요청해, 규칙을 바꿔야 하는 상황이 생겼다. 이것을 변경하기 위해 파견된 개발자가 qualifiesForScholarship() 프로퍼티를 확인하고 calculatePointsFromPassedCourses()에서 값을 수정하는 것을 확인해 코드를 수정했다. 그런데 의도치 않게 isPassing()도 비슷한 프로퍼티라 생각해, 이와 관련된 동작을 수정해 문제가 발생할 것이다.

일반적으로 private 함수는 두 가지 이상의 역할을 하지 않기 때문에, 이런 관습에 따라 생각했기 때문이다.

책임에 따라 StudentIsPassingValidator와 StudentQualifiesForScholarshipValidator 클래스를 구분해 만들었다면 혹은 코틀린의 확장 함수를 활용하면, 두 함수는 Student 클래스 아래에 두면서, 각각의 부석 관리하는 서로 다른 모듈 파일에 배치할 수도 있을 것이다.

```kotlin
// accreditations 모듈
fun Student.qualifiesForScholarship(): Boolean {
	...
}

// scholarship 모듈
fun Student.calculatePointsFromPassedCourses(): Boolean {
	...
}
```

- 헬퍼 함수는 private으로 만들지 않고, 다음과 같이 만드는 것이 일반적이다.
- 두 부서에서 모두 사용하는 일반적인 public 함수로 헬퍼 함수를 만든다. 공통 부분은 두 부서에서 모두 사용하므로, 이를 함부로 수정해서는 안되게 규약을 정한다.
- 헬퍼 함수를 각각의 부서 모듈에 따라 2개 만든다.

**단일 책임 원칙은 우리에게 두 사실을 알려준다.**

- 서로 다른 곳에서 사용하는 knowledge는 독립적으로 변경할 가능성이 많다. 따라서 비슷한 처리를 하더라도, 완전히 다른 knowledge로 취급하는 것이 좋다.
- 다른 knowledge는 분리해 두는 것이 좋다. 그렇지 않으면, 재사용해서는 안되는 부분을 재사용하려는 유혹이 발생한다.

**정리**
1. 모든 것은 변화한다. 따라서 공통 knowledge가 있따면, 이를 추출해 이런 변화에 대비해야 한다.
2. 여러 요소에 비슷한 부분이 있는 경우, 변경이 필요할 때 실수가 발생할 수 있지만 이런 부분은 추출하는 것이 좋다.
3. 의도하지 않은 수정을 피하거나 다른 곳에서 조작하는 부분이 있따면, 분리해 사용하는 것이 좋다.
4. 비슷해 보이는 코드 모두 추출하는 극단적인 경향은 좋지 않다. 항상 균형이 중요하다.

**나의 질문**
> 하위 버전은 그대로 남기며, 변경된 상위 버전 API를 만드는 작업이 있다. V1 -> V2 api를 만들 때, 원래 코드를 복사해서 작업을 하는지, 혹은 다른 방법을 사용하는지?

<br>
<hr>
<br>

### 아이템 20: 일반적인 알고리즘을 반복해서 구현하지 말라

- 많은 개발자는 같은 알고리즘을 여러번 반복해 구현한다. 예로 복잡하거나 간단한 알고리즘이 있을 수 있다.

```kotlin
val percent = when {
  numberFromUser > 100 -> 100
  numberFromUser < 0 -> 0
  else -> numberFromUser
}
```

**이 알고리즘은 사실 stdlib의 coerceIn 확장 함수로 이미 존재한다. 따로 구현하지 않아도 된다.**

`val percent = numberFromUser.coerceIn(0, 100)` 이렇게 이미 있는 것을 활용하면, 코드가 짧아지는 것 외에 아래 장점이 있다.

- 알고리즘을 만들지 않아도 되기 때문에. 코드 작성 속도가 빨라진다.
- 구현을 따로 읽지 않아도, 무엇을 하는지 확실하게 알 수 있다.
- 직접 구현할 때 발생할 수 있는 실수를 줄일 수 있다.
- 제작자들이 한 번만 최적화하면, 이런 함수를 활용하는 모든 곳이 최적화의 혜택을 받을 수 있다.

**표준 라이브러리 살펴보기**

일반적인 알고리즘은 대부분 이미 다른 사람들이 정의해 놓았다. 대표적으로 stdlib가 있다.

```kotlin
override fun saveCallResult(item: SourceResponse) {
	var sourceList = ArrayList<SourceEntity>()
	item.sources.forEach {
		var sourceEntity = SourceEntity()
		sourceEntity.id = it.id
		sourceEntity.category = it.category
		sourceEntity.country = it.country
		sourceEntity.description = it.description
		sourceList.add(sourceEntity)
	}
	db.insertSources(sourcesList)
}
```

- 앞의 코드에서 forEach를 사용하는 것은 사실 좋지 않다. 
- 이런 코드는 for을 사용하는 것과 아무 차이가 없다. sourceEntity 를 처리하는 과정이 어설프다. 
- 이는 코틀린에서 작성된 코드는 더이상 볼 수 없는 자바빈(JavaBean) 패턴이기 때문이다. 이런 형태보다 팩토리 메소드를 활용하거나, 기본 생성자를 활용하는 것이 좋다. 
- 그래도 위와 같은 패턴을 써야겠다면 다음과 같이 apply를 활용해 모든 단일 객체들의 프로퍼티를 암묵적으로 설정하는 것이 좋다.

```kotlin
override fun saveCallResult(item: SourceResponse) {
	val sourceEntries = item.sources.map(::sourceToEntry)
	db.insertSources(sourceEntries)
}

private fun sourceTnEntry(source: Source) = SourceEntity() {
	.apply {
		id = source.id
		category = source.category
		country = source.country
		description = source.description
	}
}
```

**나만의 유틸리티 구현하기**
상황에 따라 표준 라이브러리에 없는 알고리즘이 필요할 수도 있다. 예를 들어 컬렉션에 있는 모든 숫자의 곱을 계산하는 라이브러리가 필요하다면 유틸리티 함수로 정의하는 것이 좋다.

```kotlin
fun Iterable<Int>.product() = 
    fold(1) { acc, i -> acc * i }
```
**여러 번 사용되지 않아도 이렇게 따로 정의하는 것이 좋다.**

- 동일한 결과를 얻는 함수를 여러 번 만드는 것은 잘못된 일이다.
- 모든 함수는 테스트되어야 하고, 기억되어야 하며, 유지보수되어야 한다.
- 따라서 함수를 만들 때는 이러한 비용이 들어갈 수 있다는 것을 전제해야 한다.

**많이 사용되는 알고리즘을 추출하는 방법으로는 톱레벨 함수, 프로퍼티 위임, 클래스 등이 있다. 확장 함수는 이런 방법들과 비교해서, 다음과 같은 장점을 갖고 있다.**

- 함수는 상태를 유지하지 않으므로, 행위를 나타내기 좋다. 특히 side-effect(상태 변경)가 없는 경우에 더 좋다.
- 톱레벨 함수와 비교해, 확장 함수는 구체적인 타입이 있는 객체에만 사용을 제한할 수 있어 좋다.
- 수정할 객체를 아규먼트로 전달받아 사용하는 것보다 확장 리시버로 사용하는 것이 가독성 측면에서 좋다.
- 확장 함수는 객체에 정의한 함수보다 객체를 사용할 때, 자동 완성 기능 등으로 제안이 이루어지므로 쉽게 찾을 수 있다.

**정리**
- 일반적인 알고리즘을 반복해 만들지 말자. 대부분 stdlib에 정의되어 있다.
- stdlib에 없는 일반적인 알고리즘이 필요하거나, 특정 알고리즘을 반복해 사용하는 경우 프로젝트 내부에 직접 정의해라
- 일반적으로 이런 알고리즘들은 확장 함수로 정의하는 것이 좋다.

**나의 생각**
- 유틸성 로직을 작성하고자, 확장함수를 만들었지만...이미 표준 라이브러리에 모두 존재했던 경험이 다수 존재한다.
- Kotlin Collections에 아주 유용한 함수들이 존재한다..

<br>
<hr>
<br>

### 아이템 21: 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라

> 코틀린은 코드 재사용과 관련해 `프로퍼티 위임`이라는 새로운 기능을 제공한다.
> 프로퍼티 위임을 사용하면 일반적인 프로퍼티의 행위를 추출해 재 사용할 수 있다.

**대표적인 예로 지연 프로퍼티(lazy)가 있다.**
> lazy 프로퍼티는 처음 사용하는 요청이 들어올 때 초기화되는 프로퍼티이다.

```kotlin
// lazy
val value by lazy { createValue() }
```

```kotlin
// observable 패턴
var items: List<Item> by
	Delegates.observable(listOf()) { _, _, _ -> 
		notifyDataSetChanged()
}

var key: String? by
	Delegates.observable(null) { _. old, new ->
		Log.e("key changed from $old to $new")
}
```

- 프로퍼티 위임을 활용하면 다양한 패턴들(뷰, 리소스 바인딩, 의존성 주입, 데이터 바인딩 등)을 만들 수 있다.
- 일반적으로 이런 패턴들을 사용할 때 자바 등에서는 어노테이션을 많이 활용해야 하지만, kotlin은 프로퍼티 위임을 사용해 간단하고 type-safe하게 구현할 수 있따.

```kotlin
// 안드로이드에서의 뷰와 리소스 바인딩
private val button: Button by bindView(R.id.button)
private val textSize by bindDimension(R.dimen.font_size)
private val doctor: Doctor by argExtra(DOCTOR_ARG)

// kotlin 에서의 종속성 주입
private val presenter: MainPresenter by inject()
private val repository: NetworkRepository by inject()
private val vm: MainViewModel by viewModel()

// 데이터 바인딩
private val port by bindConfiguration("port")
private val token: String by preferences.bind(TOKEN_KEY)
```

어떻게 이런 코드가 가능하고, 프로퍼티 위임을 어떻게 활용할 수 있는지 살펴볼 수 있게, 간단한 프로퍼티 델리게이트를 만들어 보자.
예를 들어, 일부 프로퍼티가 사용될 때, 간단한 로그를 출력하고 싶다고 가정할 때, 가장 기본적인 방법은 getter, setter에서 로그를 출력하는 방법이다.

```kotlin
var token: String? = null
	get() {
		print("token returned value $field")
		return field
	}
	set(value) {
		print("token changed from $field to $value")
		field = value
	}

var attempts: Int = 0
	get() {
		print("attempts returned value $field")
		return field
	}
	set(value) {
		print("attempts changed from $field to $value")
		field = value
	}
```

두 프로퍼티는 타입이 다르지만, 내부적으로 거의 같은 처리를 한다. 또한 프로젝트에서 자주 반복될 것 같은 패턴처럼 보인다.
`프로퍼티 위임은 다른 객체의 메서드를 활용해 프로퍼티의 접근자(getter, setter)를 만드는 방식`
**게터는 getValue, 세터는 setValue함수를 사용해 만들어야하며 객체를 만든 뒤에는 by 키워드를 사용해 연결해주면 된다.**

```kotlin
var token: String? by LoggingProperty(null)
var attempts: Int by LoggingProperty(0)

private class LoggingProperty<T>(var value: T) {
	operator fun getValue(
		thisRef: Any?,
		prop: KProperty<*>
	): T {
		print("${prop.name} returned value $value")
		return value
	}

	operator fun setValue(
		thisRef: Any?,
		prop: KPRoperty<*>,
		newValue: T
	) {
		val name = prop.name
		print("name changed from $value to $newValue")
		value = newValue
	}
}
```

위코드는다음과 비슷한 형태로 컴파일된다.(프로퍼티가 top 레벨에서 사용될 때는 this 대신 null로 바뀐다)

```kotlin
@JvmField
private val 'token$delegate' = LoggingProperty<String>(null)
var token: String?
	get() = 'token$delegate'.getValue(this, ::token)
	set(value) {
		'token$delegate'.setValue(this, ::token, value)
	}
```

- getValue와 setValue는 단순하게 값만 처리하게 바뀌는 것이 아니라, 컨텍스트(this)와 프로퍼티 레퍼런스의 경계도 함께 사용하는 형태로 바뀐다.
- 프로퍼티에 대한 레퍼런스는 이름, 어노테이션과 관련된 정보 등을 얻을 때 사용된다. 그리고 컨텍스트는 함수가 어떤 위치에서 사용되는지와 관련된 정보를 제공해 준다.
- 이런 정보로 인해 getValue와 setValue가 여러 개 있어도 문제가 없다. 컨텍스트를 활용해 적절한 메서드가 선택되기 때문이다.

예를 들어여러종류의뷰와 함께 사용할 수 있는 델리게이트가필요한 경우를생각해보자. 이는 다음과 같이 구현해, 컨텍스트의 종류에 따라 적절한 메서드가 선택되게 만들 수 있다.

```kotlin
class SwipeRefreshBinderDelegate(val id: Int) {
	private var cache: SwipeRefreshLayout? = null

	operator fun getValue(
		activity: Activity,
		prop: KProperty<*>,
	): SwipeRefreshLayout {
	return cache?: activity
		.findViewById<SwipeRefreshLayout>(id)
		.also { cache = it }
	}

	operator fun getValue(
		fragment: Fragment,
		prop: KProperty<*>
	): SwipeRefreshLayout {
		return cache?: fragment.view
		.findViewById<SwipeRefreshLayout>(id)
		.also { cache = it }
	}
}
```

- 객체를 프로퍼티 위임하려면 val의 경우 getValue, var의 경우 getValue와 setValue 연산이 필요하다.
- 이런 연산은 멤버 함수로도 만들 수 있지만, 확장 함수로도 만들 수 있다.
- 예를 들어 다음 코드는 Map<String, *> 를 사용하는 예다.

```kotlin
val map: Map<String, Any> = mapOf(
    "name" to "Marcin",
    "kotlinProgrammer" to true
)
val name by map
print(name) // Marcin
```

이는 코틀린 stdlib에 다음과 같은 확장 함수가 정의되어 있어 사용할 수 있다.

```kotlin
inline operator fun <V, V1 : V> Map<in String, V>
.getValue(thisRef: Any?, property: KProperty<*>): V1 =
getOrImplicitDefault(property.name) as V1
```

코틀린 stdlib에서 다음과 같은 프로퍼티 델리게이터를 알아두면 좋다.

- lazy
- Delegates.observable
- Delegates.vetoable
- Delegates.notNull

## 정리

- 프로퍼티 델리게이트는 프로퍼티와 관련된 다양한 조작을 할 수 있으며, 컨텍스트와 관련된 대부분의 정보를 갖는다.
- 이런 특징으로 인해 다양한 프로퍼티의 동작을 추출해 재사용할 수 있다.
- 프로퍼티 위임은 프로퍼티 패턴을 추출하는 일반적인 방법이라 많이 사용된다.

<br>
<hr>
<br>

### 아이템 22: 일반적인 알고리즘을 구현할 때 제네릭을 사용하라


<br>
<hr>
<br>
