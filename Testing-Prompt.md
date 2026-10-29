# Task: Testing Audit and Test Suite Build-Out per Vladimir Khorikov, "Unit Testing: Principles, Practices, and Patterns"

You are a senior software engineer with a deep understanding of Khorikov's book. You will perform a complete testing audit of this codebase, build a test suite that follows the book's principles, run it, and fix discovered defects iteratively until the entire suite is reliably green.

Work autonomously. Follow the phases and rules below strictly. Wherever the book and a convenient shortcut conflict, the book wins.

---

## Mandatory Core Principles (from the book)

1. **The four pillars of a good test:** protection against regressions, resistance to refactoring, fast feedback, maintainability. Resistance to refactoring is **non-negotiable**: a test may only fail when observable behavior changes, never because implementation details changed.
2. **Classical (Detroit) school:** A unit is a unit of behavior, not a class. Tests are isolated from each other, not classes from each other. Internal collaborators are used as real objects.
3. **Observable behavior vs. implementation details:** Test only through the public API that serves a client's goal. No tests of private methods. Never expose private state for testing. No code pollution (production code that exists only for tests).
4. **Mocks only for unmanaged out-of-process dependencies** (e.g., SMTP, message bus, third-party APIs) whose interactions are observable externally. Managed dependencies (your own database) are used for real in integration tests. Mock at the edge of the system, i.e., the last type in your code before the external dependency. Never mock concrete classes; never mock types you don't own (mock your own adapter instead).
5. **Stubs vs. mocks:** Never verify interactions with stubs.
6. **Preferred test styles:** output-based over state-based over communication-based.
7. **Code quadrants:**
   - *Domain model & algorithms* (high complexity/domain significance, few collaborators) → **unit tests**, thorough, including edge cases.
   - *Controllers/orchestration* (low complexity, many collaborators) → **integration tests**.
   - *Trivial code* (getters/setters, plain DTOs, simple pass-throughs) → **no dedicated test**.
   - *Overcomplicated code* (complex AND many collaborators) → **refactor first** (Humble Object, Functional Core / Imperative Shell, separating decisions from execution), then test according to its new quadrant.
8. **Integration tests:** Per business scenario, typically one happy path that exercises as many out-of-process interactions as possible, plus the edge cases that cannot be covered by unit tests. Prefer the fail-fast principle in production code.
9. **Database tests:** Use a real database of the same type as production (no in-memory substitute), schema managed via migrations, clean up data **at the start** of each test, never run tests in parallel against the same database.
10. **Test structure:** AAA pattern (Arrange, Act, Assert), exactly one Act line, no if statements or loops in tests, no multiple AAA sections. Test names describe behavior in plain language that a domain expert would understand (e.g., `Delivery_with_a_past_date_is_invalid`), not the method name.
11. **Fixtures:** No shared mutable state via constructors/setup methods. Use private factory methods or test data builders.
12. **No leaking domain knowledge:** Expected values are never computed by re-implementing the algorithm in the test; they are stated as concrete values.
13. **Time and randomness** are injected as explicit values, not accessed as ambient context.
14. **Coverage** is a negative indicator, not a target. Branch coverage is more meaningful than line coverage. Never write tests solely to raise coverage.
15. **UI / end-to-end:** Extract presentation logic out of the UI via the Humble Object pattern (ViewModel, Presenter, Controller) and unit- or integration-test it there. The UI itself gets a small number of end-to-end tests covering the critical user journeys. Every screen/view must be exercised by at least one E2E path. No brittle selectors that depend on layout details.

---

## Phase 0: Preparation

- Work on a new Git branch `test-audit-khorikov`. Commit after each completed phase and after each individual bug fix, with a meaningful message.
- Identify language(s), frameworks, build system, existing test frameworks, mocking libraries, UI technology, database(s), and external dependencies.
- Make sure the build and existing tests run locally. Document the baseline (number of tests, pass/fail, runtime, coverage if measurable).
- If no test framework exists: choose the most established one for the stack and briefly justify the choice.

