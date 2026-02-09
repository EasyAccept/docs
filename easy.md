# Easy Language Documentation

## Requirements

EasyAccept is a tool developed to help development teams create acceptance tests. Such tests are black box and aim to capture the functional requirements for a software system as expressed by a client. A client is typically a non-IT person with a need to be solved through a software system. EasyAccept is being developed with the following requirements in mind:

- It should be easy for a client to write acceptance tests him or herself. Since knowing what to test needs some training (e.g. to test limit conditions, etc.), this task will usually be performed together with an experienced software person.
- Even if a software person helps a client to write tests, the client must be able to thoroughly understand the written tests without any help (after learning a bit about test language syntax).
- EasyAccept should be reasonably easy to retrofit into existing software, as long as business logic has been separated from user interface considerations in the software to be tested.
- It should be easy to automate the execution of acceptance tests.
- It should be easy to test error conditions and non-error conditions.

## Basic Decisions

In order to satisfy the above requirements, the following decisions were taken:

- Tests are written using a simple script language. A script can call business logic and check that it produces the proper output.
- The script language is _not_ object-oriented. It seems easier not to burden the client with the notion of objects, encapsulation, state, etc.

## Examples

### Executing business logic

A script is written in a text file. A command is written on a single line. A command is simply
executed. For example:

```easy
createUser key=key1 name="John Doe" birthdate=1957/02/04
```

will simply call the createUser business method passing three parameters to it.

### Checking business logic execution

Special built-in commands can be used to check that a (business logic) command worked correctly. For
example:

```easy
createUser key=key1 name="John Doe" birthdate=1957/02/04
expect "John Doe" getUserName key=key1
expectWithin .01 2345.67 getSalary key=key1
```

In the above, the first line calls a business logic command. EasyAccept will accept that it has
functioned correctly if it does not produce an error (an Exception, in programmer parlance). The
next line also calls business logic (getUserName with parameter key1) but checks that it returned
the string "John Doe". The third line checks that the salary is correct, with a precision of one
cent.

### Using variables

It is sometimes necessary to obtain the result returned by a command in order to use it in the test
script. For example, suppose that the `createUser` command chooses a record key internally (say a
database OID) that is unknown to the tester. The following script shows how to deal with the
situation using variables (assuming that `createUser` returns the chosen record key):

```easy
key=createUser name="John Doe" birthdate=1957/02/04
expect "John Doe" getUserName key=${key}
```

The syntax `${varName}` is substituted by the variable's value.

The scope of a variable is the set of scripts being executed, that is, from the time of variable
definition until the end of the current EasyAccept execution.

Here is another example that checks whether two users with the same attributes generated different
keys in the database:

```easy
key1=createUser name="John Doe" birthdate=1957/02/04
key2=createUser name="John Doe" birthdate=1957/02/04
expectDifferent ${key1} echo ${key2}
```

### Checking error conditions

A special built-in command can be used to check that a (business logic) command produces an error
(using an exception). For example:

```easy
expectError "Unacceptable date." createUser name="John Doe" birthdate=1957/02/30
```

In the above line, a business logic command is called (createUser). EasyAccept will accept that it
has functioned correctly if it produces an error (an Exception, in programmer parlance) and if the
Exception's error message is "Unacceptable date."

### Checking voluminous output

When you want to use the expect built-in command but the string to be checked is large, it may be
better to leave the string in a text file and have the business logic command produce output in
another file. Then, the built-in command `equalFiles` can be used to check the command's output.

```easy
# this shows that John Doe exists
expect "John Doe" getUserName key=${key1}
produceReport key=${key1} outputFile=rep.txt
equalFiles file1=expected-report.txt file2=rep.txt
```

In the above example, the command `produceReport` will produce a report concerning John Doe and the
report will be left in file rep.txt. The next line checks that the rep.txt file is equal to the
expected-report.txt file. This last file (the expected report) should be produced beforehand (by
hand, for example) and should contain exactly the output desired for the `produceReport` command.

### Executing scripts

With EasyAccept you can execute a script from within another. The current script's execution pauses
until the new script executes to completion.

```easy
# runs the script script2.txt from within the current script using the current thread
executeScript newThread=false scriptFile=script2.txt
```

