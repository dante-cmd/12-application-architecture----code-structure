**Polymorphism** comes from Greek meaning *"many forms."* In programming, it is the ability of different objects to respond to the **same method or action in their own unique way**.

Instead of writing separate, conditional code for every specific type of object, polymorphism lets you write clean, flexible code that works with a single unified interface.

---

### A Real-World Analogy

Think of the **Power Button** on an electronic device.

* When you press the power button on a **TV**, it turns on the screen and speakers.
* When you press the power button on a **Laptop**, it boots the OS and lights up the keyboard.
* When you press the power button on a **Smart Lamp**, it lights up the bulb.

You (the caller) don't need to know the complex internal mechanics of each device to turn it on—you just trigger the same action (`turnOn()`), and each device handles it according to its own nature.

---

### The Two Main Types of Polymorphism

#### 1. Dynamic (Runtime) Polymorphism — *Method Overriding*

Occurs when a subclass provides its own implementation of a method defined in its parent class or interface. The decision of which method to run happens at **runtime**.

```python
# Base / Parent concept
class PaymentMethod:
    def process_payment(self, amount):
        pass

# Specific implementations
class CreditCard(PaymentMethod):
    def process_payment(self, amount):
        print(f"Charging ${amount} to Credit Card via payment gateway.")

class PayPal(PaymentMethod):
    def process_payment(self, amount):
        print(f"Redirecting to PayPal to authorize ${amount}.")

# Polymorphic behavior in action
def checkout(payment_method: PaymentMethod, amount: float):
    # The checkout function doesn't care WHICH payment method it is!
    payment_method.process_payment(amount)

# Usage:
checkout(CreditCard(), 100)  # Output: Charging $100 to Credit Card...
checkout(PayPal(), 50)      # Output: Redirecting to PayPal...

```

#### 2. Static (Compile-Time) Polymorphism — *Method Overloading*

Occurs when multiple methods in the same class share the **same name**, but accept **different parameters** (types or counts). The compiler determines which version to execute before the program runs.

*(Note: Supported natively in languages like Java, C++, and C#).*

```java
class Calculator {
    // Same method name, different parameter types/counts
    int add(int a, int b) {
        return a + b;
    }

    double add(double a, double b) {
        return a + b;
    }

    int add(int a, int b, int c) {
        return a + b + c;
    }
}

```

---

### Why Polymorphism Matters

| Benefit | How It Helps |
| --- | --- |
| **Extensibility (Open/Closed Principle)** | You can add new features (e.g., adding a `Crypto` payment method) without modifying existing checkout code. |
| **Maintainability** | Eliminates massive `if/else` or `switch` statements checking for object types. |
| **Decoupling** | High-level code depends on abstractions (interfaces) rather than hardcoded concrete implementations. |