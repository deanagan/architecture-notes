Handling null haphazardly is often called Sir Antony Hoare's "billion-dollar mistake." In modern C#, relying on traditional null checks or returning raw null values when an alternative exists is considered an anti-pattern. Because C# has evolved dramatically over its versions, our defensive strategies should too.
Here is a comprehensive guide to the Null Anti-Pattern in C#, structured to easily migrate into your Architecture Decision Record (ADR).
1. The Evolutionary Context
Modern C# provides advanced compiler features to push null errors from runtime to compile time. When drafting an ADR, it's critical to note why old strategies are now considered anti-patterns based on available features:
2. Anti-Pattern 1: Returning null for Collections
Returning a null reference instead of an empty collection forces the caller to write defensive boilerplates, risking a NullReferenceException if they try to iterate or use LINQ.
Bad Code
public class OrderService
{
    // ANTI-PATTERN: Returns null if no orders match the criteria
    public IEnumerable<Order> GetOrdersByCustomer(int customerId)
    {
        var orders = _database.Find(o => o.CustomerId == customerId);
        if (orders == null || !orders.Any())
        {
            return null; 
        }
        return orders;
    }
}

// Consumer Side: Forced to write defensive boilerplates
var orders = orderService.GetOrdersByCustomer(42);
if (orders != null) // If missed, foreach throws NullReferenceException
{
    foreach (var order in orders) { /* ... */ }
}

Good Code
public class OrderService
{
    // BEST PRACTICE: Always return an empty collection expression
    public IEnumerable<Order> GetOrdersByCustomer(int customerId)
    {
        var orders = _database.Find(o => o.CustomerId == customerId);
        
        // C# 12+ collection expression syntax creates a memory-efficient empty array/enumerable
        return orders ?? []; 
    }
}

// Consumer Side: Safe, clean, and expressive
foreach (var order in orderService.GetOrdersByCustomer(42))
{
    // Runs cleanly even if 0 results exist, no null checks required
}

3. Anti-Pattern 2: Missing or Ignored Nullable Reference Types (NRT)
Treating all reference types as implicitly nullable leads to structural uncertainty. Developers either over-engineer with defensive if (x == null) conditions everywhere, or under-engineer and suffer runtime crashes.
Bad Code
// Project context: <Nullable>disable</Nullable> or implicitly ignored
public class UserProfile
{
    public string FirstName { get; set; }
    public string MiddleName { get; set; } // Might be absent, but no distinction in type system
    public string LastName { get; set; }
}

public void PrintUser(UserProfile user)
{
    // Is middle name safe? Is last name safe? Total guessing game.
    Console.WriteLine($"{user.FirstName} {user.MiddleName.Substring(0, 1)}. {user.LastName}"); 
}

Good Code
// Project context: <Nullable>enable</Nullable> and <WarningsAsErrors>nullable</WarningsAsErrors>
public class UserProfile
{
    public string FirstName { get; set; } = string.Empty; // Non-nullable, compiler enforces default
    public string? MiddleName { get; set; }               // Explicitly allows null
    public string LastName { get; set; } = string.Empty;
}

public void PrintUser(UserProfile user)
{
    // The compiler enforces safety check on MiddleName using the null-conditional operator
    string initial = user.MiddleName?.Substring(0, 1) ?? string.Empty;
    
    Console.WriteLine($"{user.FirstName} {initial} {user.LastName}");
}

4. Anti-Pattern 3: The "Magic Value" or Raw Null Object (Behavioral Absence)
When a dependency is missing (e.g., a logging or caching mechanism), passing null turns every method invocation into a potential minefield of null checks.
Bad Code
public class PaymentProcessor
{
    private readonly ILogger _logger;

    // Passing null here means we have to shield every single call
    public PaymentProcessor(ILogger logger)
    {
        _logger = logger; 
    }

    public void Process()
    {
        // Excessive clutter throughout domain logic
        if (_logger != null) 
        {
            _logger.Log("Processing payment...");
        }
        
        // ... Core core logic ...
    }
}

Good Code (Null Object Pattern)
Instead of executing conditional branches, provide a deliberate, concrete "do-nothing" implementation of the interface.
public interface ILogger 
{ 
    void Log(string message); 
}

// Concrete implementation that safely does nothing
public class NullLogger : ILogger
{
    public void Log(string message) { /* No-op */ }
}

public class PaymentProcessor
{
    private readonly ILogger _logger;

    // Use a fallback to ensure _logger is NEVER null
    public PaymentProcessor(ILogger? logger)
    {
        _logger = logger ?? new NullLogger(); 
    }

    public void Process()
    {
        // Safe, clean, zero conditionals required
        _logger.Log("Processing payment..."); 
    }
}

