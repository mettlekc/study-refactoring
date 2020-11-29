# Chapter 7. 객체 간의 기능 이동

기능을 어디에 넣을지 판단하는 것

##### Overview

필드이동 > 메서드 이동  
- 기능 위치 고민

클래스 추출 / 클래스 내용 직접 삽입 
- 클래스의 기능 정리

대리 객체 은폐 
- 모듈의 기능 정리

과잉중개 메서드 제거 
- 인터페이스 변경이 잦을 때

외래 클래스 메서드 추가 / 국소적 상속확장 클래스 사용 
- 원본 코드에 접근 할 수 없는 상황에서 수정 불가능한 클래스로 기능을 이동해야 할 경우 

### 메서드 이동

![move-method](move-method-01.png)

#### When
- 클래스 기능이 너무 많을 때
- 클래스 간 과하게 연동되어 의존성이 지나칠때
- 메서드를 많이 참조한다고 생각하는 객체 기준으로 진행 여부 결정

#### How
- 원본 메서드에 사용된 모든 기능 검사
- 원본 클래스의 하위클래스/상위클래스에서 그 메서드에 대한 다른 선언이 있는지 검사
- 그 메서드를 대상 클래스 안에 선언
- 원본 메서드의 코드를 대상 메서드에 복사 후 대상 클래스 안에서 잘 돌아가게끔 대상 메서드를 수정
- 대상 클래스 컴파일
- 원본 객체에서 대상 객채를 참조할 방법 확인
- 원본 메서드를 위임 메서드로 전환
- 컴파일과 테스트 실시
- 원본 메서드를 삭제하든 위임 메서드로 사용하든 선택
- 원본 메서드를 삭제할 때는 기존의 참조를 전부 대상 메서드 참조로 수정
- 컴파일과 테스트 실시

#### Example

```java
class Account {
    double overdraftCharge() {
        if (_type.isPremium()) {
            double result = 10;
            if (_daysOverdrawn > 7) result += (_daysOverdrawn - 7) * 0.85;
            return result;
        } else { return _daysOverdrawn * 1.75; }
    }
    
    double bankCharge() {
        double result = 4.5;
        if (_daysOverdrawn > 0) result += overdraftCharge();
        return result;
    }
    
    private AccountType _type;
    private int _daysOverdrawn;
}
```

목표 overdraftCharge()를 AccountType 클래스로 이동

Step 1. AccoutType 클래스로 메서드 복사
```java
class AccountType {
    double overdraftCharge(int daysOverdrawn) {
            if (isPremium()) {
                double result = 10;
                if (daysOverdrawn > 7) result += (daysOverdrawn - 7) * 0.85;
                return result;
            } else { return daysOverdrawn * 1.75; }
        }
}
```

Step 2. 적절한 작업 수행
- 기능 전체를 대상 클래스로 이동
- 대상 클래스에서 원본 클래스를 참조 생성 또는 사용
- 원본 객체를 대상 객체에 매개변수로 전달
- 기능이 변수라면 변수를 매개변수로 전달

Step 3. 원본 메서드의 코드를 위임코드로 변경
```java
class Account {
    double overdraftCharge() {
        return _type.overdraftCharge(_daysOverdrawn);
    }
    ...
}
```

Step 4. 컴파일 / 테스트

Step 5. 원본 클래스의 메서드 삭제를 위한 수정
```java
class Account {
    
    ...
    
    double bankCharge() {
        double result = 4.5;
        if (_daysOverdrawn > 0) result += _type.overdraftCharge(_daysOverdrawn);
        return result;
    }
    
    private AccountType _type;
    private int _daysOverdrawn;
}
```

Step 6. 원본 클래스에 메서드 정의 삭제

추가. 변수 한개 이상 전달 할 경우, 객체 전체를 전달해야 한다

### 필드 이동
#### When
- 필드가 자신이 속한 클래스보다 다른 클래스에서 더 많이 사용 될때
- 메서드를 더 많이 참조해서 정보를 이용할 때, 메서드는 현재 위치가 적절하다면 필드만 이동
- 클래스 추출 시 필드를 우선 옮기고, 메서드를 이동

#### How
- 필드가 public일 경우 캡슐화 기법 실시 (필드 자체 캡슐화)
- 컴파일 / 테스트
- 대상 클래스 안에 읽ㄱ/쓰기 메서드와 함께 필드를 작성
- 대상 클래스 컴파일
- 원본 객체에서 대상 객채를 참조할 방법 정하기
- 원본 클래스에서 필드를 삭제
- 원본 필드를 참조하는 모든 부분을 대상 클래스에 있는 적절한 메서드를 참조하게 수정
- 컴파일 / 테스트

