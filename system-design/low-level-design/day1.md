# Day 01 — All 5 SOLID Principles
## 30-Day LLD Series 

---

## What is SOLID?

SOLID is a set of **5 design principles** that make your code:
- Easier to understand
- Easier to change
- Easier to test
- Easier to extend without breaking things

Every senior developer knows SOLID.
Every good codebase follows SOLID.
This is where clean code begins.

```
S — Single Responsibility Principle  (SRP)
O — Open / Closed Principle          (OCP)
L — Liskov Substitution Principle    (LSP)
I — Interface Segregation Principle  (ISP)
D — Dependency Inversion Principle   (DIP)
```

Learn all 5 today. One by one. With full code.

---
---

# S — Single Responsibility Principle (SRP)

## What is it?

> A class should have **one, and only one, reason to change.**

Every class must do exactly **one job**.
Not two. Not five. One.

If your class calculates, formats, saves, AND emails —
that is 4 reasons to change. That is 4 ways to break.

## Bad Code

```python
# BAD — God class doing everything
class Invoice:
    def __init__(self, invoice_id, items):
        self.invoice_id = invoice_id
        self.items = items

    # job 1: calculate
    def calculate_total(self):
        return sum(item["price"] for item in self.items)

    # job 2: format
    def format_invoice(self):
        lines = [f"Invoice #{self.invoice_id}"]
        for item in self.items:
            lines.append(f"  {item['name']}: ${item['price']}")
        lines.append(f"Total: ${self.calculate_total()}")
        return "\n".join(lines)

    # job 3: save
    def save_to_file(self, path):
        with open(path, "w") as f:
            f.write(self.format_invoice())

    # job 4: email
    def send_email(self, recipient):
        print(f"Sending to {recipient}:\n{self.format_invoice()}")

# Problems:
# Change email logic  → edit Invoice → risk breaking calculate
# Change format       → edit Invoice → risk breaking save
# Want to test only calculate? You still need format + save + email to exist
```

## Good Code

```python
# GOOD — One class, one job

class InvoiceCalculator:
    """Only job: calculate money."""

    def calculate_total(self, items):
        return sum(item["price"] for item in items)

    def calculate_tax(self, items, rate=0.18):
        return round(self.calculate_total(items) * rate, 2)

    def calculate_grand_total(self, items, rate=0.18):
        return self.calculate_total(items) + self.calculate_tax(items, rate)


class InvoiceFormatter:
    """Only job: format the invoice into text."""

    def __init__(self):
        self.calc = InvoiceCalculator()

    def format_as_text(self, invoice_id, items):
        lines = [f"Invoice #{invoice_id}"]
        for item in items:
            lines.append(f"  {item['name']}: ${item['price']}")
        lines.append(f"Total: ${self.calc.calculate_grand_total(items)}")
        return "\n".join(lines)


class InvoiceSaver:
    """Only job: save the invoice to disk."""

    def __init__(self):
        self.formatter = InvoiceFormatter()

    def save(self, invoice_id, items, path):
        content = self.formatter.format_as_text(invoice_id, items)
        with open(path, "w") as f:
            f.write(content)
        print(f"Saved to {path}")


class InvoiceEmailer:
    """Only job: send the invoice by email."""

    def __init__(self):
        self.formatter = InvoiceFormatter()

    def send(self, invoice_id, items, recipient):
        content = self.formatter.format_as_text(invoice_id, items)
        print(f"Emailing {recipient}:\n{content}")


# Usage
items = [
    {"name": "Python Book", "price": 299},
    {"name": "Keyboard",    "price": 999},
]

calc    = InvoiceCalculator()
saver   = InvoiceSaver()
emailer = InvoiceEmailer()

print("Total:", calc.calculate_grand_total(items))
saver.save(101, items, "invoice.txt")
emailer.send(101, items, "user@example.com")
```

## Output

