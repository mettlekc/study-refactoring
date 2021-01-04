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