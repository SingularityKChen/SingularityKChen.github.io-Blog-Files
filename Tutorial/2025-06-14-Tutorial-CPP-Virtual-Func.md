---
layout: post
title: "[Tutorial] Understanding Virtual and Pure Virtual Functions in C++"
description: "Virtual functions are a fundamental concept in C++ that enable runtime polymorphism, allowing objects to be treated uniformly while exhibiting different behaviors based on their actual type. This guide provides an in-depth exploration of virtual functions, including their mechanics, best practices, and advanced use cases with pure virtual functions and abstract classes."
categories: [Tutorial]
tags: [CPP]
last_updated: 2025-06-09 16:49:00 GMT+8
excerpt: "Virtual functions are a fundamental concept in C++ that enable runtime polymorphism, allowing objects to be treated uniformly while exhibiting different behaviors based on their actual type. This guide provides an in-depth exploration of virtual functions, including their mechanics, best practices, and advanced use cases with pure virtual functions and abstract classes."
redirect_from:
  - /2025/06/14/
---

# TL;DR

- **Virtual Functions** enable runtime polymorphism, allowing derived classes to override base class behavior.
- **Pure Virtual Functions** define interfaces; derived classes must implement them.
- **Abstract Classes** cannot be instantiated; they contain at least one pure virtual function.
- **Design Patterns** like Strategy, Factory, and Template Method rely on virtual and pure virtual functions for flexible system design.

# 1. Virtual Functions: The Core of Runtime Polymorphism

## 1.1 What Are Virtual Functions?

A **virtual function** is a member function in a base class that you expect to override in derived classes. It enables **runtime polymorphism**, where the function called is determined by the object’s type at runtime, not the pointer/reference type. This mechanism allows for **dynamic dispatch**, a cornerstone of object-oriented design.

## 1.2 When to Use Virtual Functions

| Scenario | Description |
|---------|-------------|
| **Runtime Polymorphism** | Call derived class methods via base class pointers/references. |
| **Interface Definition** | Base class defines an interface for derived classes. |
| **Template Method Pattern** | Base class defines an algorithm skeleton with customizable steps. |
| **Virtual Destructors** | Prevent memory leaks when deleting derived objects via base pointers. |

## 1.3 How Virtual Functions Work Under the Hood

Virtual functions rely on a **virtual table (vtable)** and **virtual pointer (vptr)** mechanism:
- **vtable**: A hidden static array of function pointers associated with a class. Each entry points to a virtual function.
- **vptr**: A hidden pointer in each object that points to its class’s vtable.

### Example:
```cpp
class Base {
public:
    virtual void foo() { std::cout << "Base::foo\n"; }
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void foo() override { 
        std::cout << "Derived::foo\n"; 
    }
};

int main() {
    Derived d;
    Base* b = &d;
    b->foo(); // Output: Derived::foo
}
```

- At runtime, `b->foo()` resolves to `Derived::foo` via the vtable.
- The compiler ensures the vptr points to the correct vtable for the object’s actual type.

> **Key Insight**: Virtual functions introduce indirection (via vtable), which adds runtime overhead but enables flexible polymorphic behavior.

# 2. Pure Virtual Functions and Abstract Classes

## 2.1 What Is a Pure Virtual Function?

A **pure virtual function** is a virtual function declared with `= 0` in the base class, indicating that it has **no default implementation**. It serves as a placeholder for behavior that derived classes **must** implement.

```cpp
class Base {
public:
    virtual void foo() = 0; // Pure virtual function
};
```

- **Abstract Class Requirement**: A class containing at least one pure virtual function becomes an **abstract class** and cannot be instantiated.
- **Derived Class Obligation**: Any concrete (non-abstract) derived class **must override** all pure virtual functions.

## 2.2 Abstract Classes: Design and Behavior

An **abstract class** acts as a blueprint for derived classes. It can contain:
- **Pure virtual functions** (mandatory overrides)
- **Concrete virtual functions** (optional overrides)
- **Non-virtual functions** (inherited as-is)

### Example: Intermediate Abstract Class
If you want to create an intermediate class like `test_base_class` that does **not** implement the pure virtual functions from `base_class`, you can simply inherit without overriding:
```cpp
class base_class {
public:
    virtual void do_something() = 0; // Pure virtual
    virtual ~base_class() = default;
};

class test_base_class : public base_class {
    // No override → still abstract
};
```