```
Total: 1535.64
Saved to invoice.txt
Emailing user@example.com:
Invoice #101
  Python Book: $299
  Keyboard: $999
Total: $1535.64
```

## Key Rule

```
Ask yourself: "What does this class do?"

If your answer has the word AND in it —
your class is doing too much. Split it.

Bad:  "Invoice calculates AND formats AND saves AND emails."
Good: "InvoiceCalculator calculates the total."
```

---
---

# O — Open / Closed Principle (OCP)

## What is it?

> A class should be **open for extension** but **closed for modification.**

You should be able to **add new behavior** without editing existing, working code.

When you add a new feature, you write a new class.
You do NOT go back and change old classes.

## Bad Code

```python
# BAD — every new discount type requires editing this function

class DiscountService:
    def apply_discount(self, price, discount_type):
        if discount_type == "seasonal":
            return price * 0.90          # 10% off
        elif discount_type == "loyalty":
            return price * 0.85          # 15% off
        elif discount_type == "student":
            return price * 0.80          # 20% off
        # Problem: adding "new_user" discount means
        # opening this file and editing working code.
        # Every edit risks breaking existing discounts.
        else:
            return price

# To add a new discount:
# 1. Open this file
# 2. Add a new elif
# 3. Hope you don't break the others
# This violates OCP.
```

## Good Code

```python
# GOOD — add new discounts without touching old code

from abc import ABC, abstractmethod

class Discount(ABC):
    """Base interface. Every discount must implement apply()."""

    @abstractmethod
    def apply(self, price: float) -> float:
        ...

    def describe(self) -> str:
        return f"{self.__class__.__name__}"


class SeasonalDiscount(Discount):
    """10% off during seasonal sale."""
    def apply(self, price):
        return price * 0.90


class LoyaltyDiscount(Discount):
    """15% off for loyal customers."""
    def apply(self, price):
        return price * 0.85


class StudentDiscount(Discount):
    """20% off for students."""
    def apply(self, price):
        return price * 0.80


class NewUserDiscount(Discount):
    """25% off for first-time users."""
    def apply(self, price):
        return price * 0.75


class NoDiscount(Discount):
    """No discount applied."""
    def apply(self, price):
        return price


# The checkout function never changes.
# It works with ANY discount past, present, or future.
def checkout(price: float, discount: Discount) -> float:
    final = discount.apply(price)
    print(f"{discount.describe()}: ${price} → ${final:.2f}")
    return final


# Usage
base_price = 1000.0

checkout(base_price, SeasonalDiscount())    # existing
checkout(base_price, LoyaltyDiscount())     # existing
checkout(base_price, StudentDiscount())     # existing
checkout(base_price, NewUserDiscount())     # NEW — zero old code changed
checkout(base_price, NoDiscount())          # default
```

## Output

```
SeasonalDiscount: $1000.0 → $900.00
LoyaltyDiscount:  $1000.0 → $850.00
StudentDiscount:  $1000.0 → $800.00
NewUserDiscount:  $1000.0 → $750.00
NoDiscount:       $1000.0 → $1000.00
```

## Key Rule

```
Every time you write a new elif to handle a new case —
ask yourself: "Can I make this a new class instead?"

If yes → make a new class. That is OCP.

Old code stays closed.   (no edits, no risk)
New code stays open.     (add freely, safely)
```

---
---

# L — Liskov Substitution Principle (LSP)

## What is it?

> If S is a subtype of T, then objects of type T may be **replaced with objects of type S** without breaking the program.

In plain English:
**A child class must be usable anywhere the parent class is used, with no surprises.**

If you swap a subclass in and something breaks —
you are violating LSP.

## Bad Code

