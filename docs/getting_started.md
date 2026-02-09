# Getting Started

This directory contains comprehensive documentation for the EasyAccept acceptance testing framework.

## Documentation Structure

### Core Documentation Files

1. **[easy.md](./easy.md)** - Easy Language Specification
    - Complete reference for the EasyAccept script language syntax
    - Language concepts, examples, and parameter types
    - Quick reference table of built-in commands
    - **Start here** for understanding how to write test scripts

2. **[implementations.md](./implementations.md)** - Implementations Guide
    - Overview of available EasyAccept implementations
    - Comparison between C# and Java implementations
    - Common testing patterns used across implementations
    - Guide to choosing the right implementation for your technology stack

### Language-Specific Guides

Located in the `languages/` subdirectory:

- **[languages/csharp.md](./languages/csharp.md)** - C# Implementation Guide
    - Installation via NuGet
    - Creating facades in C#
    - Complete API reference with examples
    - Integration with .NET projects

- **[languages/java.md](./languages/java.md)** - Java Implementation Guide
    - Installation and setup
    - Creating façades in Java
    - Complete API reference with examples
    - Integration with Java projects

## Quick Navigation

### If you want to...

- **Learn the Easy language syntax**: Start with [easy.md](./easy.md)
- **Find documentation for a specific command**: See [easy.md](./easy.md)
- **Choose between C# or Java**: Read [implementations.md](./implementations.md)
- **Get started with C#**: Follow [languages/csharp.md](./languages/csharp.md)
- **Get started with Java**: Follow [languages/java.md](./languages/java.md)

## Documentation Flow for New Users

1. Start with [implementations.md](./implementations.md) to choose your implementation
2. Follow the implementation guide for your language:
    - C#: [languages/csharp.md](./languages/csharp.md)
    - Java: [languages/java.md](./languages/java.md)
3. Reference [easy.md](./easy.md) for script language syntax details
4. Use [easy.md](./easy.md) when working with specific commands

## Key Concepts

### Facade Pattern

The core concept in EasyAccept is the **facade** - a public interface that exposes your business logic to the test scripts. The facade should:

- Have public methods that correspond to test script commands
- Accept parameters matching the Easy language types (string, int, double, boolean, etc.)
- Return values that can be validated with built-in commands like `expect`

### Easy Language

The Easy language is a simple, non-technical scripting format designed to be readable by business stakeholders. It's not object-oriented and supports:

- Simple commands with named parameters
- Variables for storing results
- Built-in assertion commands
- Comments starting with `#`

### Built-in Commands

EasyAccept provides 13 built-in commands for testing:

| Command           | Purpose                              |
| ----------------- | ------------------------------------ |
| `expect`          | Assert exact output                  |
| `expectDifferent` | Assert output differs from value     |
| `expectWithin`    | Assert floating-point with tolerance |
| `expectError`     | Assert exception is thrown           |
| `equalFiles`      | Compare file contents                |
| `echo`            | Concatenate/echo values              |
| `repeat`          | Execute command N times              |
| `executeScript`   | Execute another script               |
| `stackTrace`      | Debug exceptions                     |
| `threadPool`      | Create thread pool                   |
| `stringDelimiter` | Change string delimiter              |
| `quit`            | Stop execution                       |

For detailed documentation, see [easy.md](./easy.md).

## File Organization

```
docs/
├── getting_started.md          (this file)
├── easy.md                     (Easy language specification)
├── implementations.md          (Guide to implementations)
└── languages/
    ├── csharp.md               (C# implementation guide)
    └── java.md                 (Java implementation guide)
```

## Contributing

When updating documentation:

1. **easy.md** - Update for changes to language syntax or concepts
2. **implementations.md** - Update for new implementations or major pattern changes
3. **languages/\*.md** - Update for language-specific setup or API changes
