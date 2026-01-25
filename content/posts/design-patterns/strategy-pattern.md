+++
title = 'The Strategy Pattern'
date = 2026-01-26T00:00:00+05:30
draft = false
tags = ['design-patterns', 'oop', 'python', 'go', 'typescript']
summary = 'Behavioral design pattern for selecting algorithms at runtime using composition over inheritance, with implementations in Python, Go, and TypeScript.'
+++

# The Strategy Pattern

## Introduction

The **Strategy Pattern** is a behavioral design pattern that enables selecting an algorithm's behavior at runtime. Instead of implementing a single algorithm directly, code receives runtime instructions specifying which of a family of algorithms to use.

This document explores the Strategy Pattern using the classic "Duck" example from *Head First Design Patterns*, with implementations in Python, Go, and TypeScript.

---

## The Problem

Imagine you're building a duck simulation. You might start with a `Duck` base class and use inheritance:

```text
Duck
├── MallardDuck
├── RedheadDuck
├── RubberDuck
└── DecoyDuck
```

But problems arise quickly:

- **Rubber ducks don't fly** — but if `fly()` is in the base class, they inherit it
- **Decoy ducks don't quack** — they're silent wooden ducks
- **Code duplication** — multiple duck types might share the same flying behavior
- **Rigidity** — changing behavior requires modifying classes

Inheritance creates a tight coupling that makes the system fragile and hard to extend.

---

## The Solution: Strategy Pattern

Instead of baking behaviors into a class hierarchy, we:

1. **Define behavior interfaces** — `FlyBehavior` and `QuackBehavior`
2. **Create concrete implementations** — `FlyWithWings`, `FlyNoWay`, `Quack`, `Squeak`, etc.
3. **Compose behaviors into ducks** — each duck *has* a fly behavior and *has* a quack behavior

This follows the principle: **"Favor composition over inheritance."**

### Class Diagram

```text
┌─────────────────────┐          ┌──────────────────────┐
│   <<interface>>     │          │    <<interface>>     │
│    FlyBehavior      │          │    QuackBehavior     │
├─────────────────────┤          ├──────────────────────┤
│ + fly(): string     │          │ + quack(): string    │
└─────────────────────┘          └──────────────────────┘
         △                                  △
         │                                  │
    ┌────┴────┐                    ┌────────┼────────┐
    │         │                    │        │        │
┌───┴───┐ ┌───┴───┐          ┌─────┴─┐ ┌────┴───┐ ┌──┴───┐
│FlyWith│ │FlyNo  │          │ Quack │ │ Squeak │ │ Mute │
│Wings  │ │Way    │          │       │ │        │ │Quack │
└───────┘ └───────┘          └───────┘ └────────┘ └──────┘

                    ┌─────────────────┐
                    │      Duck       │
                    ├─────────────────┤
                    │ flyBehavior     │──────> FlyBehavior
                    │ quackBehavior   │──────> QuackBehavior
                    ├─────────────────┤
                    │ performFly()    │
                    │ performQuack()  │
                    │ display()       │
                    └─────────────────┘
                             △
              ┌──────────────┼──────────────┐
              │              │              │
        ┌─────┴─────┐ ┌──────┴─────┐ ┌──────┴────┐
        │  Mallard  │ │   Rubber   │ │   Decoy   │
        │   Duck    │ │   Duck     │ │   Duck    │
        └───────────┘ └────────────┘ └───────────┘
```

---

## Benefits

1. **Reuse without inheritance baggage** — Behaviors can be shared across unrelated classes
2. **Open/Closed Principle** — Add new behaviors without modifying existing code
3. **Runtime flexibility** — Swap behaviors on the fly
4. **Easier testing** — Behaviors can be tested in isolation

---

## Implementation in Python

Python uses Abstract Base Classes (ABC) to define interfaces:

