### 분류 부호를 클래스로 전환
#### When
기능에 영향을 미치는 숫자형 분류 부호가 든 클래스가 있을 경우, 숫자를 클래스로 전환 (Enumeration Class)

단, Switch 문과 같은 조건문 수행시 '조건문 재정의 전환'으로 리팩토링 해야 할 경우 이후에 나오는 하위클래스로 전환 또는 상태/전략 패턴으로 전환 기법을 적용하여 분류부호를 처리 필요  

#### How
- 분류 부호의 종류를 판단할 새 클래스 작성
  - 분류 부호를 처리할 필드 선언 (code)
  - 인수를 처리한 static 변수 선언 (A,B,AB,O)
  - 분류 부호를 사용 중인 원본 클래스의 내용을 새 클래스를 사용할 수 있도록 변경
  - 컴파일/테스트
  - 새로 작성된 클래스에 필요한 메서드 작성
  - 원본 클래스에 대응되는 메서드로 변경
  - 컴파일/테스트
  - 원본 클래스에 기존 인터페이스 삭제
  - 컴파일/테스트

#### Example
원본 :
```java
class Person {
    public static final int O = 0;
    public static final int A = 1;
    public static final int B = 2;
    public static final int AB = 3;
    
    public int _bloodGroup;
    
    public Person(int bloodGroup) { _bloodGroup = bloodGroup; }
    
    public void setBloodGroup(int arg) { _bloodGroup = arg; }
    
    public int getBloodGroup() { return _bloodGroup; }
}
```
1. 새 클래스 작성
```java
class BloodGroup {
    public static final BloodGroup O = new BloodGroup(0);
    public static final BloodGroup A = new BloodGroup(1);
    public static final BloodGroup B = new BloodGroup(2);
    public static final BloodGroup AB = new BloodGroup(3);
    private static final BloodGroup[] _values = {O,A,B,AB};
    
    private final int _code;
    
    private BloodGroup(int code) {
        _code = code;
    }
    
    private int getCode() { return _code; }
    public static BloodGroup code(int arg) { return _values[arg]; }
}
```
2. 원본 클래스 수정
```java
class Person {
    public static final BloodGroup O = BloodGroup.O.getCode();
    public static final BloodGroup A = BloodGroup.A.getCode();
    public static final BloodGroup B = BloodGroup.B.getCode();
    public static final BloodGroup AB = BloodGroup.AB.getCode();
    
    public BloodGroup _bloodGroup;
    
    public Person(int bloodGroup) { _bloodGroup = BloodGroup.code(bloodGroup); }
    
    public void setBloodGroup(int arg) { _bloodGroup = BloodGroup.code(arg); }
    
    public int getBloodGroup() { return _bloodGroup.getCode(); }
}
```

3. 원본 클래스를 사용하는 클래스 수정
원본 :
```java
Person thePerson = new Person(Person.A);
thePerson.getBloodTypeGroupCode();
```
변경 :
```java
Person thePerson = new Person(BloodGroup.A);
thePerson.getBloodGroup().getCode();
```

4. 원본 클래스 static 변수 삭제
```java
class Person {
    
    public BloodGroup _bloodGroup;
    
    public Person(BloodGroup bloodGroup) { _bloodGroup = bloodGroup; }
    
    public void setBloodGroup(BloodGroup arg) { _bloodGroup = arg; }
    
    public int getBloodGroup() { return _bloodGroup; }
}
```