# Day 02 — DRY, KISS, YAGNI
## 30-Day LLD Series

---

## What Are These Principles?

Day 2 covers **3 core design principles** that every developer must follow.

```
DRY   — Don't Repeat Yourself
KISS  — Keep It Simple, Stupid
YAGNI — You Aren't Gonna Need It
```

These are not patterns. These are not frameworks.
These are **rules of thinking** that separate junior code from senior code.

Learn them today. Apply them forever.

---
---

# DRY — Don't Repeat Yourself

## What is it?

> Every piece of knowledge must have a **single, unambiguous, authoritative representation** in a system.

In plain English:
**If you wrote the same logic in two places — you have a bug waiting to happen.**

When the rule changes, you update one place.
If you have copies, you update some places and forget others.
That is how bugs are born.

---

## Bad Code — Repeated Logic Everywhere

```python
# BAD — same logic copy-pasted across multiple functions
# When the password rule changes, you must find and fix ALL copies.

class UserService:

    def register(self, username, email, password):
        # validation duplicated here
        if len(username) < 3:
            raise ValueError("Username must be at least 3 characters")
        if len(username) > 20:
            raise ValueError("Username must be at most 20 characters")
        if "@" not in email:
            raise ValueError("Email must contain @")
        if "." not in email.split("@")[-1]:
            raise ValueError("Email domain must contain a dot")
        if len(password) < 8:
            raise ValueError("Password must be at least 8 characters")
        if not any(c.isupper() for c in password):
            raise ValueError("Password must contain an uppercase letter")
        if not any(c.isdigit() for c in password):
            raise ValueError("Password must contain a digit")

        print(f"Registering user: {username}")


    def update_profile(self, username, email):
        # username and email validation DUPLICATED again
        if len(username) < 3:
            raise ValueError("Username must be at least 3 characters")
        if len(username) > 20:
            raise ValueError("Username must be at most 20 characters")
        if "@" not in email:
            raise ValueError("Email must contain @")
        if "." not in email.split("@")[-1]:
            raise ValueError("Email domain must contain a dot")

        print(f"Updating profile for: {username}")


    def change_password(self, new_password, confirm_password):
        # password validation DUPLICATED again
        if len(new_password) < 8:
            raise ValueError("Password must be at least 8 characters")
        if not any(c.isupper() for c in new_password):
            raise ValueError("Password must contain an uppercase letter")
        if not any(c.isdigit() for c in new_password):
            raise ValueError("Password must contain a digit")
        if new_password != confirm_password:
            raise ValueError("Passwords do not match")

        print("Password changed successfully")


    def reset_password(self, token, new_password, confirm_password):
        # password validation DUPLICATED yet again
        if len(new_password) < 8:
            raise ValueError("Password must be at least 8 characters")
        if not any(c.isupper() for c in new_password):
            raise ValueError("Password must contain an uppercase letter")
        if not any(c.isdigit() for c in new_password):
            raise ValueError("Password must contain a digit")
        if new_password != confirm_password:
            raise ValueError("Passwords do not match")

        print(f"Password reset with token: {token}")


# The problem:
# Manager says: "Add special character requirement to passwords."
# You now have to find and update 3 separate places.
# Miss one → inconsistent validation → security bug.
```

---

## Good Code — DRY Applied