```python
from abc import ABC, abstractmethod


# ===== Fly Behaviors =====
class FlyBehavior(ABC):
    @abstractmethod
    def fly(self):
        pass


class FlyWithWings(FlyBehavior):
    def fly(self):
        return "I'm flying with wings!"


class FlyNoWay(FlyBehavior):
    def fly(self):
        return "I can't fly."


# ===== Quack Behaviors =====
class QuackBehavior(ABC):
    @abstractmethod
    def quack(self):
        pass


class Quack(QuackBehavior):
    def quack(self):
        return "Quack!"


class Squeak(QuackBehavior):
    def quack(self):
        return "Squeak!"


class MuteQuack(QuackBehavior):
    def quack(self):
        return "..."


# ===== Duck Base Class =====
class Duck(ABC):
    def __init__(self, fly_behavior: FlyBehavior, quack_behavior: QuackBehavior):
        self.fly_behavior = fly_behavior
        self.quack_behavior = quack_behavior

    def perform_fly(self):
        return self.fly_behavior.fly()

    def perform_quack(self):
        return self.quack_behavior.quack()

    # Behaviors can be changed at runtime!
    def set_fly_behavior(self, fly_behavior: FlyBehavior):
        self.fly_behavior = fly_behavior

    def set_quack_behavior(self, quack_behavior: QuackBehavior):
        self.quack_behavior = quack_behavior

    @abstractmethod
    def display(self):
        pass


# ===== Concrete Ducks =====
class MallardDuck(Duck):
    def __init__(self):
        super().__init__(FlyWithWings(), Quack())

    def display(self):
        return "I'm a Mallard duck"


class RubberDuck(Duck):
    def __init__(self):
        super().__init__(FlyNoWay(), Squeak())

    def display(self):
        return "I'm a rubber duck"


class DecoyDuck(Duck):
    def __init__(self):
        super().__init__(FlyNoWay(), MuteQuack())

    def display(self):
        return "I'm a decoy duck"


# ===== Demo =====
if __name__ == "__main__":
    mallard = MallardDuck()
    print(mallard.display())
    print(mallard.perform_fly())
    print(mallard.perform_quack())

    print()

    rubber = RubberDuck()
    print(rubber.display())
    print(rubber.perform_fly())
    print(rubber.perform_quack())

    print()

    # Changing behavior at runtime
    print("Giving the rubber duck a rocket pack...")
    
    class FlyWithRocket(FlyBehavior):
        def fly(self):
            return "I'm flying with a rocket! 🚀"

    rubber.set_fly_behavior(FlyWithRocket())
    print(rubber.perform_fly())
```

### Output

```text
I'm a Mallard duck
I'm flying with wings!
Quack!

I'm a rubber duck
I can't fly.
Squeak!

Giving the rubber duck a rocket pack...
I'm flying with a rocket! 🚀
```

---

## Implementation in Go

Go doesn't have classes or inheritance. Instead, it uses **interfaces** and **struct composition**, which naturally aligns with the Strategy Pattern:

