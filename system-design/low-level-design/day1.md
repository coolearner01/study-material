# Day 01 — Single Responsibility Principle (SRP)
## 30-Day LLD Twitter Series | @startfin

---

## What is SRP?

**Single Responsibility Principle** is the **S** in SOLID.

> A class should have **one, and only one, reason to change.**

This means every class you write should do **exactly one job**.
Not two. Not five. One.

If your class handles multiple things — calculating, formatting, saving, emailing —
then every time any one of those things changes, you are forced to open and edit that same class.
That is risky. That causes bugs. That breaks things that were working fine.

SRP says: split the jobs. One class, one responsibility.

---

## Why Does This Matter?

Imagine you have a class called `Invoice`.
It calculates the total. It formats the output. It saves to a file. It sends an email.

Now your manager says:
- "Change the email format."
- "Switch from file storage to a database."
- "Add GST to the calculation."
- "Format the output as JSON instead of plain text."

Every single one of these changes forces you to open the SAME class.
Every change risks breaking the OTHER parts of that class.
You touched the email logic — now the calculation is broken.
You touched the formatter — now saving to file fails.

This is the problem SRP solves.

---

## The Bad Code — God Class

```python
# BAD EXAMPLE — Do NOT do this
# This class has 4 responsibilities crammed into one

class Invoice:
    def __init__(self, invoice_id, items):
        self.invoice_id = invoice_id
        self.items = items  # list of dicts: [{"name": "Book", "price": 199}, ...]

    # Responsibility 1: Calculation
    def calculate_total(self):
        total = 0
        for item in self.items:
            total += item["price"]
        return total

    # Responsibility 2: Formatting
    def format_invoice(self):
        lines = [f"Invoice ID: {self.invoice_id}"]
        for item in self.items:
            lines.append(f"  - {item['name']}: ${item['price']}")
        lines.append(f"Total: ${self.calculate_total()}")
        return "\n".join(lines)

    # Responsibility 3: Saving to file
    def save_to_file(self, filepath):
        content = self.format_invoice()
        with open(filepath, "w") as f:
            f.write(content)
        print(f"Invoice saved to {filepath}")

    # Responsibility 4: Sending email
    def send_email(self, recipient_email):
        content = self.format_invoice()
        print(f"Sending email to {recipient_email}...")
        print(f"Content:\n{content}")
        # imagine actual email logic here


# Usage
items = [
    {"name": "Python Book", "price": 299},
    {"name": "Keyboard",    "price": 999},
    {"name": "Mouse",       "price": 499},
]
invoice = Invoice(101, items)
invoice.save_to_file("invoice_101.txt")
invoice.send_email("customer@example.com")
```

### Why is this bad?

```
How many reasons does this class have to change?

1. Calculation logic changes     →  edit Invoice
2. Output format changes         →  edit Invoice
3. Storage changes (file → DB)   →  edit Invoice
4. Email logic changes           →  edit Invoice

Answer: 4 reasons.
SRP rule: maximum 1.

This class is VIOLATING SRP.
```

---

## The Good Code — SRP Applied

We split the God class into **4 focused classes**.
Each one does exactly one thing.

