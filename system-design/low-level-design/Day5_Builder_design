# Day 06 — Builder Pattern
## 30-Day LLD  Series 
### Language: Java

---

## What is the Builder Pattern?

Imagine you walk into a burger joint. You don't just say "burger" — you say "whole wheat bun, double patty, no onions, extra cheese, chipotle sauce." The kitchen builds your order step by step, and you only get the final burger when you say "done." The Builder Pattern works the same way: it lets you construct a complex object **step by step**, keeping the construction process separate from the final object, so you never end up with a half-built mess.

---

## Why Does This Matter?

**WITHOUT Builder Pattern:**
- Constructors with 8+ parameters become unreadable — `new Order(null, true, false, 3, null, "express", null, 99.0)` tells you nothing
- You can't enforce required vs optional fields at compile time
- Adding a new field forces you to update every constructor call in the codebase
- Objects can exist in an invalid, half-initialised state
- Teams argue about which constructor overload to use

**WITH Builder Pattern:**
- Object construction reads like plain English — `new Order.Builder("ORD-01").quantity(3).express(true).build()`
- Required fields are enforced — `build()` throws if they are missing
- Adding optional fields is a one-line change, zero impact on existing callers
- The final object is always fully valid and immutable
- Construction logic is centralised in one place

---

## Bad Code — The Anti-Pattern

```java
// ─── BAD: Telescoping Constructor Anti-Pattern ───────────────────────────

public class Order {

    // BAD: all fields are public and mutable — anyone can corrupt state after construction
    public String orderId;
    public String customerId;
    public String productName;
    public int quantity;
    public double pricePerUnit;
    public boolean isExpress;
    public boolean isGiftWrapped;
    public String deliveryAddress;
    public String promoCode;

    // BAD: constructor with 9 parameters — caller must remember the exact order
    public Order(String orderId,
                 String customerId,
                 String productName,
                 int quantity,
                 double pricePerUnit,
                 boolean isExpress,
                 boolean isGiftWrapped,
                 String deliveryAddress,
                 String promoCode) {
        this.orderId         = orderId;
        this.customerId      = customerId;
        this.productName     = productName;
        this.quantity        = quantity;
        this.pricePerUnit    = pricePerUnit;
        this.isExpress       = isExpress;
        this.isGiftWrapped   = isGiftWrapped;
        this.deliveryAddress = deliveryAddress;
        this.promoCode       = promoCode;
    }

    // BAD: overloaded "convenience" constructors that silently set defaults
    // — callers have no idea what default values are applied
    public Order(String orderId, String customerId, String productName,
                 int quantity, double pricePerUnit) {
        this(orderId, customerId, productName, quantity, pricePerUnit,
             false, false, null, null); // BAD: null address — valid object? Not really.
    }

    @Override
    public String toString() {
        return "Order{" +
               "orderId='"       + orderId         + "'" +
               ", customer='"    + customerId      + "'" +
               ", product='"     + productName     + "'" +
               ", qty="          + quantity        +
               ", price="        + pricePerUnit    +
               ", express="      + isExpress       +
               ", gift="         + isGiftWrapped   +
               ", address='"     + deliveryAddress + "'" +
               ", promo='"       + promoCode       + "'" +
               "}";
    }
}

class BadMain {
    public static void main(String[] args) {

        // BAD: what does 'true, false' mean here? You must open the class to check.
        Order o1 = new Order("ORD-001", "C-42", "Laptop", 1, 999.99,
                              true, false, "221B Baker St", "SAVE10");

        // BAD: uses the short constructor — delivery address is null, no error thrown
        // This order will fail at the shipping stage — bug found in production, not here
        Order o2 = new Order("ORD-002", "C-99", "Mouse", 2, 29.99);

        System.out.println(o1);
        System.out.println(o2);
    }
}
```