```python
# BAD — Penguin breaks the Bird contract

class Bird:
    def fly(self):
        return "I am flying!"

    def describe(self):
        return f"I am a {self.__class__.__name__} and I {self.fly()}"


class Eagle(Bird):
    def fly(self):
        return "soaring at 3000 feet"      # fine, Eagle can fly


class Penguin(Bird):
    def fly(self):
        # PROBLEM: Penguin cannot fly.
        # But it is forced to implement fly() because it inherits from Bird.
        raise NotImplementedError("Penguins cannot fly!")


def make_bird_fly(bird: Bird):
    print(bird.describe())      # works for Eagle
                                # CRASHES for Penguin


make_bird_fly(Eagle())          # OK
make_bird_fly(Penguin())        # RuntimeError — LSP violated
```

## Good Code

```python
# GOOD — separate capabilities, no broken substitution

from abc import ABC, abstractmethod


class Bird(ABC):
    """Base class — only what ALL birds share."""

    @abstractmethod
    def eat(self):
        ...

    @abstractmethod
    def breathe(self):
        ...

    def describe(self):
        return f"I am a {self.__class__.__name__}"


class FlyingBird(Bird):
    """Mixin for birds that can fly."""

    @abstractmethod
    def fly(self):
        ...


class SwimmingBird(Bird):
    """Mixin for birds that can swim."""

    @abstractmethod
    def swim(self):
        ...


class Eagle(FlyingBird):
    def eat(self):       return "Eagle eats fish"
    def breathe(self):   return "Eagle breathes air"
    def fly(self):       return "Eagle soars at 3000 ft"


class Duck(FlyingBird, SwimmingBird):
    def eat(self):       return "Duck eats seeds"
    def breathe(self):   return "Duck breathes air"
    def fly(self):       return "Duck flies low"
    def swim(self):      return "Duck paddles across pond"


class Penguin(SwimmingBird):
    def eat(self):       return "Penguin eats krill"
    def breathe(self):   return "Penguin breathes air"
    def swim(self):      return "Penguin dives at 25 mph"
    # No fly() — Penguin is honest about its capabilities


def let_it_fly(bird: FlyingBird):
    print(bird.fly())

def let_it_swim(bird: SwimmingBird):
    print(bird.swim())


eagle   = Eagle()
duck    = Duck()
penguin = Penguin()

let_it_fly(eagle)       # Eagle soars at 3000 ft
let_it_fly(duck)        # Duck flies low
# let_it_fly(penguin)   # Type error at compile time — caught early, never crashes

let_it_swim(duck)       # Duck paddles across pond
let_it_swim(penguin)    # Penguin dives at 25 mph
```

## Output

```
Eagle soars at 3000 ft
Duck flies low
Duck paddles across pond
Penguin dives at 25 mph
```

## Key Rule

```
Before making class B extend class A, ask:
"Can B do EVERYTHING A promises, without surprise?"

If NO  → do not inherit. Use composition or separate interfaces.
If YES → inheritance is fine.

The child must NEVER do less than the parent promised.
```

---
---

# I — Interface Segregation Principle (ISP)

## What is it?

> **Clients should not be forced to depend on interfaces they do not use.**

Do not create one big fat interface that forces every class to implement things it does not need.

Split big interfaces into small, focused ones.
A class only implements what is relevant to it.

## Bad Code

```python
# BAD — one fat interface forces Robot to implement eat() and sleep()

from abc import ABC, abstractmethod

class Worker(ABC):
    @abstractmethod
    def work(self):   ...

    @abstractmethod
    def eat(self):    ...      # robots don't eat

    @abstractmethod
    def sleep(self):  ...      # robots don't sleep

    @abstractmethod
    def charge(self): ...      # humans don't charge


class HumanWorker(Worker):
    def work(self):   print("Human is working")
    def eat(self):    print("Human is eating")
    def sleep(self):  print("Human is sleeping")
    def charge(self): pass     # forced to implement — does nothing, misleading


class RobotWorker(Worker):
    def work(self):   print("Robot is working")
    def eat(self):    raise NotImplementedError("Robots don't eat")   # lie
    def sleep(self):  raise NotImplementedError("Robots don't sleep") # lie
    def charge(self): print("Robot is charging")

# Robot is FORCED to implement eat() and sleep() even though it never uses them.
# This is ISP violation.
```