```go
package main

import "fmt"

// ===== Fly Behaviors =====
type FlyBehavior interface {
  Fly() string
}

type FlyWithWings struct{}

func (f FlyWithWings) Fly() string {
  return "I'm flying with wings!"
}

type FlyNoWay struct{}

func (f FlyNoWay) Fly() string {
  return "I can't fly."
}

type FlyWithRocket struct{}

func (f FlyWithRocket) Fly() string {
  return "I'm flying with a rocket! 🚀"
}

// ===== Quack Behaviors =====
type QuackBehavior interface {
  Quack() string
}

type RegularQuack struct{}

func (q RegularQuack) Quack() string {
  return "Quack!"
}

type Squeak struct{}

func (q Squeak) Quack() string {
  return "Squeak!"
}

type MuteQuack struct{}

func (q MuteQuack) Quack() string {
  return "..."
}

// ===== Duck =====
type Duck struct {
  Name          string
  FlyBehavior   FlyBehavior
  QuackBehavior QuackBehavior
}

func (d *Duck) PerformFly() string {
  return d.FlyBehavior.Fly()
}

func (d *Duck) PerformQuack() string {
  return d.QuackBehavior.Quack()
}

func (d *Duck) Display() string {
  return fmt.Sprintf("I'm a %s", d.Name)
}

// ===== Duck Constructors =====
func NewMallardDuck() *Duck {
  return &Duck{
    Name:          "Mallard duck",
    FlyBehavior:   FlyWithWings{},
    QuackBehavior: RegularQuack{},
  }
}

func NewRubberDuck() *Duck {
  return &Duck{
    Name:          "Rubber duck",
    FlyBehavior:   FlyNoWay{},
    QuackBehavior: Squeak{},
  }
}

func NewDecoyDuck() *Duck {
  return &Duck{
    Name:          "Decoy duck",
    FlyBehavior:   FlyNoWay{},
    QuackBehavior: MuteQuack{},
  }
}

// ===== Demo =====
func main() {
  mallard := NewMallardDuck()
  fmt.Println(mallard.Display())
  fmt.Println(mallard.PerformFly())
  fmt.Println(mallard.PerformQuack())

  fmt.Println()

  rubber := NewRubberDuck()
  fmt.Println(rubber.Display())
  fmt.Println(rubber.PerformFly())
  fmt.Println(rubber.PerformQuack())

  fmt.Println()

  // Changing behavior at runtime
  fmt.Println("Giving the rubber duck a rocket pack...")
  rubber.FlyBehavior = FlyWithRocket{}
  fmt.Println(rubber.PerformFly())
}
```

### Key Go Differences

- **Implicit interfaces**: Types satisfy interfaces automatically if they have the right methods (structural typing)
- **No inheritance**: One `Duck` struct with different configurations instead of subclasses
- **Flatter hierarchy**: The pattern becomes simpler and more explicit

---

## Implementation in TypeScript

TypeScript supports both class-based OOP and functional approaches:

### Class-Based Approach

```typescript
// ===== Fly Behaviors =====
interface FlyBehavior {
  fly(): string;
}

class FlyWithWings implements FlyBehavior {
  fly(): string {
    return "I'm flying with wings!";
  }
}

class FlyNoWay implements FlyBehavior {
  fly(): string {
    return "I can't fly.";
  }
}

class FlyWithRocket implements FlyBehavior {
  fly(): string {
    return "I'm flying with a rocket! 🚀";
  }
}

// ===== Quack Behaviors =====
interface QuackBehavior {
  quack(): string;
}

class Quack implements QuackBehavior {
  quack(): string {
    return "Quack!";
  }
}

class Squeak implements QuackBehavior {
  quack(): string {
    return "Squeak!";
  }
}

class MuteQuack implements QuackBehavior {
  quack(): string {
    return "...";
  }
}

// ===== Duck Base Class =====
abstract class Duck {
  constructor(
    protected flyBehavior: FlyBehavior,
    protected quackBehavior: QuackBehavior
  ) {}

  performFly(): string {
    return this.flyBehavior.fly();
  }

  performQuack(): string {
    return this.quackBehavior.quack();
  }

  setFlyBehavior(flyBehavior: FlyBehavior): void {
    this.flyBehavior = flyBehavior;
  }

  setQuackBehavior(quackBehavior: QuackBehavior): void {
    this.quackBehavior = quackBehavior;
  }

  abstract display(): string;
}

// ===== Concrete Ducks =====
class MallardDuck extends Duck {
  constructor() {
    super(new FlyWithWings(), new Quack());
  }

  display(): string {
    return "I'm a Mallard duck";
  }
}

class RubberDuck extends Duck {
  constructor() {
    super(new FlyNoWay(), new Squeak());
  }

  display(): string {
    return "I'm a Rubber duck";
  }
}

class DecoyDuck extends Duck {
  constructor() {
    super(new FlyNoWay(), new MuteQuack());
  }

  display(): string {
    return "I'm a Decoy duck";
  }
}

// ===== Demo =====
const mallard = new MallardDuck();
console.log(mallard.display());
console.log(mallard.performFly());
console.log(mallard.performQuack());

console.log();

const rubber = new RubberDuck();
console.log(rubber.display());
console.log(rubber.performFly());
console.log(rubber.performQuack());

console.log();

// Changing behavior at runtime
console.log("Giving the rubber duck a rocket pack...");
rubber.setFlyBehavior(new FlyWithRocket());
console.log(rubber.performFly());
```

