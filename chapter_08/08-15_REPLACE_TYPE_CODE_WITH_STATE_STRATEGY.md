### 분류 부호를 상태/전략 패턴으로 전환

분류 부호가 클래스의 기능에 영향을 주지만

하위 클래스 전환에 어려움이 있을 경우

#### Why

분류 부호가 객체 수명주기 동안 변할 때나, 다른 이유로 하위클래스로 만들 수 없을 때

상태패턴 - 상태별 데이터를 이동하고 객체를 변화하는 상태로 생각할 때
전략패턴 - 조건문을 재정의로 전환으로 하나의 알고리즘을 단순화해야 할 때는 전략이 적절

#### How
- 분류 부호를 자체 캡슐화
- 분류 부호의 목적을 나타내는 이름으로 새 클래스를 작성 (상태 객체)
- 상태 객체에 각 분류 부호에 해당하는 하위 클래스를 추가
- 상태 객체 안에 분류 코드를 반환하는 abstract 메서드 호출을 작성. 이후 하위 클래스 추상화 구현
- 컴파일
- 원본 클래스 안에 새 상태 객체를 나타내는 필드 선언
- 원본 클래스에 있는 분류 부호 판단 코드를 상태 객체에 위임하게 수정
- 원본 클래스의 분류 부호 쓰기 메서드를 적절한 상태 객체 하위 클래스의 인스턴스를 할당하게 수정
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
    
    int payAmount() {
        switch (_type) {
            case ENGINEER :
                return _monthlySalary;
            case SALESMAN :
                return _monthlySalary + _commission;
            case MANAGER :
                return _monthlySalary + _bonus;
            default :
                throw new RuntimeException("사원이 잘못됨");                
        }
    }
    
}
```

1.분류 부호 자체 캠슐화
```java
class Employee {
    private int _type;
    static final int ENGINEER = 0;
    static final int SALESMAN = 1;
    static final int MANAGER = 2;
    
    Employee (int type) {
        _type = type;
    }
    
    int getType() { return _type; }
    
    int payAmount() {
        switch (getType()) {
            case ENGINEER :
                return _monthlySalary;
            case SALESMAN :
                return _monthlySalary + _commission;
            case MANAGER :
                return _monthlySalary + _bonus;
            default :
                throw new RuntimeException("사원이 잘못됨");                
        }
    }
    
}
```

2.상태 클래스 선언
```java
abstract class EmployeeType {
    abstract int getTypeCode();
}
```

3.분류 부호별 하위 클래스 작성
```java
class Engineer extends EmployeeType {
    int getTypeCode() { return Employee.ENGINEER; }
} 
``` 

4.컴파일

5.Employee 클래스에 실제 연결
```java
class Employee {
    private int _type;
     
    int getType() { return _type.getTypeCode(); }
    
    void setType(int arg) {
        switch (arg) {
            case ENGINEER :
                _type = new Engineer();
                break;
            case SALESMAN :
                _type = new Salesman();
                break;
            ....
        }
    }    
}
``` 

6. 분류 부호의 정보를 하위 클래스로 복사
```java
class Employee {
    void setType(int arg) {
        _type = EmployeeType.newType(arg);
    }
}

class EmployeeType {
    static EmployeeType new Type(int code) {
        switch (arg) {
            case ENGINEER :
                _type = new Engineer();
                break;
            case SALESMAN :
                _type = new Salesman();
                break;
            ....
        }
    }
    
    static final int ENGINEER = 0;
    static final int SALESMAN = 1;
    static final int MANAGER = 2;
}
```

7. Employee 클래스의 분류 부호 정의 삭제, 하위 클래스 참조
```java
class Employee {
    ... 
    int payAmount() {
        switch (getType()) {
            case EmployeeType.ENGINEER :
                return _monthlySalary;
            case EmployeeType.SALESMAN :
                return _monthlySalary + _commission;
            case EmployeeType.MANAGER :
                return _monthlySalary + _bonus;
            default :
                throw new RuntimeException("사원이 잘못됨");                
        }
    }
    
}
```

이후에 payAmount 메서드를 조건문을 재정의로 전환 기법 적용 가능