If you want the new script's execution to be run in a new thread, you can flag the argument
`newThread` with true. EasyAccept takes a new thread from the thread pool, which must have been
previously created with the `threadPool` command.

```easy
threadPool poolSize=5
# runs the script script2.txt in a new thread taken from the thread pool
executeScript newThread=true scriptFile=script2.txt
```

### Debugging

When any command produces an unexpected exception and you would like to examine a stack trace of the
situation that led to the exception, use the `stacktrace` command as shown below.

```easy
# the following command produces an exception
someCommand param=someValue
```

In this case, in order to see details of the exception that was produced, temporarily use the
following command during debugging:

```easy
# the following command produces an exception
stackTrace someCommand param=someValue
```

### Miscellaneous commands

The repeat command can be used to execute a given command a specific number of times. The quit
command closes EasyAccept.

```easy
resetPlayerScore
# adds 6x200 points to the player's score
repeat numberOfTimes=6 playerScores points=200
expect 1200 getPlayerScore
# quits EasyAccept
quit
```

## The Language Syntax

A script resides in a file. Each line consists of name=value pairs. The following are examples of
acceptable name=value pairs:

```easy
name=value
value
name=
name=""
=value
```

In the second case, there is no name; in the third case, the value is null; in the fourth case, the
value is empty; the fifth case is illegal.

In a line, the first name-value pair represents a command to be executed (the value). If a name is
given, then that variable name will receive the value returned by the command. The command must
match a method available in the business logic. Parameters are passed as given in the other
name=value pairs. The names themselves (to the left of the = sign) are not used and serve only as
documentation in the tests. The parameter order is important and must match the method signature.

A method appropriate to the parameter types given will be found in the business logic. The following
parameter types are acceptable and automatic conversion will be provided from a string to the
parameter value of appropriate type:

- String
- boolean
- char
- byte
- short
- int
- long
- float
- double

A line starting with # is a comment. The line continuation character is \. A \ character itself must
be given as \\.

The default string delimiter is ". This may be changed using the `stringDelimiter` command.

## Built-in Commands Overview

EasyAccept has several built-in commands used to perform special testing actions. Below is a quick reference table, with detailed documentation for each command following.

### Command Quick Reference

| **COMMAND**         | **DESCRIPTION**                                                                                                                                                                                                                                  | **C#** | **Java** |
| :------------------ | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----: | :------: |
| **stringDelimiter** | Built-in command that changes the string delimiter to the given delimiter. By default, the string delimiter is ".                                                                                                                                |   ✗    |    ✓     |
| **expect**          | Built-in command that is used to check that a (business logic) command produced the expected result.                                                                                                                                             |   ✓    |    ✓     |
| **expectDifferent** | Built-in command that is used to check that a (business logic) command produced a result different from a given string.                                                                                                                          |   ✓    |    ✓     |
| **expectWithin**    | Built-in command that is used to check that a (business logic) command produced the expected floating-point result within a desired precision.                                                                                                   |   ✗    |    ✓     |
| **expectError**     | Built-in command that is used to check error conditions of a (business logic) command.                                                                                                                                                           |   ✓    |    ✓     |
| **equalFiles**      | Built-in command that compares two files.                                                                                                                                                                                                        |   ✗    |    ✓     |
| **stackTrace**      | Built-in command that is used to obtain a stack trace when debugging. This is useful when unexpected exceptions occur and one wishes a stack trace to see what is happening.                                                                     |   ✗    |    ✓     |
| **quit**            | Built-in command to quit EasyAccept                                                                                                                                                                                                              |   ✓    |    ✓     |
| **executeScript**   | Built-in command that executes a given script. The current script's execution is paused until the new script executes to completion. Either the current thread or a new thread taken from the thread pool may be used to execute the new script. |   ✗    |    ✓     |
| **repeat**          | Built-in command that is used to repeat a given command's execution a specific number of times.                                                                                                                                                  |   ✗    |    ✓     |
| **threadPool**      | Built-in command that creates a thread pool to execute scripts.                                                                                                                                                                                  |   ✗    |    ✓     |
| **echo**            | Built-in command that returns the concatenation of its parameters                                                                                                                                                                                |   ✓    |    ✓     |

## Built-in Commands - Detailed Reference

### echo

