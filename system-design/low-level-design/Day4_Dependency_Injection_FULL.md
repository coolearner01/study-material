# Day 04 — Dependency Injection (DI)
## 30-Day LLD Series 
### Language: Java

---

## What is Dependency Injection?

Before understanding DI, understand what a **dependency** is.

```java
class OrderService {
    private MySQLDatabase db = new MySQLDatabase();  // dependency
}
```

`OrderService` **depends on** `MySQLDatabase`.
`MySQLDatabase` is a **dependency** of `OrderService`.

**The problem:** `OrderService` is creating its own dependency.
It controls what database it uses.
You cannot change it without editing `OrderService`.
You cannot test it without a real MySQL connection.

**Dependency Injection** flips this:

```java
class OrderService {
    private Database db;  // dependency

    public OrderService(Database db) {  // injected from outside
        this.db = db;
    }
}
```

Now `OrderService` does not create its dependency.
Someone **outside** creates it and **injects** it in.
`OrderService` does not care if it is MySQL, PostgreSQL, or a fake test database.

> **Dependency Injection means: do not create your dependencies inside a class. Receive them from outside.**

---

## Why Does This Matter?

```
Without DI:
  - Class controls its own dependencies
  - Swapping a database = editing every class that uses it
  - Testing requires a real database, real email server, real payment gateway
  - Classes are tightly coupled — change one, break many

With DI:
  - Dependencies are swapped without touching the class
  - Testing uses fake/mock dependencies — no real servers needed
  - Classes are loosely coupled — change one implementation, nothing else breaks
  - Each class has one job — creating dependencies is someone else's job
```

---

## 3 Types of Dependency Injection

```
1. Constructor Injection   → dependency passed via constructor  (PREFERRED)
2. Setter Injection        → dependency passed via setter method
3. Interface Injection     → dependency passed via interface method
```

We will cover all three with full code.

---

## Bad Code — No Dependency Injection

```java
// BAD — every dependency is created inside the class
// Hard to test. Hard to swap. Tightly coupled.

public class MySQLDatabase {
    public void save(String data) {
        System.out.println("[MySQL] Saving: " + data);
    }

    public String find(int id) {
        System.out.println("[MySQL] Finding record: " + id);
        return "record_" + id;
    }
}

public class GmailService {
    public void sendEmail(String to, String subject, String body) {
        System.out.println("[Gmail] Sending to " + to
            + " | Subject: " + subject
            + " | Body: "    + body);
    }
}

public class StripePayment {
    public boolean charge(double amount, String cardToken) {
        System.out.println("[Stripe] Charging $" + amount
            + " on card " + cardToken);
        return true;
    }
}

public class OrderService {

    // PROBLEM 1: hardcoded to MySQL
    private MySQLDatabase db      = new MySQLDatabase();

    // PROBLEM 2: hardcoded to Gmail
    private GmailService  emailer = new GmailService();

    // PROBLEM 3: hardcoded to Stripe
    private StripePayment payment = new StripePayment();

    public void placeOrder(String item, double price,
                           String email, String cardToken) {

        boolean charged = payment.charge(price, cardToken);
        if (!charged) {
            System.out.println("Payment failed.");
            return;
        }

        db.save("Order: " + item + " | Price: $" + price);
        emailer.sendEmail(email, "Order Confirmed",
            "Your order for " + item + " is confirmed!");

        System.out.println("Order placed: " + item);
    }
}

// Usage
OrderService service = new OrderService();
service.placeOrder("Java Book", 499.0, "alice@example.com", "tok_visa_123");

// Problems with this code:
// 1. Want to switch from MySQL to PostgreSQL?
//    Edit OrderService. Risk breaking it.
// 2. Want to test without real Stripe charges?
//    Impossible — Stripe is hardcoded inside.
// 3. Want to test without real emails being sent?
//    Impossible — Gmail is hardcoded inside.
// 4. Want to use a different payment gateway for different regions?
//    Cannot. It is locked to Stripe.
```

**Output:**
```
[Stripe] Charging $499.0 on card tok_visa_123
[MySQL]  Saving: Order: Java Book | Price: $499.0
[Gmail]  Sending to alice@example.com | Subject: Order Confirmed | Body: ...
Order placed: Java Book
```