```python
# GOOD — single source of truth for every rule

import re


# ── Validators — each rule lives in exactly ONE place ─────────────────────

def validate_username(username: str) -> None:
    """
    Single source of truth for username rules.
    Change the rule here once. It applies everywhere automatically.
    """
    if not isinstance(username, str):
        raise ValueError("Username must be a string")
    if len(username) < 3:
        raise ValueError("Username must be at least 3 characters")
    if len(username) > 20:
        raise ValueError("Username must be at most 20 characters")
    if not username.isalnum():
        raise ValueError("Username must contain only letters and numbers")


def validate_email(email: str) -> None:
    """
    Single source of truth for email rules.
    """
    if not isinstance(email, str):
        raise ValueError("Email must be a string")
    pattern = r"^[\w\.-]+@[\w\.-]+\.\w{2,}$"
    if not re.match(pattern, email):
        raise ValueError(f"Invalid email format: {email}")


def validate_password(password: str) -> None:
    """
    Single source of truth for password rules.
    Manager says add special character? Edit HERE. Nowhere else.
    """
    if len(password) < 8:
        raise ValueError("Password must be at least 8 characters")
    if not any(c.isupper() for c in password):
        raise ValueError("Password must contain at least one uppercase letter")
    if not any(c.islower() for c in password):
        raise ValueError("Password must contain at least one lowercase letter")
    if not any(c.isdigit() for c in password):
        raise ValueError("Password must contain at least one digit")
    if not any(c in "!@#$%^&*()_+-=[]{}|;':\",./<>?" for c in password):
        raise ValueError("Password must contain at least one special character")


def validate_passwords_match(password: str, confirm: str) -> None:
    if password != confirm:
        raise ValueError("Passwords do not match")


# ── Service — calls validators, never repeats rules ───────────────────────

class UserService:

    def register(self, username: str, email: str, password: str) -> None:
        validate_username(username)
        validate_email(email)
        validate_password(password)
        # All rules enforced. Zero duplication.
        print(f"[REGISTER] User '{username}' created successfully.")


    def update_profile(self, username: str, email: str) -> None:
        validate_username(username)
        validate_email(email)
        print(f"[UPDATE] Profile for '{username}' updated.")


    def change_password(self, new_password: str, confirm: str) -> None:
        validate_password(new_password)
        validate_passwords_match(new_password, confirm)
        print("[CHANGE] Password changed successfully.")


    def reset_password(self, token: str, new_password: str, confirm: str) -> None:
        if not token:
            raise ValueError("Reset token is required")
        validate_password(new_password)
        validate_passwords_match(new_password, confirm)
        print(f"[RESET] Password reset using token '{token}'.")


# ── Usage ──────────────────────────────────────────────────────────────────

service = UserService()

# Valid operations
service.register("alice123", "alice@example.com", "Secret@99")
service.update_profile("bob456", "bob@example.com")
service.change_password("NewPass@1", "NewPass@1")
service.reset_password("tok_abc123", "Reset@Pass1", "Reset@Pass1")

print()

# Validation errors caught correctly
test_cases = [
    ("ab",          "x@x.com",    "Secret@99",  "username too short"),
    ("alice123",    "notanemail", "Secret@99",  "bad email"),
    ("alice123",    "a@b.com",    "weakpass",   "weak password"),
]

for uname, email, pwd, desc in test_cases:
    try:
        service.register(uname, email, pwd)
    except ValueError as e:
        print(f"Caught ({desc}): {e}")
```

---

## Output

```
[REGISTER] User 'alice123' created successfully.
[UPDATE]   Profile for 'bob456' updated.
[CHANGE]   Password changed successfully.
[RESET]    Password reset using token 'tok_abc123'.

Caught (username too short): Username must be at least 3 characters
Caught (bad email):          Invalid email format: notanemail
Caught (weak password):      Password must contain at least one uppercase letter
```

---

## Real-World DRY — Constants and Config

```python
# BAD — magic numbers and strings scattered everywhere

class OrderService:
    def apply_tax(self, amount):
        return amount * 1.18          # 0.18 appears here

class InvoiceService:
    def calculate_gst(self, amount):
        return amount * 0.18          # 0.18 appears again

class ReportService:
    def tax_summary(self, amount):
        tax = amount * 0.18           # 0.18 appears again
        return f"Tax at 18%: {tax}"   # "18%" hardcoded in string too

# Manager: "GST changed to 20%."
# You: searching codebase for every 0.18 and "18%"
# Missing one = wrong invoices = compliance issue


# GOOD — single source of truth for every constant

class TaxConfig:
    GST_RATE         = 0.18
    GST_DISPLAY_NAME = "GST (18%)"
    MAX_DISCOUNT_PCT = 0.30
    FREE_SHIPPING_MIN= 500

class OrderService:
    def apply_tax(self, amount):
        return amount * TaxConfig.GST_RATE       # one source

class InvoiceService:
    def calculate_gst(self, amount):
        return amount * TaxConfig.GST_RATE       # same source

class ReportService:
    def tax_summary(self, amount):
        tax = amount * TaxConfig.GST_RATE
        return f"{TaxConfig.GST_DISPLAY_NAME}: {tax}"

# Manager: "GST changed to 20%."
# You: change TaxConfig.GST_RATE = 0.20
# Done. One line. Every service updated instantly.
```

---

## DRY — Key Rules

```
1. If you wrote the same logic twice — extract it into a function.

2. If you wrote the same constant twice — extract it into a config class.

3. If you wrote the same class structure twice — extract it into a base class.

4. The DRY test:
   "If this rule changes, how many files do I need to edit?"
   Answer should always be: ONE.

5. DRY does NOT mean "never write similar-looking code."
   It means "never duplicate knowledge."
   Two functions that both loop over a list are not DRY violations
   if they represent different pieces of knowledge.

Warning: Over-applying DRY creates premature abstraction.
Apply it when duplication is REAL, not just cosmetic.
```