**Output:**
```
Order{orderId='ORD-001', customer='C-42', product='Laptop', qty=1, price=999.99, express=true, gift=false, address='221B Baker St', promo='SAVE10'}
Order{orderId='ORD-002', customer='C-99', product='Mouse', qty=2, price=29.99, express=false, gift=false, address='null', promo='null'}
```

![Bad code — telescoping constructor anti-pattern](../../data/system-desing/lld-series/day6/image1.png)

---

## Good Code — Builder Pattern Applied

### Step 1 — Create the immutable product class with a nested Builder

```java
// ─── Order.java ─────────────────────────────────────────────────────────────

public final class Order {  // final: this object should not be subclassed

    // All fields are private and final — the object is immutable after construction
    private final String orderId;
    private final String customerId;
    private final String productName;
    private final int    quantity;
    private final double pricePerUnit;
    private final boolean isExpress;
    private final boolean isGiftWrapped;
    private final String  deliveryAddress;
    private final String  promoCode;

    // Private constructor — only the inner Builder can call this
    private Order(Builder builder) {
        this.orderId          = builder.orderId;
        this.customerId       = builder.customerId;
        this.productName      = builder.productName;
        this.quantity         = builder.quantity;
        this.pricePerUnit     = builder.pricePerUnit;
        this.isExpress        = builder.isExpress;
        this.isGiftWrapped    = builder.isGiftWrapped;
        this.deliveryAddress  = builder.deliveryAddress;
        this.promoCode        = builder.promoCode;
    }

    // Getters — no setters, object is read-only
    public String  getOrderId()         { return orderId; }
    public String  getCustomerId()      { return customerId; }
    public String  getProductName()     { return productName; }
    public int     getQuantity()        { return quantity; }
    public double  getPricePerUnit()    { return pricePerUnit; }
    public boolean isExpress()          { return isExpress; }
    public boolean isGiftWrapped()      { return isGiftWrapped; }
    public String  getDeliveryAddress() { return deliveryAddress; }
    public String  getPromoCode()       { return promoCode; }
    public double  totalPrice()         { return quantity * pricePerUnit; }

    @Override
    public String toString() {
        return String.format(
            "Order[id=%s | customer=%s | product=%s | qty=%d | " +
            "total=₹%.2f | express=%s | gift=%s | address=%s | promo=%s]",
            orderId, customerId, productName, quantity, totalPrice(),
            isExpress, isGiftWrapped, deliveryAddress, promoCode
        );
    }

    // ── Step 2: Static nested Builder class ─────────────────────────────────
    public static class Builder {

        // Required fields — set in Builder constructor to enforce mandatory data
        private final String orderId;
        private final String customerId;
        private final String productName;
        private final int    quantity;
        private final double pricePerUnit;

        // Optional fields — sensible defaults so callers skip what they don't need
        private boolean isExpress        = false;
        private boolean isGiftWrapped    = false;
        private String  deliveryAddress  = "DEFAULT_WAREHOUSE";
        private String  promoCode        = null;

        // Required fields go in the Builder constructor — impossible to forget them
        public Builder(String orderId, String customerId,
                       String productName, int quantity, double pricePerUnit) {

            // Validation at construction time — fail fast before the object exists
            if (orderId == null || orderId.isBlank())
                throw new IllegalArgumentException("orderId cannot be blank");
            if (quantity <= 0)
                throw new IllegalArgumentException("quantity must be > 0");
            if (pricePerUnit < 0)
                throw new IllegalArgumentException("price cannot be negative");

            this.orderId      = orderId;
            this.customerId   = customerId;
            this.productName  = productName;
            this.quantity     = quantity;
            this.pricePerUnit = pricePerUnit;
        }

        // Each setter returns 'this' — enables method chaining
        public Builder express(boolean express) {
            this.isExpress = express;
            return this;
        }

        public Builder giftWrapped(boolean giftWrapped) {
            this.isGiftWrapped = giftWrapped;
            return this;
        }

        public Builder deliveryAddress(String address) {
            if (address == null || address.isBlank())
                throw new IllegalArgumentException("address cannot be blank");
            this.deliveryAddress = address;
            return this;
        }

        public Builder promoCode(String code) {
            this.promoCode = code;
            return this;
        }

        // build() is the single exit point — the final object is always valid here
        public Order build() {
            return new Order(this);
        }
    }
}
```

