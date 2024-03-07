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



### 아이템 25: 공통 모듈을 추출해서 여러 플랫폼에서 재사용하라



### 아이템 26: 함수 내부의 추상화 레벨을 통일하라
