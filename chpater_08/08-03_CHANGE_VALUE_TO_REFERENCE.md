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