### Step 2 — OrderService uses the builder without knowing construction details

```java
// ─── OrderService.java ──────────────────────────────────────────────────────

public class OrderService {

    // Service works with the finished Order — never with raw fields
    public void processOrder(Order order) {
        System.out.println("Processing: " + order);
        System.out.printf("  → Total charged: ₹%.2f%n", order.totalPrice());
        if (order.isExpress()) {
            System.out.println("  → Express lane: dispatching within 2 hours");
        }
        if (order.isGiftWrapped()) {
            System.out.println("  → Gift wrap: added to packing queue");
        }
    }
}
```

### Step 3 — Main shows readable, self-documenting construction

```java
// ─── Main.java ──────────────────────────────────────────────────────────────

public class Main {
    public static void main(String[] args) {

        OrderService service = new OrderService();

        // Full order — every field named, order doesn't matter
        Order laptopOrder = new Order.Builder("ORD-001", "C-42", "Laptop", 1, 999.99)
                .express(true)
                .giftWrapped(false)
                .deliveryAddress("221B Baker St, London")
                .promoCode("SAVE10")
                .build();

        // Minimal order — only required fields, defaults apply automatically
        Order mouseOrder = new Order.Builder("ORD-002", "C-99", "Mouse", 2, 29.99)
                .build();

        // Gift order — mix of optional fields, readable at a glance
        Order giftOrder = new Order.Builder("ORD-003", "C-77", "Headphones", 1, 149.00)
                .giftWrapped(true)
                .deliveryAddress("10 Downing Street, London")
                .build();

        service.processOrder(laptopOrder);
        System.out.println("---");
        service.processOrder(mouseOrder);
        System.out.println("---");
        service.processOrder(giftOrder);

        // Demonstrates fail-fast validation
        System.out.println("\n-- Validation test --");
        try {
            Order bad = new Order.Builder("", "C-00", "Keyboard", 1, 49.99).build();
        } catch (IllegalArgumentException e) {
            System.out.println("Caught expected error: " + e.getMessage());
        }
    }
}
```

![Good code — Builder Pattern applied](../../data/system-desing/lld-series/day6/image2.png)

---

## Output

```
Processing: Order[id=ORD-001 | customer=C-42 | product=Laptop | qty=1 | total=₹999.00 | express=true | gift=false | address=221B Baker St, London | promo=SAVE10]
  → Total charged: ₹999.00
  → Express lane: dispatching within 2 hours
---
Processing: Order[id=ORD-002 | customer=C-99 | product=Mouse | qty=2 | total=₹59.98 | express=false | gift=false | address=DEFAULT_WAREHOUSE | promo=null]
  → Total charged: ₹59.98
---
Processing: Order[id=ORD-003 | customer=C-77 | product=Headphones | qty=1 | total=₹149.00 | express=false | gift=true | address=10 Downing Street, London | promo=null]
  → Total charged: ₹149.00
  → Gift wrap: added to packing queue

-- Validation test --
Caught expected error: orderId cannot be blank
```

---

## Bonus — Director Class + Multi-Variant Builders

Senior developers often pair Builder with a **Director** class. The Director knows *how* to build common configurations, so callers don't repeat the same chained calls across the codebase.

This is how Lombok's `@Builder`, OkHttp's `Request.Builder`, and Retrofit's client all work internally.

