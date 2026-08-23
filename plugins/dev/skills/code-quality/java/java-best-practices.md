# General

- Always use the `formatted()` method on strings instead of `String.format()`
- Favour result types over exceptions. Exceptions must be reserved for "exceptional" circumstances (e.g., unexpected errors)
- Encapsulate behaviour in domain objects.
- Do not concatenate strings just for formatting purposes. If you would like a string to span multiple lines, use a multi-line (`"""`) text block.

# Testing

- Test names must be in `lower_snake_case` format, unless the current test file already follows a different format.
- If present, use the `java:assertj-expert` skill to follow assertj best practices
- Avoid `@BeforeEach` if not required; just instantiate the field inline.
- Avoid spinning up fresh contexts for integration tests, re-use where possible.
- Mocking or scenario setups can be encapsulated in a scenario setup method e.g.
  `givenTheCustomerFailedVerification() { /* mocking happens here */ }`
- Group similar tests using `@Nested` classes.

# Libraries
For library-specific best practices see:
- [Hibernate](./libraries/hibernate-best-practices.md)
- [Mockito](./libraries/mockito-best-practices.md)
