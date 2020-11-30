# Chapter 6. 메서드 정리

##### Overview

### 메서드 추출
```java
void printOwing(double amount) {
    printBanner();
    
    // 세부 정보 출력
    System.out.println("name:" + _name);
    System.out.println("amount:" + amount);
}
```
```java
void printOwing(double amount) {
    printBanner();
    printDetails(amount);
}

void printDetails(double amount) {
    System.out.println("name:" + _name);
    System.out.println("amount:" + amount);
}
```

#### When
- 메서드가 너무 길거나 주석을 달아야만 의도를 이해할 수 있을 때
- 직관적인 이름의 간결한 메서드가 좋은 이유
  1. 메서드가 적절히 잘게 쪼개져 있으면 다른 메서드에서 쉽게 사용 가능 (재사용성)
  2. 상위 계층의 메서드에 주석 같은 더 많은 정보 처리 가능
  3. 재정의 쉬움
  
#### How
- 목적에 맞는 이름의 새 메서드 생성
  - 좋은 메서드 명 = 원리가 아닌 기능을 나타내는 이름
  - 한 두줄의 코드를 추출하는 당위성은 새 메서드 명을 통해 그 코드의 기능(목적)을 잘 들어 낼 수 있을 때
- 기존 메서드에서 빼낸 코드를 새로 생성한 메서드로 복사
- 빼낸 코드에서 기존 메서드의 모든 지역변수 참조를 찾는다. 그것들을 새로 생성한 메서드의 지역변수 또는 매개변수로 사용
- 빼낸 코드 안에서만 사용되는 임시변수가 있는지 파악해서 있다면 그것들을 새로 생성한 메서드 안에 임시변수로 선언
- 추출 코드에 의해 변경되는 지역변수가 있는지 파악
  - 하나의 변수 음 
  - 두개 변수 이상, 임시변수 분리 등의 기법 적용, 이후 임시 변수 제거하려면 임시변수 메서드 호출로 전환기법 사용
- 빼낸 코드에서 읽어들이 지역변수를 대상 메서드에 매개변수로 전달
- 모든 지역변수 처리르 완료했으면 컴파일 실시
- 원본 메서드 안에서 빼낸 코드 부분을 새로 생성한 메서드 호출로 수정
- 컴파일 / 테스트

#### Example
##### 지역변수 사용 안 함
```java
void printOwing() {
    Enumeration e = _orders.elements();
    double outstanding = 0.0;
    
    // 배너 출력
    System.out.println("***********************");
    System.out.println("*******고객 외상********");
    System.out.println("***********************");
    
    // 외상액 계산
    while (e.hasMoreElements()) {
        Order each = (Order) e.nextElement();
        outstanding += each.getAmount();
    }
    
    // 세부 내역 출력
    System.out.println("고객명: " + _name);
    System.out.println("외상액: " + outstanding);
}
````
```java
void printOwing() {
    Enumeration e = _orders.elements();
    double outstanding = 0.0;
    
    printBanner();
    
    // 외상액 계산
    while (e.hasMoreElements()) {
        Order each = (Order) e.nextElement();
        outstanding += each.getAmount();
    }
    
    // 세부 내역 출력
    System.out.println("고객명: " + _name);
    System.out.println("외상액: " + outstanding);
}

void printBanner() {
    // 배너 출력
    System.out.println("***********************");
    System.out.println("*******고객 외상********");
    System.out.println("***********************");
}
````

##### 지역변수 사용
```java
void printOwing() {
    Enumeration e = _orders.elements();
    double outstanding = 0.0;
    
    printBanner();
    
    // 외상액 계산
    while (e.hasMoreElements()) {
        Order each = (Order) e.nextElement();
        outstanding += each.getAmount();
    }
    
    // 세부 내역 출력
    System.out.println("고객명: " + _name);
    System.out.println("외상액: " + outstanding);
}
```
```java
void printOwing() {
    Enumeration e = _orders.elements();
    double outstanding = 0.0;
    
    printBanner();
    
    // 외상액 계산
    while (e.hasMoreElements()) {
        Order each = (Order) e.nextElement();
        outstanding += each.getAmount();
    }
    
    printDetails(outstanding);

}

void printDetails(double outstanding) {
    // 세부 내역 출력
    System.out.println("고객명: " + _name);
    System.out.println("외상액: " + outstanding);
}
```

