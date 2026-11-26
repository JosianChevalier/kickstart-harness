---
name: gate-api
description: The REST and API checks of a stage review, loaded by the gate-api subagent.
---

# Gate: API

This gate checks the contract the server offers to its clients: the URLs, the methods, the status codes, the shapes of requests and responses, and what an existing client sees change. It does not check authorization, concurrency or the inside of the code: other gates do.

## Running this gate

- You are the `gate-api` subagent: go to the next section. Your prompt says how you work and how you report.
- You are in the main session, and the user asked for this gate: do not review the commits yourself. The agent that wrote the work does not review it. Send the `gate-api` subagent with `task`. Give it the north star (the stage's, or the need in one line), the base commit (the stage's, in the notebook; without one, `git merge-base HEAD <default branch>`), and the commits since. Give the user your synthesis of its report, not the raw report. An issue the user wants fixed is a new commit, in the normal cycle; never amend.

## Doctrine

- The API is a contract with clients the server does not control. A client is often the user's own front end: they know what it needs. What a client needs is a question for the user, not a guess.
- The project's existing API is the convention. A new endpoint follows the style of its neighbours (URL shape, naming case, error format, pagination), even where you know another style. An inconsistency with the neighbours is an issue.
- A change that breaks an existing client is an issue unless the stage says it is wanted.
- The contract shows only what the client needs. The database model is not the API.

## What you check

On each new or changed endpoint (look for the route decorators: `@router.get`, `@app.post`, `path()`, `@api_view`):

1. **Resources.** The URL names a thing, not an action: `POST /orders/{id}/refunds`, not `POST /refundOrder`. Collections in the plural. Nesting only for a real ownership. An action that is not a resource (`/orders/{id}/cancel`) is acceptable when the project does it; report it only when it breaks the neighbours' style.
2. **Methods.** `GET` changes nothing. `PUT` replaces the whole resource, and two identical `PUT` give the same result. `PATCH` changes some fields. `DELETE` twice gives the same state. `POST` creates or runs an action. A `GET` that writes is an issue.
3. **Status codes.** `201 Created` with the new resource (and its `Location` when the project uses it) on creation, `204 No Content` with no body. `400` or `422` for invalid input, as the project does. `404` for a missing resource, `409 Conflict` for a state that forbids the action (the order is already refunded). An error answered with `200` and an error in the body is an issue. A domain error that ends in `500` is an issue.
4. **Errors.** One error format across the API, the project's own (for example Problem Details, RFC 9457). The client can tell the cases apart by a stable code, not by parsing a message.
5. **Request shape.** The body is a declared schema (a Pydantic model, a serializer). Required and optional fields are explicit. Unknown fields are refused or ignored, as the project does.
6. **Response shape.** A declared response schema (`response_model`, a serializer), not the ORM object returned as it is. No internal field leaks: internal ids, flags, hashes. Field names follow the project's case. Dates in ISO 8601 with the timezone. Money as integer minor units or a decimal string, with its currency, never a `float`.
7. **Collections.** A list that can grow is paginated, with a maximum page size. Its order is stable, so a page does not repeat or skip items.
8. **Compatibility.** For a changed endpoint: a field removed or renamed, a type changed, a new required field in the request, a status code changed, an error format changed. Each one breaks a client. Additive changes are safe: a new optional field, a new endpoint.
9. **Retries.** A `POST` that moves money or creates something a client may retry: does the contract let the client retry safely (an `Idempotency-Key` header, or a client-chosen id)? The storage side of it belongs to `gate-data-integrity`.
10. **Documentation.** The generated schema (OpenAPI) says what the endpoint does: the status codes it returns, the response model. A code returned but not declared is an issue.

## Examples of issues

```
b4c5d6e feat(refunds): refund an order
  issues:  api/refunds.py:30 answers 200 with {"error": "already refunded"} when the order is already
           refunded. The front end must parse the body to know it failed. 409 Conflict, in the
           project's error format, as api/orders.py:51 does.
           api/refunds.py:34 returns the SQLAlchemy `Refund` object: the response carries
           `internal_note` and `provider_token`. A response model with the fields the client needs.
  limits:  the front end's use of the endpoint is not in this repository.
```

## Not in this gate

- Input validation and what leaks through the endpoint: `gate-security`.
- Whether the business rule behind it sits in the right layer: `gate-design`.