---
---

# KISS — Keep It Simple, Stupid

## What is it?

> **Most systems work best if they are kept simple rather than made complicated.**
> Simplicity should be a key goal in design.
> Unnecessary complexity should be avoided.

In plain English:
**The simplest solution that works is almost always the right solution.**

Complex code is hard to read.
Hard to read = hard to debug.
Hard to debug = hard to maintain.
Hard to maintain = slow team = expensive product.

Simple code is fast to write, fast to read, fast to fix.

---

## Bad Code — Over-Engineered "Hello World"

```python
# BAD — a junior developer who just read a design patterns book

from abc import ABC, abstractmethod
from typing import Optional


class AbstractGreetingStrategy(ABC):
    @abstractmethod
    def generate_salutation(self) -> str: ...

    @abstractmethod
    def generate_farewell(self) -> str: ...


class FormalGreetingStrategy(AbstractGreetingStrategy):
    def generate_salutation(self) -> str:
        return "Good day"

    def generate_farewell(self) -> str:
        return "Kind regards"


class CasualGreetingStrategy(AbstractGreetingStrategy):
    def generate_salutation(self) -> str:
        return "Hey"

    def generate_farewell(self) -> str:
        return "Later"


class GreetingContext:
    def __init__(self, strategy: AbstractGreetingStrategy):
        self._strategy = strategy

    def set_strategy(self, strategy: AbstractGreetingStrategy):
        self._strategy = strategy

    def get_strategy(self) -> AbstractGreetingStrategy:
        return self._strategy


class GreetingFactory:
    _registry = {}

    @classmethod
    def register(cls, key: str, strategy_class):
        cls._registry[key] = strategy_class

    @classmethod
    def create(cls, key: str) -> AbstractGreetingStrategy:
        if key not in cls._registry:
            raise KeyError(f"Unknown strategy: {key}")
        return cls._registry[key]()


GreetingFactory.register("formal", FormalGreetingStrategy)
GreetingFactory.register("casual", CasualGreetingStrategy)


class GreetingServiceBuilder:
    def __init__(self):
        self._style: Optional[str] = None
        self._name:  Optional[str] = None

    def with_style(self, style: str) -> "GreetingServiceBuilder":
        self._style = style
        return self

    def with_name(self, name: str) -> "GreetingServiceBuilder":
        self._name = name
        return self

    def build(self) -> "GreetingService":
        if not self._style:
            raise ValueError("Style is required")
        if not self._name:
            raise ValueError("Name is required")
        strategy = GreetingFactory.create(self._style)
        context  = GreetingContext(strategy)
        return GreetingService(context, self._name)


class GreetingService:
    def __init__(self, context: GreetingContext, name: str):
        self._context = context
        self._name    = name

    def greet(self) -> str:
        strat = self._context.get_strategy()
        return f"{strat.generate_salutation()}, {self._name}!"


# 60 lines of code to print "Hey, Alice!"
service = (GreetingServiceBuilder()
           .with_style("casual")
           .with_name("Alice")
           .build())
print(service.greet())
```

---

## Good Code — KISS Applied

```python
# GOOD — simple, readable, does the job perfectly

def greet(name: str, formal: bool = False) -> str:
    """Greet a person formally or casually."""
    if formal:
        return f"Good day, {name}."
    return f"Hey, {name}!"


# That is it. 4 lines. Works perfectly.
print(greet("Alice"))           # Hey, Alice!
print(greet("Alice", formal=True))  # Good day, Alice.


# When do you add complexity? Only when the REQUIREMENTS demand it.
# Not before. Not in anticipation. Now. See YAGNI below.
```

---

## Real-World KISS — Data Processing