##### 지역변수를 다시 대입
```java
void printOwing() {
    Enumeration e = _orders.elements();
    double outstanding = 0.0;
    
    printBanner();
    
    // 외상액 계산
    while (e.hasMoreElements()) {
        Order each = (Order) e.nextElement();
        outstanding += each.getAmount();
    }
    
    printDetails(outstanding);

}
```
Step 1. 계산 부분 추출
```java
void printOwing() {
    printBanner();
    double outstanding =  getOutstanding();
    printDetails(outstanding);
}

double getOutstanding() {
    Enumeration e = _orders.elements();
    double outstanding = 0.0;
    // 외상액 계산
    while (e.hasMoreElements()) {
        Order each = (Order) e.nextElement();
        outstanding += each.getAmount();
    }
    return outstanding;
}
```
Step 2. 반환 값 이름 변경
```java
double getOutstanding() {
    Enumeration e = _orders.elements();
    double result = 0.0;
    // 외상액 계산
    while (e.hasMoreElements()) {
        Order each = (Order) e.nextElement();
        result += each.getAmount();
    }
    return result;
}
```
Step 3. 매개변수로 이전 값 전달하도록 되어 있을 경우 
```java
void printOwing(double previousAmount) {
    Enumeration e = _orders.elements();
    double outstanding =  previousAmount * 1.2;
    printBanner();
    
    // 외상액 계산
    while (e.hasMoreElements()) {
        Order each = (Order) e.nextElement();
        outstanding += each.getAmount();
    }
        
    printDetails(outstanding);
}
```
```java
void printOwing(double previousAmount) {
    double outstanding =  previousAmount * 1.2;
    printBanner();
    outstanding = getOutstanding(outstanding);
    printDetails(outstanding);
}

double getOutstanding(double initialValue) {
    double result = initialValue;
    Enumeration e = _orders.elements();
    // 외상액 계산
    while (e.hasMoreElements()) {
        Order each = (Order) e.nextElement();
        result += each.getAmount();
    }
    return result;
}
```
```java
void printOwing(double previousAmount) {
    printBanner();
    outstanding = getOutstanding(previousAmount * 1.2);
    printDetails(outstanding);
}
```

### 메서드 내용 직접 삽입
```java
int getRating() {
    return (moreThanFiveLateDeliveries()) ? 2 : 1;
}
boolean moreThanFiveLateDeliveries() {
    return _numberOfLateDeliveries > 5;
}
```
```java
int getRating() {
    return (_numberOfLateDeliveries > 5) ? 2 : 1;
}
```
#### When
- 메서드 기능이 지나치게 단순 할 때

#### How
- 메서드가 재정의되어 있지 않은지 확인
  - 메서드가 하위클래스에 재정의되어 있다면 진행 금지
- 그 메서드를 호출하는 부분을 모두 찾자
- 각 호출 부분을 메서드 내용으로 교체
- 테스트 수행
- 메서드 정의 삭제

***재귀 처리 방법, 여러 개의 반환 지점 처리 방법, 접근 메서드가 없을 때 다른 객체 안에 직접 삽입하는 방법 등....***

### 임시변수 내용 직접 삽입
```java
double basePrice = anOrder.basePrice();
return (basePrice > 1000)
```
```java
return (anOrder.basePrice() > 1000)
```

### 임시변수를 메서드 호출로 전환
```java
double basePrice = _quantity * _itemPrice;
if (basePrice > 1000)
    return basePrice * 0.95;
else
    return basePrice * 0.98;
```
```java
if (basePrice() > 1000)
    return basePrice() * 0.95;
else
    return basePrice() * 0.98;

double basePrice() {
    return _quantity * _itemPrice;
}
```
#### When
- 임시변수 문제점
  - 일시적이고 적용이 국소적 범위로 제한
  - 자신이 속한 메서드 안에서만 인식, 접근에 제한
- 메서드 추출 적용 전에 적용
  
#### How

#### Example
```java
double getPrice() {
    int basePrice = _quantity * _itemPrice;
    double discountFactor;
    if (basePrice > 1000) discountFactor = 0.95;
    else discountFactor = 0.98;
    return basePrice * discountFactor;
}
```

Step 1. final 선언
```java
double getPrice() {
    final int basePrice = _quantity * _itemPrice;
    final double discountFactor;
    if (basePrice > 1000) discountFactor = 0.95;
    else discountFactor = 0.98;
    return basePrice * discountFactor;
}
```

Step 2. 메서드 호출 변경
```java
double getPrice() {
    return basePrice() * discountFactor();
}

private int basePrice() {
    return _quantity * _itemPrice;
}

private double discountFactor() {
    if (basePrice() > 1000) return 0.95;
    else return 0.98;
}
```

