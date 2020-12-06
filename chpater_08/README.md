# Chapter 08. 데이터 체계화
데이터 연동을 간편하게 하기 위한 방법

필드 자체 캡슐화 (Self Encapsulate Field)

데이터 값을 객체로 전환 (Replace Data Value with Object)
- 불명확한 데이터를 뚜렷한 객체로 변경 가능
- 프로그램 여러 부분에 사용될 인스턴스라면 참조 전환(Change Value to Reference)으로 참조 객체 생성

배열을 객체로 전환 (Replace Array with Object)
- 배열이 데이터 구조 역할을 할 때
- 객체로 변환 이후 장점은 변환 이후에 메서드 이동을 적용해서 새로 생긴 객체에 기능을 추가하기 쉬워짐

매직 넘버
- 프로그래밍 코드 안에 사용되는 상수, 배열 크기, 문자열 안의 글자 위치, 변환 계수 등 특정한 각종 숫자 값
- 마법 숫자를 기호 상수로 전환 (Replace Magic Number with Symbolic Constant) 기법으로 제거

클래스의 단방향 연결을 양방향으로 전환

클래스의 양방향 연결을 단방향으로 전환

관측 데이터 복제

필드 캡슐화

컬렉션 캡슐화

레코드를 데이터 클래스로 전환

분류 부호를 클래스로 전환

분류 부호를 하위 클래스로 전환

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
 
### 데이터 값을 객체로 전환

데이터 항목에 데이터 기능을 더 추가해야 할 때는,
데이터 항목을 객체로 만들자

#### When
- 데이터 항목이 점차 복잡해 질 때 > '중복코드', '잘못된 소속'

#### How
- 데이터 

#### Example
AS-IS
```java
class Order {
    ...
    public Order(String customer) {
        _customer = customer;
    }
    
    public String getCustomer() {
        return _customer;
    }
    
    public void setCustomer(String arg) {
        _customer = arg;
    }
    
    private String _customer;
    ...
}
```
```java
private static int numberOfOrderFor(Collection orders, String customer) {
    int result = 0;
    Iterator iter = order.iterator();
    while(iter.hasNext()) {
        Order each = (Order) iter.next();
        if(each.getCustomer().equals(customer)) result++;
    }
    return result;
}
```
TO-BE
```java
class Customer {
    public Customer(String name) {
        _name = name;
    }
    
    public String getName() {
        return _name;
    }
    
    private final String _name;
}
```

```java
class Order {
    ...
    public Order(String customer) {
        _customer = new Customer(customer);
    }
    
    public String getCustomer() {
        return _customer;
    }
    
    public String getCustomerName() {
            return _customer.getName();
    }
    
    public void setCustomer(String arg) {
        _customer = new Customer(arg);
    }
    
    private Customer _customer;
    ...
}
```

한계 :
- 신용등급, 주소 등을 추가하려면, Customer가 값 객체로 취급되기 때문에 어려움
- 값을 참조로 전환을 적용, 한 고객의 모든 주문이 하나의 Customer 객체를 사용하도록 변경

### 값을 참조로 전환

#### When
- 클래스에 같은 인스턴스가 많이 들어 있어서 이것들을 하나의 객체로 바꿔야 할 때
- 객체 분류
  - 참조 객체
  - 값 객체

#### How
- 생성자를 팩토리 메서드 전환
- 컴파일/테스트
- 참조 객체로의 접근을 담당할 객체를 정함
  - 정적 딕셔러니, 레지스트리 객체 등 이용
  - 참조 객체로의 접근을 둘 이상의 객체로 담당
- 객체를 미리 생성할지 사용하기 직전에 생성할지 결정
- 참조 객체를 반환하게 팩토리 메서드를 수정
- 컴파일/테스트

#### Example
원본
```java
class Customer {
    public Customer(String name) {
        _name = name;
    }
    public String getName() { return _name; }
    
    private final String _name;
}
```
```java
class Order {
    ...
    public Order(String customer) {
        _customer = new Customer(customer);
    }
    
    public String getCustomer() {
        return _customer;
    }
    
    public String getCustomerName() {
            return _customer.getName();
    }
    
    public void setCustomer(String arg) {
        _customer = new Customer(arg);
    }
    
    private Customer _customer;
    ...    
}
```
```java
private static int numberOfOrdersFor(Collection orders, String customer) {
    int result = 0;
    Iterator iter = orders.iterator();
    while (iter.hasNext()) {
     Order each = (Order) iter.next();
     if (each.getCustomerName().equals(customer)) result++;
    }
    return result;
}
```
변경
```java
class Customer {
    // 1차 변경
    //public static Customer create(String name) {
    //    return new Customer(name);
    //}
    
    public static Customer getNamed(String name) {
        return (Customer) _instance.get(name);
    }
    
    private Customer (String name) {
        _name = name;
    }
    
    // 미리 생성 방식
    static void loadCustomers() {
        new Customer("test 1").store();
        new Customer("test 2").store();
        new Customer("test 3").store();
        new Customer("test 4").store();        
    }
    private void store() {
        _instance.put(this.getName(), this);
    }
    
    // 객체 접근 방식
    private static Dictinary _instance = new Hashtable();
    ...
}
```
```java
class Order {
    public Order(String customer) {
        _customer = Customer.create(customer);
    }
    
    ...
}
```