**Description:** Returns the concatenation of its parameters, useful for creating computed values or just echoing values to verify their existence.

**Syntax:**

```easy
echo param1 param2 ... paramN
```

---

### expect

**Description:** Used to check that a business logic command produced the expected result. The command will succeed if the returned value matches the expected string exactly.

**Syntax:**

```easy
expect "expected_value" commandName param1=value1 param2=value2
```

**Behavior:**

- Passes if the command returns exactly the expected string
- Fails with detailed error message if value does not match
- Case-sensitive comparison

---

### expectDifferent

**Description:** Used to check that a business logic command produced a result different from a given string.

**Syntax:**

```easy
expectDifferent "unexpected_value" commandName param1=value1 param2=value2
```

**Behavior:**

- Passes if the command returns a value different from the given string
- Fails if the returned value matches the provided value
- Case-sensitive comparison

---

### expectWithin

**Description:** Used to check that a business logic command produced the expected floating-point result within a desired precision. This is useful for comparing floating-point numbers where exact equality might be problematic due to rounding.

**Syntax:**

```easy
expectWithin precision expected_value commandName param1=value1
```

**Behavior:**

- Precision is specified as an absolute tolerance value
- Useful for floating-point comparisons
- Both expected and actual values are converted to double

---

### expectError

**Description:** Used to check error conditions. This command verifies that a business logic command produces an exception/error with a specific error message.

**Syntax:**

```easy
expectError "error_message" commandName param1=value1
```

**Behavior:**

- Passes if the command throws an exception with the matching error message
- Fails if no exception is thrown
- Fails if exception message does not match

---

### equalFiles

**Description:** Compares two files for equality. This is useful for checking voluminous output from commands by comparing against a reference file.

**Syntax:**

```easy
equalFiles file1=path/to/expected.txt file2=path/to/actual.txt
```

**Behavior:**

- Compares two files byte-by-byte
- Passes if files are identical
- Fails with details about first difference found
- Relative paths are typically resolved from the working directory

---

### stackTrace

**Description:** Displays a stack trace when an exception occurs. This is particularly useful during debugging to understand the sequence of calls that led to an error.

**Syntax:**

```easy
stackTrace commandName param1=value1 param2=value2
```

**Behavior:**

- Prints full exception stack trace to help identify the cause
- Useful for troubleshooting unexpected errors
- Should be removed once issue is resolved

---

### quit

**Description:** Terminates the EasyAccept test execution.

**Syntax:**

```easy
quit
```

**Behavior:**

- Stops all test execution immediately
- Useful for conditional test termination

---

### executeScript

**Description:** Executes another EasyAccept script file. The current script pauses until the executed script completes.

**Syntax:**

```easy
executeScript scriptFile=path/to/script.txt newThread=false
executeScript scriptFile=path/to/script.txt newThread=true
```

**Behavior:**

- `newThread=false`: Executes in current thread, blocking until complete
- `newThread=true`: Executes in new thread from pool (requires threadPool command)
- Useful for modular test organization

---

### repeat

**Description:** Executes a given command multiple times. Useful for scenarios requiring repeated test actions.

**Syntax:**

```easy
repeat numberOfTimes=N commandName param1=value1
```

**Behavior:**

- Repeats the specified command exactly N times
- Useful for testing accumulation or repeated operations
- Each iteration is independent

---

### threadPool

**Description:** Creates a thread pool for executing scripts in parallel using the `executeScript` command with `newThread=true`.

**Syntax:**

```easy
threadPool poolSize=N
```

**Behavior:**

- Must be called before using `executeScript` with `newThread=true`
- Creates a fixed-size thread pool
- Threads are reused for multiple script executions

---

### stringDelimiter

**Description:** Changes the string delimiter used in the script from the default double quote (`"`) to a custom delimiter. This is useful when the default delimiter conflicts with content.

**Syntax:**

```easy
stringDelimiter newDelimiter=|
commandName param=|string containing " quotes|
```

**Behavior:**

- Changes delimiter for entire remaining script execution
- Useful when strings contain the default delimiter
- Common alternatives: `|`, `~`, or any single character

---

## Getting Started

To start using EasyAccept, choose your implementation:

- [C# Implementation](./languages/csharp.md)
- [Java Implementation](./languages/java.md)
