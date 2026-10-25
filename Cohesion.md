# Cohesion

Cohesion is **the degree to which the members of a class work together to fulfil a single, well-defined responsibility**.

> [!IMPORTANT]
> A class should have **one responsibility** and **one reason to change**. If a class has more responsibilities, SRP is violated (S in [SOLID](SOLID.md)). The same idea applies to methods, on a smaller scale.

---

#### TOC

- [High vs low cohesion](#high-vs-low-cohesion)
- [The 7 types](#the-7-types)
- [Coincidental cohesion](#coincidental-cohesion)
- [Logical cohesion](#logical-cohesion)
- [Temporal cohesion](#temporal-cohesion)
- [Procedural cohesion](#procedural-cohesion)
- [Communicational cohesion](#communicational-cohesion)
- [Sequential cohesion](#sequential-cohesion)
- [Functional cohesion](#functional-cohesion)
- [Fixing low cohesion](#fixing-low-cohesion)

---

## High vs low cohesion

| Advantages of **high** cohesion                                                     | Disadvantages of **low** cohesion                                                     |
| ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| **Improved readability**: clear, focused classes with a single, well-defined responsibility | **Increased code duplication**: members that belong together get split across classes |
| **Better error isolation**: a change is less likely to affect other parts           | **Reduced functionality**: classes without a single responsibility, holding members that don't belong together |
| **Improved reliability**: less prone to errors, works more consistently              | **Hard to understand**: harder to tell what each class is responsible for             |

## The 7 types

```mermaid
flowchart LR
    A[Coincidental] --> B[Logical] --> C[Temporal] --> D[Procedural] --> E[Communicational] --> F[Sequential] --> G[Functional]
    style G fill:#d4edda,stroke:#1e8449
```

_Functional (green) is the best kind._

> [!TIP]
> To spot the type, ask: **why** are these members in the same class?

| Type            | Grouped because...                                   |
| --------------- | ---------------------------------------------------- |
| Coincidental    | No conceptual relationship at all                    |
| Logical         | Logically related, but not functionally related      |
| Temporal        | They're executed at the same _time_                  |
| Procedural      | They always happen in a specific _order_             |
| Communicational | They operate on the same _data_                      |
| Sequential      | One's _output_ becomes the next one's _input_        |
| Functional      | They all contribute to _one well-defined job_        |

## Coincidental cohesion

Coincidental cohesion is:

- Class members with **no conceptual relationship**.
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

- Members that are **logically related** (the same kind of thing), but **not functionally related** (they don't work together).

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

In the above example, keyboard, file and network are all "input", but they share no code or data. The `source` flag is also **control coupling** (see [Coupling](Coupling.md)).

## Temporal cohesion

Temporal cohesion is:

- Members grouped because they're **executed at the same time**, e.g. at start-up.

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

- Statements grouped in a single method because they **always happen in a specific order**, even though they work on different things.

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

In the above example, the steps follow an order, but they don't pass data to each other.

## Communicational cohesion

Communicational cohesion is:

- Members grouped because they **operate on the same data**.

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

- The **output** of one member becomes the **input** of another, like an assembly line.

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

- **All members contribute to a single, well-defined responsibility**. The best kind.

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

The lecture gives 2 refactorings: **extract class** and **extract method**.

### Extract class

Split a class that does several jobs into one class per job.

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

### Extract method

Split a method that does several things into smaller methods, each with one job.

_Example:_

```csharp
// -------- Bad practice ---------
public void PrintInvoice(Invoice invoice) {
    Console.WriteLine("INVOICE");
    Console.WriteLine("-------");

    double total = 0;
    foreach (Item item in invoice.Items) {
        total += item.Price;
    }
    Console.WriteLine("Total: " + total);
}
```

```csharp
// -------- Good practice ---------
public void PrintInvoice(Invoice invoice) {
    PrintHeader();
    PrintTotal(invoice);
}

private void PrintHeader() {
    Console.WriteLine("INVOICE");
    Console.WriteLine("-------");
}

private void PrintTotal(Invoice invoice) {
    double total = 0;
    foreach (Item item in invoice.Items) {
        total += item.Price;
    }
    Console.WriteLine("Total: " + total);
}
```

In the above example, the bad method mixes printing the header with calculating the total. After extracting, each method does one thing and its name says what.
