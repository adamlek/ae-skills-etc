---
name: python-clean-code
description: "This skill embodies the principles of \"Clean Code\" by Robert C. Martin (Uncle Bob), specifically adapted for modern Python. Use it to transform \"code that works\" into \"code that is clean, pythonic, and strictly typed.\""
risk: safe
source: "Adapted from Clean Code / ClawForge"
---

# Python Clean Code Skill

This skill embodies the principles of "Clean Code" adapted for Pythonic idioms, PEP 8 standards, and modern typing. Use it to transform "code that works" into code that is clean, readable, and maintainable.

## 🧠 Core Philosophy
> "Code is read much more often than it is written." — Guido van Rossum

## When to Use
Use this skill when:
- **Writing new Python code**: To ensure high quality, proper typing, and PEP 8 compliance from the start.
- **Reviewing Pull Requests**: To provide constructive, principle-based feedback on Pythonic patterns.
- **Refactoring legacy scripts**: To identify code smells and transform procedural scripts into structured, testable modules.

## 1. Meaningful Names (PEP 8 Compliant)
- **Use Intention-Revealing Names**: `elapsed_time_in_days` instead of `d`.
- **Follow Python Conventions**: `snake_case` for variables/functions, `PascalCase` for classes, `UPPER_SNAKE_CASE` for constants.
- **Avoid Disinformation**: Don't use `account_list` if it's actually a `set` or `dict`. Use `accounts`.
- **Use Pronounceable/Searchable Names**: Avoid `genymdhms`.
- **Class Names**: Use nouns (`Customer`, `WikiPage`). Avoid vague names like `Manager` or `Data`.
- **Function/Method Names**: Use verbs (`post_payment`, `delete_page`).

## 2. Functions & Type Hints
- **Small!**: Functions should be short. If it requires scrolling, it's likely too long.
- **Do One Thing**: A function should do only one thing and do it well. Extract helper functions heavily.
- **Type Hints**: Always use modern type hints (`def process_user(user_id: int) -> User:`).
- **Descriptive Names**: `is_password_valid` is better than `check_pwd`.
- **Arguments**: 0-2 is ideal. If you need 3+, group them into a `@dataclass` or use keyword-only arguments (`*`) to prevent positional errors.
- **No Side Effects**: Functions shouldn't secretly mutate global state or passed arguments unless explicitly named for it (e.g., `append_...`).

## 3. Comments and Docstrings
- **Don't Comment Bad Code—Rewrite It**: Most inline comments are a sign of failure to express intent in code.
- **Explain Yourself in Code**: 
  ```python
  # Check if employee is eligible for full benefits
  if employee.flags & HOURLY and employee.age > 65:
  ```
  *Refactored:*
  ```python
  if employee.is_eligible_for_full_benefits():
  ```
- **Docstrings (`"""..."""`)**: Use for public modules, classes, and functions to explain *what* and *why*, not *how*.
- **Inline Comments (`#`)**: Reserve for explaining complex algorithms, regex intent, or "why we can't do X" (e.g., working around a library bug).

## 4. Formatting & Tooling
- **The Newspaper Metaphor**: High-level concepts at the top, details at the bottom.
- **Embrace Formatters**: Code should look like it was formatted by `black` or `ruff`. Don't argue over formatting; automate it.
- **Vertical Density**: Group related lines without empty spaces; separate distinct thoughts with a single blank line.
- **Distance**: Declare variables right before they are used, not at the top of the function.

## 5. Objects and Data Structures
- **Data Abstraction**: Use properties (`@property`) instead of explicit getters and setters (`get_name()`, `set_name()`).
- **The Law of Demeter**: A module should not know about the innards of the objects it manipulates. Avoid chained lookups like `a.b.c.do_something()`.
- **Data Structures**: Use `@dataclass`, `NamedTuple`, or Pydantic models for data-heavy objects without business logic, rather than standard dictionaries or generic classes.

## 6. Error Handling
- **Use Exceptions instead of Return Codes**: Don't return `False` or `-1` for an error state. Raise a specific exception (`ValueError`, `KeyError`, or a custom exception).
- **Don't Return `None` for Errors**: It forces the caller to check for `None` everywhere. If a missing value is expected, type it explicitly as `User | None` (or `Optional[User]`).
- **Try/Except/Finally/Else**: Keep the `try` block as small as possible. Use `else` for code that should only run if no exception occurred.
  ```python
  try:
      data = fetch_data()
  except ConnectionError:
      log.error("Failed to fetch")
      raise
  else:
      process(data)
  ```

## 7. Unit Tests
- **Use Pytest**: Favor `pytest` and its fixtures over standard `unittest` classes.
- **The Three Laws of TDD**:
  1. Don't write production code until you have a failing unit test.
  2. Don't write more of a unit test than is sufficient to fail.
  3. Don't write more production code than is sufficient to pass the failing test.
- **F.I.R.S.T. Principles**: Fast, Independent, Repeatable, Self-Validating, Timely.

## 8. Classes
- **Small!**: Classes should have a single responsibility (SRP).
- **Encapsulation in Python**: Python doesn't have strict `private` modifiers. Use a single leading underscore (`_variable`) to signal internal implementation details that shouldn't be accessed externally.
- **Protocols and ABCs**: Use `typing.Protocol` for duck-typing interfaces, or `abc.ABC` for strict inheritance, rather than relying on deep, rigid class hierarchies.

## 9. Smells and Heuristics
- **Mutable Default Arguments**: NEVER use mutable structures as defaults (`def foo(items: list = []):`). Use `items: list | None = None`.
- **God Objects/Files**: `utils.py` or `helpers.py` that become dumping grounds.
- **List Comprehension Abuse**: Nested or overly complex comprehensions. If a list comprehension is hard to read, turn it back into a `for` loop.

## 🛠️ Implementation Checklist
- [ ] Are variables and functions snake_case, and classes PascalCase?
- [ ] Is this function shorter than 20 lines and doing exactly one thing?
- [ ] Are all signatures explicitly typed using modern Python type hints?
- [ ] Are we using exceptions rather than returning `None` or error codes?
- [ ] Are data structures utilizing `@dataclass` or properties appropriately?
- [ ] Is there a focused `pytest` test for this behavior?

## Example

**User request:**

> Refactor this procedural Python script to use PEP 8 conventions, explicit type hints, focused functions, `@dataclass` for data structures, and isolated `try/except` blocks. Ensure the behavior is covered by a `pytest` suite.
