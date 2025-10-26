# 객체 지향 설계의 5원칙: **S.O.L.I.D**

## 1. SOLID란?

- **객체지향 설계의 5가지 원칙**을 뜻함.
    
- 목적: **변화에 유연하고 유지보수가 쉬운 설계 구조**를 만들기 위함.
    
- SOLID는 특정 언어나 프레임워크에 **종속되지 않으며**, 대부분의 객체지향 언어에 적용 가능함.
    

### SOLID 구성 원칙

|약어|원칙명|설명|
|---|---|---|
|SRP|Single Responsibility Principle|단일 책임 원칙|
|OCP|Open/Closed Principle|개방-폐쇄 원칙|
|LSP|Liskov Substitution Principle|리스코프 치환 원칙|
|ISP|Interface Segregation Principle|인터페이스 분리 원칙|
|DIP|Dependency Inversion Principle|의존 역전 원칙|

---

## 2. 각 원칙 정리

### SRP - 단일 책임 원칙

- **정의**: 클래스는 오직 **하나의 책임**만 가져야 한다.
    
- **의미**: 하나의 클래스는 하나의 기능만 담당하고, 변경 이유도 하나만 가져야 한다.
    
- **이점**: 유지보수성 향상, 코드 순환 참조 감소.
    
- **예시**: 청소기 클래스는 청소만 하면 됨. 빨래나 물주기는 다른 객체가 해야 함.
    

### OCP - 개방 폐쇄 원칙

- **정의**: 소프트웨어 요소는 **확장에는 열려 있고, 변경에는 닫혀 있어야** 한다.
    
- **목적**: 기존 코드를 수정하지 않고 기능을 추가할 수 있어야 함.
    
- **방법**: 상속, 추상화, 인터페이스 등을 활용해 구조 설계.
    
- **핵심 포인트**: 다형성과 추상화를 통해 유연하게 확장 가능.
    

### LSP - 리스코프 치환 원칙

- **정의**: **자식 클래스는 언제나 부모 클래스 역할을 대체할 수 있어야** 한다.
    
- **목적**: 업캐스팅된 객체가 부모 타입으로 동작할 때도 문제 없이 의도대로 작동해야 함.
    
- **주의사항**: 부모의 계약(계약 기반 설계)을 지키며 오버라이딩할 것.
    
- **예시**: `Collection` → `LinkedList` 또는 `HashSet`으로 바꿔도 `.add()` 메서드는 동일하게 작동함.
    

### ISP - 인터페이스 분리 원칙

- **정의**: **사용자에 따라 인터페이스를 분리**하라.
    
- **의미**: 하나의 범용 인터페이스보다는, 목적에 맞는 세부 인터페이스로 나누는 것이 좋다.
    
- **이점**: 불필요한 의존을 줄이고, 클라이언트의 사용 편의성 향상.
    
- **주의**: 이미 분리한 인터페이스는 가능하면 다시 수정하지 않도록 주의.
    

### DIP - 의존 역전 원칙

- **정의**: **구현이 아닌 추상화에 의존하라.**
    
- **방법**: 인터페이스나 추상 클래스를 통해 의존 관계를 맺고, 구체 클래스에 직접 의존하지 않도록 설계.
    
- **목적**: 모듈 간 결합도를 낮추고, 변경에 유연하게 대응 가능.
    
- **이점**: 테스트 용이성 증가, 유지보수성 향상.
    

---

## 3. SOLID 원칙 적용에 대한 추가 정보

- **적용 순서?**  
    → SOLID는 철자를 맞춘 것이며, 순서에 의존하지 않음.
    
- **모두 적용해야 하나?**  
    → 상황에 따라 일부만 적용해도 무방하며, **필요한 문제를 해결하기 위한 도구**로 활용.
    
- **SOLID는 OOP의 재정립**  
    → 추상화, 상속, 다형성 등 기존 OOP 개념을 실질적인 설계 기준으로 구체화한 것.
    
- **디자인 패턴과의 관계**  
    → 대부분의 디자인 패턴은 SOLID 원칙에 기반함.
    

---

## 4. 요약