### 직관적 임시변수 사용
```java
if ((platform.toUpperCase().indexOf("MAC") > -1) &&
    (browser.toUpperCase().indexOf("IE") > -1) &&
    wasInitialized() && resize > 0 ) {
        // 기능 코드
}
```
```java
final boolean isMacOs = platform.toUpperCase().indexOf("MAC") > -1;
final boolean isIEBrowser = browser.toUpperCase().indexOf("IE") > -1;
final boolean wasResized = resize > 0;

if (isMacOS && isIEBrowser && wasInitialized() && wasResized) {
        // 기능 코드
}
```
#### Example
```java
double price() {
    // 결제액(price) = 총 구매액(base price) -
    // 대량 구매 할인 (quantity discount) + 배송비(shipping)
    return _quantity * _itemPrice 
            - Math.max(0, _quantity - 500) * _itemPrice * 0.05 
            + Math.min(_quantity * _itemPrice * 0.1, 100.0); 
}
```
임시 변수 사용
```java
double price() {
    // 결제액(price) = 총 구매액(base price) -
    // 대량 구매 할인 (quantity discount) + 배송비(shipping)
    final double basePrice = _quantity * _itemPrice;
    final double quantityDiscount = Math.max(0, _quantity - 500) * _itemPrice * 0.05;
    final double shipping = Math.min(basePrice * 0.1, 100.0);
    return  basePrice - quantityDiscount + shipping; 
}
```
메서드 추출
```java
double price() {
    // 결제액(price) = 총 구매액(base price) -
    // 대량 구매 할인 (quantity discount) + 배송비(shipping)
    return  basePrice() - quantityDiscount() + shipping(); 
}

private double basePrice() {
    return _quantity * _itemPrice;
}

private double quantityDiscount() {
    return Math.max(0, _quantity - 500) * _itemPrice * 0.05;
}

private double shipping() {
    return Math.min(basePrice() * 0.1, 100.0);
}
```

### 임시변수 분리
```java
double temp = 2 * (_height + _width);
System.out.println(temp);
temp = _height * _width;
System.out.println(temp);
```
```java
final double perimeter = 2 * (_height + _width);
System.out.println(perimeter);
final double area = _height * _width;
System.out.println(area);
```
#### Example
```java
double getDistanceTravelled(int time) {
    double result;
    double acc = _primaryForce / _mass;
    int primaryTime = Math.min(time, _delay);
    result = 0.5 * acc * primaryTime * primaryTime;
    int secondaryTime = time - _delay;
    if (secondaryTime > 0) {
        double primaryVel = acc * _delay;
        acc = (_primaryForce + _secondaryForce) / _mass;
        result += primaryVel * secondaryTime + 0.5 * acc * secondaryTime * secondaryTime;
    }
    return result;
}
```
```java
double getDistanceTravelled(int time) {
    double result;
    final double primaryAcc = _primaryForce / _mass;
    int primaryTime = Math.min(time, _delay);
    result = 0.5 * primaryAcc * primaryTime * primaryTime;
    int secondaryTime = time - _delay;
    if (secondaryTime > 0) {
        double primaryVel = primaryAcc * _delay;
        double acc = (_primaryForce + _secondaryForce) / _mass;
        result += primaryVel * secondaryTime + 0.5 * acc * secondaryTime * secondaryTime;
    }
    return result;
}
```
```java
double getDistanceTravelled(int time) {
    double result;
    final double primaryAcc = _primaryForce / _mass;
    final int primaryTime = Math.min(time, _delay);
    result = 0.5 * primaryAcc * primaryTime * primaryTime;
    int secondaryTime = time - _delay;
    if (secondaryTime > 0) {
        double primaryVel = primaryAcc * _delay;
        final double secondaryAcc = (_primaryForce + _secondaryForce) / _mass;
        result += primaryVel * secondaryTime + 0.5 * secondaryAcc * secondaryTime * secondaryTime;
    }
    return result;
}
```
### 매개변수로의 값 대입 제거
```java
int discount(int inputVal, int quantity, int yearToDate) {
    if (inputVal > 50) inputVal -= 2;
    ...
}
```
```java
int discount(int inputVal, int quantity, int yearToDate) {
    int result = inputVal;
    if (inputVal > 50) result -= 2;
    ...
}
```
#### When
- 매개 변수 값을 대입하는 코드가 있을 땐 매개변수를 대신하는 임시변수 사용

#### Why
- 코드의 명료성도 떨어지고, '값을 통한 전달'과 '참조를 통한 전달'을 혼동
- '값을 통한 전달'로 통일
- 용도의 일관성으로 코드 이해가 쉬워짐

#### How
- 매개변수 대신 사용할 임시변수 선언
- 매개변수로 값을 대입하는 코드 뒤에 나오는 매개변수 참조를 전부 인시변수로 수정
- 매개변수의 값 대입을 임시변수로의 값 대임으로 수정
- 컴파일 / 테스트
  - 반환값이 둘 이상일 경우 객체화도 고려