## Phase 1: Inventory and Classification

Create `docs/TEST_AUDIT.md` with a table covering **every** class, module, and UI component:

| Element | Path | Quadrant | Dependencies (internal / managed / unmanaged) | Test strategy | Rationale |

- Every element gets a classification. "No dedicated test" is a valid strategy but must be justified (e.g., "trivial" or "covered by integration test X").
- Flag overcomplicated code and describe the planned refactoring.
- List all out-of-process dependencies, each classified as "managed" or "unmanaged".

## Phase 2: Audit of Existing Tests

Evaluate every existing test against the four pillars and the core principles. For each test, record:
- Violations (e.g., tests implementation details, mocks internal classes, verifies stub calls, multiple Act lines, shared state, leaked domain knowledge, tests private methods).
- Decision: **keep**, **rewrite**, or **delete**, each with a rationale.

Delete a test only if it provides no regression protection that isn't covered elsewhere, or if it fundamentally violates resistance to refactoring and cannot reasonably be rewritten.

## Phase 3: Refactoring for Testability

- Carry out the refactorings planned in Phase 1 for overcomplicated code (Humble Object, separating decision logic from side effects).
- **Do not change observable behavior** while doing so. If that behavior would otherwise be unprotected, secure it beforehand with higher-level tests.
- Document every change to a public interface.

## Phase 4: Build the Test Suite

Following the strategy from Phase 1:
1. Unit tests for the domain model and algorithms, including edge cases and error paths.
2. Integration tests for controllers/orchestration, with a real managed database and mocks only for unmanaged dependencies.
3. Tests for presentation logic (ViewModels/Presenters).
4. A small number of end-to-end UI tests for the critical user journeys, such that every screen is exercised at least once.

Every new test must satisfy all structural rules (AAA, naming, no logic in tests, fixtures).

## Phase 5: Run and Fix Iteratively

Run the complete suite. For **every** failing test:

1. **Classify the root cause** and record it in the fix log:
   - (a) Defect in production code
   - (b) Defect in the test
   - (c) Environment/infrastructure problem
   - (d) Unclear whether the behavior is intended
2. **Act:**
   - (a) Apply a minimal, targeted fix to production code. The failing test serves as the regression test.
   - (b) Correct the test, but **never** weaken assertions, adjust expected values to match incorrect actual behavior, disable/skip tests, or artificially raise timeouts to force green.
   - (c) Fix the environment and document the cause.
   - (d) **Change nothing.** Mark the test as an open question and list it in the final report with a description of the actual and the presumed intended behavior.
3. **After every fix, rerun the complete suite**, not just the affected test, to catch knock-on failures.
4. **Per-defect stop rule:** After 3 unsuccessful fix attempts for the same defect, revert those changes, document the defect as open (root-cause hypothesis, attempts made, outcome), and move on to the next defect.
5. **Flakiness:** At the end, run the suite three times in a row. Any non-deterministic test counts as a defect (usually caused by time, randomness, ordering, or shared state) and is handled under the same rules.

Repeat Phase 5 until all tests are green or only documented open items remain.

## Definition of Done

- Every element in the inventory has an implemented, justified test strategy.
- All tests satisfy the core principles. No test verifies implementation details.
- The suite passes fully three times in a row.
- No test has been disabled, skipped, or weakened in what it asserts in order to achieve green.
- The build is clean, with no new warnings.

## Final Report (append to `docs/TEST_AUDIT.md`)

1. Baseline vs. final state (test count by type, runtime, branch coverage as information, not as a target).
2. Deleted and rewritten tests, with rationale.
3. Refactorings performed.
4. Fix log: every defect found, with classification, root cause, fix, and commit.
5. Open items: unclear behavioral questions and unresolved defects.
6. Recommendations for architecture and testability beyond the scope of this task.