#### Example

```java
class Account {
    private AccountType _type;
    private double _interestRate;
    
    double interestForAmount_days(double amount, int days) {
        return _interestRate * amount * days / 365;
    }
    
}
```

목표. rate 필드를 AccountType 클래스로 이동

Step 1. 필드와 set/get 생성
```java
class AccountType {
    private double _interestRate;
    
    void setInterestRate(double arg) {
        _interestRate = arg;
    }
    
    double getInterestRate() {
        return _interestRate;
    }
}
```

Step 2. 컴파일 / 테스트

Step 3. 원본 클래스 안의 메서드들을 AccountType 클래스를 사용하도록 참조를 변경, 원본에서 interestRate 필드 삭제

```java
class Account {
    double interestForAmount_days(double amount, int days) {
        return _type.getInterestRate * amount * days / 365;
    } 
}
```

참고. 필드 자체 캡슐화
```java
class Account {
    
    private double _interestRate;
    
    double interestForAmount_days(double amount, int days) {
        return getInterestRate * amount * days / 365;
    }
    
    private void setInterestRate(double arg) {
        _interestRate = arg;
    }
    
    private double getInterestRate() {
        return _interestRate;
    }
}
```

### 클래스 추출
#### When
- 두 클래스가 처리해야 할 기능이 하나의 클래스에 들어 있을 때, 기능을 다른 클래스로 분리
- 데이터의 일부분과 메서드의 일부분이 한덩이
- 함께 변화하거나 서로 유난히 의존적인 데이터의 일부분
- 메서드를 하나 제거하면 어떻게 될지, 다른 필드와 메서드를 추가하는 건 합리적이지 않은지 자문

#### How
- 클래스의 기능 분리 방법 정하기
- 분리한 기능을 넣을 새 클래스 작성
- 원본 클래스에서 새 클래스로의 링크 생성
- 필드 이동 적용
- 필드 옮길 때 마다 컴파일 / 테스트 수행
- 메서드 이동을 실시 / 테스트 수행
- 인터페이스 줄이기 (양방향 링크를 단방향으로 조정)
- 여러 곳에서 클래스에 접근 할 수 있게 할지 보고 공개 여부 결정

#### Example
```java
class Person {
    ...
    public String getName() { return _name; }
    public String getTelephoneNumber() { return ("(" + _officeAreaCode + ")" + _officeNumber); }
    
    public String getOfficeAreaCode() { return _name; }
    public String setOfficeAreaCode(String arg) { _officeAreaCode = arg; }
    
    public String getOfficeNumber() { return _name; }
    public String setOfficeNumber(String arg) { _officeNumber = arg; }
    
    private String _name;
    private String _officeAreaCode;
    private String _officeNumber;
    ...
}
```

목표. 전화번호 기능 분리
```java
class TelephoneNumber {    
}
```

Step 1. 원본 클래스에 TelephoneNumber 클래스로 링크 작성
```java
class Person {
    ...
    private TelephoneNumber _officeTelephone = new TelephoneNumber();
    ...
}
```

Step 2. 필드 이동 수행
```java
class TelephoneNumber {
    public String getAreaCode() { return _name; }
    public String setAreaCode(String arg) { _areaCode = arg; }
    
    private String _areaCode;
}
```
```java
class Person {
    ...
    public String getTelephoneNumber() { return ("(" + getOfficeAreaCode() + ")" + _officeNumber); }
        
    public String getOfficeAreaCode() { return _officeTelephone.getAreaCode(); }
    public String setOfficeAreaCode(String arg) { _officeTelephone.setAreaCode(arg); }
    ...
```

Step 3. 컴파일 / 테스트

Step 4. Telephone Number 메서드에 대해서도 동일하게 수행

Step 5. 컴파일 / 테스트

### 클래스 내용 직접 삽입
#### When
- 클래스에 기능이 너무 적을 때
- 주로 리펙터링 실시 후 남은 기능이 거의 없어졌을 때 사용

#### How
- 원본 클래스의 public 프로토콜 메서드를 합칠 클래스에 선언 후 이 메서드를 전부 원본 클래스에 위임
- 원본 클래스의 모든 참조를 합칠 클래스 참조로 수정
- 컴파일 / 테스트
- 메서드 이동, 필드 이동 실시
- 원본 클래스 삭제

