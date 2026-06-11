---
name: assertj-expert
description: 
---

# AssertJ Assertion Rules

## Rule 1 — Never pass a boolean expression to `assertThat`

`assertThat(boolean)` without a terminal assertion is a no-op. Assert on the subject, not on a boolean derived from it.

```java
// WRONG — passes silently
assertThat(actual.equals(expected));
assertThat(list.size() == 3);
assertThat(str.contains("foo"));

// RIGHT
assertThat(actual).isEqualTo(expected);
assertThat(list).hasSize(3);
assertThat(str).contains("foo");
```

| You wrote                                       | Use instead                          |
| ----------------------------------------------- | ------------------------------------ |
| `assertThat(list.size()).isEqualTo(n)`          | `assertThat(list).hasSize(n)`        |
| `assertThat(list.isEmpty()).isTrue()`           | `assertThat(list).isEmpty()`         |
| `assertThat(map.containsKey(k)).isTrue()`       | `assertThat(map).containsKey(k)`     |
| `assertThat(map.get(k)).isEqualTo(v)`           | `assertThat(map).containsEntry(k,v)` |
| `assertThat(str.contains(s)).isTrue()`          | `assertThat(str).contains(s)`        |
| `assertThat(str.startsWith(s)).isTrue()`        | `assertThat(str).startsWith(s)`      |
| `assertThat(opt.isPresent()).isTrue()`          | `assertThat(opt).isPresent()`        |
| `assertThat(opt.get()).isEqualTo(v)`            | `assertThat(opt).contains(v)`        |
| `assertThat(x != null).isTrue()`                | `assertThat(x).isNotNull()`          |

## Rule 2 — Configuration methods go before the terminal assertion

Placed after, they are silently ignored:

- `.as(...)` / `.describedAs(...)`
- `.withFailMessage(...)` / `.overridingErrorMessage(...)`
- `.usingComparator(...)` / `.usingElementComparator(...)`
- `.usingRecursiveComparison()`
- `.withRepresentation(...)`

```java
// WRONG
assertThat(actual).isEqualTo(expected).as("user id");

// RIGHT
assertThat(actual).as("user id").isEqualTo(expected);
```

## Rule 3 — Pick the right containment assertion

Choose by answering: **does order matter?** and **must the collection contain only these elements?**

| Need                                            | Use                              |
| ----------------------------------------------- | -------------------------------- |
| Some elements present, others allowed           | `contains(a, b)`                 |
| Exactly these elements, exact order             | `containsExactly(a, b, c)`       |
| Exactly these elements, any order               | `containsExactlyInAnyOrder(...)` |
| Only these elements, duplicates collapsed       | `containsOnly(...)`              |
| Consecutive elements, in this order             | `containsSequence(...)`          |
| These elements in this order, gaps allowed      | `containsSubsequence(...)`       |
| At least one of these                           | `containsAnyOf(...)`             |
| Each appears exactly once                       | `containsOnlyOnce(...)`          |

When the test means "the result should be these N things", default to `containsExactlyInAnyOrder`.

## Rule 4 — Exceptions: never `try`/`catch`/`fail`

```java
// WRONG
try {
  service.call();
  fail("expected exception");
} catch (IllegalStateException e) {
  assertThat(e.getMessage()).contains("boom");
}

// RIGHT — pick one style per project
assertThatThrownBy(() -> service.call())
    .isInstanceOf(IllegalStateException.class)
    .hasMessageContaining("boom");

assertThatExceptionOfType(IllegalStateException.class)
    .isThrownBy(() -> service.call())
    .withMessageContaining("boom")
    .withNoCause();

Throwable thrown = catchThrowable(() -> service.call());
assertThat(thrown).isInstanceOf(IllegalStateException.class)
                  .hasMessageContaining("boom");

// To inspect typed fields on the exception:
TextException e = catchThrowableOfType(TextException.class,
                                       () -> service.call());
assertThat(e.line).isEqualTo(5);

// To assert no exception is thrown:
assertThatNoException().isThrownBy(() -> service.call());
```

For common exception types, use the shortcut form rather than passing the class:

```java
// assertThatNullPointerException(), assertThatIllegalArgumentException(),
// assertThatIllegalStateException(), assertThatIOException()
assertThatIllegalArgumentException()
    .isThrownBy(() -> service.call("bad"))
    .withMessageContaining("bad");

// catchNullPointerException(), catchIllegalArgumentException(),
// catchIllegalStateException(), catchIOException(),
// catchIndexOutOfBoundsException(), catchReflectiveOperationException(),
// catchRuntimeException()
IOException e = catchIOException(() -> service.read());
assertThat(e).hasMessageContaining("disk");
```

For cause and root cause, navigate with `.cause()` / `.rootCause()` (with `assertThat`) or `.havingCause()` / `.havingRootCause()` (with `assertThatExceptionOfType`):

