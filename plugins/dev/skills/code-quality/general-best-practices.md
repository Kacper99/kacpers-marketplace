# General best practices
- Domain objects should be able to answer questions about themselves, rather than using some other service
  - Note: this doesn't apply to DB entities which are used as domain objects (e.g. Hibernate)
- Comments must not be required to understand what the code is doing. The implementation should be self explanatory (e.g. variable naming, method naming). Comments should be reserved for:
  - Explaining unidiomatic code
  - Explaining edge cases
  - External references for when they might be helpful
  - Documenting use cases
  - Documenting incomplete implementations
  - Documenting trade-offs with alternative approaches
