# Architecture best practices
- Domain objects should be able to answer questions about themselves, rather than using some other service
  - Note: this doesn't apply to DB entities which are used as domain objects (e.g. Hibernate)