## Good Code

```python
# GOOD — small, focused interfaces. Each class picks only what it needs.

from abc import ABC, abstractmethod


class Workable(ABC):
    @abstractmethod
    def work(self): ...


class Eatable(ABC):
    @abstractmethod
    def eat(self): ...


class Sleepable(ABC):
    @abstractmethod
    def sleep(self): ...


class Chargeable(ABC):
    @abstractmethod
    def charge(self): ...


# Human needs: work + eat + sleep
class HumanWorker(Workable, Eatable, Sleepable):
    def work(self):
        print("Human is working hard")

    def eat(self):
        print("Human is eating lunch")

    def sleep(self):
        print("Human is sleeping 8 hours")


# Robot needs: work + charge
class RobotWorker(Workable, Chargeable):
    def work(self):
        print("Robot is working 24/7")

    def charge(self):
        print("Robot is charging battery")


# Android (future) needs: work + eat (simulated) + charge
class AndroidWorker(Workable, Eatable, Chargeable):
    def work(self):
        print("Android is working")

    def eat(self):
        print("Android simulates eating for social interaction")

    def charge(self):
        print("Android is recharging")


# Usage
def start_shift(worker: Workable):
    worker.work()

def lunch_break(worker: Eatable):
    worker.eat()

def end_of_day(worker: Sleepable):
    worker.sleep()

def recharge_station(worker: Chargeable):
    worker.charge()


human   = HumanWorker()
robot   = RobotWorker()
android = AndroidWorker()

start_shift(human)    # Human is working hard
start_shift(robot)    # Robot is working 24/7
start_shift(android)  # Android is working

lunch_break(human)    # Human is eating lunch
lunch_break(android)  # Android simulates eating
# lunch_break(robot)  # Type error — Robot has no eat(). Caught early.

recharge_station(robot)    # Robot is charging battery
recharge_station(android)  # Android is recharging
end_of_day(human)          # Human is sleeping 8 hours
```

## Output

```
Human is working hard
Robot is working 24/7
Android is working
Human is eating lunch
Android simulates eating for social interaction
Robot is charging battery
Android is recharging
Human is sleeping 8 hours
```

## Key Rule

```
Ever write this?

    def eat(self):
        raise NotImplementedError("Robots don't eat")

That is ISP being violated.

The class is being FORCED to implement something it does not need.

Fix: split the interface. Each class only signs up for what it actually does.
```

---
---

# D — Dependency Inversion Principle (DIP)

## What is it?

> 1. High-level modules should not depend on low-level modules. Both should depend on **abstractions**.
> 2. Abstractions should not depend on details. Details should depend on abstractions.

In plain English:
**Your business logic should not care about HOW things are done.**
It should only care about WHAT needs to be done — via an interface.

This lets you swap databases, email providers, payment gateways —
without touching a single line of your core business logic.

## Bad Code

```python
# BAD — OrderService is hardwired to specific implementations

class MySQLDatabase:
    def save(self, order):
        print(f"[MySQL] Saving order {order['id']} to MySQL database")

class GmailEmailService:
    def send(self, to, message):
        print(f"[Gmail] Sending email to {to}: {message}")

class OrderService:
    def __init__(self):
        # PROBLEM: hardcoded to MySQL and Gmail
        # Want to switch to PostgreSQL? Edit OrderService.
        # Want to switch to SendGrid? Edit OrderService.
        # Want to test without real DB and email? You can't.
        self.db    = MySQLDatabase()
        self.email = GmailEmailService()

    def place_order(self, order):
        self.db.save(order)
        self.email.send(order["customer_email"], f"Order {order['id']} confirmed!")
        print(f"Order {order['id']} placed.")
```

## Good Code