---

## Type 1 — Constructor Injection (Preferred)

```java
// GOOD — dependencies are injected via the constructor

// ── Step 1: Define interfaces (abstractions) ───────────────────────────────

public interface Database {
    void save(String data);
    String find(int id);
}

public interface EmailService {
    void sendEmail(String to, String subject, String body);
}

public interface PaymentGateway {
    boolean charge(double amount, String cardToken);
}


// ── Step 2: Implement the interfaces concretely ───────────────────────────

public class MySQLDatabase implements Database {
    @Override
    public void save(String data) {
        System.out.println("[MySQL] Saving: " + data);
    }

    @Override
    public String find(int id) {
        System.out.println("[MySQL] Finding: " + id);
        return "mysql_record_" + id;
    }
}

public class PostgreSQLDatabase implements Database {
    @Override
    public void save(String data) {
        System.out.println("[PostgreSQL] Saving: " + data);
    }

    @Override
    public String find(int id) {
        System.out.println("[PostgreSQL] Finding: " + id);
        return "pg_record_" + id;
    }
}

public class GmailService implements EmailService {
    @Override
    public void sendEmail(String to, String subject, String body) {
        System.out.println("[Gmail] To: " + to
            + " | Subject: " + subject + " | " + body);
    }
}

public class SendGridService implements EmailService {
    @Override
    public void sendEmail(String to, String subject, String body) {
        System.out.println("[SendGrid] To: " + to
            + " | Subject: " + subject + " | " + body);
    }
}

public class StripeGateway implements PaymentGateway {
    @Override
    public boolean charge(double amount, String cardToken) {
        System.out.println("[Stripe] Charged $" + amount
            + " on " + cardToken);
        return true;
    }
}

public class RazorpayGateway implements PaymentGateway {
    @Override
    public boolean charge(double amount, String cardToken) {
        System.out.println("[Razorpay] Charged ₹" + amount
            + " on " + cardToken);
        return true;
    }
}


// ── Step 3: Inject dependencies via constructor ───────────────────────────

public class OrderService {

    private final Database       db;
    private final EmailService   emailer;
    private final PaymentGateway payment;

    // All three dependencies injected from outside
    // OrderService does not know or care which implementations are used
    public OrderService(Database db,
                        EmailService emailer,
                        PaymentGateway payment) {
        this.db      = db;
        this.emailer = emailer;
        this.payment = payment;
    }

    public boolean placeOrder(String item, double price,
                              String email, String cardToken) {

        // 1. Charge the customer
        boolean charged = payment.charge(price, cardToken);
        if (!charged) {
            System.out.println("[ORDER] Payment failed for: " + item);
            return false;
        }

        // 2. Persist the order
        db.save("item=" + item + ", price=" + price + ", buyer=" + email);

        // 3. Notify the customer
        emailer.sendEmail(
            email,
            "Order Confirmed: " + item,
            "Thank you! Your order for " + item
                + " worth $" + price + " is confirmed."
        );

        System.out.println("[ORDER] Successfully placed: " + item);
        return true;
    }
}


// ── Step 4: Wire dependencies from outside ────────────────────────────────

public class Main {
    public static void main(String[] args) {

        // Production: MySQL + Gmail + Stripe
        System.out.println("=== Production (MySQL + Gmail + Stripe) ===");
        OrderService prodService = new OrderService(
            new MySQLDatabase(),
            new GmailService(),
            new StripeGateway()
        );
        prodService.placeOrder(
            "Java Book", 499.0, "alice@example.com", "tok_visa_001"
        );

        System.out.println();

        // India: PostgreSQL + SendGrid + Razorpay
        // OrderService code = ZERO changes
        System.out.println("=== India Stack (PostgreSQL + SendGrid + Razorpay) ===");
        OrderService indiaService = new OrderService(
            new PostgreSQLDatabase(),
            new SendGridService(),
            new RazorpayGateway()
        );
        indiaService.placeOrder(
            "Spring Boot Course", 1999.0, "bob@example.in", "tok_razorpay_002"
        );
    }
}
```