```java
// ─── OrderDirector.java ──────────────────────────────────────────────────────

/**
 * Director encapsulates pre-set build "recipes".
 * New recipes are added here — zero impact on Order or its Builder.
 */
public class OrderDirector {

    // Builds a standard, no-frills warehouse order
    public static Order standardOrder(String orderId, String customerId,
                                      String product, int qty, double price) {
        return new Order.Builder(orderId, customerId, product, qty, price)
                .deliveryAddress("Central Warehouse, Mumbai")
                .build();
    }

    // Builds a premium order — express + gift wrap always applied
    public static Order premiumGiftOrder(String orderId, String customerId,
                                          String product, int qty, double price,
                                          String address) {
        return new Order.Builder(orderId, customerId, product, qty, price)
                .express(true)
                .giftWrapped(true)
                .deliveryAddress(address)
                .promoCode("PREMIUM5")
                .build();
    }

    // Builds a bulk B2B order — large qty, no gift wrap, specific address
    public static Order bulkB2BOrder(String orderId, String customerId,
                                      String product, int qty, double price,
                                      String warehouseAddress) {
        return new Order.Builder(orderId, customerId, product, qty, price)
                .deliveryAddress(warehouseAddress)
                .promoCode("B2B-BULK")
                .build();
    }
}

// ─── DirectorMain.java ───────────────────────────────────────────────────────

public class DirectorMain {
    public static void main(String[] args) {
        OrderService service = new OrderService();

        Order std = OrderDirector.standardOrder(
            "ORD-010", "C-11", "Keyboard", 5, 45.00);

        Order gift = OrderDirector.premiumGiftOrder(
            "ORD-011", "C-22", "Watch", 1, 4999.00, "Park Street, Kolkata");

        Order bulk = OrderDirector.bulkB2BOrder(
            "ORD-012", "CORP-01", "Monitor", 50, 299.00, "APMC Warehouse, Navi Mumbai");

        service.processOrder(std);
        System.out.println("---");
        service.processOrder(gift);
        System.out.println("---");
        service.processOrder(bulk);
    }
}
```

**Real-world systems using this pattern:**
- **OkHttp** — `Request.Builder`, `OkHttpClient.Builder`
- **Retrofit** — `Retrofit.Builder`
- **Lombok** — `@Builder` generates the entire inner class at compile time
- **Spring's MockMvcRequestBuilders** — test request construction
- **Elasticsearch Java client** — query DSL is entirely builder-chained

![Director class — pre-set build recipes / architecture flow](../../data/system-desing/lld-series/day6/image3.png)

---

## Unit Tests

