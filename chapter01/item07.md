# 객체 생성과 파괴


### item07. 다 쓴 객체 참조를 해제하라

자바에서는 가비지 컬렉터가 메모리를 회수하지만, **자칫 메모리 관리에 더 이상 신경쓰지 않아도 된다고 오해할 수 있다.**

메모리 누수를 위한 코드를 작성해보자.

```
class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object object) {
        ensureCapacity();
        elements[size++] = elements;
    }
    
    public Object pop() {
        if (size == 0) {
            throw new RuntimeException();
        }
        return elements[--size];
    }
    
    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

**이 스택에서 커졌다가 줄어들었을 때 스택에서 꺼내진 객체들은 가바지 컬렉터가 회수 하지 않는다.** 이 스택이 그 객체들의 다 쓴 참조를 여전히 가지고 있는 것인데, 다 쓴 참조란 앞으로 다시 쓰지 않을 참조를 뜻한다.

그럼 어떻게 해결 하면 될까?

```
public Object pop() {
    if (size == 0) {
        throw new RuntimeException();
    }
    Object result = elements[--size];
    elements[size] = null;
    return result;
}
```

해당 참조를 다 썼을 때 null 처리 하면 된다.

다 쓴 참조를 null 처리 하면 실수로 null 처리한 참조를 사용하려 할 때 NPE를 던지며 프로그램이 종료 되게 되어 오류를 조기에 발견 할 수 있다.  
객체를 쓰자마자 일일히 null 처리 할 필요는 전혀 없으며 객체 참조를 null 처리 하는 일은 예외적이 경우여야 한다.  
**자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야 한다.**

캐시 또한 메모리 누수가 일어 날 수 있다.

객체 참조를 캐시에 넣고 이 캐시를 그냥 두는 경우인데, 캐시를 만들 때 유효 기간을 (ttl) 두면 문제가 해결 된다.

마지막으로 리스너 혹은 콜백의 경우에도 약한 참조로 저장하면 가비지 컬렉터가 즉시 수거해간다.

```
 WeakReference<Stack> mStack = new WeakReference<>(new Stack());
        Stack stack = mStack.get();
        stack = null;
```

약한 참조는 WeakReference로 wrapping 한 객체를 뜻하며 mStack.get() 메소드로 객체를 가져 올 수 있다.

stack에 null 값을 주게 되면 weakly reachable 상태가 되며 weakly reachable 인 객체는 다음 GC때 반드시 제거된다.

> 메모리 누수는 겉으로는 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 있다. 이런 누수는 철저한 코드 리뷰나 힙 프로파일러 같은 디버깅 도구를 동원해야만 발견되기도 한다. 그래서 이런 종류의 문제는 예방법을익혀주는 것이 매우 중요하다.