#### Example
```java
class Person {
    ...
    public String getName() { return _name; }
    
    public String getTelephoneNumber() { return _officeTelephone.getTelephoneNumber(); }
    public String getOfficeTelephone() { return _officeTelephone; }
    
    private String _name;
    private TelephoneNumber _officeTelephone = new TelephoneNumber();
    ...
}

class TelephoneNumber {
    ...
    public String getTelephoneNumber() {
        return ("(" + _areaCode + ")" + _number);
    }
    
    String getAreaCode() {
        return _areaCode;
    }
    
    String setAreaCode(String arg) {
        _areaCode = arg;
    }
    
    String getNumber() {
        return _number;
    }
    
    void setNumber(String arg) {
        _number = arg;
    }
    private String _number;
    private String _areaCode;
    ...
}
```

목표. TelephoneNumber 클래스에 모든 외부 공개 메서드르 Person 클래스에 선언
```java
class Person {
    ...
    String getAreaCode() {
        return _officeTelephone.getAreaCode();
    }
    
    String setAreaCode(String arg) {
        _officeTelephone.setAreaCode(arg);
    }
    
    String getNumber() {
        return _officeTelephone.getNumber();
    }
    
    void setNumber(String arg) {
        _officeTelephone.setNumber(arg);
    }
    ...
}
```


### 대리 객체 은폐
#### When
- 클라이언트가 객체의 대리 클래스를 호출 할 때
- 캡슐화

#### How
- 대리 객체에 들어 있는 각 메서드를 대상으로 서버에 위임 메서드 작성
- 클라이언트를 수정해서 서버를 호출
- 각 메서드를 수정 할 때마다 컴파일 / 테스트
- 대리 객체를 읽고 써야 할 클라이언트가 하나도 남지 않게 되면, 서버에서 대리 객체가 사용하는 읽기/쓰기 메서드 삭제
- 컴파일 / 테스트

#### Example
```java
class Person {
    Department _department;
    
    public Department getDepartment() {
        return _department;
    }
    
    public void setDepartment(Department arg) {
        _department = arg;
    }
}

class Department {
    private String _chargetCode;
    private Person _manager;
    
    public Department(Person manager) {
        _manager = manager;
    }
    
    public Person getManager() {
        return _manager;
    }
}
```
```java
manager = john.getDepartment().getManager();
```

수정 후,

```java
public Person getManager() {
    return _department.getManger();
}
```
```java
manager = john.getManager();
```

### 과잉 중개 메서드 제거
#### When
- 클래스에 자잘한 위임이 너무 많을 때
- 캡슐화의 단점
- 은폐의 정도에 대한 고찰

#### How
- 대리 객체에 대한 접근 메서드 작성
- 대리 메서드를 클라이언트가 사용할 때마다 서버에서 메서드를 제거하고 클라이언트에서 호출을 대리 객체에서의 메서드 호출로 교체
- 메서드를 수정할 때마다 테스트 수행

#### Example
```java
class Person {
    Department _department;
    public Person getManager() { return _department.getManager(); }
}
...
class Department {
    private Person _manager;
    public Department(Person manager) {
        _manager = manager;
    }
}
```
```java
manager = john.getManager();
```

Person 객체가 위임으로 비대해질 경우, '대리 객체 은폐' 반대로 수행


### 외래 클래스 메서드 추가
#### When
- 사용 중인 서버 클래스에 메서드를 추가해야 하는데 그 클래스를 수정할 수 없을 때

#### How
- 필요한 기능의 메서드를 클라이언트 클래스 안에 작성
- 서버 클래스의 인스턴스를 첫 번째 매개변수로 만들기
- 그 메서드에 '서버 클래스에 있을 외래 메서드'같은 주석 표기

#### Example
```java
Date newStart = new Date(previousEnd.getYear(), previousEnd.getMonth(), previous.getDate() + 1)
```

```java
Date newStart = nextDay(previousEnd);

private static Date nextDay(Date arg) {
    return new Date(arg.getYear(), arg.getMonth(), arg.getDate() + 1);
}
```

### 국소적 상속확장 클래스 사용
#### When
- 사용 중인 서버 클래스에 여러개의 메서드를 추가해야 하는데 클래스를 수정 할 수 없을 때

#### How
- 상속확장 클래스를 작성한 후 원본 클래스의 하위클래스나 래퍼 클래스로 만들기
- 상속화장 클래스에 변환 생성자 메서드 작성
- 상속확장 클래스에 새 기능 추가
- 필요한 위치마다 원본 클래스를 상속확장 클래스로 수정
- 이 클래스용으로 정의된 외래 메서드를 전부 상속확장 클래스로 이동

#### Example
하위 클래스 사용
```java
class MfDateSub extends Date {
    public MfDateSub nextDa() ...
    public int dayOfYear() ...
}
```
Wrapper 클래스 사용
```java
class mfDateWrap {
    private Date _original;
}
```