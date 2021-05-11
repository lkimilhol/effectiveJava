# 객체 생성과 파괴


### item09. try-finally보다는 try-with-resources를 사용하라

우리는 대체적으로 try-finally을 사용하여 자원을 제대로 닫힘을 보장했다.

```
try {
    return br.readLine();
} finally {
    br.close()
}
```

하지만 이런 방식은 자원이 둘 이상이라면 너무 지저분하게 된다.

try-finally 문을 제대로 사용한다고 하더라도 예외는 항상 try 블록과 finally 블록에서 발생 할 수 있다.

만약 두번째 예외가 발생하게 된다면 두 번째 예외가 첫 번째 예외를 모두 삼켜버리게 된다. 이렇게 되면 첫 번째 예외의 정보는 남지 않게 된다.

이러한 문제점을 try-with-resources 문을 통하여 해결이 가능하다. 이 구조를 사용하려면 해당 자원이 AutoClosable 인터페이스를 구현해야 한다.

```
try (InputStream in = new FileInputStream(src);
    OutputStream out = new FileOutputStream(dst)) {
    //...
} catch (IOException e) {
    return xx;
}
```

코드가 훨씬 간결해진 것을 볼 수 있다.

> 꼭 회수해야 하는 자원을 다룰 때는 try-finally 말고, try-with-resources를 사용하자. 예외는 없다. 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다. try-finally로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도, try-with-resources로는 정확하고 쉽게 자원을 회수할 수 있다.