```python
# GOOD — DIP applied. OrderService depends on abstractions, not concretions.

from abc import ABC, abstractmethod


# ── Abstractions (interfaces) ──────────────────────────────────────────────
class Database(ABC):
    """What a database must be able to do."""

    @abstractmethod
    def save(self, data: dict) -> None: ...

    @abstractmethod
    def find(self, record_id: int) -> dict: ...


class EmailService(ABC):
    """What an email service must be able to do."""

    @abstractmethod
    def send(self, to: str, subject: str, body: str) -> None: ...


class PaymentGateway(ABC):
    """What a payment gateway must be able to do."""

    @abstractmethod
    def charge(self, amount: float, card_token: str) -> bool: ...


# ── Low-level implementations ──────────────────────────────────────────────
class MySQLDatabase(Database):
    def save(self, data):
        print(f"[MySQL] Saved record: {data}")

    def find(self, record_id):
        print(f"[MySQL] Found record #{record_id}")
        return {"id": record_id}


class PostgreSQLDatabase(Database):
    def save(self, data):
        print(f"[PostgreSQL] Saved record: {data}")

    def find(self, record_id):
        print(f"[PostgreSQL] Found record #{record_id}")
        return {"id": record_id}


class InMemoryDatabase(Database):
    """Used in unit tests — no real database needed."""

    def __init__(self):
        self._store = {}

    def save(self, data):
        self._store[data["id"]] = data
        print(f"[InMemory] Saved: {data}")

    def find(self, record_id):
        return self._store.get(record_id, {})


class GmailService(EmailService):
    def send(self, to, subject, body):
        print(f"[Gmail] To: {to} | Subject: {subject} | Body: {body}")


class SendGridService(EmailService):
    def send(self, to, subject, body):
        print(f"[SendGrid] To: {to} | Subject: {subject} | Body: {body}")


class FakeEmailService(EmailService):
    """Used in unit tests — no real emails sent."""

    def __init__(self):
        self.sent = []

    def send(self, to, subject, body):
        self.sent.append({"to": to, "subject": subject, "body": body})
        print(f"[FakeEmail] Recorded email to {to}")


class StripeGateway(PaymentGateway):
    def charge(self, amount, card_token):
        print(f"[Stripe] Charged ${amount} on card {card_token}")
        return True


class FakePaymentGateway(PaymentGateway):
    """Used in unit tests — no real charges."""

    def charge(self, amount, card_token):
        print(f"[FakePayment] Simulated charge of ${amount}")
        return True


# ── High-level business logic ──────────────────────────────────────────────
class OrderService:
    """
    Core business logic.
    Does NOT know about MySQL, Gmail, or Stripe.
    Only knows about Database, EmailService, PaymentGateway interfaces.
    """

    def __init__(
        self,
        db:      Database,
        emailer: EmailService,
        payment: PaymentGateway,
    ):
        self._db      = db
        self._emailer = emailer
        self._payment = payment

    def place_order(self, order: dict) -> bool:
        # 1. Charge customer
        charged = self._payment.charge(
            amount=order["total"],
            card_token=order["card_token"],
        )
        if not charged:
            print("Payment failed. Order not placed.")
            return False

        # 2. Save order
        self._db.save({"id": order["id"], "items": order["items"], "total": order["total"]})

        # 3. Notify customer
        self._emailer.send(
            to=order["customer_email"],
            subject=f"Order #{order['id']} Confirmed",
            body=f"Thank you! Your order total is ${order['total']}.",
        )

        print(f"Order #{order['id']} placed successfully.")
        return True


# ── Production wiring ──────────────────────────────────────────────────────
print("=== PRODUCTION ===")
prod_service = OrderService(
    db=MySQLDatabase(),
    emailer=GmailService(),
    payment=StripeGateway(),
)
prod_service.place_order({
    "id": 1001,
    "items": ["Python Book", "Keyboard"],
    "total": 1298,
    "card_token": "tok_visa_xxxx",
    "customer_email": "alice@example.com",
})

# ── Swap to PostgreSQL + SendGrid — zero changes to OrderService ───────────
print("\n=== PRODUCTION (Different Stack) ===")
alt_service = OrderService(
    db=PostgreSQLDatabase(),
    emailer=SendGridService(),
    payment=StripeGateway(),
)
alt_service.place_order({
    "id": 1002,
    "items": ["Mouse"],
    "total": 499,
    "card_token": "tok_visa_yyyy",
    "customer_email": "bob@example.com",
})

# ── Testing — no real DB, email, or payment needed ─────────────────────────
print("\n=== UNIT TEST (Fake dependencies) ===")
fake_email   = FakeEmailService()
test_service = OrderService(
    db=InMemoryDatabase(),
    emailer=fake_email,
    payment=FakePaymentGateway(),
)
test_service.place_order({
    "id": 9999,
    "items": ["Test Item"],
    "total": 100,
    "card_token": "tok_test",
    "customer_email": "test@example.com",
})
print("Emails captured in test:", fake_email.sent)
```

