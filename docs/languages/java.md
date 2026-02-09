# Getting Started with Java

EasyAccept for Java is the original implementation of the acceptance testing framework. This guide will help you get started with EasyAccept in your Java projects.

## Installation

### Download the Library

Download the latest EasyAccept JAR file from the [SourceForge project page](https://sourceforge.net/projects/easyaccept/files/).

### Add to Your Project

#### Using Maven

Add the dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>easyaccept</groupId>
    <artifactId>easyaccept</artifactId>
    <version>1.0</version>
    <scope>test</scope>
</dependency>
```

#### Using Gradle

Add to your `build.gradle`:

```gradle
testImplementation 'easyaccept:easyaccept:1.0'
```

#### Manual Setup

1. Download the EasyAccept JAR file
2. Add it to your project's classpath
3. Ensure Java 8 or higher is installed

For more information, visit [EasyAccept on SourceForge](https://easyaccept.sourceforge.net/).

## Understanding the Easy Language

The Easy language is a simple, client-readable scripting format designed for acceptance testing. Test scripts are plain text files with commands that call your business logic and verify results. The language is intentionally non-technical to be accessible to business stakeholders.

### Basic Concepts

The Easy language supports:

- **Commands**: Method calls to your business logic
- **Parameters**: Named parameters in `name=value` format
- **Assertions**: Built-in commands like `expect`, `expectDifferent`, `expectError`, and `expectWithin`
- **Variables**: Store and reuse command results with `${variableName}` syntax
- **Built-in Commands**: `echo`, `quit`, `executeScript`, `repeat`, `threadPool`, `stackTrace`

### Example Easy Script

```easy
# Create a user
userId=createUser name="John Doe" birthdate=1990/01/15

# Verify the user was created
expect "John Doe" getUserName key=${userId}

# Check salary is correct (within precision)
expectWithin 0.01 50000.00 getSalary key=${userId}

# Check for error on invalid data
expectError "Invalid date format" createUser name="Jane Doe" birthdate=1990/13/45
```

For complete documentation on the Easy language, including all built-in commands, refer to the [Easy Language Specification](../easy.md).

## Creating Your Facade

The facade is the bridge between your business logic and EasyAccept. It exposes public methods that will be called from test scripts. The facade should encapsulate your business logic while keeping the interface simple and testable.

### Facade Requirements

1. **Public methods only** - Only public methods can be called from test scripts
2. **Simple parameter types** - Support: `String`, `boolean`, `char`, `byte`, `short`, `int`, `long`, `float`, `double`
3. **Simple return types** - Either `void` or `String`
4. **No side effects in output** - Your facade should communicate through return values, not System.out.println()
5. **Throw exceptions for errors** - Use exceptions to signal test failures
6. **Separate business logic from UI** - EasyAccept tests the business logic, not the user interface

### Example Facade

```java
import java.util.HashMap;
import java.util.Map;

public class UserFacade {
    private Map<String, User> users = new HashMap<>();
    private int nextId = 1;

    public String createUser(String name, String birthdate) {
        if (name == null || name.isEmpty()) {
            throw new RuntimeException("Name cannot be empty");
        }

        // Validate birthdate format (YYYY/MM/DD)
        if (!isValidDate(birthdate)) {
            throw new RuntimeException("Invalid date format");
        }

        String userId = String.valueOf(nextId++);
        users.put(userId, new User(userId, name, birthdate));
        return userId;
    }

    public String getUserName(String key) {
        if (!users.containsKey(key)) {
            throw new RuntimeException("User not found");
        }
        return users.get(key).getName();
    }

    public String getUserBirthdate(String key) {
        if (!users.containsKey(key)) {
            throw new RuntimeException("User not found");
        }
        return users.get(key).getBirthdate();
    }

    public void deleteUser(String key) {
        if (!users.containsKey(key)) {
            throw new RuntimeException("User not found");
        }
        users.remove(key);
    }

    private boolean isValidDate(String date) {
        String[] parts = date.split("/");
        if (parts.length != 3) return false;
        try {
            int year = Integer.parseInt(parts[0]);
            int month = Integer.parseInt(parts[1]);
            int day = Integer.parseInt(parts[2]);
            return month >= 1 && month <= 12 && day >= 1 && day <= 31;
        } catch (NumberFormatException e) {
            return false;
        }
    }

    private static class User {
        private String id;
        private String name;
        private String birthdate;

        public User(String id, String name, String birthdate) {
            this.id = id;
            this.name = name;
            this.birthdate = birthdate;
        }

        public String getName() { return name; }
        public String getBirthdate() { return birthdate; }
    }
}
```

## Basic Usage

### Step 1: Create Your Facade

Create a class with public methods that implement your business logic:

```java
public class CalculatorFacade {
    public String add(int a, int b) {
        return String.valueOf(a + b);
    }

    public String multiply(int a, int b) {
        return String.valueOf(a * b);
    }

    public String divide(int a, int b) {
        if (b == 0) {
            throw new RuntimeException("Division by zero");
        }
        return String.valueOf(a / b);
    }
}
```

### Step 2: Create Test Scripts

Create `.txt` or `.easy` files with your test commands:

```easy
# tests/calculator.txt
add param1=5 param2=3
expect "8" add param1=5 param2=3

multiply param1=4 param2=7
expect "28" multiply param1=4 param2=7

expectError "Division by zero" divide param1=10 param2=0
expect "2" divide param1=10 param2=5
```

### Step 3: Execute Tests Using the API

```java
import java.util.ArrayList;
import java.util.List;
import easyaccept.EasyAcceptFacade;

public class TestRunner {
    public static void main(String[] args) throws Exception {
        // Create list of test script files
        List<String> files = new ArrayList<String>();
        files.add("tests/calculator.txt");

        // Instantiate your facade
        CalculatorFacade facade = new CalculatorFacade();

        // Instantiate EasyAccept facade
        EasyAcceptFacade eaFacade = new EasyAcceptFacade(facade, files);

        // Execute the tests
        eaFacade.executeTests();

        // Print the complete results
        System.out.println(eaFacade.getCompleteResults());
    }
}
```

### Step 4: Run from Command Line (Alternative)

You can also run tests directly from the command line:

```bash
java -classpath easyaccept.jar:. easyaccept.EasyAccept \
  com.example.CalculatorFacade \
  tests/calculator.txt
```

The exit code will be:

- `0` if all tests passed
- Non-zero if any tests failed

## API Overview

The `EasyAcceptFacade` class provides the following key methods:

### Execution

- **`void executeTests()`** - Executes all test scripts provided during initialization

### Results Retrieval

- **`String getCompleteResults()`** - Returns detailed results including failure descriptions
- **`String getSummarizedResults()`** - Returns summary of passed/failed test counts
- **`String getScriptCompleteResults(String scriptFile)`** - Results for a specific script
- **`String getScriptSummarizedResults(String scriptFile)`** - Summary for a specific script
- **`List<Result> getScriptResults(String scriptFile)`** - Get all Result objects for a script

### Statistics

- **`int getTotalNumberOfPassedTests()`** - Count of all passed tests
- **`int getTotalNumberOfNotPassedTests()`** - Count of all failed tests
- **`int getScriptNumberOfPassedTests(String scriptFile)`** - Count for a specific script
- **`int getScriptNumberOfNotPassedTests(String scriptFile)`** - Count for a specific script
- **`int getTotalNumberOfTests()`** - Total test count
- **`List<Result> getScriptFailures(String scriptFile)`** - Get failed test details

## Complete Example

Here's a complete example of setting up and running acceptance tests:

### Facade Implementation

```java
import java.util.HashMap;
import java.util.Map;

public class BankingFacade {
    private Map<String, Account> accounts = new HashMap<>();
    private int nextId = 1000;

    public String createAccount(String owner, String initialBalance) {
        String accountId = String.valueOf(nextId++);
        double balance = Double.parseDouble(initialBalance);
        accounts.put(accountId, new Account(accountId, owner, balance));
        return accountId;
    }

    public String getBalance(String accountId) {
        if (!accounts.containsKey(accountId)) {
            throw new RuntimeException("Account not found");
        }
        return String.valueOf(accounts.get(accountId).getBalance());
    }

    public void deposit(String accountId, String amount) {
        if (!accounts.containsKey(accountId)) {
            throw new RuntimeException("Account not found");
        }
        double depositAmount = Double.parseDouble(amount);
        if (depositAmount < 0) {
            throw new RuntimeException("Deposit amount must be positive");
        }
        accounts.get(accountId).deposit(depositAmount);
    }

    public void withdraw(String accountId, String amount) {
        if (!accounts.containsKey(accountId)) {
            throw new RuntimeException("Account not found");
        }
        double withdrawAmount = Double.parseDouble(amount);
        if (withdrawAmount < 0) {
            throw new RuntimeException("Withdrawal amount must be positive");
        }
        Account account = accounts.get(accountId);
        if (account.getBalance() < withdrawAmount) {
            throw new RuntimeException("Insufficient funds");
        }
        account.withdraw(withdrawAmount);
    }

    private static class Account {
        private String id;
        private String owner;
        private double balance;

        public Account(String id, String owner, double balance) {
            this.id = id;
            this.owner = owner;
            this.balance = balance;
        }

        public double getBalance() { return balance; }
        public void deposit(double amount) { balance += amount; }
        public void withdraw(double amount) { balance -= amount; }
    }
}
```

### Test Runner

```java
import java.util.ArrayList;
import java.util.List;
import easyaccept.EasyAcceptFacade;

public class BankingTestRunner {
    public static void main(String[] args) throws Exception {
        List<String> testFiles = new ArrayList<>();
        testFiles.add("tests/banking.txt");

        BankingFacade facade = new BankingFacade();
        EasyAcceptFacade easyAccept = new EasyAcceptFacade(facade, testFiles);

        easyAccept.executeTests();

        // Display results
        System.out.println(easyAccept.getCompleteResults());

        // Check statistics
        System.out.println("Total Passed: " + easyAccept.getTotalNumberOfPassedTests());
        System.out.println("Total Failed: " + easyAccept.getTotalNumberOfNotPassedTests());

        // Exit with appropriate code
        System.exit(easyAccept.getTotalNumberOfNotPassedTests() == 0 ? 0 : 1);
    }
}
```

### Test Script (tests/banking.txt)

```easy
# Create accounts
accountId1=createAccount owner="Alice" initialBalance=1000

# Verify initial balance
expect "1000.0" getBalance accountId=accountId1

# Test deposit
deposit accountId=accountId1 amount=500
expect "1500.0" getBalance accountId=accountId1

# Test withdrawal
withdraw accountId=accountId1 amount=300
expect "1200.0" getBalance accountId=accountId1

# Test error conditions
expectError "Account not found" getBalance accountId=9999
expectError "Insufficient funds" withdraw accountId=accountId1 amount=5000
expectError "Deposit amount must be positive" deposit accountId=accountId1 amount=-100
```

## Advanced Features

### Working with Variables

Store command results and reuse them in subsequent commands:

```easy
# Create a user and store the ID
userId=createUser name="Bob Smith" birthdate=1985/06/20

# Use the variable in subsequent commands
expect "Bob Smith" getUserName key=${userId}
expect "1985/06/20" getUserBirthdate key=${userId}

# Use variables in multiple commands
deposit accountId=${accountId} amount=200
withdraw accountId=${accountId} amount=100
expect "500.0" getBalance accountId=${accountId}
```

### Using Echo for Debugging

The `echo` command helps you output values and debug your tests:

```easy
echo "Starting balance test"
accountId=createAccount owner="Charlie" initialBalance=1000
echo "Created account:" ${accountId}
echo "Performing deposit..."
deposit accountId=${accountId} amount=500
echo "New balance:"
echo getBalance accountId=${accountId}
```

### Handling Error Conditions

Use `expectError` to verify that your business logic properly throws exceptions with the correct message:

```easy
expectError "User not found" getUserName key=999
expectError "Division by zero" divide param1=10 param2=0
expectError "Insufficient funds" withdraw amount=1000000
```

### Floating-Point Precision Testing

Use `expectWithin` to test floating-point results with a specified precision:

```easy
expect "3.14159" calculatePi
expectWithin 0.00001 3.14159 calculatePi
expectWithin 0.01 3.14 calculatePi
```

### Executing Multiple Test Files

You can execute multiple test script files in sequence:

```java
List<String> testFiles = new ArrayList<>();
testFiles.add("tests/user_creation.txt");
testFiles.add("tests/user_deletion.txt");
testFiles.add("tests/user_update.txt");

EasyAcceptFacade easyAccept = new EasyAcceptFacade(facade, testFiles);
easyAccept.executeTests();
```

### Executing Scripts from within Scripts

Use `executeScript` to run a test script from within another test script:

```easy
# Run another script using the current thread
executeScript newThread=false scriptFile=tests/setup.txt

# Or run in a new thread (requires threadPool)
threadPool poolSize=5
executeScript newThread=true scriptFile=tests/parallel_test.txt
```

### Repeating Commands

Use `repeat` to execute a command multiple times:

```easy
# Execute the same command 10 times
repeat numberOfTimes=10 incrementCounter
```

## Comparing with Other Approaches

If you're migrating from Java to C#, check out the [C# Getting Started Guide](./csharp.md). The API is similar but adapted to .NET conventions.

## Original Documentation

For more detailed information about EasyAccept, visit:

- **SourceForge Project**: [https://sourceforge.net/projects/easyaccept/](https://sourceforge.net/projects/easyaccept/)
- **Documentation**: [https://easyaccept.sourceforge.net/](https://easyaccept.sourceforge.net/)
- **Easy Language Specification**: [easy.md](../easy.md)
- **Built-in Commands:** See the [Easy Language Specification](../easy.md) for all commands
- **All Implementations**: [implementations.md](../implementations.md)

## Troubleshooting

### Method Not Found Exception

Ensure:

- The method name matches exactly (case-sensitive)
- The number of parameters matches
- Parameter order in the test matches the method signature
- Parameter types are supported

### Syntax Errors in Scripts

Check your test script files for:

- Correct command syntax
- Proper parameter naming (e.g., `paramName=value`)
- Valid string delimiters (default is `"`)
- Valid line continuation (use `\` for multiline commands)

### Class Not Found Exception

When using command-line execution:

- Ensure the facade class is on the classpath
- Use the fully qualified class name
- Check that EasyAccept JAR is in the classpath

### Exception Messages in Tests

Remember:

- Exception messages must match exactly (case-sensitive)
- Use `expectError` for testing expected exceptions
- The message is taken from the exception's message

## Best Practices

1. **Keep facades simple** - Focus on business logic, not presentation
2. **Use meaningful method names** - Make test scripts readable to business stakeholders
3. **Return strings from methods** - Simplifies result verification in tests
4. **Separate test concerns** - Create focused test scripts for different features
5. **Use variables effectively** - Reuse IDs and results to create realistic test flows
6. **Document test scripts** - Use comments (lines starting with `#`) to explain test intent
7. **Handle errors explicitly** - Throw exceptions with clear messages for error cases

## Getting Help

- Check the [Easy Language Specification](../easy.md) for language details
- See the [Easy Language Specification](../easy.md) for all built-in command documentation
- See [All Implementations](../implementations.md) for other language options
- Visit the [SourceForge project page](https://sourceforge.net/projects/easyaccept/) for downloads and forums
- Review the [original documentation](https://easyaccept.sourceforge.net/) for additional information
