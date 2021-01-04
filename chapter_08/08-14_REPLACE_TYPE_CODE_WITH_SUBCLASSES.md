### 분류 부호를 하위 클래스로 전환
클래스 기능에 영향을 주는 (조건식 : switch-case, if-else)

변경불가 분류 부호가 있을땐 (code)

분류 부호를 하위 클래스로 만들자

#### Why

조건 문이 있을 경우 조건문을 재정의로 전환을 실시해서 변경 필요
- 다형성 구조의 경우 내부 의존성에 대한 결합이 축소됨
- 다형화된 기능이 상속 구조로 변경 필요
- 분류 부호를 하위 클래스로 구성 하여 상속구조로 변경 가능

기법 적용시 제한사항
- 분류 부호의 값이 객체 생성 후 변할 때 
- 다른 이유로 분류 부호를 이미 하위클래스로 만들었을 때
- 분류 부호를 상태/전략 패턴으로 전환을 실시해야 할 때

기법 적용이 필요한 상황
- 조건문이 있을 경우 
  - 조건문을 재정의로 전환하기 위한 사전작업
  - 조건문이 없을 경우, 분류 부호를 클래스로 전환 기법 적용
- 특정 분류 부호의 객체에만 관련된 기능이 있을 때 (다형성)
  - 메서드 햐항 / 필드 하향 기법을 적용해서 그런 기능이 특정 경우에만 관련되어 있음을 분명히 할 수 있음

장점
- 다형적인 기능 관련 데이터가 클래스 자체로 이동
  - 변형된 새 기능 추가 시 하위클래스 하나만 추가하면 됨  

#### How
- 분류 부호 자체 캡슐화
- 각 분류 부호 값마다 그에 해당하는 하위 클래스 작성. 하위 클래스 안에 관련 값을 반환하는 분류 부호 읽기 메서드를 재정의
- 각 분류 부호 값을 하위 클래스로 만들 때마다 컴파일과 테스트 실시
- 상위 클래스의 분류 부호 필드 삭제. 분류 부호 읽기 메서드와 쓰기 메서드를 abstract 타입으로 선언
- 컴파일/테스트

#### Example
원본 :
```java
class Employee {
    private int _type;
    static final int ENGINEER = 0;
    static final int SALESMAN = 1;
    static final int MANAGER = 2;
    
    Employee (int type) {
        _type = type;
    }    
}
```

1. 필드 자체 캡슐화 적용
```java
int getType() { return _type; }
```

2. 생성시 분류 부호를 매개변수로 받을 경우, 생성자 메서드를 팩토리 매서드로 변경
```java
Employee create(int type) {
    return new Employee(type);
}

private Employee (int type) {
    _type = type;
}
```

3. 객체 생성할 수 있도록, 팩토리 매서드 변경
```java
Employee create(int type) {
    if (type == ENGINEER) return new Engineer();
    else return new Employee(type);
}
```

4. 분류 부호 필드를 abstract 타입으로 변경하고, 하위 클래스에서 처리
```java
abstract int gettype();

Employee create(int type) {
    if (type == ENGINEER) return new Engineer();
    else if (type == SALESMAN) return new Salesman();
    else if (type == MANAGER) return new Manager();
    else throw new IllegalArgumentException("분류 부호 값이 잘못 됨");
}
```
