# SOLID

## Single Responsibility Principle (SRP) - Nguyên tắc trách nhiệm đơn lẻ

Một class chỉ nên làm một nhiệm vụ duy nhất.

```java
class OrderService {
    void createOrder() {}
}

class EmailService {
    void sendEmail() {}
}

class LoggerService {
    void writeLog() {}
}

```

## Open/Closed Principle (OCP) - Nguyên tắc mở/đóng.

Class nên mở rộng được nhưng không sửa đổi code cũ.

* kế thừa và strategy pattern
* Hay dùng strategty pattern để tăng tính mở rộng tránh lỗi.

VD: push event

interface pushEvent();

class DebeziumKafka impl pushEvent() 
class sendEvent impl pushEvent()

## Liskov Substitution Principle (LSP) - Nguyên tắc thay thế Liskov

Class con phải thay thế được class cha mà không làm sai logic

```java
❌ Sai: Class con phá vỡ logic class cha

class Bird {
    void fly() {}
}

class Ostrich extends Bird {
    void fly() { throw new UnsupportedOperationException("Cannot fly"); }
}

✔️ Đúng — tách interface

interface Flyable { void fly(); }
class Sparrow implements Flyable { public void fly() {} }
class Ostrich {} // không implements
```

## Interface Segregation Principle (ISP) - Nguyên tắc phân tách giao diện

Không ép class implement phương thức không cần thiết.

```java
❌ Sai — interface quá to

interface Worker {
    void code();
    void design();
}

✔️ Đúng — chia nhỏ

interface Developer { void code(); }
interface Designer { void design(); }
```

## Dependency Inversion Principle (DIP) - Nguyên tắc đảo ngược phụ thuộc

Phụ thuộc abstraction (interface), không phụ thuộc implementation cụ thể.

* Module cấp cao (business logic) không được phụ thuộc trực tiếp vào module cấp thấp (implementation cụ thể).
* Cả hai phải phụ thuộc vào abstraction (interface).

Abstraction không được phụ thuộc vào chi tiết, mà chi tiết phải phụ thuộc vào abstraction.

```java
❌ Sai — phụ thuộc trực tiếp vào Email 
(nếu logic nghiệp vụ yc đổi phương thức gửi -> sms -> sửa code cũ -> vi phạm OCP)

class NotificationService {
    private Email email = new Email();
}


✔️ Đúng — dùng interface

@ApplicationScoped
public class EmailNotifier implements Notifier {
    public void send(String message) { ... }
}

@ApplicationScoped
public class SmsNotifier implements Notifier {
    public void send(String message) { ... }
}


Quarkus:

@ApplicationScoped
class NotificationService {

    @Inject
    Notifier notifier; // Inject bất kỳ implementation nào
}
```