```python
# BAD — unnecessary complexity for a simple task

from functools import reduce
from typing import Iterator, Generator, Callable, TypeVar

T = TypeVar("T")

class DataPipelineStage(ABC):
    @abstractmethod
    def process(self, data: Iterator) -> Generator: ...

class FilterStage(DataPipelineStage):
    def __init__(self, predicate: Callable):
        self._predicate = predicate
    def process(self, data):
        return (item for item in data if self._predicate(item))

class TransformStage(DataPipelineStage):
    def __init__(self, transformer: Callable):
        self._transformer = transformer
    def process(self, data):
        return (self._transformer(item) for item in data)

class DataPipeline:
    def __init__(self):
        self._stages = []
    def add_stage(self, stage: DataPipelineStage):
        self._stages.append(stage)
        return self
    def execute(self, data):
        result = iter(data)
        for stage in self._stages:
            result = stage.process(result)
        return list(result)

# All this just to get the squares of even numbers
pipeline = DataPipeline()
pipeline.add_stage(FilterStage(lambda x: x % 2 == 0))
pipeline.add_stage(TransformStage(lambda x: x ** 2))
result = pipeline.execute(range(1, 11))
print(result)


# GOOD — Python already gives you this in one readable line

numbers = range(1, 11)
result  = [x ** 2 for x in numbers if x % 2 == 0]
print(result)
# [4, 16, 36, 64, 100]

# Same result. One line. No abstractions. No classes.
# When your requirements grow, THEN you add the pipeline.
```

---

## Real-World KISS — API Response Handling

```python
# BAD — 40 lines of abstraction for reading JSON

from dataclasses import dataclass
from typing import Generic, TypeVar, Optional

T = TypeVar("T")

@dataclass
class ApiResponse(Generic[T]):
    data:    Optional[T]
    error:   Optional[str]
    success: bool
    status_code: int

class ResponseParser(Generic[T]):
    def parse(self, raw: dict) -> ApiResponse[T]:
        return ApiResponse(
            data=raw.get("data"),
            error=raw.get("error"),
            success=raw.get("success", False),
            status_code=raw.get("status_code", 200),
        )

class UserResponseParser(ResponseParser[dict]):
    def parse(self, raw: dict) -> ApiResponse[dict]:
        response = super().parse(raw)
        if response.data:
            response.data["full_name"] = (
                f"{response.data.get('first', '')} "
                f"{response.data.get('last', '')}"
            ).strip()
        return response

parser   = UserResponseParser()
raw_data = {"data": {"first": "Alice", "last": "Smith"}, "success": True, "status_code": 200}
response = parser.parse(raw_data)
print(response.data["full_name"])


# GOOD — just read the dictionary

raw_data  = {"data": {"first": "Alice", "last": "Smith"}, "success": True}
user      = raw_data["data"]
full_name = f"{user['first']} {user['last']}"
print(full_name)   # Alice Smith
```

---

## KISS — Key Rules

```
1. Write the simplest code that solves the current problem.

2. Before adding abstraction, ask:
   "Am I solving a problem that actually exists right now?"
   If NO → do not add it.

3. Signs you are violating KISS:
   - New team member needs 30 minutes to understand a simple function
   - You have abstract base classes for things that only have one implementation
   - You have 5 layers of indirection to print a string
   - You are using a design pattern because it felt right, not because it solved a problem

4. The KISS test:
   "Can a developer who has never seen this code understand it in under 2 minutes?"
   If NO → simplify.

5. Simple does not mean sloppy.
   Clean, well-named, well-structured code is simple.
   Cryptic one-liners are NOT simple.
   Magic is NOT simple.

6. Complexity budget:
   Your codebase has a complexity budget.
   Spend it on the hard business problems.
   Do not waste it on unnecessary abstractions.
```

---
---

# YAGNI — You Aren't Gonna Need It

## What is it?

> Always implement things when you **actually need** them, never when you just **foresee** that you might need them.

In plain English:
**Do not build features that nobody has asked for.**

Developers love to think ahead.
"What if we need multi-currency support later?"
"What if we need to support 10 database types?"
"What if we need a plugin system?"

YAGNI says: **build it when someone actually asks for it. Not before.**

The code you don't write has zero bugs.
The code you don't write needs zero maintenance.
The code you don't write takes zero time.

---

## Bad Code — Speculative Over-Engineering

