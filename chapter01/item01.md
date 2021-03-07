# 객체 생성과 파괴


### item01. 생성자 대신 정적 팩터리 메서드를 고려하라.

***

클래스는 생성자와 별도로 정적 팩터리 메서드를 제공 할 수 있다.

```
class Test{
    Test() {} //생성자
    
    public static void testMethod() {} // 정적 팩터리 메서드
}
```

### 정적 팩터리 메서드를 사용하는 장점

1. 이름을 가질 수 있다.  

생성자는 클래스와 같은 이름을 가지기 때문에 어떤 기능을 제공하는지 알 수 없다.  
   **생성자인 BigInteger(int, int, Random) 과 BigInteger.probablePrime 중 어느 쪽이 '값이 소수인 BigInteger를 반환한다'는 의미를 더 잘 설명할 것 같은지 생각해보라.**   
   위의 test 클래스 또한 마찬가지이다. 하지만 정적 팩터리 메서드를 사용한다면 기능에 맞게 이름을 지어 줄 수 있다.
      
```
class Test {
   Test() {}

   public static int plusOne(int a) {
      return a + 1;
   }
}
      
public class Main {
   public static void main(String[] args) {
      System.out.println(Test.plusOne(1));
   }
}
```
정적 팩터리 메세드를 활용하여 plusOne 이라는 함수를 만들었고, 이름을 통해 어떤 역할을 하는지 알 수 있다.


2. 호출 될 때마다 인스턴스를 새로 생성하지는 않아도 된다.

static으로 선언 된 정적 메서드는 인스턴스를 새로 생성하지 않아도 호출이 가능하다.  
**Boolean.valueOf(boolean) 메서드는 객체를 아예 생성하지 않는다.**  만약 생성 비용이 큰 객체가 자주 요청되는 상화이라면 성능을 상당히 끌어올려 준다.     

이런 클래스를 인스턴스 통제 클래스라고 한다. 그렇다면 통제 클래스를 사용하는 이유는 무엇일까?  
인스턴스를 통제하면 클래스를 싱글턴으로 만들 수도, 인스턴스화 불가로 만들수도 있으며 불변 값 클래스에서 동치인 인스턴스가 단 하나뿐임을 보장 할 수 있다.     
(a == b 일 때만 a.equals(b)가 성립)

```
System.out.println(Test.plusOne(1));
```

위의 예제에서 Test.plusOne 메소드를 new 하지 않고 호출한 예제이다.

3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

반환할 객체의 클래스를 자유롭게 선택할 수 있게 하는 '엄청난 유연성'을 가지게 되는데, 쉽게 생성자는 리턴값이 없기에 하위 클래스를 리턴 할 수 없지만 정적 팩터리 메서드는 하위 클래스를 리턴해 줄 수 있다.

```
class Test {
    Test() {}
    
    public static Son returnSon() {
        return new Son();
    }
}

class Son extends Test{}

public class Main {
    public static void main(String[] args) {
        Son son = Test.returnSon();
    }
}
```

4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.

우리는 메소드 오버로딩을 활용하여 다른 하위 타입의 클래스를 반환 할 수 있다.

```
class Test {
    Test() {}

    public static TestClass1 testMethod(String code) {
        return new TestClass1(code);
    }

    public static TestClass2 testMethod(String code, String name) {
        return new TestClass2(code, name);
    }
}

class TestClass1 extends Test{
    TestClass1(String code) {
        System.out.println("코드: " + code);
    }
}
class TestClass2 extends Test{
    TestClass2(String code, String name) {
        System.out.println("코드: " + code + " " + "이름: " + name);
    }
}

public class Main {
    public static void main(String[] args) {
        TestClass1 testClass1 = Test.testMethod("A0001");
        TestClass2 testClass2 = Test.testMethod("A0002", "김자바");
    }
}

// 출력
코드: A0001
코드: A0002 이름: 김자바
```

5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

이해하기 어려울 수 있는데, 정적 팩토리 메소드를 구현하고, 이 메서드가 실행 되는 시점에 객체의 클래스가 정해진다는 의미이다.

예를들어보자.

```
class Test {
    public static Test connection() {
        Test t = new Test();
        //이 클래스에서 실행 될 때 FQCN을 읽어온다.
        //FQCN에 해당하는 클래스 인스턴스를 생성한다.
        //t 변수를 해당 인스턴스를 가르키도록 수정한다.
        return t;
    }
}
```
FQCN이란 클래스가 속한 패키지명을 모두 포함한 이름을 말한다.

메서드를 만들고 나서 Test를 상속받은 클래스들 중 하나의 인스턴스를 생성해서 그 인스턴스를 가르키도록 할 수 있다는 의미이다.

책에서 JDBC를 예로 든 것은, getConnection()이라는 메서드를 호출 했을 때 각 드라이버가 다를텐데(오라클, mysql 등등) 그 드라이버를 어떻게 결정할지는 실행중에 결정 한다는 의미이다. 

### 정적 팩터리 메서드를 사용하는 단점

1. 상속을 하려면 public이나 protected생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.

정적 팩터리 메서드 하나만 있다고 할지라도 생성자는 자동으로 컴파일 타임에 구현해주니 무슨 문제라고 생각 할 수 있지만, 여기서 말하는 내용은 생성자가 private로 되어 있을 경우이다.

예시로 Collection 프레임워크에서 제공하는 편의성 구현체(java.util.collections)는 상속 할 수 없다(생성자가 private).

하지만 이 클래스는 상속 할 필요가 없어 되려 이 점이 장점으로 볼 수도 있다. (상속 할 필요가 없으니 상속하지 못하게 막는 것)

2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

생성자는 자바독 상단에 보여주지만 정적 팩터리 메서드는 이를 찾기가 어렵다는 내용.

해당 설명은 설명에 관한 내용이라 개인적으로는 딱히 단점인지 잘 모르겠다.


### 핵심 정리

정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋다.  
그렇다고 하더라도 정적 팩터리를 사용하는 게 유리한 경우가 더 많으므로 무작정 public 생성자를 제공하던 습관이 있다면 고치자.