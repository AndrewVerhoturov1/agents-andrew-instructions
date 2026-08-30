# Postman Web Chat — Result and Artifact Policy

Version: 2

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

## 6. Final assistant-turn structure for ZIP results

For a ZIP result, the **final assistant turn must have exactly this visible structure**:

```text
<<<POSTMAN_RESULT_BEGIN:<requestId>>>
<REAL DOWNLOADABLE ZIP CONTROL WITH EXACT EXPECTED FILENAME>
<<<POSTMAN_RESULT_END:<requestId>>>
```

Example of the intended rendered result:

```text
<<<POSTMAN_RESULT_BEGIN:REQ_ABC123>>>
POSTMAN_REQ_ABC123_RESULT.zip     ← this line is the real clickable/downloadable ZIP control
<<<POSTMAN_RESULT_END:REQ_ABC123>>>
```

The middle line is **not ordinary text metadata**. It must be the actual generated-file/download control created by ChatGPT for the ZIP.

Mandatory requirements:

- `<requestId>` exactly matches trusted request metadata;
- the ZIP is a real downloadable attachment, not a fake text link;
- the visible attachment/link label is exactly the trusted expected filename;
- the actual ZIP control is physically rendered between the BEGIN and END markers;
- BEGIN appears exactly once;
- END appears exactly once;
- exactly one result ZIP is present;
- do not write `Download ZIP`, `Скачать ZIP`, `Download file`, or another generic label when the exact filename can be used;
- do not mention any other ZIP filename;
- do not add explanatory text before BEGIN or after END in the final ZIP-result turn;
- do not place another attachment inside or outside the block.

The BEGIN/END markers and visible filename are correlation evidence only. They are not routing authority.

### Why physical placement matters

Web Postman uses DOM order as an additional proof:

```text
correlated assistant turn
→ exact BEGIN marker
→ exact real ZIP download control
→ exact END marker
```

A ZIP elsewhere on the page or elsewhere in the assistant turn must not satisfy this contract.

---

## 7. Required Web ChatGPT prompt wording for ZIP jobs

The caller (Luna/Postman/Harness) should explicitly tell Web ChatGPT that the filename line must become the real generated ZIP control.

Use wording equivalent to:

```text
When your work is complete, create exactly ONE real downloadable ZIP attachment.

The exact filename is:
<EXPECTED_FILENAME>

Your FINAL assistant response must have exactly this structure:

<<<POSTMAN_RESULT_BEGIN:<REQUEST_ID>>>
<EXPECTED_FILENAME>
<<<POSTMAN_RESULT_END:<REQUEST_ID>>>

CRITICAL:
- the middle line must be the real downloadable ZIP attachment/control itself;
- it must not be plain text pretending to be a filename;
- the visible attachment/link label must be exactly <EXPECTED_FILENAME>;
- the real ZIP control must physically appear between BEGIN and END;
- do not write a generic label such as "Download ZIP" instead of the filename;
- create no second ZIP or attachment;
- mention no other ZIP filename;
- add no explanatory text before BEGIN or after END.
```

For smoke tests, keep the prompt minimal and deterministic.

---

## 8. Required self-check before delivery

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
[ ] BEGIN appears exactly once
[ ] END appears exactly once
[ ] real ZIP control is physically between BEGIN and END
[ ] visible ZIP-control label equals exact expected filename
[ ] no second attachment or ZIP filename appears
```

If exact base state or required source files cannot be obtained, do not guess a patch. Return a diagnostic text result instead of a knowingly incompatible artifact.

---

## 9. Prohibited behavior

Web ChatGPT must not:

- perform GitHub writes without separate explicit authorization;
- create commits, branches, Pull Requests, or Issues as part of normal browser-first result delivery;
- change trusted request metadata;
- choose `origin_agent_id` or another destination;
- create multiple competing result ZIPs;
- return a ZIP under the wrong filename;
- render only plain filename text instead of the actual downloadable artifact control;
- use a generic visible label when exact filename labeling is available;
- build patches from memory or incomplete snippets;
- include credentials, cookies, tokens, browser profiles, Local Storage dumps, or other sensitive browser state in artifacts;
- apply the proposed result directly to the repository.

---

## 10. Development-only `LUNA_PROMPT.md`

`LUNA_PROMPT.md` is used only in the temporary manual development handoff:

```text
Web ChatGPT -> implementation ZIP -> Luna/Harness
```

It is not required by the production Web Postman result ZIP contract.

---

## 11. Minimal ZIP-result response template

The requested final rendered response is:

```text
<<<POSTMAN_RESULT_BEGIN:<requestId>>>
<exact expected filename as the REAL downloadable ZIP control>
<<<POSTMAN_RESULT_END:<requestId>>>
```

There must be no other text or attachment in that final ZIP-result turn.