```java
// ─── OrderBuilderTest.java ──────────────────────────────────────────────────

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;
import static org.junit.jupiter.api.Assertions.*;

class OrderBuilderTest {

    // ── Happy Path ──────────────────────────────────────────────────────────

    @Test
    @DisplayName("Build order with all fields set — all values stored correctly")
    void testFullOrderBuild() {
        Order order = new Order.Builder("ORD-001", "C-01", "Laptop", 1, 999.99)
                .express(true)
                .giftWrapped(true)
                .deliveryAddress("221B Baker St")
                .promoCode("SAVE10")
                .build();

        assertEquals("ORD-001",     order.getOrderId());
        assertEquals("C-01",        order.getCustomerId());
        assertEquals("Laptop",      order.getProductName());
        assertEquals(1,             order.getQuantity());
        assertEquals(999.99,        order.getPricePerUnit(), 0.001);
        assertTrue(order.isExpress());
        assertTrue(order.isGiftWrapped());
        assertEquals("221B Baker St", order.getDeliveryAddress());
        assertEquals("SAVE10",      order.getPromoCode());
    }

    @Test
    @DisplayName("Build minimal order — optional fields use correct defaults")
    void testMinimalOrderUsesDefaults() {
        Order order = new Order.Builder("ORD-002", "C-02", "Mouse", 2, 29.99)
                .build();

        // Verifies that defaults are applied without the caller setting them
        assertFalse(order.isExpress());
        assertFalse(order.isGiftWrapped());
        assertEquals("DEFAULT_WAREHOUSE", order.getDeliveryAddress());
        assertNull(order.getPromoCode());
    }

    @Test
    @DisplayName("Total price calculation — quantity * pricePerUnit is correct")
    void testTotalPriceCalculation() {
        Order order = new Order.Builder("ORD-003", "C-03", "Pen", 10, 5.00)
                .build();

        // 10 × 5.00 = 50.00
        assertEquals(50.00, order.totalPrice(), 0.001);
    }

    @Test
    @DisplayName("Builder chaining returns same builder instance — fluent API works")
    void testBuilderChainingReturnsSelf() {
        Order.Builder builder = new Order.Builder("ORD-004", "C-04", "Bag", 1, 199.99);

        // Each setter must return the same builder to allow chaining
        assertSame(builder, builder.express(true));
        assertSame(builder, builder.giftWrapped(false));
        assertSame(builder, builder.deliveryAddress("Some Street"));
        assertSame(builder, builder.promoCode("XYZ"));
    }

    // ── Failure / Validation Path ───────────────────────────────────────────

    @Test
    @DisplayName("Blank orderId — builder throws IllegalArgumentException immediately")
    void testBlankOrderIdThrows() {
        // Verifies fail-fast: invalid state is caught before any object is created
        assertThrows(IllegalArgumentException.class, () ->
            new Order.Builder("", "C-05", "Phone", 1, 599.00)
        );
    }

    @Test
    @DisplayName("Zero quantity — builder throws IllegalArgumentException")
    void testZeroQuantityThrows() {
        assertThrows(IllegalArgumentException.class, () ->
            new Order.Builder("ORD-006", "C-06", "Tablet", 0, 299.00)
        );
    }

    @Test
    @DisplayName("Negative price — builder throws IllegalArgumentException")
    void testNegativePriceThrows() {
        assertThrows(IllegalArgumentException.class, () ->
            new Order.Builder("ORD-007", "C-07", "Case", 1, -10.00)
        );
    }

    @Test
    @DisplayName("Blank delivery address — builder throws IllegalArgumentException")
    void testBlankAddressThrows() {
        assertThrows(IllegalArgumentException.class, () ->
            new Order.Builder("ORD-008", "C-08", "Cable", 3, 9.99)
                    .deliveryAddress("   ")
                    .build()
        );
    }

    // ── Edge Cases ──────────────────────────────────────────────────────────

    @Test
    @DisplayName("Same builder can NOT be reused — each build() call creates a new Order")
    void testBuilderProducesIndependentObjects() {
        Order.Builder builder = new Order.Builder("ORD-009", "C-09", "Charger", 1, 19.99);

        Order o1 = builder.express(true).build();
        Order o2 = builder.express(false).build();

        // Two builds from the same builder must be distinct objects
        assertNotSame(o1, o2);
        assertTrue(o1.isExpress());
        assertFalse(o2.isExpress());
    }

    @Test
    @DisplayName("Director standardOrder — address set to Central Warehouse correctly")
    void testDirectorStandardOrder() {
        Order order = OrderDirector.standardOrder("ORD-010", "C-10", "Desk", 1, 899.00);

        assertEquals("Central Warehouse, Mumbai", order.getDeliveryAddress());
        assertFalse(order.isExpress());
        assertFalse(order.isGiftWrapped());
    }

    @Test
    @DisplayName("Director premiumGiftOrder — express and gift wrap always true")
    void testDirectorPremiumGiftOrder() {
        Order order = OrderDirector.premiumGiftOrder(
            "ORD-011", "C-11", "Watch", 1, 4999.00, "Park St, Kolkata");

        assertTrue(order.isExpress());
        assertTrue(order.isGiftWrapped());
        assertEquals("PREMIUM5", order.getPromoCode());
    }
}
```

