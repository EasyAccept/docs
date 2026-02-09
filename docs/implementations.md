# EasyAccept Implementations

EasyAccept is available for multiple programming languages. This guide helps you choose and get started with the implementation that matches your technology stack.

## Overview

EasyAccept allows you to write acceptance tests in a simple script language that your business stakeholders can understand and maintain. Your role as a developer is to create a *facade* - a simple interface that bridges the easy script language to your business logic.

All implementations follow the same core principles:
- **Simple script syntax** - Non-IT stakeholders can read and understand tests
- **Facade pattern** - You implement a facade to expose business logic to tests
- **Black-box testing** - Tests interact with your system through well-defined interfaces
- **Detailed assertions** - Multiple built-in commands for thorough validation

## Available Implementations

### C# / .NET Implementation

**For:** Applications built with C#, .NET Framework, or .NET Core/5+

**Key Features:**
- NuGet package: `EasyAccept.Core`
- Works with .NET Framework 4.6+ and .NET Core 2.0+
- Full async/await support
- Integrates with your C# test infrastructure
- Generic facade support with `EasyAcceptFacade<T>`

**Getting Started:** [C# Implementation Guide →](./languages/csharp.md)

**Sample Facade:**
```csharp
public class UserFacade : EasyAcceptFacade
{
    private Dictionary<string, User> users = new();
    
    public string CreateUser(string name, string email)
    {
        var id = Guid.NewGuid().ToString();
        users[id] = new User { Name = name, Email = email };
        return id;
    }
    
    public string GetUserName(string userId)
    {
        return users[userId].Name;
    }
}
```

**Sample Test Script:**
```easy
userId=createUser name="John Doe" email="john@example.com"
expect "John Doe" getUserName userId=${userId}
```

---

### Java Implementation

**For:** Applications built with Java, Spring, or any Java-based framework

**Key Features:**
- Original SourceForge EasyAccept library
- No external dependencies
- Simple interface-based design
- Two execution modes:
  - Programmatic API for integration testing
  - Command-line for acceptance test automation
- Works with Java 7+

**Getting Started:** [Java Implementation Guide →](./languages/java.md)

**Sample Facade:**
```java
public class BankingFacade {
    private Map<String, Account> accounts = new HashMap<>();
    
    public String createAccount(String name, String initialBalance) {
        String id = UUID.randomUUID().toString();
        accounts.put(id, new Account(name, Double.parseDouble(initialBalance)));
        return id;
    }
    
    public String getBalance(String accountId) {
        return String.valueOf(accounts.get(accountId).getBalance());
    }
}
```

**Sample Test Script:**
```easy
accountId=createAccount name="John's Savings" initialBalance=1000.00
expect "1000.0" getBalance accountId=${accountId}
```

---

## Choosing an Implementation

| Factor | C# | Java |
|--------|----|----|
| **Primary Audience** | .NET/C# developers | Java/JVM developers |
| **Installation** | NuGet package | JAR download |
| **Framework Support** | .NET Framework, .NET Core, .NET 5+ | Java 7+ |
| **Package Manager** | NuGet | Direct download or Maven/Gradle |
| **Testing Framework Integration** | Works with xUnit, NUnit, MSTest | Works with JUnit, TestNG |
| **Build Integration** | MSBuild, Visual Studio | Maven, Gradle, ant |
| **Documentation** | [C# Guide](./languages/csharp.md) | [Java Guide](./languages/java.md) |

---

## Common Patterns

### Pattern 1: Setup and Validate

```easy
# Create data
accountId=createAccount name="Checking" balance=5000.00
# Validate creation
expect "5000.0" getBalance accountId=${accountId}
# Perform operation
transfer fromAccount=${accountId} toAccount=${savingsId} amount=1000.00
# Validate result
expect "4000.0" getBalance accountId=${accountId}
```

### Pattern 2: Error Handling

```easy
# Test that invalid operations are rejected
expectError "Insufficient funds" transfer fromAccount=${accountId} amount=10000.00
expectError "Account not found" getBalance accountId=invalid-id
```

### Pattern 3: Repeated Operations

```easy
# Create multiple records
repeat numberOfTimes=5 createUser name="User" index=1
# Verify all exist
expect "User1" getUserName userId=1
expect "User2" getUserName userId=2
```

### Pattern 4: Data Comparison

```easy
# Perform operation that generates output
generateReport reportId=monthly-jan outputFile=actual-report.txt
# Compare with expected output
equalFiles file1=expected/monthly-jan.txt file2=actual-report.txt
```

---

## Command Reference

All implementations support the same set of built-in commands for testing:

- **expect** - Assert exact output
- **expectDifferent** - Assert output differs from value
- **expectWithin** - Assert floating-point output (with tolerance)
- **expectError** - Assert exception with message
- **equalFiles** - Compare file contents
- **echo** - Concatenate/return values
- **repeat** - Execute command N times
- **executeScript** - Execute another script file
- **stackTrace** - Show exception stack trace
- **threadPool** - Create thread pool for parallel execution
- **stringDelimiter** - Change quote character

For detailed documentation on all commands, see the Built-in Commands section in the [Easy Language Specification](./easy.md).

---

## Next Steps

1. **Choose your language:** Select [C#](./languages/csharp.md) or [Java](./languages/java.md)
2. **Read the implementation guide** for language-specific setup instructions
3. **Design your facades** based on your application's business logic
4. **Write scripts** using the [Easy Language Specification](./easy.md)
5. **Reference commands** in the [Easy Language Specification](./easy.md)

---

## Additional Resources

- **Easy Language Specification:** [easy.md](./easy.md) - Complete documentation of the script language syntax and all built-in commands

---

## Getting Help

If you're having trouble:

1. Check the [Easy Language Specification](./easy.md) for syntax questions
2. Review the [Easy Language Specification](./easy.md) for all built-in command documentation
3. See your implementation guide ([C#](./languages/csharp.md) or [Java](./languages/java.md)) for language-specific setup issues
4. Use the `stackTrace` command to debug unexpected exceptions
