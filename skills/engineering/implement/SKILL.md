---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets.

If the user passes a ticket reference, fetch it from the issue tracker and state its title before starting. If the reference is ambiguous, ask.

Do not default to TDD. Implement the work directly, then run the relevant existing tests to verify, and state which changes are not covered by existing tests. Call the Skill tool with "tdd" only when the user explicitly asked for TDD or the ticket touches high-risk/complex logic: protocol parsing & encoding/decoding, crypto/authentication, routing & load-balancing decisions, concurrency & state machines, configuration migration.

Run typechecking regularly, single test files regularly, and the full test suite once at the end.

Once done, call the Skill tool with "code-review" to review the work.

Commit your work to the current branch.
