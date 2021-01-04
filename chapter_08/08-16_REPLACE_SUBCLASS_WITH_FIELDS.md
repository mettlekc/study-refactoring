### 하위클래스를 필드로 전환
여러 하위 클래스가 상수 데이터를 반환하는 메서드만 다를 때

각 하위클래스의 메서드를 상위클래스 필드로 전환하고 하위 클래스는 전부 삭제

> 단순한 재정의는 삭제하고 상위 클래스에 맡기자

#### Why
하위 클래스의 불필요한 복잡함을 제거할 수 있다
- 불필요한 메서드는 새로운 하위 클래스 정의 시 복잡도가 증가된다

#### Example

원본 :
```java
abstract class Person {
    abstract boolean isMale();
    abstract char getCode();
}

class Male extends Person {
    boolean isMale() { return true; }
    char getCode() { return 'M'; }
}

class Female extends Person {
    boolean isMale() { return false; }
    char getCode() { return 'F'; }
}
```

1.팩토리 매서드 패턴으로 변경
```java
class Person {
    static Person createMale() {
        return new Male();
    }
    static Person createFeMale() {
        return new Female();
    }
}
```

2.하위 클래스 생성을 팩토리 매서드로 전환

3.각 상수 메서드별 필드 선언
```java
class Person {
    private final boolean _isMale;
    private final char _code;
    
    protected Person(boolean isMale, char code) {
        ...
    }
}
```

4.생성 메서드 추가
```java
class Male {
    Male() {
        super(ture, 'M');
    }
}
```

5.컴파일/테스트

6.상위 클래스에 읽기 메서드를 넣어 필드를 사용하게하고 하위클래스 메서드 삭제
```java
class Person... {
    boolean isMale() {
        return _isMale;
    }
}
```
<pre><code><del>class Male {</del>
<del>    boolean isMale() {</del>
<del>      return true;</del>
<del>    }</del>
<del>}</del>
</pre></code>