**Output:**
```
=== Production (MySQL + Gmail + Stripe) ===
[Stripe]     Charged $499.0 on tok_visa_001
[MySQL]      Saving: item=Java Book, price=499.0, buyer=alice@example.com
[Gmail]      To: alice@example.com | Subject: Order Confirmed: Java Book | ...
[ORDER]      Successfully placed: Java Book

=== India Stack (PostgreSQL + SendGrid + Razorpay) ===
[Razorpay]   Charged ₹1999.0 on tok_razorpay_002
[PostgreSQL]  Saving: item=Spring Boot Course, ...
[SendGrid]   To: bob@example.in | Subject: Order Confirmed: Spring Boot Course | ...
[ORDER]      Successfully placed: Spring Boot Course
```

`OrderService` was not touched. Not one line changed.
Only the injected implementations changed.
**That is the power of DI.**

---

## Type 2 — Setter Injection

```java
// Setter injection — dependencies set after object construction
// Use when: dependency is optional, or needs to be changed after creation

public class NotificationService {

    private EmailService emailService;    // optional — may not always send email
    private SMSService   smsService;      // optional — may not always send SMS

    // Setters — called only when needed
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }

    public void setSmsService(SMSService smsService) {
        this.smsService = smsService;
    }

    public void notify(String user, String message) {
        if (emailService != null) {
            emailService.sendEmail(user, "Notification", message);
        } else {
            System.out.println("[NOTIFY] No email service configured — skipping email");
        }

        if (smsService != null) {
            smsService.sendSMS(user, message);
        } else {
            System.out.println("[NOTIFY] No SMS service configured — skipping SMS");
        }
    }
}

public interface SMSService {
    void sendSMS(String phone, String message);
}

public class TwilioSMS implements SMSService {
    @Override
    public void sendSMS(String phone, String message) {
        System.out.println("[Twilio] SMS to " + phone + ": " + message);
    }
}


// Usage
NotificationService ns = new NotificationService();

// Scenario 1: only email
ns.setEmailService(new GmailService());
ns.notify("alice@example.com", "Your package is shipped!");

System.out.println();

// Scenario 2: email + SMS
ns.setSmsService(new TwilioSMS());
ns.notify("+91-9999999999", "OTP: 847291");
```

**Output:**
```
[Gmail]  To: alice@example.com | Subject: Notification | Your package is shipped!
[NOTIFY] No SMS service configured — skipping SMS

[Gmail]  To: +91-9999999999 | Subject: Notification | OTP: 847291
[Twilio] SMS to +91-9999999999: OTP: 847291
```

---

## Type 3 — Interface Injection

```java
// Interface injection — the dependency injects itself via an interface the class implements
// Less common in Java but useful for self-contained injection contracts

public interface DatabaseInjectable {
    void injectDatabase(Database db);
}

public interface EmailInjectable {
    void injectEmailService(EmailService emailService);
}

public class ReportService implements DatabaseInjectable, EmailInjectable {

    private Database     db;
    private EmailService emailService;

    @Override
    public void injectDatabase(Database db) {
        this.db = db;
        System.out.println("[ReportService] Database injected: "
            + db.getClass().getSimpleName());
    }

    @Override
    public void injectEmailService(EmailService emailService) {
        this.emailService = emailService;
        System.out.println("[ReportService] EmailService injected: "
            + emailService.getClass().getSimpleName());
    }

    public void generateAndSendReport(String reportName, String recipient) {
        String data = db.find(1);
        System.out.println("[REPORT] Generated: " + reportName + " | Data: " + data);
        emailService.sendEmail(recipient, "Report: " + reportName,
            "Please find your report attached.");
    }
}

// Injector — responsible for injecting dependencies
public class Injector {

    public static void inject(ReportService service,
                               Database db,
                               EmailService emailService) {
        service.injectDatabase(db);
        service.injectEmailService(emailService);
    }
}


// Usage
ReportService reportService = new ReportService();
Injector.inject(reportService, new MySQLDatabase(), new SendGridService());
reportService.generateAndSendReport("Monthly Sales", "manager@company.com");
```

**Output:**
```
[ReportService] Database injected: MySQLDatabase
[ReportService] EmailService injected: SendGridService
[MySQL]  Finding: 1
[REPORT] Generated: Monthly Sales | Data: mysql_record_1
[SendGrid] To: manager@company.com | Subject: Report: Monthly Sales | ...
```