#### Example
```java
int discount(int inputVal, int quantity, int yearToDate) {
    if (inputVal > 50) inputVal -= 2;
    if (quantity > 100) inputVal -= 1;
    if (yearToDate > 10000) inputVal -= 4;
    return inputVal;
}
```
```java
int discount(final int inputVal, final int quantity, final int yearToDate) {
    int result = inputVal;
    if (inputVal > 50) result -= 2;
    if (quantity > 100) result -= 1;
    if (yearToDate > 10000) result -= 4;
    return result;
}
```

자바에서 값을 통한 전달
```java
class Param {
    public static void main(String[] args) {
        int x = 5;
        triple(x);
        System.out.println("triple 메서드 실행 후 x 값: " + x);
    }
    private static void triple(int arg) {
        arg = arg * 3;
        System.out.println("triple 메서드 안의 arg 값" + arg)
    }
}
```

```text
triple 메서드 안의 arg 값 : 15
triple 메서드 실행 후 x 값 : 5
```

```java
class Param {
    public static void main (String[] args) {
        Date d1 = new Date("1 Apr 98");
        nextDateUpdate(d1);
        System.out.println("nextDay 메서드 실행 후 d1 값 " + d1);
        
        Date d2 = new Date("1 Apr 98");
        nextDateReplace(d2);
        System.out.println("nextDay 메서드 실행 후 d2 값 " + d2);
    }
    
    private static void nextDateUpdate(Date arg) {
        arg.setDate(arg.getDate() + 1);
        System.out.println("nextDay 메서드 안의 arg 값: " + arg);
    }
    
    private static void nextDateReplace(Date arg) {
        arg = new Date(arg.getYear(), arg.getMonth(), arg.getDate() +1);
        System.out.println("nextDay 메서드 안의 arg 값: " + arg);
    }
}
```
```text
nextDay 메서드 안의 arg 값: Thu Apr 02 00:00:00 EST 1998 
nextDay 메서드 실행 후 d1 값: Thu Apr 02 00:00:00 EST 1998 
nextDay 메서드 안의 arg 값: Thu Apr 02 00:00:00 EST 1998 
nextDay 메서드 실행 후 d2 값: Thu Apr 01 00:00:00 EST 1998
```
### 메서드를 메서드 객체로 전환
```java
Class Order {
    ...
    double price() {
        double primaryBasePrice;
        double secondaryBasePrice;
        double tetiaryBasePrice;
        // 긴 계산 코드
        ...
    }
}
```
#### When
- 지역변수가 많아 메서드를 쪼개기 어려울 때
- 분해가 필요한 메서드를 분해 할 수 없을 때

#### Pros
- 모든 지역변수가 메서드 객체의 속성이 됨

#### How
- 전환 할 메서드의 이름과 같은 이름으로 새 클래스 작성
- 그 클래스에 원본 메서드가 들어 있던 객체를 나타내는 final 필드를 추가하고, 원본 메서드 안의 각 임시변수와 매개변수에 해당하는 속성을 추가
- 새 클래스에 원본 객체와 각 매개변수를 받는 생성자 메서드를 작성
- 새 클래스에 compute라는 이름의 메서드를 작성
- 원본 메서드 내용을 compute 메서드 안에 복사, 원본 객체에 있는 메서드를 호출할 땐 원본 객체를 나타내는 필드를 사용
- 컴파일
- 원본 메서드를 새 객체 생성과 compute 메서드 호출을 담당하는 메서드로 바꾸기

#### Example
```java
Class Account {
    int gamma(int inputVal, int quantity, int yearToDate) {
        int importantValue1 = (inputVal * quantity) + delta();
        int importantValue2 = (inputVal * yearToDate) + 100;
        if ((yeatToDate - importantValue1) > 100)
            importantValue2 -= 20;
        int importantValue3 = importantValue2 * 7;
        
        // 기타 작업
        
        return importantValue3 - 2 * importantValue1;
    }
    ...
}
```

```java
class Gamma {
    ...
    private final Account _account;
    private final int inputVal;
    private final int quantity;
    private final int yearToDate;
    private final int importantValue1;
    private final int importantValue2;
    private final int importantValue3;
    ...
}
```

```java
Gamma(Account source, int inputValArg, int quantityArg, int yearToDateArg) {
    _account = source;
    inputVal = inputValArg;
    quantity = quantityArg;
    yearToDate = yearToDateArg;
}
```

```java
int compute() {
    int importantValue1 = (inputVal * quantity) + delta();
    int importantValue2 = (inputVal * yearToDate) + 100;
    if ((yeatToDate - importantValue1) > 100)
        importantValue2 -= 20;
    int importantValue3 = importantValue2 * 7;
    
    // 기타 작업
    
    return importantValue3 - 2 * importantValue1;    
}
```
- 인자 전달 없이 compute 메서드에서 손쉽게 수정 가능
### 알고리즘 전환
#### When
#### How
#### Example