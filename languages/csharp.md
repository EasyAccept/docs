# Getting Started with CSharp

EasyAccept for C# is a powerful library that allows you to integrate acceptance testing into your .NET projects. This guide will help you get started with the [EasyAccept.Core](https://www.nuget.org/packages/EasyAccept.Core/) NuGet package.

## Installation

### Via NuGet Package Manager

Install the EasyAccept.Core NuGet package:

```bash
dotnet add package EasyAccept.Core
```

Or using the Package Manager Console:

```powershell
Install-Package EasyAccept.Core
```

## Understanding the Easy Language

Before using the C# library, it's important to understand the EasyAccept test script language. The Easy language is a simple, client-readable scripting format used to write acceptance tests. Test scripts are plain text files with commands that call your business logic and verify results.

### Basic Concepts

The Easy language supports:

- **Commands**: Calls to your business logic methods
- **Assertions**: Built-in commands like `expect`, `expectDifferent`, and `expectError`
- **Variables**: Store and reuse command results
- **Echo**: Print output for debugging

### Example Easy Script

```easy
# Create a user and store the ID
userId=createUser name="John Doe" email="john@example.com"

# Verify the user was created
expect "John Doe" getUserName userId=${userId}

# Check that an error is thrown for invalid email
expectError "Invalid email format" createUser name="Jane Doe" email="invalid"
```

For complete documentation on the Easy language, including all built-in commands, refer to the [Easy Language Specification](../easy.md).

## Creating Your Facade

The facade is the bridge between your business logic and EasyAccept. It exposes public methods that can be called from test scripts.

### Facade Requirements

1. **Public methods only** - Only public methods can be called from test scripts
2. **Simple parameter types** - Support: `string`, `bool`, `char`, `byte`, `short`, `int`, `long`, `float`, `double`
3. **Simple return types** - Either `void` or `string`
4. **No side effects in output** - Your facade should communicate through return values, not by printing
5. **Throw exceptions for errors** - Use exceptions to signal test failures

### Example Facade

```csharp
public class CalculatorFacade
{
    private Dictionary<string, int> _values = new Dictionary<string, int>();

    public string Add(int a, int b)
    {
        return (a + b).ToString();
    }

    public void SetValue(string name, int value)
    {
        _values[name] = value;
    }

    public string GetValue(string name)
    {
        if (!_values.ContainsKey(name))
            throw new Exception($"Value '{name}' not found");

        return _values[name].ToString();
    }

    public void ValidatePositive(int value)
    {
        if (value < 0)
            throw new Exception("Value must be positive");
    }
}
```

## Basic Usage

### Step 1: Create Your Facade

Create a class with public methods that implement your business logic.

### Step 2: Create Test Scripts

Create `.easy` files with your test commands:

```easy
# tests/calculator.easy
add param1=5 param2=3
expect "8" Add param1=5 param2=3

setValue name="pi" value=31415
expect "31415" getValue name="pi"

validatePositive value=5
expectError "Value must be positive" validatePositive value=-1
```

### Step 3: Execute Tests

```csharp
using EasyAccept.Core;
using System.Collections.Generic;

var facade = new CalculatorFacade();
var testFiles = new List<string> { "tests/calculator.easy" };

var easyAccept = new EasyAcceptFacade<CalculatorFacade>(facade, testFiles);
easyAccept.ExecuteTests();

// Get results
Console.WriteLine(easyAccept.GetCompleteResults());
```

## API Overview

The `EasyAcceptFacade<T>` class provides these key methods:

### Execution

- **`ExecuteTests()`** - Executes all test scripts provided during initialization

### Results Retrieval

- **`GetCompleteResults()`** - Returns detailed results including failures
- **`GetSummarizedResults()`** - Returns summary of passed/failed tests
- **`GetScriptCompleteResults(string scriptFile)`** - Results for a specific script
- **`GetScriptSummarizedResults(string scriptFile)`** - Summary for a specific script

### Statistics

