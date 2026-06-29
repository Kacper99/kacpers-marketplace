# General

- Always use the `formatted()` method on strings instead of `String.format()`
- Favour result types over exceptions.
- Encapsulate behaviour in domain objects. (This does not apply to hibernate entities)
- Only use hibernate entities for the repository layer, DO NOT expore them into the domain.
- Do not do string concatenation `"foo" +"bar"` just for code formatting purposes

# Testing

- Test names must be in `lower_snake_case` format, unless the current test file already follows a different format.
- If present, use the `java:assertj-expert` skill to follow assertj best practices
- Prefer fakes over mocks where convenient and reasonable
- Do not use `@Mock` to define a mock and instead use the `mock()` static factory.
- `@BeforEach` if not requried, just instantiate the field inline.
- Avoid spinning up fresh contexts for integration tests, re-use where possible.
- Mocking or scenario setups can be encapsulated in scenario setup method e.g.
  `givenTheCustomerFailedVerification
() {} // mocking happens here }`
- Group similar tests using `@Nested` classes.
