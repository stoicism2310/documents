# Design Pattern

## Singleton

Quarkus - không cần tự implement Singleton pattern bằng code Java (private constructor, static getInstance…).
➡  chỉ cần dùng @Singleton hoặc @ApplicationScoped, Quarkus sẽ quản lý lifecycle, threading, và injection.

```java
import jakarta.inject.Singleton;
import org.jboss.logging.Logger;

@Singleton
public class AppLogger {
    private static final Logger LOG = Logger.getLogger(AppLogger.class);
    public void info(String message) {
        LOG.info(message);
    }
    public void error(String message, Throwable t) {
        LOG.error(message, t);
    }
}
```

```java
import jakarta.inject.Inject;
@Singleton
public class OrderService {
    @Inject
    AppLogger logger;
    public void processOrder() {
        logger.info("Start process order");
    }
}
```

- singleton: chỉ cần 1 object trong jvm, (logger, cached, config read-only)
tránh sử dụng trong các biến global mutable state -> concurency (đồng thời)