- **`GetTotalNumberOfPassedTests()`** - Count of all passed tests
- **`GetTotalNumberOfNotPassedTests()`** - Count of all failed tests
- **`GetScriptNumberOfPassedTests(string scriptFile)`** - Count for a specific script
- **`GetScriptNumberOfNotPassedTests(string scriptFile)`** - Count for a specific script
- **`GetScriptFailures(string scriptFile)`** - Get failed test details

### Results Listening

- **`AddResultsListener(IResultsListener listener)`** - Register a listener for test results

## Complete Example

Here's a complete example of setting up and running acceptance tests:

```csharp
using EasyAccept.Core;
using System;
using System.Collections.Generic;

public class UserFacade
{
    private Dictionary<int, string> _users = new Dictionary<int, string>();
    private int _nextId = 1;

    public string CreateUser(string name)
    {
        if (string.IsNullOrEmpty(name))
            throw new Exception("Name cannot be empty");

        int id = _nextId++;
        _users[id] = name;
        return id.ToString();
    }

    public string GetUserName(int userId)
    {
        if (!_users.ContainsKey(userId))
            throw new Exception($"User {userId} not found");

        return _users[userId];
    }

    public void DeleteUser(int userId)
    {
        if (!_users.ContainsKey(userId))
            throw new Exception($"User {userId} not found");

        _users.Remove(userId);
    }
}

public class Program
{
    public static void Main()
    {
        var facade = new UserFacade();
        var testFiles = new List<string> { "tests/user.easy" };

        var easyAccept = new EasyAcceptFacade<UserFacade>(facade, testFiles);
        easyAccept.ExecuteTests();

        // Display results
        Console.WriteLine(easyAccept.GetCompleteResults());

        // Check statistics
        Console.WriteLine($"Passed: {easyAccept.GetTotalNumberOfPassedTests()}");
        Console.WriteLine($"Failed: {easyAccept.GetTotalNumberOfNotPassedTests()}");
    }
}
```

**Test Script (tests/user.easy):**

```easy
# Create a user
userId=createUser name="Alice"
expect "1" echo ${userId}

# Verify user exists
expect "Alice" getUserName userId=1

# Try to get non-existent user
expectError "User 2 not found" getUserName userId=2

# Delete user and verify
deleteUser userId=1
expectError "User 1 not found" getUserName userId=1
```

## Advanced Features

### Working with Variables

The Easy language allows you to capture command results in variables and reuse them:

```easy
result=createUser name="Bob"
expect "Bob" getUserName userId=${result}
```

### Using Echo for Debugging

The `echo` command helps you debug and understand test flow:

```easy
echo "Starting user creation test"
userId=createUser name="Charlie"
echo "Created user with ID:" ${userId}
```

### Handling Errors

Use `expectError` to verify that your business logic properly throws exceptions:

```easy
expectError "User not found" getUserName userId=999
```

### Multiple Assertions

Chain multiple assertions to test different aspects:

```easy
createUser name="David"
expect "David" getUserName userId=1
createUser name="Eve"
expect "Eve" getUserName userId=2
```

## Repository and Examples

For more examples and the complete source code, visit the [EasyAccept C# repository](https://github.com/EasyAccept/easyaccept-csharp).

You can find test examples in the repository that demonstrate:

- Basic command execution
- Variable handling
- Error handling
- Multiple test files
- Results aggregation

## Troubleshooting

### Method Not Found Exception

Ensure:

- The method name matches exactly (case-sensitive)
- The number of parameters matches
- Parameter types are supported (string, int, bool, etc.)

### Syntax Errors in Scripts

Check your `.easy` files for:

- Correct command syntax
- Proper parameter naming (e.g., `paramName=value`)
- String delimiters (default is `"`)

### Facade Issues

Remember:

- Facade methods must be public
- Return types must be void or string
- Throw exceptions for error conditions
- Don't print directly; use return values instead

## Getting Help

- Check the [Easy Language Specification](../easy.md) for language details
- See the [Easy Language Specification](../easy.md) for all built-in command documentation
- See [All Implementations](../implementations.md) for other language options
- Review the [GitHub repository](https://github.com/EasyAccept/easyaccept-csharp) for examples
- Visit the [NuGet package page](https://www.nuget.org/packages/EasyAccept.Core/) for version information
