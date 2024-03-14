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

### 질문
- 시맨틱 버저닝.. 정말 잘 지키고 있는가?

---

<br>
<br>

### 아이템 29: 외부 API를 랩(wrap)해서 사용하라

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

- 클래스를 이해하는데 좋다.
- 유지 보수하기 쉽다.
- 보이는 요소가 적은 경우 유지보수하고 테스트할 것이 적어진다.
- 변경이 일어날 경우 기존껄 수정하는것 보다 새로 추가하는게 쉽다.
- 기존에 제공하던 API를 축소하면 여러 사용자의 분노를 느끼기 쉬우니 처음부터 작은 인터페이스 유지가 좋다.
- 접근 제어자를 잘 활용해야 외부에서의 임의 수정을 최소화 할 수 있다.

```kotlin
// 외부에서 set 접근 가능한 경우
var elementsAdded: Int = 0

// 외부에서 set 접근하지 못하도록 만든 경우
var elementsAdded: Int = 0
  private set
```

#### 결론

- 접근자의 가시성을 제한해서 모든 프로퍼티를 캡슐화하는것이 좋다.
- 가시성 제한될수록 클래스의 변경을 쉽게 추적 가능
- 프로퍼티의 상태를 더 쉽게 이해할 수 있다.
- 동시성(concurrency)을 처리할 때 중요

## 가시성 한정자 사용하기

- public
- private
- protected
- internal

---

<br>
<br>

### 아이템 31: 문서로 규약을 정의하라

## 주석을 써야할까?

- 현재는 주석 없이도 읽을 수 있는 코드를 작성해야 하는 프로그래밍이 일반적
- 로버트 마틴 - 클린 코드 내용 `극단적인 것은 언제나 좋지 않습니다.`
- 주석을 통한 내용의 규약을 설명은 필요

## KDoc 형식

- KDoc은 `/**`로 시작해 `*/`으로 끝난다.
- 첫 번째 부분은 요소에 대한 요약 설명
- 두 번째 부분은 상세 설명
- 이어지는 줄은 태그와의 조합

### 주요 태그

- @param <name> 함수 파라미터 또는 클래스, 프로퍼티, 함수 타입 파라미터를 문서화
- @return 함수의 리턴 값을 문서화
- @throws 메서드 내부에서 발생할 수 있는 예외에 대한 문서화
- @see 특정한 클래스 또는 메서드에 대한 링크 추가. 문서 내용 중간에서 링크가 필요하다면 `[]`로 대체 가능

---

### 아이템 32: 추상화 규약을 지켜라

- 프로그램을 안정적으로 유지하고 싶다면, 규약을 지켜라!
- 규약을 깰 수 밖에 없다면, 이를 잘 문서화하자.

---