> ✅ **Key Insight**: You are **not required** to explicitly redeclare pure virtual functions in `test_base_class`. The inheritance is implicit. However, **explicitly declaring them** (e.g., `virtual void do_something() override = 0;`) improves readability and makes the abstract nature of the class more obvious.

## 2.3 Default Implementations for Pure Virtual Functions

Although rare, **pure virtual functions can have default implementations**. This allows derived classes to **reuse** the base implementation if desired.

### Example:
```cpp
class base_class {
public:
    virtual void do_something() = 0; // Pure virtual declaration
    virtual ~base_class() = default;
};

// Default implementation
void base_class::do_something() {
    std::cout << "Default implementation\n";
}

class Derived : public base_class {
public:
    void do_something() override {
        base_class::do_something(); // Call default behavior
        std::cout << "Derived extension\n";
    }
};
```

> 📌 **Important**: Even with a default implementation, the class remains abstract because of the `= 0` in the declaration.

## 2.4 Pure Virtual Destructors

A pure virtual destructor is special. While you can declare it as pure, you **must provide a default implementation**, or your program will fail to link.

### Example:
```cpp
class base_class {
public:
    virtual ~base_class() = 0; // Pure virtual destructor
};

// Required implementation
base_class::~base_class() {
    // Cleanup common to all derived classes
}
```

> ⚠️ **Critical Rule**: A pure virtual destructor ensures derived classes must define their own destructors, but the base class still needs an implementation to allow proper destruction during object lifetime.

# 3. Design Patterns Using Virtual and Pure Virtual Functions

## 3.1 Strategy Pattern: Dynamic Behavior Switching

**Problem**:  
You want to define a family of algorithms (e.g., payment methods), encapsulate each one, and make them interchangeable at runtime without modifying the context that uses them.

**Solution**:  
Use the **Strategy Pattern** with an abstract class (`PaymentStrategy`) defining a common interface. Concrete strategies (`CreditCard`, `PayPal`) implement specific behaviors.

### Example:
```cpp
// Abstract strategy interface
class PaymentStrategy {
public:
    virtual void pay(int amount) = 0; // Pure virtual function
    virtual ~PaymentStrategy() = default;
};

// Concrete strategy 1
class CreditCard : public PaymentStrategy {
public:
    void pay(int amount) override {
        std::cout << "Paid $" << amount << " by Credit Card\n";
    }
};

// Concrete strategy 2
class PayPal : public PaymentStrategy {
public:
    void pay(int amount) override {
        std::cout << "Paid $" << amount << " via PayPal\n";
    }
};

// Context using the strategy
class ShoppingCart {
public:
    void setPaymentStrategy(PaymentStrategy* strategy) {
        this->strategy = strategy;
    }

    void checkout(int totalAmount) {
        if (strategy) {
            strategy->pay(totalAmount);
        } else {
            std::cout << "No payment method selected.\n";
        }
    }

private:
    PaymentStrategy* strategy = nullptr;
};
```

### Usage:
```cpp
int main() {
    ShoppingCart cart;
    
    CreditCard cc;
    PayPal pp;

    cart.setPaymentStrategy(&cc);
    cart.checkout(100); // Output: Paid $100 by Credit Card

    cart.setPaymentStrategy(&pp);
    cart.checkout(200); // Output: Paid $200 via PayPal
}
```

**Key Advantages**:
- **Flexibility**: Add new payment methods without changing `ShoppingCart`.
- **Avoids Conditionals**: No need for `if/else` or `switch` statements.
- **Separation of Concerns**: Payment logic is decoupled from business logic.

**Real-World Use Cases**:
- Payment gateways in e-commerce.
- Sorting/searching algorithms.
- Compression formats (e.g., ZIP, GZIP).

## 3.2 Factory Pattern: Centralized Object Creation

**Problem**:  
You want to encapsulate object creation logic, allowing subclasses to decide which class to instantiate. This promotes loose coupling between object users and their concrete types.

**Solution**:  
Use the **Factory Pattern** with an abstract class (`Product`) and a factory method that returns a base class pointer.

### Example:
```cpp
// Abstract product interface
class Product {
public:
    virtual std::string name() const = 0; // Pure virtual function
    virtual ~Product() = default;
};

// Concrete product 1
class Chair : public Product {
public:
    std::string name() const override {
        return "Chair";
    }
};

// Concrete product 2
class Table : public Product {
public:
    std::string name() const override {
        return "Table";
    }
};

// Factory class
class ProductFactory {
public:
    static Product* createProduct(const std::string& type) {
        if (type == "chair") {
            return new Chair();
        } else if (type == "table") {
            return new Table();
        } else {
            return nullptr;
        }
    }
};
```

