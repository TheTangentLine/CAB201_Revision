# SOLID

SOLID is an acronym for five principles intended to make source code more understandable, flexible and maintainable.

---

#### TOC

- [S - Single responsibility principle](#s---single-responsibility-principle)
- [O - Open-closed principle](#o---open-closed-principle)
- [L - Liskov substitution principle](#l---liskov-substitution-principle)
- [I - Interface segregation principle](#i---interface-segregation-principle)
- [D - Dependency inversion principle](#d---dependency-inversion-principle)

---

## S - Single responsibility principle

S in SOLID states that:

- There should be **one and only one** reason for a class to change.
- In other words, every class should have **only one responsibility**.

_Example:_

```csharp
// -------- Good practice ---------
class Student {
    public void Study() {
        Console.WriteLine("I'm studying.");
    }
}

class Teacher {
    public void Teach() {
        Console.WriteLine("I'm teaching.");
    }
}

// -------- Bad practice ---------
class Person {
    public void Study() {
        Console.WriteLine("I'm studying.");
    }
    public void Teach() {
        Console.WriteLine("I'm teaching.");
    }
}
```

In the above example, "Student" only has 1 job (study), same for "Teacher" (teach). If we want to change the Study method later, only Student is affected. With the Person approach, one class does everything, so every change touches it.

## O - Open-closed principle

O in SOLID states that:

- Software entities (classes, modules, functions,...) should be **open for extension** but **closed for modification**.
- In other words, add new code instead of changing old code that already works.

_Example:_

```csharp
// Old class
class Athlete {
    protected int km_H;
    public void Walk() {
        km_H = 10;
        Console.WriteLine("Walking...");
    }
}

// Now, they want the Athlete to walk a bit faster

// -------- Good practice ---------
class FastAthlete : Athlete {
    public void WalkFaster() {
        km_H = 15;
        Console.WriteLine("Walking faster...");
    }
}

// -------- Bad practice ---------
class Athlete {
    protected int km_H;
    public void Walk() {
        km_H = 15;
        Console.WriteLine("Walking...");
    }
}
```

In the above example, the good practice **extends** Athlete with a new class, so the old Athlete is never touched and nothing using it can break. The bad practice directly modifies Walk, so everywhere that relied on 10 km/h now changes. And if one day people want it slower, we have to modify Walk again.

## L - Liskov substitution principle

L in SOLID states that:

- A child class must be able to **replace** its parent class without breaking anything.
- In other words, if Duck is a Bird, then anywhere a Bird works, a Duck must work too.

> [!NOTE]
> This is what makes **polymorphism** safe: we can write `Bird b = new Duck();` and call `b.Fly()` without caring which bird it actually is.

_Example:_

```csharp
class Bird {
    public virtual void Fly() {
        Console.WriteLine("Flying...");
    }
}

// -------- Good practice ---------
class Duck : Bird {
    public override void Fly() {
        Console.WriteLine("Duck flying...");
    }
}

Bird duck = new Duck();
duck.Fly(); // "Duck flying..." -> works fine

// -------- Bad practice ---------
class Penguin : Bird {
    public override void Fly() {
        throw new NotImplementedException("Penguins can't fly!");
    }
}

Bird penguin = new Penguin();
penguin.Fly(); // crash!
```

In the above example, Duck can stand in for Bird with no surprises. Penguin can't, because it breaks Fly. Penguin is still a bird, but it shouldn't inherit Fly. A better design is to move Fly into a separate `FlyingBird` class that only flying birds inherit.

## I - Interface segregation principle

I in SOLID states that:

- A class should **not be forced** to implement methods it doesn't use.
- In other words, many small interfaces are better than one big interface.

> [!IMPORTANT]
> This one is very important in **enterprise**: codebases are huge and many teams share the same interfaces. With one big interface, adding a single method forces every class (and every team) to change.

_Example:_

```csharp
// -------- Good practice ---------
interface IPrinter { void Print(); }
interface IScanner { void Scan(); }
interface IFax     { void Fax(); }

class SuperMachine : IPrinter, IScanner, IFax {
    public void Print() { Console.WriteLine("Printing..."); }
    public void Scan()  { Console.WriteLine("Scanning..."); }
    public void Fax()   { Console.WriteLine("Faxing..."); }
}

class Printer : IPrinter {
    public void Print() { Console.WriteLine("Printing..."); }
}

// -------- Bad practice ---------
interface ISuperMachine {
    void Print();
    void Scan();
    void Fax();
}

class Printer : ISuperMachine {
    public void Print() { Console.WriteLine("Printing..."); }
    public void Scan()  { throw new NotImplementedException(); } // can't scan
    public void Fax()   { throw new NotImplementedException(); } // can't fax
}
```

In the above example, the bad Printer is stuck with Scan and Fax even though it can't do them. With small interfaces, the SuperMachine picks all three, and the Printer only picks what it can actually do.

## D - Dependency inversion principle

> [!NOTE]
> Hardest to explain, easiest to implement.

D in SOLID states that:

- Depend on **interfaces**, not concrete classes.
- In other words, the high-level code (_what_ we want to do: send a notification) shouldn't care about the low-level details (_how_ it's done: Email, WhatsApp,...).

_Example:_

```csharp
// The "I" in front is just the C# naming convention for interfaces
interface IMessage {
    void Send(string text);
}

class Email : IMessage {
    public void Send(string text) {
        Console.WriteLine("Email: " + text);
    }
}

class WhatsApp : IMessage {
    public void Send(string text) {
        Console.WriteLine("WhatsApp: " + text);
    }
}

// -------- Good practice ---------
class Notification {
    private IMessage message;
    public Notification(IMessage message) {
        this.message = message;
    }
    public void Notify(string text) {
        message.Send(text);
    }
}

new Notification(new Email()).Notify("Hi!");    // send by Email
new Notification(new WhatsApp()).Notify("Hi!"); // switch to WhatsApp, Notification unchanged

// -------- Bad practice ---------
class Notification {
    private Email email = new Email();
    public void Notify(string text) {
        email.Send(text);
    }
}
```

In the above example, the good Notification only knows about IMessage, so switching between Email and WhatsApp is a one-word change. The bad Notification is stuck with Email, so switching to WhatsApp means rewriting the class.