### Functional Approach

A more idiomatic TypeScript/JavaScript approach using functions and plain objects:

```typescript
// ===== Behavior Types =====
type FlyBehavior = () => string;
type QuackBehavior = () => string;

// ===== Fly Behaviors =====
const flyWithWings: FlyBehavior = () => "I'm flying with wings!";
const flyNoWay: FlyBehavior = () => "I can't fly.";
const flyWithRocket: FlyBehavior = () => "I'm flying with a rocket! 🚀";

// ===== Quack Behaviors =====
const quack: QuackBehavior = () => "Quack!";
const squeak: QuackBehavior = () => "Squeak!";
const muteQuack: QuackBehavior = () => "...";

// ===== Duck Type =====
interface Duck {
  name: string;
  fly: FlyBehavior;
  quack: QuackBehavior;
}

// ===== Factory Functions =====
const createMallard = (): Duck => ({
  name: "Mallard duck",
  fly: flyWithWings,
  quack: quack,
});

const createRubberDuck = (): Duck => ({
  name: "Rubber duck",
  fly: flyNoWay,
  quack: squeak,
});

const createDecoyDuck = (): Duck => ({
  name: "Decoy duck",
  fly: flyNoWay,
  quack: muteQuack,
});

// ===== Demo =====
const mallard = createMallard();
console.log(`I'm a ${mallard.name}`);
console.log(mallard.fly());
console.log(mallard.quack());

console.log();

const rubber = createRubberDuck();
console.log(`I'm a ${rubber.name}`);
console.log(rubber.fly());
console.log(rubber.quack());

console.log();

// Swap behavior (immutable style - create new object)
console.log("Giving the rubber duck a rocket pack...");
const rocketRubberDuck: Duck = {
  ...rubber,
  fly: flyWithRocket,
};
console.log(rocketRubberDuck.fly());
```

---

## Comparison Across Languages

| Aspect | Python | Go | TypeScript |
|--------|--------|-----|------------|
| **Interface Definition** | ABC with `@abstractmethod` | `interface` keyword | `interface` keyword |
| **Interface Implementation** | Explicit (`class X(Interface)`) | Implicit (structural) | Explicit (`implements`) |
| **Inheritance** | Supported | Not supported | Supported |
| **Idiomatic Style** | Class-based or duck typing | Composition with structs | Class-based or functional |
| **Runtime Behavior Swap** | Setter methods | Direct field assignment | Setter methods or spread operator |

---

## When to Use the Strategy Pattern

✅ **Use when:**

- You have multiple algorithms for a specific task
- You need to switch algorithms at runtime
- You have a class with multiple conditional behaviors
- You want to avoid inheritance hierarchies

❌ **Avoid when:**

- You only have a couple of simple algorithms that rarely change
- The overhead of extra classes/interfaces isn't justified
- Your language has simpler alternatives (e.g., passing functions directly)

---

## Related Patterns

- **State Pattern**: Similar structure, but for managing object state transitions
- **Decorator Pattern**: Also uses composition, but for adding responsibilities
- **Factory Pattern**: Often used together to create strategy objects

---

## References

- *Head First Design Patterns* by Eric Freeman & Elisabeth Robson
- *Design Patterns: Elements of Reusable Object-Oriented Software* by Gang of Four
