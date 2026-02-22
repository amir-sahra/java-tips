# 🧼 Clean Code in Java — Common Developer Mistakes

Clean code is not about perfection; it is about clarity, predictability, and ease of change. In Java projects, a handful of recurring mistakes make codebases hard to maintain and expensive to scale. This guide shows the most common issues and the practical fixes, with simple Java examples you can apply immediately.

## ✅ What “clean” means in Java

Clean Java code is:
- Easy to read without extra context
- Easy to change without breaking other parts
- Easy to test in isolation
- Consistent with the rest of the codebase

## 🧭 Common mistakes and how to fix them

### 🧠 1) Vague or misleading names

Bad names hide intent and force readers to scan the entire method.

**Before**

```java
public List<User> f(List<User> u) {
    List<User> r = new ArrayList<>();
    for (User x : u) {
        if (x.getA() > 18) {
            r.add(x);
        }
    }
    return r;
}
```

**After**

```java
public List<User> findAdultUsers(List<User> users) {
    List<User> adults = new ArrayList<>();
    for (User user : users) {
        if (user.getAge() > 18) {
            adults.add(user);
        }
    }
    return adults;
}
```

### 🧱 2) Methods that do too much

Large methods blur responsibilities and make testing difficult.

**Before**

```java
public Invoice createInvoice(Order order) {
    if (order == null || order.getItems().isEmpty()) {
        throw new IllegalArgumentException("Invalid order");
    }
    BigDecimal total = BigDecimal.ZERO;
    for (OrderItem item : order.getItems()) {
        total = total.add(item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())));
    }
    Invoice invoice = new Invoice(order.getId(), total);
    invoice.setCreatedAt(Instant.now());
    invoice.setStatus(InvoiceStatus.CREATED);
    return invoice;
}
```

**After**

```java
public Invoice createInvoice(Order order) {
    validateOrder(order);
    BigDecimal total = calculateTotal(order.getItems());
    return buildInvoice(order, total);
}

private void validateOrder(Order order) {
    if (order == null || order.getItems().isEmpty()) {
        throw new IllegalArgumentException("Invalid order");
    }
}

private BigDecimal calculateTotal(List<OrderItem> items) {
    BigDecimal total = BigDecimal.ZERO;
    for (OrderItem item : items) {
        total = total.add(item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())));
    }
    return total;
}

private Invoice buildInvoice(Order order, BigDecimal total) {
    Invoice invoice = new Invoice(order.getId(), total);
    invoice.setCreatedAt(Instant.now());
    invoice.setStatus(InvoiceStatus.CREATED);
    return invoice;
}
```

### 🧩 3) Magic numbers and hard-coded values

When values appear without context, intent is lost.

**Before**

```java
if (user.getFailedAttempts() >= 5) {
    user.lock();
}
```

**After**

```java
private static final int MAX_FAILED_ATTEMPTS = 5;

if (user.getFailedAttempts() >= MAX_FAILED_ATTEMPTS) {
    user.lock();
}
```

### 🎯 4) Boolean parameters that hide intent

Booleans at call sites are ambiguous.

**Before**

```java
emailService.send(user, true);
```

**After**

```java
emailService.sendWelcomeEmail(user);
```

### ⚠️ 5) Overusing null instead of modeling intent

Returning null pushes uncertainty to every caller.

**Before**

```java
public User findByEmail(String email) {
    return repository.findByEmail(email);
}
```

**After**

```java
public Optional<User> findByEmail(String email) {
    return repository.findByEmail(email);
}
```

### 🧯 6) Catching broad exceptions without context

Swallowing exceptions hides failures and breaks debugging.

**Before**

```java
try {
    paymentGateway.charge(request);
} catch (Exception e) {
    return PaymentResult.failed();
}
```

**After**

```java
try {
    paymentGateway.charge(request);
    return PaymentResult.success();
} catch (GatewayTimeoutException e) {
    return PaymentResult.retryableFailure();
} catch (GatewayRejectedException e) {
    return PaymentResult.failed();
}
```

### 📦 7) Data clumps and long parameter lists

Many parameters often indicate a missing abstraction.

**Before**

```java
public User createUser(String firstName, String lastName, String email, String phone, String countryCode) {
    return new User(firstName, lastName, email, phone, countryCode);
}
```

**After**

```java
public User createUser(UserProfile profile) {
    return new User(profile.getFirstName(), profile.getLastName(), profile.getEmail(),
            profile.getPhone(), profile.getCountryCode());
}
```

### 🔁 8) Duplicated logic across layers

Repeated logic makes changes error-prone.

**Before**

```java
public BigDecimal calculateDiscount(BigDecimal total) {
    if (total.compareTo(new BigDecimal("1000")) >= 0) {
        return total.multiply(new BigDecimal("0.10"));
    }
    return BigDecimal.ZERO;
}
```

**After**

```java
public BigDecimal calculateDiscount(BigDecimal total) {
    return discountPolicy.discountFor(total);
}
```

### 🏗️ 9) Mixing domain logic with infrastructure

Domain code becomes untestable and tightly coupled.

**Before**

```java
public void register(User user) {
    entityManager.persist(user);
    emailSender.send(user.getEmail());
}
```

**After**

```java
public void register(User user) {
    userRepository.save(user);
    notificationService.sendWelcome(user);
}
```

### 🧩 10) Poor class cohesion

Classes that handle unrelated responsibilities are hard to reason about.

**Before**

```java
public class ReportService {
    public byte[] generatePdf(Report report) { return new byte[0]; }
    public void sendReportEmail(User user, byte[] pdf) {}
    public void archiveReport(byte[] pdf) {}
}
```

**After**

```java
public class ReportGenerator {
    public byte[] generatePdf(Report report) { return new byte[0]; }
}

public class ReportNotifier {
    public void sendReportEmail(User user, byte[] pdf) {}
}

public class ReportArchive {
    public void archiveReport(byte[] pdf) {}
}
```

## ✅ Practical checklist

- Use names that reveal intent
- Keep methods short and focused
- Extract constants for meaningful values
- Avoid ambiguous booleans in method calls
- Prefer Optional for “maybe” results
- Catch specific exceptions and map them to outcomes
- Reduce parameter lists with value objects
- Eliminate duplication with shared policies or services
- Separate domain logic from infrastructure
- Keep classes cohesive and focused

## 🧾 Summary

Clean code in Java is about reducing surprise and maximizing clarity. If your code explains itself, changes are safer, tests are simpler, and teammates move faster. Start by fixing one mistake at a time, and your codebase will steadily become more maintainable.