### Usage:
```cpp
int main() {
    Product* p1 = ProductFactory::createProduct("chair");
    Product* p2 = ProductFactory::createProduct("table");

    if (p1) std::cout << "Created: " << p1->name() << "\n"; // Output: Created: Chair
    if (p2) std::cout << "Created: " << p2->name() << "\n"; // Output: Created: Table

    delete p1;
    delete p2;
}
```

**Key Advantages**:
- **Centralized Creation**: All object creation logic resides in one place.
- **Extensibility**: Add new product types without modifying existing code.
- **Decoupling**: Clients depend on the `Product` interface, not concrete types.

**Real-World Use Cases**:
- GUI toolkits (e.g., `createButton()`, `createWindow()`).
- Database connection factories (MySQL, PostgreSQL).
- Game entity spawning systems.

## 3.3 Interface Abstraction: Enforcing Contracts

**Problem**:  
You need to define a standardized API that multiple implementations must follow, ensuring consistency across different modules or plugins.

**Solution**:  
Use an **abstract class with pure virtual functions** to define an interface (`ILogger`). Derived classes (`ConsoleLogger`, `FileLogger`) must implement the required methods.

### Example:
```cpp
// Abstract interface
class ILogger {
public:
    virtual void log(const std::string& message) = 0; // Pure virtual function
    virtual ~ILogger() = default;
};

// Concrete implementation 1
class ConsoleLogger : public ILogger {
public:
    void log(const std::string& message) override {
        std::cout << "[Console] " << message << "\n";
    }
};

// Concrete implementation 2
class FileLogger : public ILogger {
public:
    void log(const std::string& message) override {
        std::ofstream file("app.log", std::ios::app);
        file << "[File] " << message << "\n";
    }
};
```

### Usage:
```cpp
void logMessage(ILogger* logger, const std::string& msg) {
    logger->log(msg); // Polymorphic dispatch
}

int main() {
    ConsoleLogger console;
    FileLogger file;

    logMessage(&console, "User logged in");  // Output: [Console] User logged in
    logMessage(&file, "System error");      // Output: [File] System error in app.log
}
```

**Key Advantages**:
- **Contract Enforcement**: Ensures all loggers implement `log()`.
- **Plug-and-Play**: Swap logging implementations without code changes.
- **Dependency Inversion**: High-level modules depend on abstractions, not concretions.

**Real-World Use Cases**:
- Logging frameworks (e.g., log4cpp).
- Cross-platform APIs (e.g., `IFileSystem` for Windows/Linux/Mac).
- Plugin architectures (e.g., IDE extensions).

# 4. Best Practices and Common Pitfalls

## 4.1 Virtual Destructors

- If a base class is intended for inheritance, its destructor **must be virtual**.
- For pure virtual destructors, always provide a default implementation.

## 4.2 Function Signature Consistency

- Overriding a virtual function requires **exact signature matching**, including `const` qualifiers and parameter lists.
- Use `override` to ensure correct overrides and `final` to prevent further overrides.

## 4.3 Avoid Unnecessary Virtual Functions

- Use virtual functions only when runtime polymorphism is needed.
- For compile-time polymorphism, prefer templates or CRTP (Curiously Recurring Template Pattern).

## 4.4 Performance Considerations

- Virtual functions introduce indirection (via vtable), which adds runtime overhead.
- Avoid virtual functions in performance-critical code (e.g., hot paths, embedded systems).

## 4.5 Inline and Header-Only Definitions

- If you define a pure virtual function with a default implementation and mark it `inline`, ensure it's placed in a header file.

# 5. Summary

| Concept | Key Takeaways |
|--------|---------------|
| **Virtual Functions** | Enable runtime polymorphism through vtable/vptr mechanism. |
| **Pure Virtual Functions** | Define interfaces; derived classes must implement them. |
| **Abstract Classes** | Cannot be instantiated; may mix pure and concrete functions. |
| **Design Patterns** | Strategy, Factory, and Interface patterns benefit greatly. |
| **Best Practices** | Use `override`, `final`, and virtual destructors; avoid mismatched signatures. |
| **Performance** | Introduce runtime overhead; use sparingly in performance-critical code. |
| **Alternatives** | Prefer templates, lambdas, or composition for compile-time or lightweight polymorphism. |

By mastering virtual and pure virtual functions, you unlock the power of polymorphism, enabling flexible and extensible class hierarchies. However, always weigh their benefits against their costs to build efficient, maintainable systems.