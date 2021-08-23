#  클래스와 인터페이스


### item23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라.

* 태그란 무엇일지 부터 확인해보자.
```java
class Figure{
    enum Shape {RECTANGLE, CIRCLE};
    
    //태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;
    
    double length;
    double width;
    
    double radius;
}
```

위의 코드의 문제점은 무엇일까? 바로 쓸데없는 코드들이 많다는 것이다.

열거 필드와 태그 필드, 그리고 length와 width는 사각형에서만 사용되고 radius는 원에서만 사용 되므로 선언 되는 모양에 따라 사용되는 필드가 나뉜다.



* 추가적인 로직을 구현하게 되면 결국 쓸데 없는 코드들이 늘어나기만 할 것


* 이보다는 계층구조를 활용해야 한다.


```java
class Figure{
    abstract double area();
}

class Circle extends Figure {}

class Rectangle extends Figure {}
```

* 위와 같은 방식으로 선언하면 계층 구조를 활용한 것이고 서로 쓸데 없는 코드들이 생성되지 않도록 분류가 가능하다.

> 태그 달린 클래스를 써야 하는 상황은 거의 없다. 새로운 클래스를 작성하는 데 태그 필드가 등장한다면 태그를 없애고 계층구조로 대체하는 방법을 생각해보자. 기존 클래스가 태그 필드를 사용하고 있다면 계층구조로 리팩터링하는 걸 고민해보자.