```python
# GOOD EXAMPLE — SRP Applied correctly
# Each class has ONE job and ONE reason to change


# ─────────────────────────────────────────────
# Class 1: Only knows how to CALCULATE
# ─────────────────────────────────────────────
class InvoiceCalculator:
    """
    Responsibility: Calculate the total price of an invoice.
    Reason to change: Only if calculation rules change (e.g., add tax, discount).
    """

    def calculate_total(self, items):
        total = 0
        for item in items:
            total += item["price"]
        return total

    def calculate_tax(self, items, tax_rate=0.18):
        total = self.calculate_total(items)
        return round(total * tax_rate, 2)

    def calculate_grand_total(self, items, tax_rate=0.18):
        total = self.calculate_total(items)
        tax   = self.calculate_tax(items, tax_rate)
        return total + tax


# ─────────────────────────────────────────────
# Class 2: Only knows how to FORMAT
# ─────────────────────────────────────────────
class InvoiceFormatter:
    """
    Responsibility: Format invoice data into a readable string.
    Reason to change: Only if output format changes (e.g., plain text → JSON → HTML).
    """

    def __init__(self):
        self.calculator = InvoiceCalculator()

    def format_as_text(self, invoice_id, items):
        lines = []
        lines.append(f"{'='*40}")
        lines.append(f"  INVOICE #{invoice_id}")
        lines.append(f"{'='*40}")
        for item in items:
            lines.append(f"  {item['name']:<20} ${item['price']:>6}")
        lines.append(f"{'─'*40}")
        total = self.calculator.calculate_total(items)
        tax   = self.calculator.calculate_tax(items)
        grand = self.calculator.calculate_grand_total(items)
        lines.append(f"  {'Subtotal':<20} ${total:>6}")
        lines.append(f"  {'Tax (18%)':<20} ${tax:>6}")
        lines.append(f"  {'Grand Total':<20} ${grand:>6}")
        lines.append(f"{'='*40}")
        return "\n".join(lines)

    def format_as_json(self, invoice_id, items):
        import json
        return json.dumps({
            "invoice_id": invoice_id,
            "items":      items,
            "subtotal":   self.calculator.calculate_total(items),
            "tax":        self.calculator.calculate_tax(items),
            "grand_total":self.calculator.calculate_grand_total(items),
        }, indent=2)


# ─────────────────────────────────────────────
# Class 3: Only knows how to SAVE
# ─────────────────────────────────────────────
class InvoiceSaver:
    """
    Responsibility: Persist the invoice to storage.
    Reason to change: Only if storage mechanism changes (file → database → cloud).
    """

    def __init__(self):
        self.formatter = InvoiceFormatter()

    def save_to_file(self, invoice_id, items, filepath):
        content = self.formatter.format_as_text(invoice_id, items)
        with open(filepath, "w") as f:
            f.write(content)
        print(f"[SAVED] Invoice #{invoice_id} → {filepath}")

    def save_to_json(self, invoice_id, items, filepath):
        content = self.formatter.format_as_json(invoice_id, items)
        with open(filepath, "w") as f:
            f.write(content)
        print(f"[SAVED] Invoice #{invoice_id} as JSON → {filepath}")


# ─────────────────────────────────────────────
# Class 4: Only knows how to NOTIFY
# ─────────────────────────────────────────────
class InvoiceEmailer:
    """
    Responsibility: Send the invoice to the customer.
    Reason to change: Only if notification channel changes (email → SMS → WhatsApp).
    """

    def __init__(self):
        self.formatter = InvoiceFormatter()

    def send_email(self, invoice_id, items, recipient):
        content = self.formatter.format_as_text(invoice_id, items)
        print(f"[EMAIL] Sending to {recipient}...")
        print(content)
        print(f"[EMAIL] Sent successfully.")

    def send_sms(self, invoice_id, items, phone):
        total = InvoiceCalculator().calculate_grand_total(items)
        print(f"[SMS] To {phone}: Invoice #{invoice_id} total is ${total}. Thank you!")


# ─────────────────────────────────────────────
# Usage — clean, clear, separated
# ─────────────────────────────────────────────
if __name__ == "__main__":
    invoice_id = 101
    items = [
        {"name": "Python Book", "price": 299},
        {"name": "Mechanical Keyboard", "price": 999},
        {"name": "Wireless Mouse",      "price": 499},
    ]

    # Calculate
    calc  = InvoiceCalculator()
    print("Subtotal :", calc.calculate_total(items))
    print("Tax      :", calc.calculate_tax(items))
    print("Grand    :", calc.calculate_grand_total(items))

    # Format
    fmt = InvoiceFormatter()
    print(fmt.format_as_text(invoice_id, items))

    # Save
    saver = InvoiceSaver()
    saver.save_to_file(invoice_id, items, "invoice_101.txt")
    saver.save_to_json(invoice_id, items, "invoice_101.json")

    # Notify
    emailer = InvoiceEmailer()
    emailer.send_email(invoice_id, items, "customer@example.com")
    emailer.send_sms(invoice_id, items, "+91-9999999999")
```

---

## Output When You Run This

```
Subtotal : 1797
Tax      : 323.46
Grand    : 2120.46

========================================
  INVOICE #101
========================================
  Python Book          $   299
  Mechanical Keyboard  $   999
  Wireless Mouse       $   499
────────────────────────────────────────
  Subtotal             $  1797
  Tax (18%)            $323.46
  Grand Total          $2120.46
========================================

[SAVED] Invoice #101 → invoice_101.txt
[SAVED] Invoice #101 as JSON → invoice_101.json

[EMAIL] Sending to customer@example.com...
... (full invoice printed)
[EMAIL] Sent successfully.

[SMS] To +91-9999999999: Invoice #101 total is $2120.46. Thank you!
```

---

## Side-by-Side Comparison

```
BEFORE SRP (God Class)               AFTER SRP (4 Classes)
─────────────────────────────────    ─────────────────────────────────
class Invoice:                       class InvoiceCalculator:
    def calculate_total()                def calculate_total()
    def format_invoice()                 def calculate_tax()
    def save_to_file()                   def calculate_grand_total()
    def send_email()
                                     class InvoiceFormatter:
1 class                                  def format_as_text()
4 responsibilities                       def format_as_json()
4 reasons to change
All code in one place            →   class InvoiceSaver:
Changing email breaks saving         def save_to_file()
Changing format breaks tests         def save_to_json()
Hard to test in isolation
Hard to reuse                        class InvoiceEmailer:
                                         def send_email()
                                         def send_sms()

                                     4 classes
                                     1 responsibility each
                                     1 reason to change each
                                     Easy to test in isolation
                                     Easy to reuse independently
```

---

## Unit Tests — How SRP Makes Testing Easy

