# SOLID & Design Principles — Explained with Exercises

## Part 1: The SOLID Principles

SOLID is five principles for writing maintainable, extensible object-oriented code. Each one answers a specific "why does this code rot?" question.

---

### S — Single Responsibility Principle (SRP)

**A class should have only one reason to change.**

If a class does multiple things, changing any one of them risks breaking the others — and multiple teams/requirements will fight over the same file.

```python
# ❌ Violates SRP — persistence, formatting, and business logic mixed
class Invoice:
    def __init__(self, items):
        self.items = items

    def calculate_total(self):          # business logic
        return sum(i.price for i in self.items)

    def save_to_database(self):         # persistence
        db.execute("INSERT ...")

    def generate_pdf(self):             # presentation
        render_pdf(self)
```

```python
# ✅ Each class has one job
class Invoice:
    def calculate_total(self):
        return sum(i.price for i in self.items)

class InvoiceRepository:
    def save(self, invoice): ...

class InvoicePdfRenderer:
    def render(self, invoice): ...
```

**📝 Exercise 1:** This class has multiple responsibilities — split it:
```python
class Employee:
    def calculate_pay(self): ...
    def report_hours(self): ...
    def save(self): ...
```
Ask: *who would ask for a change to each method?* (CFO → pay logic, Operations → hour reports, DBA → persistence. Three actors = three classes.)

---

### O — Open/Closed Principle (OCP)

**Open for extension, closed for modification.** You should add new behavior by adding new code, not editing existing, tested code.

```python
# ❌ Every new discount type requires editing this method
def discount(customer):
    if customer.type == "student":
        return 0.2
    elif customer.type == "senior":
        return 0.3
    # next week: add "veteran"... edit again, retest everything
```

```python
# ✅ New discounts = new classes, zero changes to existing code
class Discount(ABC):
    @abstractmethod
    def rate(self) -> float: ...

class StudentDiscount(Discount):
    def rate(self): return 0.2

class SeniorDiscount(Discount):
    def rate(self): return 0.3

def apply_discount(price, discount: Discount):
    return price * (1 - discount.rate())
```

Typical tools: **strategy pattern, polymorphism, dependency injection.**

**📝 Exercise 2:** Refactor this area calculator to OCP — adding a `Triangle` shouldn't require touching the calculator:
```python
def area(shape):
    if shape.name == "circle":
        return 3.14159 * shape.radius ** 2
    elif shape.name == "rectangle":
        return shape.width * shape.height
```

---

### L — Liskov Substitution Principle (LSP)

**Subclasses must be substitutable for their base class without breaking correctness.** If code works with the base type, it must work with any subtype.

The classic violation:

```python
class Rectangle:
    def set_width(self, w): self.width = w
    def set_height(self, h): self.height = h
    def area(self): return self.width * self.height

class Square(Rectangle):   # ❌ mathematically a square IS a rectangle...
    def set_width(self, w):
        self.width = self.height = w   # ...but this breaks client expectations
    def set_height(self, h):
        self.width = self.height = h

# This function is correct for Rectangle but silently wrong for Square:
def test_area(r: Rectangle):
    r.set_width(5)
    r.set_height(4)
    assert r.area() == 20   # fails for Square (returns 16)
```

**Fix:** don't force the inheritance — model both as a shared `Shape` abstraction with an `area()` method, or make Rectangle immutable so there's nothing to set.

Rule of thumb: *a subclass shouldn't strengthen preconditions, weaken postconditions, or throw exceptions the base never throws.*

**📝 Exercise 3:** A team writes:
```python
class Bird:
    def fly(self): ...
class Penguin(Bird):
    def fly(self):
        raise NotImplementedError()
```
Explain why this violates LSP and redesign the hierarchy (hint: `Flyable` capability, composition).

---

### I — Interface Segregation Principle (ISP)

**Clients shouldn't be forced to depend on methods they don't use.** Many small, role-specific interfaces beat one fat interface.

```python
# ❌ Fat interface
class Worker(ABC):
    @abstractmethod
    def work(self): ...
    @abstractmethod
    def eat(self): ...

class Robot(Worker):
    def eat(self): ...  # forced to implement a meaningless method
```

```python
# ✅ Segregated
class Workable(ABC):
    @abstractmethod
    def work(self): ...

class Eatable(ABC):
    @abstractmethod
    def eat(self): ...

class Human(Workable, Eatable): ...
class Robot(Workable): ...
```

**📝 Exercise 4:** A document system has this interface:
```python
class Machine(ABC):
    def print(self): ...
    def scan(self): ...
    def fax(self): ...
```
A cheap printer only prints. Redesign with segregated interfaces so `OldPrinter` isn't full of `raise NotImplementedError`.

---

### D — Dependency Inversion Principle (DIP)

**High-level modules shouldn't depend on low-level modules; both should depend on abstractions.** Don't let your business logic `new` up concrete infrastructure.

