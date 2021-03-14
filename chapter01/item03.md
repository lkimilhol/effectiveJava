# 객체 생성과 파괴


### item03. private 생성자나 열거 타입으로 싱글턴임을 보증하라

싱글턴이란 무엇인지 먼저 알아보자.

> 인스턴스 하나만 생성 하도록 하는 클래스. 생성된 인스턴스를 어디에서든지 참조할 수 있도록 하는 패턴. 인스턴스를 계속해서 재생성 할 경우 자원낭가 될 수 있으므로, 인스턴스를 한번 생성하여 계속해서 재사용 하게 한다. 또한 이 인스턴스는 전역변수 처럼 어디서든 사용 할 수 있는 장점이 있다.

> 객체와 인스턴스의 차이는 무엇일까?
> 객체(Object)는 소프트웨어 세계에 구현할 대상이고, 이를 구현하기 위한 설계도가 클래스(Class)이며, 이 설계도에 따라 소프트웨어 세계에 구현된 실체가 인스턴스(Instance)이다.

싱글턴을 생성하는 방법은 여러가지가 있지만, 책에 나온 내용을 토대로 구현해보도록 하자.

아래는 public static final 필드 방식의 싱글턴이다.

1. 
```
class Test {
    public static final Test instance = new Test();

    private Test() {}
}
```

이처럼 instance 1번만 생성하여 getInstance 메서드를 통해, 생성된 인스턴스를 리턴 받는 형태이다.

우리는 다음 코드를 이용해 Test클래스의 인스턴스에 접근한다.

```
Test.instance
```

위 Test 클래스의 장점은 해당 클래스가 싱글턴임이 명백히 드러난다는 점, 그리고 간결함이다.


이제 이 인스턴스를 정적 팩터리 메소드를 통해 get 할 수 있도록 한다.

2. 
```
class Test {
    private static final Test instance = new Test();

    private Test() {}
    
    public static Test getInstance () { return instance;}
}
```

정적 팩터리 방식의 장점은 (getInstance 메소드를 만든 것) 언제든지 싱글턴이 아니게 변경이 가능하다는 점이다.

또한 정적 팩터리 메서드를 제네릭 싱글턴 팩터리로 만들 수 있으며, 정적 팩터리의 메소드 참조를 공급자로 사용이 가능하다는 점이다.

```
Supplier<Test> testSupplier = () -> Test.getInstance();
Test t = testSupplier.get(); //어떤 점이 큰 장점이지는 와닿지 않는다.
```

장점들이 굳이 필요하지 않다면 public 필드 방식이 좋다.

둘 중 하나의 방식으로 만든 싱글턴 클래스를 직렬화하려면 단순히 Serializable을 구현한다고 선건하는 것으로는 부족하다.

모든 인스턴스 필드를 일시적이라고 선언하고 readResolve 메서드를 제공해야 한다. 이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화 할 때 새로운 인스턴스가 만들어진다.

>기본적인 자바 직렬화 또는 역직렬화 과정에서 별도의 처리가 필요할 때는 writeObject와 readObject 메서드를 클래스 내부에 선언해주면 된다. 물론 해당 클래스는 Serializable 인터페이스를 구현한 직렬화 대상 클래스여야 한다. 직렬화 과정에서는 writeObject가 역직렬화 과정에서는 readObject 메서드가 자동으로 호출된다.

```
private Object readResolve() {
    return instance;    //진짜 Test를 반환하고, 가짜 Test는 가비지 컬렉터에 맡긴다.
}
```

3. 열거 타입 방식의 싱글턴
```
public enum Test{
    INSTANCE;
}
```

public 필드와 비슷하지만 더 간결하고 추가 노력 없이 직렬화할 수 있다. 또한 Enum 자체가 스레드 세이프하도록 되어있다. 

그리고 복잡한 직렬화 상황이나, 리플렉션의 경우에도 제2의 인스턴스가 생기는 일을 방지한다.

이펙티브 자바에서는 열거 타입 방식의 싱글턴이 가장 좋은 방법이라고 소개하고 있다.

```
public class Main{
    public static void main(String [] args) {
        EnumSingleton enumSingleton1 = EnumSingleton.INSTANCE.getInstance();
        enumSingleton1.test();
    }
}

enum EnumSingleton {

    INSTANCE("Initial class info");

    private String info;

    private EnumSingleton(String info) {
        this.info = info;
    }

    public EnumSingleton getInstance() {
        return INSTANCE;
    }

    public void test() {
        System.out.println("test");
    }

    // getters and setters
}
```
 