---

## Testing Without DI vs With DI

```java
// ── Fake implementations for testing ──────────────────────────────────────
// No real database. No real emails. No real charges.
// Tests run in milliseconds with zero external setup.

import java.util.ArrayList;
import java.util.List;

public class FakeDatabase implements Database {
    private final List<String> savedRecords = new ArrayList<>();

    @Override
    public void save(String data) {
        savedRecords.add(data);
        System.out.println("[FakeDB] Saved: " + data);
    }

    @Override
    public String find(int id) {
        return "fake_record_" + id;
    }

    public List<String> getSavedRecords() {
        return savedRecords;
    }
}

public class FakeEmailService implements EmailService {
    private final List<String> sentEmails = new ArrayList<>();

    @Override
    public void sendEmail(String to, String subject, String body) {
        sentEmails.add(to + "|" + subject);
        System.out.println("[FakeEmail] Recorded email to: " + to);
    }

    public List<String> getSentEmails() {
        return sentEmails;
    }

    public boolean wasEmailSentTo(String recipient) {
        return sentEmails.stream().anyMatch(e -> e.startsWith(recipient));
    }
}

public class FakePaymentGateway implements PaymentGateway {
    private boolean shouldSucceed;
    private int     chargeCount = 0;

    public FakePaymentGateway(boolean shouldSucceed) {
        this.shouldSucceed = shouldSucceed;
    }

    @Override
    public boolean charge(double amount, String cardToken) {
        chargeCount++;
        System.out.println("[FakePayment] Simulated charge of $" + amount
            + " | Success: " + shouldSucceed);
        return shouldSucceed;
    }

    public int getChargeCount() { return chargeCount; }
}


// ── Unit Tests ────────────────────────────────────────────────────────────

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class OrderServiceTest {

    private FakeDatabase       fakeDb;
    private FakeEmailService   fakeEmailer;
    private FakePaymentGateway fakePayment;
    private OrderService       orderService;

    @BeforeEach
    void setUp() {
        fakeDb      = new FakeDatabase();
        fakeEmailer = new FakeEmailService();
        fakePayment = new FakePaymentGateway(true);    // payment succeeds

        // inject fakes — no real MySQL, Gmail, or Stripe needed
        orderService = new OrderService(fakeDb, fakeEmailer, fakePayment);
    }

    @Test
    void order_should_save_to_database_on_success() {
        orderService.placeOrder(
            "Java Book", 499.0, "alice@example.com", "tok_test"
        );

        assertEquals(1, fakeDb.getSavedRecords().size());
        assertTrue(fakeDb.getSavedRecords().get(0).contains("Java Book"));
    }

    @Test
    void order_should_send_confirmation_email_on_success() {
        orderService.placeOrder(
            "Java Book", 499.0, "alice@example.com", "tok_test"
        );

        assertTrue(fakeEmailer.wasEmailSentTo("alice@example.com"));
        assertEquals(1, fakeEmailer.getSentEmails().size());
    }

    @Test
    void order_should_charge_payment_gateway_once() {
        orderService.placeOrder(
            "Java Book", 499.0, "alice@example.com", "tok_test"
        );

        assertEquals(1, fakePayment.getChargeCount());
    }

    @Test
    void order_should_not_save_or_email_when_payment_fails() {
        // inject a payment gateway that always fails
        FakePaymentGateway failingPayment = new FakePaymentGateway(false);
        OrderService failService = new OrderService(
            fakeDb, fakeEmailer, failingPayment
        );

        boolean result = failService.placeOrder(
            "Java Book", 499.0, "alice@example.com", "tok_test"
        );

        assertFalse(result);
        assertEquals(0, fakeDb.getSavedRecords().size());    // nothing saved
        assertEquals(0, fakeEmailer.getSentEmails().size()); // no email sent
    }

    @Test
    void order_should_return_true_on_successful_placement() {
        boolean result = orderService.placeOrder(
            "Spring Course", 999.0, "bob@example.com", "tok_test_2"
        );

        assertTrue(result);
    }
}
```

---

