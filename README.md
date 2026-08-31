# agents-andrew-instructions

Public, version-controlled instructions for Andrew's agents and Web ChatGPT workflows.

The purpose of this repository is to keep reusable agent policies in one public location so models can read them through ordinary public web access.

## Canonical policies

- [`policies/github-public-first.md`](policies/github-public-first.md) — general GitHub least-privilege policy.
- [`policies/postman-webchat-result-artifact.md`](policies/postman-webchat-result-artifact.md) — canonical browser-first Postman request identity and TEXT/ZIP result policy.
- [`templates/postman-github-bootstrap.md`](templates/postman-github-bootstrap.md) — canonical Luna/Postman request and ZIP-rendering template.
- [`templates/chatgpt-custom-instructions.md`](templates/chatgpt-custom-instructions.md) — small reusable Custom Instructions block.

## Canonical public URLs

Postman result/artifact policy:

`https://raw.githubusercontent.com/AndrewVerhoturov1/agents-andrew-instructions/main/policies/postman-webchat-result-artifact.md`

Postman bootstrap template:

`https://raw.githubusercontent.com/AndrewVerhoturov1/agents-andrew-instructions/main/templates/postman-github-bootstrap.md`

General GitHub policy:

`https://raw.githubusercontent.com/AndrewVerhoturov1/agents-andrew-instructions/main/policies/github-public-first.md`

## Postman request identity

Every new browser-first request uses one immutable key created by the initiating Harness model before Postman registration:

```text
REQ_YYYYMMDDTHHMMSSZ_NNNN
```

Example:

```text
REQ_20260831T043812Z_4827
```

The timestamp is UTC to the second and `NNNN` is exactly four random digits. Runtime validates and registers the exact key, rejects collisions, and never rewrites it.

The same key is reused in:

```text
initiating request
→ Postman Runtime registry
→ Web ChatGPT prompt first line
→ user-turn recovery/search
→ correlated assistant result
→ BEGIN/END envelope
→ manifest requestId
→ ZIP filename
→ journal/logs
→ durable result directory
→ originating Harness delivery
```

Do not use the automatically generated ChatGPT title as request authority. The exact request key in the user turn is the durable chat recovery anchor.

## ZIP naming

One ZIP:

```text
POSTMAN_<requestId>_RESULT.zip
```

If trusted request metadata explicitly expects several ZIPs, keep the same request ID and number only the artifacts:

```text
POSTMAN_<requestId>_RESULT-01.zip
POSTMAN_<requestId>_RESULT-02.zip
```

## Browser-first result structure

For a single ZIP result, the expected rendered assistant turn is:

```text
<<<POSTMAN_RESULT_BEGIN:<requestId>>>
<real clickable ZIP control with exact expected filename>
<<<POSTMAN_RESULT_END:<requestId>>>
```

The actual ZIP control must physically appear between BEGIN and END and display the exact expected filename.

## Policy layering

1. Platform/system safety requirements remain in force.
2. Explicit task-specific authorization defines what the current task may change.
3. The initiating model creates and registers the canonical request key.
4. Postman trusted request metadata defines repository/base/scope/expected filename.
5. The Postman result/artifact policy defines browser-first result delivery.
6. The general GitHub public-first policy applies unless another write workflow is explicitly authorized.

## Browser-first design principle

Normal Postman delivery does not require GitHub writes:

```text
initiating Harness Agent
→ immutable REQ
→ Postman Runtime
→ Web ChatGPT
→ correlated assistant TEXT/ZIP
→ browser download
→ validation
→ durable local result
→ Runtime READY
→ originating Harness Agent
```