```python
# ❌ Business logic glued to a specific database
class OrderService:
    def __init__(self):
        self.db = MySQLDatabase()   # concrete dependency

    def place_order(self, order):
        self.db.insert(order)
```

```python
# ✅ Depend on an abstraction; inject the implementation
class OrderRepository(ABC):
    @abstractmethod
    def insert(self, order): ...

class MySQLOrderRepository(OrderRepository): ...
class InMemoryOrderRepository(OrderRepository): ...   # perfect for tests

class OrderService:
    def __init__(self, repo: OrderRepository):
        self.repo = repo

    def place_order(self, order):
        self.repo.insert(order)
```

Benefits: swap implementations, mock in unit tests, business logic stays pure.

**📝 Exercise 5:** Refactor this to follow DIP so the app can support email, SMS, and push notifications without modification:
```python
class NotificationService:
    def __init__(self):
        self.smtp = SmtpClient()
    def notify_user(self, user, msg):
        self.smtp.send(user.email, msg)
```

---

## Part 2: Core Design Principles Beyond SOLID

These are the principles SOLID is built on — often more useful day-to-day.

### DRY — Don't Repeat Yourself
Every piece of knowledge should have a single representation. Duplicated logic = bugs fixed in one place but not another.
> ⚠️ Nuance: don't deduplicate *coincidental* similarity. Two pieces of code that look alike but change for different reasons should stay separate (premature DRY is a real anti-pattern).

**📝 Exercise 6:** Find the violation: two report classes both contain the same 15 lines of "fetch data, filter by date, format currency." Decide whether it's true duplication or coincidence, and fix.

### KISS — Keep It Simple, Stupid
The simplest solution that meets the actual requirement wins. Don't build a plugin architecture for a script run twice.

### YAGNI — You Aren't Gonna Need It
Don't add functionality until it's actually required. Speculative flexibility is code you'll maintain forever for a need that never arrives.

### Composition Over Inheritance
Inheritance is the strongest coupling there is. Prefer assembling behavior from small, swappable parts.
```python
# ❌ Deep hierarchy: Animal → Bird → FlyingBird → Duck
# ✅ class Duck: def __init__(self, fly_behavior, quack_behavior)
```

### Encapsulate What Varies
Identify the parts of your system that change frequently, and isolate them behind interfaces. (This is the engine behind OCP, Strategy, and Factory.)

### Law of Demeter ("Don't talk to strangers")
An object should only call methods on itself, its fields, its parameters, and objects it creates — not reach through chains:
```python
# ❌ order.get_customer().get_address().get_city()   # coupled to 3 internals
# ✅ order.shipping_city()                            # ask, don't dig
```

### Principle of Least Surprise
Code should behave the way a reasonable reader expects. `list.delete(x)` that also sends an email violates it.

### Separation of Concerns
The big-picture version of SRP: partition the system (UI / domain / infrastructure) so each part can evolve independently.

---

## Part 3: Capstone Exercises

**Exercise 7 — The Legacy Refactor.** Below is a class violating nearly everything. List every SOLID violation you can find, then refactor it:
```python
class Report:
    def __init__(self, data):
        self.data = data
        self.db = OracleDB()           # hard-coded
    def fetch_and_render(self, type):
        rows = self.db.query("SELECT * FROM t")
        if type == "html":
            return "<html>" + "".join(f"<p>{r}</p>" for r in rows) + "</html>"
        elif type == "csv":
            return "\n".join(",".join(map(str, r)) for r in rows)
        elif type == "json":
            return json.dumps(rows)
    def email_report(self, recipient):
        smtp = SMTPServer()
        smtp.send(recipient, self.fetch_and_render("html"))
```

**Exercise 8 — Design from scratch.** Design a payment system where:
- New payment methods (card, PayPal, crypto) can be added without touching checkout code (OCP)
- Checkout logic is unit-testable without a real gateway (DIP)
- Notifications after payment are optional and pluggable (SRP, ISP)

Sketch the classes and interfaces.

**Exercise 9 — Code review practice.** Take any real class you've written recently and answer for each method: *"If requirement X changes, which methods/classes break?"* If the answer is "many," find the violated principle.

**Exercise 10 — Spot the fake violation.** Sometimes SOLID is invoked to justify over-engineering. For a 40-line CLI utility, argue both sides: which SOLID principles genuinely help, and which would just add indirection? (This builds the judgment that distinguishes senior engineers.)

---

### Quick Reference

| Principle | One-liner | Smell that signals violation |
|---|---|---|
| SRP | One reason to change | Class needs changes for unrelated requirements |
| OCP | Add code, don't edit | `if/elif` on types that keeps growing |
| LSP | Subtypes are substitutable | `raise NotImplementedError` in overrides |
| ISP | Small, role-based interfaces | Clients implement methods they don't need |
| DIP | Depend on abstractions | `new ConcreteDB()` inside business logic |

Want me to provide worked solutions to any of the exercises, or adapt everything to a specific language (Java/C#/TypeScript)?