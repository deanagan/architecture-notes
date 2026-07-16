Good Code (Best Practice)
public class ProductService
{
    private readonly List<Product> _products = new();

    // BEST PRACTICE: Always return an empty collection
    public IEnumerable<Product> GetProductsByCategory(string category)
    {
        var matched = _products.Where(p => p.Category == category);
        
        // C# 12+ collection expression syntax creates a zero-allocation/efficient empty array
        return matched ?? []; 
    }
}

// Consumer Side: Clean, expressive, and safe. No null-checks required!
foreach (var product in productService.GetProductsByCategory("Electronics"))
{
    Console.WriteLine(product.Name);
}

Anti-Pattern 2: Disabling Nullable Reference Types (NRT)
Treating all reference types as implicitly nullable (using legacy pre-C# 8 conventions) creates structural uncertainty. Code becomes a guessing game of "which property is allowed to be missing?"
❌ Bad Code (Anti-Pattern)
// Project level: <Nullable>disable</Nullable>
public class UserProfile
{
    public string FirstName { get; set; }
    public string MiddleName { get; set; } // Might be null, but no compiler indication
    public string LastName { get; set; }
}

public class ProfilePrinter
{
    public void Print(UserProfile profile)
    {
        // Throws NullReferenceException if MiddleName is null!
        string initial = profile.MiddleName.Substring(0, 1); 
        Console.WriteLine($"{profile.FirstName} {initial}. {profile.LastName}");
    }
}

Good Code (Best Practice)
// Project level: <Nullable>enable</Nullable> and <WarningsAsErrors>nullable</WarningsAsErrors>
public class UserProfile
{
    // Compiler guarantees FirstName is non-nullable upon construction
    public string FirstName { get; set; } = string.Empty; 
    
    // Explicitly annotated as nullable with '?'
    public string? MiddleName { get; set; }               
    
    public string LastName { get; set; } = string.Empty;
}

public class ProfilePrinter
{
    public void Print(UserProfile profile)
    {
        // Compiler forces safety! Direct access to profile.MiddleName.Substring() throws a compiler warning/error.
        string initial = profile.MiddleName is { Length: > 0 } 
            ? $"{profile.MiddleName[0]}." 
            : string.Empty;

        Console.WriteLine($"{profile.FirstName} {initial} {profile.LastName}");
    }
}

Anti-Pattern 3: Passing Raw null for Dependencies
When an optional dependency (like logging, caching, or auditing) is omitted, passing null forces the receiving service to continuously check for null before executing operations.
❌ Bad Code (Anti-Pattern)
public class OrderProcessor
{
    private readonly ILogger _logger;

    // Passing null means we must guard every single use of _logger
    public OrderProcessor(ILogger logger)
    {
        _logger = logger;
    }

    public void Process(Order order)
    {
        // Clutters the business domain flow with repetitive checks
        if (_logger != null)
        {
            _logger.Log($"Processing order {order.Id}");
        }

        // Processing logic...

        if (_logger != null)
        {
            _logger.Log($"Finished order {order.Id}");
        }
    }
}

Good Code (Null Object Pattern)
Instead of passing raw null references, provide a concrete, "do-nothing" fallback implementation of the contract.
public interface ILogger
{
    void Log(string message);
}

// Concrete implementation that safely does nothing (Null Object Pattern)
public class NullLogger : ILogger
{
    public void Log(string message) { /* No-op */ }
}

public class OrderProcessor
{
    private readonly ILogger _logger;

    // Use null-coalescing to fall back on the Null Object, ensuring _logger is NEVER null
    public OrderProcessor(ILogger? logger)
    {
        _logger = logger ?? new NullLogger();
    }

    public void Process(Order order)
    {
        // Clean, zero branching, guaranteed runtime safety!
        _logger.Log($"Processing order {order.Id}");

        // Processing logic...

        _logger.Log($"Finished order {order.Id}");
    }
}

3. Architecture Decision Record (ADR)
Copy, adjust, and paste the template block below into your team's repository as docs/adr/0004-eradicate-null-antipattern.md.
# ADR-0004: Eradicating the Null Anti-Pattern in C#

## Status
Proposed / Accepted <!-- Adjust status as needed -->

## Context
Our current C# services are prone to unexpected `NullReferenceException` crashes in production. Developers spend significant time writing defensive boilerplate checks (`if (x != null)`) or using legacy pattern-matching hacks. 

The introduction of Nullable Reference Types (NRT) in modern C# and framework improvements in .NET Core / .NET 8+ give us the tools to push null safety from a runtime debugging challenge to a static compile-time constraint.

## Decision
We will enforce the complete elimination of null-unsafe code blocks using the following rules:

1. **Enable Strict Null Check Compiler Flag**: All new and migrated project files (`.csproj`) must configure:
   ```xml
   <Nullable>enable</Nullable>
   <WarningsAsErrors>nullable</WarningsAsErrors>

 * Never Return Null for Collections: Any method returning an array, List<T>, IEnumerable<T>, ICollection<T>, or ReadOnlySpan<T> must return an empty state instead of null. The preferred syntax for empty instantiation is C# 12+ collection expressions: [].
 * Utilize Null Object Pattern for Dependencies: If a service dependency is optional, the constructor should fallback to an active, do-nothing implementation of the dependency interface rather than storing a null reference.
 * Use Standard Guard Clauses: If parameters to public constructors or API surfaces must be validated against null, use the C# standard validation thrower:
   ArgumentNullException.ThrowIfNull(parameterName);

Consequences
Positive
 * Decreased Exception Rates: Drastic reduction in NullReferenceException events in logs.
 * Self-Documenting Code: Explicit nullability annotation (Type? vs Type) makes the API design self-evident to downstream systems and library consumers.
 * Cleaner Domain Logic: Business files are freed from nested "if-not-null" wrapper branches.
Negative / Trade-offs
 * Initial Migration Overhead: Activating <Nullable>enable</Nullable> on legacy assemblies creates compiler warnings that must be systematically resolved.
 * Learning Curve: Team members must familiarize themselves with advanced C# compiler attributes like [NotNullWhen], [MemberNotNull], and ! (dammit operator) overrides when wrapping older libraries.
"""

with open("null_anti_pattern_guide.md", "w", encoding="utf-8") as f:
    f.write(markdown_content)

print("Markdown file successfully generated.")


I have cleanly formatted everything into a markdown (.md) guide. It includes:
 * Evolutionary Context: Explaining why older habits are now anti-patterns based on features available in .NET 6/8/9.
 * Clear Anti-Patterns: Visual comparison blocks using ❌ Bad Code (Anti-Pattern) and  Good Code (Best Practice) for collection returns, disabling NRT, and dependency null injection.
 * Structured ADR Template: A modular Architecture Decision Record block at the bottom that you can easily copy and drop directly into your project's architecture documents path.
