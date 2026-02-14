---
description: Testing philosophy — behavior focus, naming, structure, and factories
globs: ["**/*.test.ts", "**/*.spec.ts", "tests/**"]
---

# Testing Principles

- Test behavior, not implementation — refactoring shouldn't break tests
- Test names: scenario + outcome ("returns null when user ID does not exist")
- Arrange, Act, Assert structure in every test
- Don't test private functions — test through the public API
- Don't mock what you don't own — wrap libraries, mock the wrapper
- Use factory functions for test data with sensible defaults and overrides
- Don't test the framework — test YOUR logic
