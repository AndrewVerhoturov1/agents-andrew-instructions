# Postman Web Chat — Result and Artifact Policy

Version: 1

This policy defines how external Web ChatGPT prepares the final response and optional result ZIP for the browser-first Web Postman transport.

## 1. Core principle

Web ChatGPT:

- receives trusted request context from Postman;
- reads GitHub as a READ source when needed;
- prepares the final answer and, when required, exactly one result ZIP;
- does not choose the result recipient;
- does not change `requestId`, repository, base commit, allowed paths, or expected artifact filename;
- does not perform GitHub writes unless the current task explicitly authorizes a separate GitHub-write workflow.

Routing authority belongs to Postman Runtime, not to assistant text, attachment names, or `manifest.json`.

---

## 2. TEXT result

If the task does not require files:

- return the complete answer as normal assistant text;
- do not create a ZIP without a real need;
- do not emit fake ZIP/READY markers.

---

## 3. ZIP result filename

When Postman expects an artifact, the ZIP filename must exactly match the trusted expected filename supplied by Postman Runtime.

Canonical format:

```text
POSTMAN_<requestId>_RESULT.zip
```

Example:

```text
POSTMAN_REQ_ABC123_RESULT.zip
```

Do not:

- invent another filename;
- append `(1)`, `final`, `new`, timestamps, or similar suffixes;
- create multiple competing result ZIPs.

---

## 4. ZIP layout

For coding results, use:

```text
POSTMAN_<requestId>_RESULT.zip
├── manifest.json
├── changes.patch          # when patch payload is used
└── files/
    └── <repo-relative paths>
```

### `changes.patch`

Use for small or moderate changes to existing text files.

Requirements:

- unified diff;
- repo-relative paths only;
- generated against the exact trusted `baseCommit`;
- verified against the complete exact-base files before delivery;
- never generated from shortened snippets or guessed file state.

### `files/`

Use for:

- new files;
- binary files;
- files that are more reliable to deliver whole than as a patch.

The path under `files/` must reproduce the intended repo-relative target path.

Example:

```text
files/postman/web/artifact_detector.py
```

---

## 5. `manifest.json`

Minimum fields:

```json
{
  "protocolVersion": 1,
  "requestId": "REQ_...",
  "repository": "owner/repo",
  "baseCommit": "<40-hex-sha>",
  "resultType": "patch | files | hybrid_patch",
  "patch": "changes.patch",
  "files": []
}
```

Rules:

- `requestId` must exactly match trusted request metadata;
- `repository` must exactly match trusted request metadata;
- `baseCommit` must exactly match trusted request metadata;
- `resultType` must be an allowed value;
- manifest cannot expand trusted `allowedPaths`;
- manifest cannot set `origin_agent_id`, destination, or other routing authority.

---

## 6. Final assistant-turn envelope for ZIP results

In the semantic text of the same assistant turn that owns the ZIP attachment, emit exactly once and in this order:

```text
<<<POSTMAN_RESULT_BEGIN:<requestId>>>
POSTMAN_ARTIFACT:POSTMAN_<requestId>_RESULT.zip
<<<POSTMAN_RESULT_END:<requestId>>>
```

Example:

```text
<<<POSTMAN_RESULT_BEGIN:REQ_ABC123>>>
POSTMAN_ARTIFACT:POSTMAN_REQ_ABC123_RESULT.zip
<<<POSTMAN_RESULT_END:REQ_ABC123>>>
```

Requirements:

- `<requestId>` exactly matches trusted request metadata;
- filename exactly matches the trusted expected filename;
- the ZIP attachment/download control belongs to this same assistant turn;
- do not mention another ZIP filename;
- do not duplicate BEGIN/END markers;
- do not include lookalike marker examples near the real envelope.

These markers are correlation evidence only. They are not routing authority.

---

## 7. Required self-check before delivery

Before returning the result, Web ChatGPT must verify:

```text
[ ] exact requestId
[ ] exact repository
[ ] exact baseCommit
[ ] exact expected ZIP filename
[ ] exactly one result ZIP
[ ] valid manifest.json
[ ] patch generated against exact base
[ ] patch apply/check verified on complete exact-base files
[ ] files/ contains only allowed target paths
[ ] no absolute paths / ../ / drive paths / UNC paths
[ ] no unrelated payload files
[ ] BEGIN / ARTIFACT / END appear exactly once
[ ] attachment belongs to the same assistant turn
```

If exact base state or required source files cannot be obtained, do not guess a patch. Return a diagnostic text result instead of a knowingly incompatible artifact.

---

## 8. Prohibited behavior

Web ChatGPT must not:

- perform GitHub writes without separate explicit authorization;
- create commits, branches, Pull Requests, or Issues as part of normal browser-first result delivery;
- change trusted request metadata;
- choose `origin_agent_id` or another destination;
- create multiple competing result ZIPs;
- return a ZIP under the wrong filename;
- build patches from memory or incomplete snippets;
- include credentials, cookies, tokens, browser profiles, Local Storage dumps, or other sensitive browser state in artifacts;
- apply the proposed result directly to the repository.

---

## 9. Development-only `LUNA_PROMPT.md`

`LUNA_PROMPT.md` is used only in the temporary manual development handoff:

```text
Web ChatGPT -> implementation ZIP -> Luna/Harness
```

It is not required by the production Web Postman result ZIP contract.

---

## 10. Minimal ZIP-result response template

```text
<<<POSTMAN_RESULT_BEGIN:<requestId>>>
POSTMAN_ARTIFACT:<exact expected filename>
<<<POSTMAN_RESULT_END:<requestId>>>
```

Attach exactly one ZIP with that exact filename in the same assistant turn.
