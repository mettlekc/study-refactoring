### 참조를 값으로 전환
#### When
- 참조 객체가 작고 수정할 수 없고 관리하기 힘들 때
- 변경이 불가해야 할 것이라 판단되는 경우
  - money(currency, value)
#### How
- 전환할 객체가 변경불가인지 변경 가능인지 검사
  - 전환 객체가 변경불가가 아니면, 변경불가가 될 때까지 getter,setter 제거 실시
  - 전환 객체가 변경불가이면 리펙토링 하지말자
- equals 메서드, hash 메서드 작성
- 컴파일/테스트
- 팩토리 메서드 삭제하고, 생성자를 public으로 만들어야 좋을지 생각
#### Example
```java
class Currency {
    ...
    private String _code;
    public String getCode() {
        return _code;
    }
    private Currency(String code) {
        _code = code;
    }
}
```
```java
Currency usd = Currency.get("USD");
```
```java
new Currency("USD").equals(new Currency("USD"));
```
- 결과 : false

변경
```java
public boolean equals(Object arg) {
    if(!(arg instanceof Currency)) return false;
    Currency other = (Currency) arg;
    return (_code.equals(other._code));
}

public int hashCode() {
    return _code.hashCode();
}
```
```java
new Currency("USD").equals(new Currency("USD"));
```
- 결과 : true