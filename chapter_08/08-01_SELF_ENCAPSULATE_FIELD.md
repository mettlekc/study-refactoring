### 필드 자체 캡슐화

필드에 접근하던 중 **필드로의 결합에 문제**가 생길 때,
필드용 getter/setter를 작성 두 메서드를 통해서만 필드에 접근하도록 수정

```java
private int _low, _high;
boolean includes (int arg) {
    return arg >= _low && arg <= _high;
}
```

```java
private int _low, _high;
boolean includes (int arg) {
    return arg >= getLow() && arg <= getHigh();
}
int getLow() {return _low;}
int getHigh() {return _high;}
```

 #### When
 - 변수 접근 방식에 대한 고찰
   - 직접 접근 방식
   - 간접 접근 방식
   - 개인의 스타일
 - 상위클래스 안의 필드에 접근하되 이 변수 접근을 하위클래스에서 계산된 값으로 재정의해야 할 때
  
 #### How
 - getter()/setter() 작성
 - 필드 참조 부분을 전부 찾아 읽기 메서드와 쓰기 메서드로 수정
   - 필드를 삭제하면, 컴파일을 통해 쉽게 확인 가능
 - 필드를 private으로 수정
 - 참조 코드를 모두 수정했는지 다시 확인
 - 테스트

#### Example 
```java
class IntRange {
    
    private int _low, _high;
    
    boolean includes(int arg) {
        return arg >= _low && arg <= _high;
    }
    
    void grow(int factor) {
        _high = _high * factor;
    }
    
    IntRange(int low, int high) {
        _low = low;
        _high = high;
    }
}
```

```java
class IntRange {
    
    private int _low, _high;
    
    boolean includes(int arg) {
        return arg >= getLow() && arg <= getHigh();
    }
    
    void grow(int factor) {
        setHigh(getHigh() * factor);
    }
    
    IntRange(int low, int high) {
        initialize(low, high);
    }
    
    private void initialize(int low, int high) {
        _low = low;
        _high = high;
    }
    
    int getLow() {
        return _low;
    }
    
    int getHigh() {
        return _high;
    }
    
    void setLow(int arg) {
        _low = arg;
    }
    
    void setHigh(int arg) {
        _high = arg;
    }
}
``` 

```java
class CappedRange extends IntRange {
    CappedRange(int low, int high, int cap) {
        super(low, high);
        _cap = cap;
    }
    
    private int _cap;
    
    int getCap() {
        return _cap;
    }
    
    int getHIgh() {
        return Math.min(super.getHigh(), getCap());
    }
}
```