- **좋은 설계란?**  
    변화에 강하고, 확장성과 유지보수가 쉬운 구조.
    
- **SOLID의 궁극적 목적?**  
    **개발 생산성 향상**과 **유지보수 비용 절감**을 위한 소프트웨어 구조 최적화.
    

---
## 1. SRP – 단일 책임 원칙

### ❌ Bad Example: 한 클래스가 여러 책임을 가짐

```java
public class Employee {
	public void calculatePay() {
		// 급여 계산 로직     
	}      
	public void saveToDatabase() {
		// DB 저장 로직     
	}      
	public void printReport() {
		// 보고서 출력 로직
	}
}
```

### ✅ Good Example: 각 책임을 별도의 클래스로 분리

```java
public class Employee {
    public void calculatePay() {
        // 급여 계산 로직
    }
}

public class EmployeeRepository {
    public void save(Employee employee) {
        // DB 저장 로직
    }
}

public class EmployeePrinter {
    public void print(Employee employee) {
        // 보고서 출력 로직
    }
}

```
---

## 2. OCP – 개방 폐쇄 원칙

### ❌ Bad Example: 새로운 기능 추가를 위해 기존 코드 수정

```java
public class DiscountService {
    public int getDiscount(String type) {
        if (type.equals("VIP")) return 20;
        else if (type.equals("Regular")) return 10;
        else return 0;
    }
}
```
### ✅ Good Example: 새로운 타입 추가 시 기존 코드를 수정하지 않음

```java
public interface DiscountPolicy {
    int getDiscount();
}

public class VIPDiscount implements DiscountPolicy {
    public int getDiscount() {
        return 20;
    }
}

public class RegularDiscount implements DiscountPolicy {
    public int getDiscount() {
        return 10;
    }
}

public class DiscountService {
    public int calculateDiscount(DiscountPolicy policy) {
        return policy.getDiscount();
    }
}
```

---

## 3. LSP – 리스코프 치환 원칙

### ❌ Bad Example: 자식 클래스가 부모의 계약을 어김

```java
public class Bird {
    public void fly() {
        System.out.println("날아갑니다.");
    }
}

public class Ostrich extends Bird {
    @Override
    public void fly() {
        throw new UnsupportedOperationException("타조는 날 수 없습니다!");
    }
}
```

### ✅ Good Example: 상위 타입으로 치환해도 문제 없음

```java
public abstract class Bird {
    public abstract void move();
}

public class Sparrow extends Bird {
    public void move() {
        System.out.println("하늘을 날아요.");
    }
}

public class Ostrich extends Bird {
    public void move() {
        System.out.println("땅을 달려요.");
    }
}
```

---

## 4. ISP – 인터페이스 분리 원칙

### ❌ Bad Example: 인터페이스가 너무 많은 기능을 강제

```java
public interface Machine {
    void print();
    void scan();
    void fax();
}

public class Printer implements Machine {
    public void print() { }
    public void scan() { throw new UnsupportedOperationException(); }
    public void fax() { throw new UnsupportedOperationException(); }
}
```

### ✅ Good Example: 기능별로 인터페이스를 분리

```java
public interface Printer {
    void print();
}

public interface Scanner {
    void scan();
}

public class SimplePrinter implements Printer {
    public void print() {
        System.out.println("출력합니다.");
    }
}
```

---

## 5. DIP – 의존 역전 원칙

### ❌ Bad Example: 고수준 모듈이 저수준 모듈에 직접 의존

```
public class MySQLDatabase {
    public void connect() {
        System.out.println("MySQL 연결");
    }
}

public class UserService {
    private MySQLDatabase db = new MySQLDatabase();

    public void registerUser() {
        db.connect();
    }
}
```

### ✅ Good Example: 고수준 모듈이 인터페이스에 의존

```java
public interface Database {
    void connect();
}

public class MySQLDatabase implements Database {
    public void connect() {
        System.out.println("MySQL 연결");
    }
}

public class UserService {
    private Database db;

    public UserService(Database db) {
        this.db = db;
    }

    public void registerUser() {
        db.connect();
    }
}
```


