
# Static SQL Injection Detector

An AST-based static analysis tool for detecting potential SQL Injection (SQLi) vulnerabilities in Python source code using taint analysis.

## Overview

SQL Injection is a security vulnerability that occurs when untrusted user input reaches SQL queries without proper parameterization or validation.

This project focuses on detecting potential SQL Injection vulnerabilities during static analysis, without executing the analyzed Python program.

The detector analyzes Python source code, identifies untrusted data sources, tracks the propagation of tainted data, and reports when tainted data reaches SQL execution sinks.

## Key Features

- **Scope Handling:** Function-local variables stay separate and won't leak..
- **AST-Based Analysis:** Parses Python source code using the Abstract Syntax Tree (AST).
- **Taint Analysis:** Tracks untrusted data through variable assignments and expressions.
- **SQL Sink Detection:** Identifies SQL execution APIs such as `execute()` and `executemany()`.
- **Dynamic Query Detection:** Analyzes potentially unsafe SQL query construction patterns, including string concatenation, f-strings, and `.format()`.
- **Source-to-Sink Tracking:** Connects the origin of untrusted input to the SQL execution point.
- **Vulnerability Reporting:** Generates structured reports containing source location, sink, and taint flow information.
- **No Runtime Execution:** The analyzed Python program is not executed during the analysis.

## How It Works

The detector follows a source-to-sink analysis approach.

```text
Python Source Code
        |
        v
   AST Parsing
        |
        v
 Identify Taint Sources
        |
        v
  Taint Propagation
        |
        v
  SQL Sink Detection
        |
        v
 Source-to-Sink Analysis
        |
        v
 Vulnerability Report
```

### 1. AST Parsing

The Python source code is parsed into an Abstract Syntax Tree.

The AST allows the detector to inspect Python statements, expressions, function calls, assignments, and other code structures without executing the program.

### 2. Taint Sources

The detector identifies inputs that may contain untrusted data.

Examples include:

- `input()`
- HTTP request data
- Environment variables
- Command-line arguments

### 3. Taint Propagation

Taint is propagated through variable assignments and expressions.

For example:

```python
user_id = input("Enter user ID: ")

query = "SELECT * FROM users WHERE id = " + user_id
```

The value obtained from `input()` is treated as tainted.

When that value is used in constructing the SQL query, the taint is propagated to the resulting expression.

### 4. SQL Execution Sinks

The detector focuses on SQL execution operations such as:

```python
cursor.execute(query)
```

and:

```python
cursor.executemany(query, data)
```

A potential SQL Injection vulnerability is reported when tainted data reaches a SQL execution sink.

### 5. Vulnerability Reporting

The detector generates a report containing information such as:

- Source file
- Line number
- Taint source
- SQL execution sink
- Tainted variable or expression
- Source-to-sink flow

## Example

### Vulnerable Code

```python
import sqlite3

user_id = input("Enter user ID: ")

query = "SELECT * FROM users WHERE id = " + user_id

cursor.execute(query)
```

### Why It Is Flagged

The data flow is:

```text
input()
   |
   v
user_id
   |
   v
SQL Query Construction
   |
   v
cursor.execute()
```

Untrusted input reaches an SQL execution sink through query construction.

The detector can identify this as a potential SQL Injection vulnerability.

### Safe Code

```python
import sqlite3

user_id = input("Enter user ID: ")

query = "SELECT * FROM users WHERE id = ?"

cursor.execute(query, (user_id,))
```

Here, the SQL query structure is separated from the user-provided value using parameterized execution.

## Detection Scope

The project primarily focuses on SQL Injection detection in Python programs.

### Sources

- User input
- Request-related input
- Environment variables
- Command-line arguments

### Query Construction Patterns

- String concatenation
- F-strings
- `.format()`-based query construction

### SQL Sinks

- `execute()`
- `executemany()`

## Project Architecture

The detector is designed around the following analysis stages:

1. Python AST parsing.
2. Source identification.
3. Taint propagation.
4. SQL sink identification.
5. Source-to-sink vulnerability analysis.
6. Structured report generation.

The analysis is performed without executing the target Python program.

## Limitations

Static analysis identifies potential vulnerabilities; it does not guarantee that every reported issue is exploitable.

Possible limitations include:

- False positives.
- Complex or indirect data flows.
- Dynamic Python features.
- SQL execution APIs not recognized by the detector.
- Analysis limited to supported source and sink patterns.

The exact limitations depend on the implemented analysis rules.

## Future Scope

- Improved interprocedural taint analysis.
- Better handling of complex Python data flows.
- Additional SQL execution APIs.
- False-positive reduction.
- Cross-Site Scripting (XSS) detection.
- Integration with development and CI/CD workflows.

## Technologies

- Python
- Python AST
- Static Analysis
- Taint Analysis
- SQL Injection Detection
- Compiler Design

## Project Status

Academic / research project focused on static SQL Injection detection using AST-based taint analysis.

