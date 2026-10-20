# Object Calisthenics: C# smell / fix pairs

## 1. One level of indentation

```csharp
// smell
foreach (var order in orders) {
    if (order.IsActive) {
        foreach (var line in order.Lines) { Process(line); }
    }
}

// fix
foreach (var order in orders)
{
    ProcessActiveLines(order);
}
```

## 2. No else

```csharp
// smell
if (status == Approved) { Publish(); } else { Reject(); }

// fix: guard clause
if (status != Approved)
{
    Reject();
    return;
}
Publish();
```
Chains on an enum/type -> `switch` expression or a dictionary of strategies.

## 3. Wrap primitives

```csharp
// smell
void Submit(long orderId, long approverId)   // easy to swap

// fix
public readonly record struct OrderId(long Value);
public readonly record struct ApproverId(long Value);
void Submit(OrderId order, ApproverId approver)
```
Validate in the constructor so invalid values can't exist.

## 4. First-class collections

```csharp
// smell: class has List<Issue> plus other fields, callers do .Where(i => i.IsBlocking).Any()
// fix
public sealed class Issues
{
    private readonly List<Issue> _issues;
    public bool HasBlocking() => _issues.Exists(issue => issue.IsBlocking);
}
```

## 5. One dot per line

```csharp
// smell
var city = order.Customer.Address.City;

// fix
var city = order.ShippingCity();   // Order delegates internally
```
Exempt: `builder.Add(...).Add(...)`, LINQ, `ConfigureAwait`, `?.` null chains, `nameof`/static access.

## 6. No abbreviations

`cfg` -> `config`, `svc` -> `service`, `resp` -> `response`, `cust` -> `customer`, `mgr` -> `manager`.

## 7. Small entities

Extract method by intent; extract class when a group of private methods shares one field subset. Large `*Task`/`*Job`/`*Handler` classes usually split into: fetch, decide, act.

## 8. Two instance variables

```csharp
// smell: 5 injected deps
// fix options:
//  - split by responsibility (query vs command side)
//  - group related deps into one collaborator (e.g. PaymentGateway wrapping client + converter)
```
Do not hide deps behind a service-locator or a "context" bag to pass the count.

## 9. No getters/setters

```csharp
// smell
if (order.Status == Status.Approved && order.Expiry < now) { ... }

// fix
if (order.IsExpiredApproval(now)) { ... }
```
Use for domain objects only. Leave EF entities, DTOs, and GraphQL types alone.