## Output

```
=== PRODUCTION ===
[Stripe]     Charged $1298 on card tok_visa_xxxx
[MySQL]      Saved record: {'id': 1001, 'items': ['Python Book', 'Keyboard'], 'total': 1298}
[Gmail]      To: alice@example.com | Subject: Order #1001 Confirmed | Body: Thank you! ...
Order #1001 placed successfully.

=== PRODUCTION (Different Stack) ===
[Stripe]     Charged $499 on card tok_visa_yyyy
[PostgreSQL] Saved record: {'id': 1002, 'items': ['Mouse'], 'total': 499}
[SendGrid]   To: bob@example.com | Subject: Order #1002 Confirmed | Body: Thank you! ...
Order #1002 placed successfully.

=== UNIT TEST (Fake dependencies) ===
[FakePayment] Simulated charge of $100
[InMemory]    Saved: {'id': 9999, 'items': ['Test Item'], 'total': 100}
[FakeEmail]   Recorded email to test@example.com
Order #9999 placed successfully.
Emails captured in test: [{'to': 'test@example.com', 'subject': 'Order #9999 Confirmed', ...}]
```

## Key Rule

```
High-level code (business logic) should never say:
    self.db    = MySQLDatabase()      ← hardcoded
    self.email = GmailService()       ← hardcoded

It should always say:
    def __init__(self, db: Database, email: EmailService):
        self.db    = db               ← injected
        self.email = email            ← injected

This is called Dependency Injection.
DI is HOW you implement DIP.
```

---
---

## All 5 Principles — Quick Reference

```
┌─────┬──────────────────────────────────────────────────────────────────┐
│     │                                                                  │
│  S  │  SRP — One class, one job, one reason to change.                │
│     │                                                                  │
│  O  │  OCP — Add new behavior via new classes. Never edit old code.   │
│     │                                                                  │
│  L  │  LSP — Subclass must fully honor the parent's contract.         │
│     │                                                                  │
│  I  │  ISP — Small focused interfaces. No class forced to implement   │
│     │        things it does not use.                                   │
│     │                                                                  │
│  D  │  DIP — Depend on interfaces, not concrete classes.              │
│     │        Inject dependencies from outside.                         │
│     │                                                                  │
└─────┴──────────────────────────────────────────────────────────────────┘
```

---

## The One-Sentence Test for Each Principle

```
SRP → "What does this class do?"
      If the answer has AND in it — split it.

OCP → "Do I need to edit old working code to add this feature?"
      If YES — design it with an interface instead.

LSP → "If I replace the parent with this child, will anything break?"
      If YES — rethink your inheritance.

ISP → "Is this class forced to implement a method it does not use?"
      If YES — split the interface.

DIP → "Does my business logic name a specific database or email service?"
      If YES — introduce an interface and inject it.
```
---
