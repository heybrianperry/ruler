# GitHub Copilot Instructions

This file contains architecture decision records (ADRs) that guide development in this repository.

---

## Standardize Input Validation in CI/CD Test Suites: Test Suites Include

All CI/CD test suites MUST include explicit input validation test cases that verify proper handling of invalid, malformed, and edge-case inputs

---

## Standardize Input Validation in CI/CD Test Suites: Input Validation Test

Input validation test suites SHOULD follow consistent naming conventions (e.g., *.test.ts) and organizational patterns to facilitate discovery and maintenance

---

## Standardize Input Validation in CI/CD Test Suites: Validation Tests Executed

Validation tests SHOULD be executed early in the CI/CD pipeline to provide fast feedback on input handling issues

---

## Standardize Input Validation in CI/CD Test Suites: Teams Use Property

Teams MAY use property-based testing or fuzzing techniques to enhance input validation coverage beyond explicit test cases

---

## Adopt TypeScript with Vitest as Standard Testing Framework: Test Files Organized

Test files MUST be organized into appropriate directories based on test type: tests/unit/, tests/integration/, or tests/e2e/

---

## Adopt TypeScript with Vitest as Standard Testing Framework: Unit Tests Colocated

Unit tests MUST be colocated with the code they test following the pattern tests/unit/<module>/<component>.test.ts

---

## Standardize Input Validation in CI/CD Test Suites: Input Validation Tests

Input validation tests MUST cover both positive cases (valid inputs accepted) and negative cases (invalid inputs rejected with appropriate error handling)

---

## Adopt TypeScript with Vitest as Standard Testing Framework: Test Files Use TypeScript

All test files MUST use TypeScript with the .test.ts file extension

---

## Adopt TypeScript with Vitest as Standard Testing Framework: Test Files Use Vitest

Test files SHOULD use Vitest as the primary test runner and assertion library

---