```java
assertThat(thrown).rootCause()
                  .isInstanceOf(SQLException.class)
                  .hasMessageContaining("constraint");

assertThatExceptionOfType(ServiceException.class)
    .isThrownBy(() -> service.call())
    .havingRootCause()
    .withMessage("connection refused");
```

## Rule 5 — Use `usingRecursiveComparison()` when the type doesn't override `equals`

```java
// WRONG — reference equality on a DTO without equals()
assertThat(actualDto).isEqualTo(expectedDto);

// RIGHT
assertThat(actualDto).usingRecursiveComparison()
                     .isEqualTo(expectedDto);

// Common refinements:
.ignoringFields("id", "createdAt")
.ignoringFieldsOfTypes(Instant.class)
.ignoringExpectedNullFields()
.withEqualsForType(closeEnough, Double.class)
```

Add `.withStrictTypeChecking()` when both sides are the **same** declared type — it catches accidental subtype substitution. Leave it **off** when comparing across types on purpose (e.g. `Entity` ↔ `Dto`), since enabling it would fail on the type mismatch alone.

```java
assertThat(actualUser).usingRecursiveComparison()
                      .withStrictTypeChecking()
                      .isEqualTo(expectedUser);
```

## Rule 6 — Floating point comparisons need an offset

```java
// WRONG
assertThat(price).isEqualTo(19.99);

// RIGHT
import static org.assertj.core.api.Assertions.within;
import static org.assertj.core.api.Assertions.withinPercentage;

assertThat(price).isCloseTo(19.99, within(0.001));
assertThat(price).isCloseTo(19.99, withinPercentage(0.1));
```

## Rule 7 — Extract and filter inside the chain, not before it

```java
// WRONG
List<String> names = users.stream().map(User::getName).toList();
assertThat(names).contains("Alice");

List<User> adults = users.stream()
    .filter(u -> u.getAge() >= 18)
    .toList();
assertThat(adults).hasSize(3);

// RIGHT
assertThat(users).extracting(User::getName).contains("Alice");
assertThat(users).filteredOn(u -> u.getAge() >= 18).hasSize(3);

// Multiple fields at once:
assertThat(users)
    .extracting(User::getName, User::getAge, u -> u.getAddress().getCity())
    .containsExactly(
        tuple("Alice", 30, "Paris"),
        tuple("Bob",   25, "Lyon"));
```

When the extracted field is itself a collection, use `flatExtracting` (alias: `flatMap`) to flatten before asserting:

```java
// teams is List<Team>, each Team has List<Player> getPlayers()
assertThat(teams).flatExtracting(Team::getPlayers)
                 .contains(alice, bob, carol);
```

When a collection should contain exactly one element, use `singleElement()` (optionally with `as(STRING)`, `as(LIST)`, etc. from `InstanceOfAssertFactories` for typed follow-up) instead of `hasSize(1)` plus indexed access.

## Rule 8 — Use `allSatisfy` / `anySatisfy` / `noneSatisfy` instead of looping

When every element (or some, or none) must hold a non-trivial property, don't loop with `forEach` and don't pre-filter just to count. On failure, the satisfy methods report all violators with context; a loop reports only the first.

```java
// WRONG — reports only the first failure
users.forEach(u -> assertThat(u.getRole()).isEqualTo(ADMIN));

// WRONG — loses per-element diagnostics
assertThat(users.stream().filter(u -> u.getRole() != ADMIN).toList()).isEmpty();

// RIGHT
assertThat(users).allSatisfy(u -> {
    assertThat(u.getRole()).isEqualTo(ADMIN);
    assertThat(u.isActive()).isTrue();
});

assertThat(users).anySatisfy(u -> assertThat(u.getName()).isEqualTo("root"));
assertThat(users).noneSatisfy(u -> assertThat(u.getRole()).isEqualTo(GUEST));
```

For single-predicate checks where you don't need full assertion error messages per element, the `*Match` variants are tighter:

```java
assertThat(users).allMatch(User::isActive, "active")
                 .anyMatch(u -> u.getName().equals("root"))
                 .noneMatch(u -> u.getRole() == GUEST);
```

## Rule 9 — Chain on one subject; use `SoftAssertions` for independent checks

```java
// AVOID
assertThat(name).isNotNull();
assertThat(name).startsWith("Fro");
assertThat(name).endsWith("do");

// PREFERRED
assertThat(name).isNotNull()
                .startsWith("Fro")
                .endsWith("do");
```

When the assertions are independent and you want all failures reported (not just the first), use `SoftAssertions`:

```java
SoftAssertions.assertSoftly(softly -> {
    softly.assertThat(user.getName()).isEqualTo("Frodo");
    softly.assertThat(user.getAge()).isEqualTo(33);
    softly.assertThat(user.getRace()).isEqualTo(HOBBIT);
});
```

## Rule 10 — Don't mix AssertJ with JUnit `assertEquals` in the same file

Argument order is opposite: `assertEquals(expected, actual)` vs `assertThat(actual).isEqualTo(expected)`. If the project uses AssertJ, remove `org.junit.jupiter.api.Assertions.*` imports.