## DI Without a Framework — Manual Wiring

```java
// You do NOT need Spring to use DI.
// You can wire everything manually in one place.
// This is called the Composition Root.

public class AppConfig {

    // All wiring happens here and only here.
    // Every other class just receives what it needs.

    public static OrderService buildOrderService() {
        Database       db      = new MySQLDatabase();
        EmailService   emailer = new GmailService();
        PaymentGateway payment = new StripeGateway();

        return new OrderService(db, emailer, payment);
    }

    public static NotificationService buildNotificationService() {
        NotificationService ns = new NotificationService();
        ns.setEmailService(new GmailService());
        ns.setSmsService(new TwilioSMS());
        return ns;
    }

    public static ReportService buildReportService() {
        ReportService rs = new ReportService();
        Injector.inject(rs, new PostgreSQLDatabase(), new SendGridService());
        return rs;
    }
}

// Main — everything built and wired in one place
public class Main {
    public static void main(String[] args) {

        OrderService       orders        = AppConfig.buildOrderService();
        NotificationService notifications = AppConfig.buildNotificationService();
        ReportService      reports       = AppConfig.buildReportService();

        orders.placeOrder(
            "Design Patterns Book", 799.0, "alice@example.com", "tok_visa"
        );

        notifications.notify("bob@example.com", "Flash sale starts now!");

        reports.generateAndSendReport("Q1 Sales", "ceo@company.com");
    }
}
```

---

## DI Comparison Table

```
┌──────────────────────┬────────────────────────┬────────────────────────┐
│                      │    WITHOUT DI          │      WITH DI           │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Swap database        │ Edit every class that  │ Change one line in     │
│                      │ uses the database      │ AppConfig. Done.       │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Test without real DB │ Impossible             │ Inject FakeDatabase    │
│                      │                        │ in setUp()             │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Test without real    │ Emails go to real      │ Inject FakeEmailService │
│ email                │ addresses              │ No emails sent         │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Test payment failure │ Must simulate at       │ FakePaymentGateway     │
│ scenario             │ Stripe API level       │ (false) — instant      │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Coupling             │ Tight — class knows    │ Loose — class knows    │
│                      │ exact implementation   │ only the interface     │
├──────────────────────┼────────────────────────┼────────────────────────┤
│ Adding new payment   │ Edit OrderService      │ New class implements   │
│ gateway              │                        │ PaymentGateway. Done.  │
└──────────────────────┴────────────────────────┴────────────────────────┘
```

---

## Project Structure

```
src/
├── interfaces/
│   ├── Database.java
│   ├── EmailService.java
│   ├── PaymentGateway.java
│   ├── SMSService.java
│   ├── DatabaseInjectable.java
│   └── EmailInjectable.java
│
├── implementations/
│   ├── MySQLDatabase.java
│   ├── PostgreSQLDatabase.java
│   ├── GmailService.java
│   ├── SendGridService.java
│   ├── StripeGateway.java
│   ├── RazorpayGateway.java
│   └── TwilioSMS.java
│
├── fakes/
│   ├── FakeDatabase.java
│   ├── FakeEmailService.java
│   └── FakePaymentGateway.java
│
├── services/
│   ├── OrderService.java
│   ├── NotificationService.java
│   └── ReportService.java
│
├── config/
│   ├── AppConfig.java
│   └── Injector.java
│
└── Main.java

test/
└── OrderServiceTest.java
```

---

## Key Takeaways

```
1. Dependency Injection = receive dependencies, do not create them.

2. Always depend on INTERFACES, not concrete classes.
   private Database db;           ← correct
   private MySQLDatabase db;      ← wrong

3. Constructor injection is preferred.
   Dependencies are explicit, visible, and required.
   The class cannot be created without its dependencies.

4. DI makes your code:
   → Testable  (inject fakes in tests)
   → Flexible  (swap implementations freely)
   → Decoupled (classes do not know about each other)

5. You do NOT need Spring for DI.
   Spring automates it. But you can wire manually.
   Understanding manual DI first makes Spring make sense.

6. The Composition Root:
   One place in your app where ALL wiring happens.
   Usually Main.java or AppConfig.java
   Everything else just receives what it needs.
```

---