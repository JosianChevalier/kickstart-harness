---
name: gate-security
description: "security gate", a security review of commits. The security checks of a stage review, used by the gate-security subagent.
---

# Gate: security

This gate checks what an attacker could do with the change: what input it trusts, and what it exposes. The application runs offline: there is no authentication, no authorization, and no client to distrust; do not check them. It does not check concurrency, design or the API contract: other gates do. When the company or the project has its own security rules, they come first; this gate does not replace them.

## Running this gate

- You are the `gate-security` subagent: go to the next section. Your prompt says how you work and how you report.
- You are in the main session, and the user asked for this gate: do not review the commits yourself. The agent that wrote the work does not review it. Send the `gate-security` subagent with `task`. Give it the north star (the stage's, or the need in one line), the base commit (the stage's, in the notebook; without one, `git merge-base HEAD <default branch>`), and the commits since. Give the user your synthesis of its report, not the raw report. An issue the user wants fixed is a new commit, in the normal cycle; never amend.

## Doctrine

- Every input from outside the code is a risk until it is validated: the body, the query string, the headers, the files, the messages, the data another system returns.
- Deny by default. A new setting or option starts closed.
- Use the established library for SQL and for cryptography. Never write your own.
- Every issue says four things: what an attacker could do, from where, what new access it gives, and the safe boundary you propose.
- A flaw in code the stage did not change, but now relies on, is reported too, marked "existing". The main session writes it in the notebook; it is handled before the session ends.

## What you check

1. **Input validation at the boundary.** The input is validated when it enters: type, format, length, range, allowed values (a Pydantic model, a serializer, a form). Look for a size limit on uploads, on lists, on page sizes.
2. **SQL injection.** This is the main risk: check every query the stage adds or changes. Outside input never enters the text of a query; it goes in as a bound parameter. Look for SQL built with an f-string, `%`, `+` or `.format()`: `execute(f"... {order_id}")`, `text(f"...")`, `cursor.execute("... %s" % value)`, `.raw()` or `.extra()` with formatting. A column name or a sort order taken from input (`ORDER BY {sort}`) cannot be a bound parameter: it must be checked against a fixed list of allowed values. An ORM query (`select(Refund).where(Refund.id == refund_id)`) is safe.
3. **Other injection.** Outside input used without sanitization in a shell command (`shell=True`, or in the arguments), in a file path (path traversal: `../`), in `pickle.loads`, `yaml.load` without `SafeLoader`, or `eval`.
4. **Secrets.** The code will be shared: a secret in the repository is public. Look for a key, a token or a password in the code, in a committed configuration file, in a test fixture that looks real, in a log line, in an error message. Secrets come from the environment or the secret store. A secret already committed stays in the history: it must be rotated, not only removed.
5. **What leaks out.** A log line or an error response with a token, a password, a card number or personal data. A stack trace or an SQL error sent to the client.
6. **Open defaults.** `DEBUG = True`, a feature flag on by default.
7. **Dependencies.** A new dependency: is it the known package (not a name one letter away), maintained, and pinned in the lock file? A dependency that runs code at install or at import is worth a line.

## Examples of issues

```
f1a2b3c feat(refunds): search refunds by reason
  issues:  ⚠ refunds/repository.py:31 runs `text(f"SELECT * FROM refunds WHERE reason LIKE '%{search}%'")`.
           A search like `' OR 1=1; DROP TABLE refunds; --` reads or destroys the whole table
           (SQL injection). Safe boundary: a bound parameter,
           `text("... WHERE reason LIKE :search")` with `{"search": f"%{search}%"}`.
           ⚠ existing: payments/client.py:9 holds the payment provider's API key in the code.
           The code will be shared: anyone who reads it can make payments. Move it to the
           environment; rotate the key.
  limits:  did not check the deployed settings: they are outside the repository.
```

## Not in this gate

- Two requests at the same time, idempotency: `gate-data-integrity`.
- The status code chosen for a refusal: `gate-api`.
