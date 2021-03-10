# 객체 생성과 파괴


### item02. 생성자에 매개변수가 많다면 빌더를 고려하라

***


일반 메서드를 만들때도 그렇지만 많은 매개변수는 좋은 메서드를 만들지 못하다는 의미와 비슷하다.

매개변수가 많아진다면 소스코드의 가독성이 나빠지며 코드 작성에도 어려워진다.

```
public class PersonInfo {
    private String name;
    private Integer age;
    private String favoriteColor;
    private String favoriteAnimal;
    private Integer favoriteNumber;

    public PersonInfo(String name, Integer age, String favoriteColor, String favoriteAnimal, Integer favoriteNumber){
        this.name = name;
        this.age = age;
        this.favoriteColor = favoriteColor;
        this.favoriteAnimal = favoriteAnimal;
        this.favoriteNumber = favoriteNumber;
    }
}
```

인스턴스를 생성 할 때 생성자를 통해 생성하는 예제이다. 우리는 생성자에 많은 매개변수들을 넣어 줘야 하며, 실제로 사용되지 않을 값에는 null을 넣어 줘야 할 것이다.

```
new PersonInfo("JDM", 20, null, null, null);
```
이 문제를 해결하기 위해 점증적 생성자 패턴(telescoping constructor pattern)을 사용해보도록 하자.

### 점증적 생성자 패턴

```
public class PersonInfo {
    private String name;
    private Integer age;
    private String favoriteColor;
    private String favoriteAnimal;
    private Integer favoriteNumber;

    public PersonInfo(String name, Integer age, String favoriteColor, String favoriteAnimal, Integer favoriteNumber){
        this.name = name;
        this.age = age;
        this.favoriteColor = favoriteColor;
        this.favoriteAnimal = favoriteAnimal;
        this.favoriteNumber = favoriteNumber;
    }
    
    //추가
    public PersonInfo(String name, Integer age) {
        this.name = name;
        this.age = age;
        this.favoriteColor = null;
        this.favoriteAnimal = null;
        this.favoriteNumber = null;
    }
    
    public PersonInfo(String name, Integer age, String favoriteColor) {
        this.name = name;
        this.age = age;
        this.favoriteColor = favoriteColor;
        this.favoriteAnimal = null;
        this.favoriteNumber = null;
    }
    
    // ...
}
```

생성자의 매개변수를 각기 다르게 작성하여 매개변수로 넘겨받지 않는 값은 null을 선언해 주었다. 이런식으로 매개변수로 우리가 알고 있는 값들은 셋팅을 해주고 그렇지 않으면 기본적으로 null 값을 주는 것이 점증적 생성자 패턴이다.

하지만 클래스의 멤버변수가 늘어난다면 생성자를 또 새로 생성해주어야 하며, 기본값을 설정하는 코드 또한 추가가 된다. 

즉 **확장하기 어렵고, 코드가 지저분해지는** 결과를 낳게 된다.

그렇다면 이 문제는 어떻게 해결이 가능할까?

이번에는 선택 매개변수가 많을 때 활용할 수 있는 자바빈즈 패턴(JavaBeans pattern)을 활용해 보도록 하자.

### 자바빈즈 패턴

매개변수가 없는 생성자로 객체를 만든 후 세터 메서드들을 호출해 원하는 매개변수의 값을 설정하는 방식이다.

```
public class PersonInfo {
    private String name = null;
    private Integer age = null;
    private String favoriteColor = "blue";
    private String favoriteAnimal = "cat";
    private Integer favoriteNumber = "15";

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public void setFavoriteColor(String favoriteColor) {
        this.favoriteColor = favoriteColor;
    }

    public void setFavoriteAnimal(String favoriteAnimal) {
        this.favoriteAnimal = favoriteAnimal;
    }

    public void setFavoriteNumber(Integer favoriteNumber) {
        this.favoriteNumber = favoriteNumber;
    }
}
```

점증적 생성자 패턴의 단점들이 사라졌다. 또한 코드 읽기가 더욱 수월해 졌다.

```
PersonInfo p = new PersonInfo();
p.setName("lkimilhol");
p.setAge(18);
p.setFavoriteColor("red");
```
우리가 알고 있는 정보들은 setter를 통하여 값을 저장할 수 있고, 모르는 경우는 기본으로 null 값이 들어가게 된다.

하지만 자바빈즈 패턴에서는 **객체 하나를 만들려면 메서드를 여러 개 호출해야 하는 단점이 있으며, 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이게 된다**.

일관성이 무너진다는 말이 어떤 의미 일까?

위의 PersonInfo 클래스에서 디폴트로 값이 정해지지 않은 name, age의 경우에 값이 셋팅 되지 않은 경우에 이 객체는 **유효하지 않은 상태**가 된다.
유효하지 않은 상태에서 만약 name 필드에 접근하려고 한다면 exception이 발생 할 것이고, 이런 경우를 **일관성이 무너진다** 라는 표현을 사용한다.

일관성이 깨진 객체가 만들어지면 버그를 심은 코드와 그 버그 때문에 런타임에 문제를 겪는 코드의 디버깅도 어렵다. 또한 **클래스를 불변으로 만들 수 없으며** 스레드 안정성을 얻으려면 추가 작업을 해줘야 한다.



