# [Java] List와 ArrayList

List와 ArrayList의 비교를 통해서 인터페이스와 구현체에 대해서 알아보도록 하겠습니다. 또한 이러한 구조의 문제점은 무엇이고 장점은 어떤 부분이 있는지도 확인해보도록 하겠습니다.

자바에서는 `전체 사용자수`, `오늘 등록한 주문수` 등 복수의 형태의 데이터를 저장하여 활용하기 위해서는 `List` 를 사용합니다. 아래는 샘플 코드입니다.

```java
List<Object> lists = new ArrayList<>();
```

그렇다면, 왜 이렇게 사용할까? 그 내용을 알아보기 위해 정리합니다.



# List

`List` 는 선형 자료구조로 순서를 가진 항목들이 모여 있습니다. 먼저 `List` 는 `Collection` 의 하위 인터페이스입니다.

```java
public interface List<E> extends Collection<E
```



# ArrayList

`ArrayList` 는 배열을 이용해서 `List` 인터페이스를 구현하였으며 `Collection` 프레임워크의 일부입니다.

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```



# 왜 그렇게 해야할까?

처음 내용으로 돌아와서 대부분의 목록 구현을 위해서는 아래와 같이 구현을 합니다.

```java
List<Object> lists = new ArrayList<>();
```

왜 이렇게 주로 선언을 해서 사용할까요? 크게 2가지 이유가 있습니다.

- 변경의 유연성
- Collection 기능 활용



## 변경의 유연성

가장 대표적인 이유는 변경에 유연합니다. 왜냐하면 `List` 는 인터페이스라는 것을 알 수 있습니다. 그래서 구현 클래스(여기서는 `new ArrayList<>()`)를 변경하여도 유연하게 대응할 수 있다는 것입니다. 아래는 `ArrayList`에서 `LinkedList`로 변경이 되었습니다. 인터페이스로 특정 구현을 분리하였기 때문에 컴파일 에러가 발생하지 않습니다.

```java
List<String> lists = new ArrayList<>();

lists = new LinkedList<>();
```

만약 `List<String>` 를 사용하는 것이 아니라 `ArrayList<String>` 를 사용하였다면 아래와 같이 에러를 만날 수 있습니다.

```java
ArrayList<String> arrayLists = new ArrayList<>();
arrayLists = new LinkedList<>();
```

```
ncompatible types: cannot infer type arguments for java.util.LinkedList<>
reason: no instance(s) of type variable(s) E exist so that java.util.LinkedList<E> conforms to java.util.ArrayList<java.lang.String>
```

에러를 해결하기 위해서는 다음과 같이 변경할 수 있습니다.

```java
LinkedList<String> arrayLists = new LinkedList<>();
arrayLists = new LinkedList<>();
```

- 기존 소스의 영향이 생기게 됩니다.

특정 구현 클래스에 종속적인이기 때문에 확장의 유연하지 않고 수정의 범위가 많아지게 됩니다.



## Collection 기능 활용

그 다음 이유는 `ArrayList`의 기능만 사용하지는 않는다는 것입니다. 아래 코드를 보게 되면 `List`는 `Collections`의 기능들을 사용할 수 있습니다.

```java
List<String> listStrings = Collections.EMPTY_LIST;
```



# 다형성

앞에서 `List` 인터페이스를 통해서 보았을때, 한 가지 단어가 생각나실 수 있습니다. 그것은 바로 `다형성` 입니다. 다형성의 뜻은 하나의 객체가 여러 가지 타입을 가질 수 있는 것을 의미합니다. `List` 라는 하나의 인터페이스가 `ÀrrayList`, `LinkedList` , `Vector` 등의 여러 가지 타입을 가질 수 있습니다. 바로 위와 같은 특징을 자바에서는 `다형성`이라고 합니다.

마치, 레고 블럭을 조립하듯이 변경이 가능한 구조입니다. 키보드와 마우스 수명이 다한다면 우리는 어떻게 하나요? 컴퓨터와 상관없이 새로운 키보드와 마우스를 구매해서 사용하면 됩니다. 이러한 장점의 가장 큰 특징이  `유연하고 변경의 용이` 하다는 부분입니다.  이 특징을 활용하여서 실제 우리가 개발하는 애플리케이션을 쉽고 유연하게 변경하면서 쉽게 개발할 수 있습니다. 



# 마무리

- `List` 는 인터페이스이며, `ArrayList` 는 구현체입니다.
  - `Lisr` 인터페이스를 활용하면 `ArrayList` 구현체가 변경되어도 유연하게 대처가 가능합니다.
- `다형성` 의 특징은 `유연하고 변경의 용이` 합니다.