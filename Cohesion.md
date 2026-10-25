# Cohesion

Cohesion measures how closely related the things **inside one class** are. Do its fields and methods all work towards the same job?

> [!IMPORTANT]
> Aim for **high cohesion**: one class, one focused job. This is the same idea as S in [SOLID](SOLID.md).

---

#### TOC

- [The 7 types, worst to best](#the-7-types-worst-to-best)
- [Coincidental cohesion](#coincidental-cohesion)
- [Logical cohesion](#logical-cohesion)
- [Temporal cohesion](#temporal-cohesion)
- [Procedural cohesion](#procedural-cohesion)
- [Communicational cohesion](#communicational-cohesion)
- [Sequential cohesion](#sequential-cohesion)
- [Functional cohesion](#functional-cohesion)
- [Fixing low cohesion](#fixing-low-cohesion)

---

## The 7 types, worst to best

```mermaid
flowchart LR
    A[Coincidental] --> B[Logical] --> C[Temporal] --> D[Procedural] --> E[Communicational] --> F[Sequential] --> G[Functional]
    style A fill:#f8d7da,stroke:#c0392b
    style B fill:#f8d7da,stroke:#c0392b
    style C fill:#fff3cd,stroke:#b7950b
    style D fill:#fff3cd,stroke:#b7950b
    style E fill:#fff3cd,stroke:#b7950b
    style F fill:#d4edda,stroke:#1e8449
    style G fill:#d4edda,stroke:#1e8449
```

_Red = bad, yellow = okay, green = good._

> [!TIP]
> To spot the type, ask: **why** are these methods in the same class?

| Type            | Grouped because...                          |
| --------------- | ------------------------------------------- |
| Coincidental    | No reason, they just ended up there         |
| Logical         | They're the same _kind_ of thing            |
| Temporal        | They run at the same _time_                 |
| Procedural      | They run in a fixed _order_                 |
| Communicational | They use the same _data_                    |
| Sequential      | One's _output_ is the next one's _input_    |
| Functional      | They all do _one single job_                |

## Coincidental cohesion

Coincidental cohesion is:

- Methods grouped for **no real reason**. The worst kind.
- In other words, the classic `Utils` / `Helpers` dumping ground.

_Example:_

```csharp
class Utils {
    public string FormatDate(DateTime date) { /* ... */ }
    public double CalculateTax(double income) { /* ... */ }
    public void SendEmail(string to, string text) { /* ... */ }
}
```

In the above example, dates, tax and email have nothing in common. Each method should live in the class it belongs to.

## Logical cohesion

Logical cohesion is:

- Methods grouped because they're the same **kind** of thing, and a **flag** picks which one runs.
- In other words, they look related, but they don't actually work together.

_Example:_

```csharp
class InputHandler {
    public string Handle(int source) {
        if (source == 1) return ReadKeyboard();
        if (source == 2) return ReadFile();
        return ReadNetwork();
    }
}
```

In the above example, keyboard, file and network input are all "input", but share no code or data. The flag is also **control coupling** (see [Coupling](Coupling.md)).

## Temporal cohesion

Temporal cohesion is:

- Methods grouped because they run at the **same time**, e.g. at start-up or shut-down.

_Example:_

```csharp
class Startup {
    public void Run() {
        InitLogger();
        LoadConfig();
        OpenDatabase();
    }
}
```

In the above example, logging, config and database have nothing in common except "they happen when the app starts".

## Procedural cohesion

Procedural cohesion is:

- Methods grouped because they must run in a **fixed order**, but they **don't pass data** to each other.

_Example:_

```csharp
class LoginFlow {
    public void Run() {
        CheckPassword();
        LoadMenu();
        PlayWelcomeSound();
    }
}
```

In the above example, the steps follow an order, but each one works on different data.

## Communicational cohesion

Communicational cohesion is:

- Methods grouped because they all work on the **same data**.

_Example:_

```csharp
class CustomerReport {
    private Customer customer;

    public void PrintDetails() { /* uses customer */ }
    public void UpdateAddress(string address) { /* uses customer */ }
    public double CalculateBalance() { /* uses customer */ }
}
```

In the above example, every method uses the same `customer`. That's a real link, but printing, updating and calculating are still 3 different jobs.

## Sequential cohesion

Sequential cohesion is:

- Methods grouped because the **output** of one is the **input** of the next, like an assembly line.

_Example:_

```csharp
class ImportPipeline {
    public void Import(string path) {
        string[] lines = ReadFile(path);
        List<Record> records = ParseLines(lines);
        SaveRecords(records);
    }
}
```

In the above example, the data flows from one step to the next. Compare this with procedural, where the steps follow an order but **don't** pass data along.

## Functional cohesion

Functional cohesion is:

- Every field and method contributes to **one single, well-defined job**. The best kind.

_Example:_

```csharp
class TaxCalculator {
    private const double TaxFreeThreshold = 18200;

    public double Calculate(double income) {
        if (income <= TaxFreeThreshold) return 0;
        return Round((income - TaxFreeThreshold) * 0.19);
    }
    private double Round(double value) { /* ... */ }
}
```

In the above example, everything in the class exists to calculate tax. Nothing could be moved out without breaking that job.

## Fixing low cohesion

The fix is almost always the same: **split the class by job**.

_Example:_

```csharp
// -------- Bad practice ---------
class ReportGenerator {
    public List<Sale> LoadSales() { /* database code */ }
    public string FormatHtml(List<Sale> sales) { /* HTML code */ }
    public void Email(string html) { /* email code */ }
}
```

```csharp
// -------- Good practice ---------
class SalesRepository {
    public List<Sale> LoadSales() { /* database code */ }
}

class SalesReportFormatter {
    public string FormatHtml(List<Sale> sales) { /* HTML code */ }
}

class EmailSender {
    public void Send(string html) { /* email code */ }
}
```

In the above example, the bad class changes whenever the database, the HTML layout **or** the email setup changes. After the split, each class has one job and one reason to change.