![Unit tests — happy path, validation, and edge cases](../../data/system-desing/lld-series/day6/image4.png)

---

## Side-by-Side Comparison

```
╔══════════════════════╦══════════════════════════════╦══════════════════════════════════╗
║ Dimension            ║ WITHOUT Builder (Bad)         ║ WITH Builder (Good)              ║
╠══════════════════════╬══════════════════════════════╬══════════════════════════════════╣
║ Coupling             ║ Caller must know every field  ║ Caller sets only what it needs   ║
║                      ║ and their exact order         ║ by name — order doesn't matter   ║
╠══════════════════════╬══════════════════════════════╬══════════════════════════════════╣
║ Testability          ║ Hard — all fields must be set ║ Easy — build minimal valid       ║
║                      ║ even in tests for irrelevant  ║ objects; set only tested fields  ║
║                      ║ scenarios                     ║                                  ║
╠══════════════════════╬══════════════════════════════╬══════════════════════════════════╣
║ Flexibility          ║ New optional field = change   ║ New optional field = one method  ║
║                      ║ every constructor call        ║ in Builder, zero caller changes  ║
╠══════════════════════╬══════════════════════════════╬══════════════════════════════════╣
║ Runtime behaviour    ║ Null fields discovered at     ║ Invalid state rejected at        ║
║                      ║ runtime, in production        ║ build() call — fail fast         ║
╠══════════════════════╬══════════════════════════════╬══════════════════════════════════╣
║ Adding new variants  ║ New constructor overload for  ║ New Director method or optional  ║
║                      ║ every combination; combinat-  ║ Builder setter — one change,     ║
║                      ║ orial explosion               ║ zero ripple effect               ║
╚══════════════════════╩══════════════════════════════╩══════════════════════════════════╝
```

<!-- IMAGE 3: This comparison table becomes the architecture / comparison slide -->

---

## When Should You Use This?

**Use Builder when:**

1. Your object has **4 or more fields** and at least some of them are optional.
2. You need to **enforce required fields** without relying on runtime null checks.
3. The object must be **immutable** after creation (thread safety, defensive design).
4. Multiple **configurations of the same object type** are created in different places.
5. You want **readable, self-documenting** construction calls that new team members understand without reading the class.
6. You're building a **DSL-style API** (query builders, HTTP clients, test fixtures).

**Do NOT use Builder when:**

- Your object has **2-3 fields and all are required** — a normal constructor is cleaner.
- The object is a **simple data transfer object (DTO)** with no validation logic — use a record or a plain POJO.
- You're inside a **tight loop creating millions of objects per second** — the extra object allocation (Builder instance) adds GC pressure; use a pool or factory instead.

---

## Project Structure

```
src/
└── main/
    └── java/
        └── com/startfin/lld/day06/
            ├── model/
            │   └── Order.java              # Immutable product + nested Builder
            ├── service/
            │   └── OrderService.java       # Consumes Order — no construction logic
            ├── director/
            │   └── OrderDirector.java      # Pre-set build recipes
            └── Main.java                   # Demo entry point + DirectorMain

src/
└── test/
    └── java/
        └── com/startfin/lld/day06/
            └── OrderBuilderTest.java       # JUnit 5 — happy, failure, edge cases
```

---

## Key Takeaways

1. Builder separates **how an object is assembled** from **what the object is** — two reasons to change that should never share a class.
2. Required fields belong in the **Builder constructor**; optional fields belong as **setter methods** — this enforces the contract at compile time.
3. Every Builder setter **returns `this`** — that one decision is what makes fluent method chaining possible.
4. The **Director class** is Builder's power-up: it centralises common configurations so teams don't copy-paste the same chain everywhere.
5. A Builder-built object should always be **immutable** — if callers can mutate it after `build()`, you have lost the safety the pattern provides.
6. When your constructor has **boolean parameters next to each other** (`new X(true, false, true)`), that is the clearest signal you need a Builder.
---