```python
import unittest

# ── Test InvoiceCalculator in complete isolation ──────────────────────────────
class TestInvoiceCalculator(unittest.TestCase):

    def setUp(self):
        self.calc  = InvoiceCalculator()
        self.items = [
            {"name": "Book",     "price": 200},
            {"name": "Pen",      "price": 50},
            {"name": "Notebook", "price": 100},
        ]

    def test_calculate_total(self):
        self.assertEqual(self.calc.calculate_total(self.items), 350)

    def test_calculate_tax(self):
        self.assertAlmostEqual(self.calc.calculate_tax(self.items, 0.18), 63.0)

    def test_calculate_grand_total(self):
        self.assertAlmostEqual(self.calc.calculate_grand_total(self.items, 0.18), 413.0)

    def test_empty_items(self):
        self.assertEqual(self.calc.calculate_total([]), 0)


# ── Test InvoiceFormatter in complete isolation ───────────────────────────────
class TestInvoiceFormatter(unittest.TestCase):

    def setUp(self):
        self.fmt   = InvoiceFormatter()
        self.items = [{"name": "Book", "price": 200}]

    def test_text_contains_invoice_id(self):
        result = self.fmt.format_as_text(42, self.items)
        self.assertIn("42", result)

    def test_text_contains_item_name(self):
        result = self.fmt.format_as_text(42, self.items)
        self.assertIn("Book", result)

    def test_json_is_valid(self):
        import json
        result = self.fmt.format_as_json(42, self.items)
        parsed = json.loads(result)           # should not raise
        self.assertEqual(parsed["invoice_id"], 42)
        self.assertEqual(parsed["subtotal"], 200)


# ── Run all tests ─────────────────────────────────────────────────────────────
if __name__ == "__main__":
    unittest.main(verbosity=2)

# OUTPUT:
# test_calculate_grand_total ... ok
# test_calculate_tax         ... ok
# test_calculate_total       ... ok
# test_empty_items           ... ok
# test_json_is_valid         ... ok
# test_text_contains_invoice_id ... ok
# test_text_contains_item_name  ... ok
#
# Ran 7 tests in 0.002s
# OK
```

Notice how you can test `InvoiceCalculator` without touching `InvoiceFormatter`.
You can test `InvoiceFormatter` without a real file or email server.
That is only possible because of SRP.

---

## Real-World Analogy

```
Think of a restaurant kitchen.

CHEF         → cooks the food.         One job.
WAITER       → serves the food.        One job.
CASHIER      → handles the billing.    One job.
MANAGER      → coordinates everyone.   One job.

Nobody asks the Chef to also:
  - deliver the food to the table
  - handle the payment
  - clean the dishes

Why? Because if the Chef is doing all of that,
and you hire a new waiter, you still need to
retrain the Chef. Change one thing, everything
is affected.

Same rule in your code.
One class = one chef = one job.
```

---

## Key Takeaways

```
1. SRP = one class, one job, one reason to change.

2. Ask yourself before writing any class:
   "What is the SINGLE thing this class is responsible for?"

3. If you cannot answer that in one sentence without using
   the word "and", your class is doing too much.

4. Signs you are violating SRP:
   - Class has 10+ methods
   - Class name ends in "Manager", "Handler", "Helper", "Utils"
   - You cannot test one feature without setting up five others
   - Changing one feature breaks a completely different feature

5. How to fix SRP violations:
   - Find the different "themes" of methods in your class
   - Each theme = one new class
   - Name each class after what it does, not what it is
```

---

## Twitter Thread — Copy Paste Ready

**Tweet 1:**
```
How many reasons does YOUR class have to change?

If the answer is more than 1 — it is violating SRP.

Here is how to fix it 🧵

Day 1/30 — LLD Series #SRP #SOLID
```

**Tweet 2:**
```
The God Class problem.

One class. 4 jobs.
Calculate → Format → Save → Email

Change the email? Risk breaking the calculator.
Change the format? Risk breaking the saver.

[POST CODE IMAGE — the BAD example above]
```

**Tweet 3:**
```
The SRP fix.

Split into 4 focused classes:

InvoiceCalculator → only math
InvoiceFormatter  → only output
InvoiceSaver      → only storage
InvoiceEmailer    → only delivery

[POST CODE IMAGE — the GOOD example above]
```

**Tweet 4:**
```
Why does this matter?

✅ Each class is testable in isolation
✅ Change email logic → zero risk to calculator
✅ Switch from file to DB → only touch InvoiceSaver
✅ Add JSON output → only touch InvoiceFormatter

One change. One class. Zero surprises.
```

**Tweet 5:**
```
Quick test to check SRP:

"What does this class do?"

If your answer has the word AND in it —
your class is doing too much.

Split it.

Day 1/30 ✅
Tomorrow: Open/Closed Principle (OCP)
Follow for all 30 days 👇

#LLD #SOLID #SRP #CleanCode #SystemDesign
#Python #SoftwareEngineering #100DaysOfCode
```

---

## Checklist Before You Post

- [ ] Read the full code above and understand it
- [ ] Copy the BAD code block → paste to ray.so → screenshot
- [ ] Copy the GOOD code block → paste to ray.so → screenshot
- [ ] Schedule Tweet 1 (hook) for 8 AM or 7 PM your timezone
- [ ] Add all 5 tweets as a thread (reply to yourself)
- [ ] Reply to every comment within first 30 minutes
- [ ] Pin the full 30-day PDF to your profile bio

---

*Day 1 of 30 — @startfin LLD Twitter Series*