```python
# BAD — building for imaginary future requirements nobody asked for

from abc import ABC, abstractmethod
from typing import Optional, List, Dict, Any
import json
import csv


class DataExporter(ABC):
    """Abstract base for all exporters."""
    @abstractmethod
    def export(self, data: List[Dict]) -> str: ...

    @abstractmethod
    def get_content_type(self) -> str: ...

    @abstractmethod
    def get_file_extension(self) -> str: ...


class JSONExporter(DataExporter):
    def export(self, data): return json.dumps(data, indent=2)
    def get_content_type(self): return "application/json"
    def get_file_extension(self): return ".json"


class CSVExporter(DataExporter):
    def export(self, data):
        if not data: return ""
        lines = [",".join(data[0].keys())]
        for row in data:
            lines.append(",".join(str(v) for v in row.values()))
        return "\n".join(lines)
    def get_content_type(self): return "text/csv"
    def get_file_extension(self): return ".csv"


class XMLExporter(DataExporter):         # nobody asked for XML
    def export(self, data):
        lines = ["<records>"]
        for row in data:
            lines.append("  <record>")
            for k, v in row.items():
                lines.append(f"    <{k}>{v}</{k}>")
            lines.append("  </record>")
        lines.append("</records>")
        return "\n".join(lines)
    def get_content_type(self): return "application/xml"
    def get_file_extension(self): return ".xml"


class ExcelExporter(DataExporter):       # nobody asked for Excel either
    def export(self, data): return "Excel export not really implemented"
    def get_content_type(self): return "application/vnd.ms-excel"
    def get_file_extension(self): return ".xlsx"


class PDFExporter(DataExporter):         # nobody asked for PDF
    def export(self, data): return "PDF export not really implemented"
    def get_content_type(self): return "application/pdf"
    def get_file_extension(self): return ".pdf"


class MarkdownExporter(DataExporter):    # nobody asked for Markdown
    def export(self, data):
        if not data: return ""
        headers = "| " + " | ".join(data[0].keys()) + " |"
        separator = "| " + " | ".join(["---"] * len(data[0])) + " |"
        rows = ["| " + " | ".join(str(v) for v in row.values()) + " |" for row in data]
        return "\n".join([headers, separator] + rows)
    def get_content_type(self): return "text/markdown"
    def get_file_extension(self): return ".md"


class ExporterFactory:
    _exporters = {
        "json":     JSONExporter,
        "csv":      CSVExporter,
        "xml":      XMLExporter,
        "excel":    ExcelExporter,
        "pdf":      PDFExporter,
        "markdown": MarkdownExporter,
    }

    @classmethod
    def create(cls, format_type: str) -> DataExporter:
        if format_type not in cls._exporters:
            raise ValueError(f"Unsupported format: {format_type}")
        return cls._exporters[format_type]()

    @classmethod
    def register(cls, format_type: str, exporter_class):
        cls._exporters[format_type] = exporter_class


class ExportService:
    def __init__(self, factory: ExporterFactory):
        self._factory = factory

    def export_users(self, users: List[Dict], fmt: str) -> str:
        exporter = self._factory.create(fmt)
        return exporter.export(users)

    def export_orders(self, orders: List[Dict], fmt: str) -> str:
        exporter = self._factory.create(fmt)
        return exporter.export(orders)

    def export_products(self, products: List[Dict], fmt: str) -> str:
        exporter = self._factory.create(fmt)
        return exporter.export(products)


# The actual requirement was:
# "Export user list as JSON for the dashboard."
# That's it. One format. One data type.
# This entire system was built for requirements that don't exist.
```

---

## Good Code — YAGNI Applied

```python
# GOOD — build exactly what was asked for
# Requirement: "Export user list as JSON for the dashboard."

import json
from typing import List, Dict


def export_users_as_json(users: List[Dict]) -> str:
    """
    Export users list as a JSON string.
    Current requirement: dashboard needs JSON only.
    """
    return json.dumps(users, indent=2)


# Usage
users = [
    {"id": 1, "name": "Alice", "email": "alice@example.com", "role": "admin"},
    {"id": 2, "name": "Bob",   "email": "bob@example.com",   "role": "user"},
    {"id": 3, "name": "Carol", "email": "carol@example.com", "role": "user"},
]

result = export_users_as_json(users)
print(result)


# Later, manager says: "We also need CSV export for the finance team."
# NOW you add CSV. Not before.

import csv
import io

def export_users_as_csv(users: List[Dict]) -> str:
    """
    Export users list as a CSV string.
    Added when finance team requested it — not speculatively.
    """
    if not users:
        return ""
    output  = io.StringIO()
    writer  = csv.DictWriter(output, fieldnames=users[0].keys())
    writer.writeheader()
    writer.writerows(users)
    return output.getvalue()


print(export_users_as_csv(users))


# If XML, Excel, PDF, Markdown are NEVER requested:
# Those 4 exporters = 0 lines of code.
# 0 lines = 0 bugs = 0 maintenance = faster delivery.
```

---

## Output

```json
[
  {
    "id": 1,
    "name": "Alice",
    "email": "alice@example.com",
    "role": "admin"
  },
  {
    "id": 2,
    "name": "Bob",
    "email": "bob@example.com",
    "role": "user"
  },
  {
    "id": 3,
    "name": "Carol",
    "email": "carol@example.com",
    "role": "user"
  }
]
```

```
id,name,email,role
1,Alice,alice@example.com,admin
2,Bob,bob@example.com,user
3,Carol,carol@example.com,user
```

---

## Real-World YAGNI — User Repository

```python
# BAD — methods nobody asked for

class UserRepository:

    def get_by_id(self, user_id: int): ...           # ✅ asked for
    def get_by_email(self, email: str): ...          # ✅ asked for

    def get_by_username(self, username: str): ...    # ❌ nobody asked
    def get_by_phone(self, phone: str): ...          # ❌ nobody asked
    def get_by_oauth_token(self, token: str): ...    # ❌ nobody asked
    def get_by_referral_code(self, code: str): ...   # ❌ nobody asked
    def export_to_csv(self): ...                     # ❌ nobody asked
    def export_to_json(self): ...                    # ❌ nobody asked
    def export_to_xml(self): ...                     # ❌ nobody asked
    def bulk_import_from_csv(self, file): ...        # ❌ nobody asked
    def sync_to_ldap(self, ldap_url: str): ...       # ❌ nobody asked
    def archive_inactive_users(self, days: int): ... # ❌ nobody asked
    def generate_analytics_report(self): ...        # ❌ nobody asked


# GOOD — only what is needed right now

class UserRepository:

    def get_by_id(self, user_id: int):
        """Fetch user by primary key."""
        return self._db.find("users", {"id": user_id})

    def get_by_email(self, email: str):
        """Fetch user by email address."""
        return self._db.find("users", {"email": email})

    # When a new feature needs a new lookup method,
    # add it at that point with a real test and real requirement.
    # Not before.
```

---

## YAGNI — Key Rules

```
1. Build what is in the current sprint. Nothing else.

2. "We might need this later" is not a requirement.
   A written ticket is a requirement. A spoken assumption is not.

3. The cost of building something unused:
   - Time to write it
   - Time to test it
   - Time to review it
   - Time to maintain it forever
   - Cognitive load on every developer who reads it
   - Risk of it interfering with future features
   Total: very high.

4. The cost of adding it later when actually needed:
   - Time to write it then
   Total: much lower. You also know the exact requirement.

5. YAGNI does NOT mean "write throwaway code."
   Write clean, good code for what is needed now.
   Just do not write clean code for what is NOT needed.

6. Signs you are violating YAGNI:
   - Methods with no callers
   - Interfaces with only one implementation "for extensibility"
   - Configuration options nobody uses
   - "Future-proof" abstractions
   - TODO comments like "// add multi-currency support here someday"
```

---
---

## All 3 Principles — Quick Reference

```
┌──────────┬────────────────────────────────────────────────────────────────┐
│          │                                                                │
│  DRY     │  Don't Repeat Yourself.                                       │
│          │  Every piece of knowledge has ONE authoritative location.     │
│          │  Change the rule once → updates everywhere.                   │
│          │                                                                │
├──────────┼────────────────────────────────────────────────────────────────┤
│          │                                                                │
│  KISS    │  Keep It Simple, Stupid.                                      │
│          │  Write the simplest solution that solves the problem.         │
│          │  Add complexity only when requirements demand it.             │
│          │                                                                │
├──────────┼────────────────────────────────────────────────────────────────┤
│          │                                                                │
│  YAGNI   │  You Aren't Gonna Need It.                                    │
│          │  Build for current requirements only.                         │
│          │  Do not build for imaginary future needs.                     │
│          │                                                                │
└──────────┴────────────────────────────────────────────────────────────────┘
```

---

## The One-Sentence Test for Each Principle

```
DRY   → "If this rule changes, how many files do I edit?"
        Answer must be ONE. If more → you have duplication.

KISS  → "Can a new developer understand this in 2 minutes?"
        If NO → simplify. Remove abstractions that solve no real problem.

YAGNI → "Who asked for this feature?"
        If nobody → delete it. Build it when someone does.
```

---

## How DRY, KISS, and YAGNI Work Together

```
YAGNI decides WHAT you build.       (only what is needed)
KISS  decides HOW you build it.     (as simply as possible)
DRY   decides HOW you structure it. (no repeated knowledge)

Start with YAGNI → scope the work
Apply KISS       → keep the solution simple
Apply DRY        → remove duplication from the simple solution

In that order. Every time.
```

---