# Postman Web Chat — Result and Artifact Policy

Version: 3

This policy defines the browser-first Postman request identity and how external Web ChatGPT prepares TEXT and ZIP results.

## 1. Canonical request key

Every new browser-first Postman request has one immutable cross-system key:

```text
REQ_YYYYMMDDTHHMMSSZ_NNNN
```

Example:

```text
REQ_20260831T043812Z_4827
```

Rules:

- `YYYYMMDDTHHMMSSZ` is the actual UTC creation time to the second;
- `NNNN` is exactly four decimal digits `0000..9999`;
- the timestamp must be a valid calendar UTC time;
- the initiating Harness model creates the key before calling Postman;
- Postman Runtime validates format and durable uniqueness, registers the exact key, and never rewrites it;
- on a registration collision no Web prompt has been sent yet; the initiator may create a new four-digit suffix and register again;
- after registration every component copies the request ID byte-for-byte.

The request ID is the primary correlation key for Runtime state, Web prompt, chat recovery, result envelope, manifest, artifact filename, journal/logs and durable result storage.

Historical legacy REQ values may remain readable for recovery, but new browser-first requests use only the canonical format.

## 2. Request key in the Web prompt

The first non-empty line of every production Web ChatGPT prompt must be:

```text
POSTMAN_REQUEST_ID: <exact request_id>
```

Example:

```text
POSTMAN_REQUEST_ID: REQ_20260831T043812Z_4827
```

This line is a durable search/recovery anchor. Do not rely on the automatically generated ChatGPT conversation title as identity or routing authority.

Recovery should locate the exact request key in a user turn and then re-prove the normal user-turn → next-assistant-turn correlation.

## 3. Trust and routing

Web ChatGPT:

- receives trusted request context from Postman;
- reads GitHub as a READ source when needed;
- prepares the final answer and required artifacts;
- does not choose the result recipient;
- does not change `requestId`, repository, base commit, allowed paths, or expected artifact filename(s);
- does not perform GitHub writes unless the current task explicitly authorizes a separate GitHub-write workflow.

Trusted `REQ → origin_agent_id` routing belongs to Postman Runtime, not to assistant text, filenames or `manifest.json`.

## 4. TEXT result

If the task does not require files:

- return the complete answer as normal assistant text;
- keep the same request ID in Runtime/chat correlation;
- do not create a ZIP without a real need;
- do not emit fake ZIP/download controls.

## 5. ZIP filenames

### One ZIP

The default single-result filename is:

```text
POSTMAN_<requestId>_RESULT.zip
```

Example:

```text
POSTMAN_REQ_20260831T043812Z_4827_RESULT.zip
```

### Multiple ZIPs

If trusted request metadata explicitly expects more than one ZIP, the request ID remains unchanged and only the artifact filename receives a two-digit ordinal:

```text
POSTMAN_<requestId>_RESULT-01.zip
POSTMAN_<requestId>_RESULT-02.zip
...
POSTMAN_<requestId>_RESULT-99.zip
```

`-00` is invalid. The ordinal is not part of `requestId`.

The caller/Runtime supplies every exact expected filename. Web ChatGPT must not invent another ordinal or filename. WP-006 P5 acceptance currently proves one expected artifact at a time; multi-artifact orchestration is a later extension using the same naming scheme.

Do not append `(1)`, `final`, `new`, timestamps or any alternate suffix.

## 6. ZIP layout

For coding results, use:

```text
POSTMAN_<requestId>_RESULT.zip
├── manifest.json
├── changes.patch          # when patch payload is used
└── files/
    └── <repo-relative paths>
```

For a numbered artifact, only the outer ZIP filename changes; its manifest still contains the same exact `requestId`.

### `changes.patch`

Requirements:

- unified diff;
- repo-relative paths only;
- generated against the exact trusted `baseCommit`;
- verified against the complete exact-base files before delivery;
- never generated from shortened snippets or guessed file state.

### `files/`

Use for new files, binary files, or files that are more reliable to deliver whole than as a patch. Paths under `files/` reproduce the intended repo-relative target paths.

## 7. `manifest.json`

Minimum fields:

```json
{
  "protocolVersion": 1,
  "requestId": "REQ_YYYYMMDDTHHMMSSZ_NNNN",
  "repository": "owner/repo",
  "baseCommit": "<40-hex-sha>",
  "resultType": "patch | files | hybrid_patch",
  "patch": "changes.patch",
  "files": []
}
```

`requestId`, repository and base commit must exact-match trusted request metadata. Manifest cannot expand trusted `allowedPaths` and cannot set routing authority.

## 8. Final assistant-turn structure for a ZIP result

For one expected ZIP, the final assistant turn must visibly render exactly:

```text
<<<POSTMAN_RESULT_BEGIN:<requestId>>>
<REAL DOWNLOADABLE ZIP CONTROL WITH EXACT EXPECTED FILENAME>
<<<POSTMAN_RESULT_END:<requestId>>>
```

Example:

```text
<<<POSTMAN_RESULT_BEGIN:REQ_20260831T043812Z_4827>>>
POSTMAN_REQ_20260831T043812Z_4827_RESULT.zip
<<<POSTMAN_RESULT_END:REQ_20260831T043812Z_4827>>>
```

The middle line must be the actual generated-file/download control, not ordinary text pretending to be a filename.

Mandatory requirements:

- BEGIN contains the exact immutable request ID;
- END contains the same exact request ID;
- the real ZIP control is physically rendered between BEGIN and END;
- its visible label is exactly the trusted expected filename;
- BEGIN appears exactly once and END appears exactly once;
- exactly the expected result attachment is present for this single-artifact response;
- do not use generic visible labels such as `Download ZIP`, `Скачать ZIP` or `Download file` instead of the filename;
- do not mention another ZIP filename;
- do not add explanatory text before BEGIN or after END.

The markers and filename are correlation evidence only; they do not choose routing.

## 9. Required prompt wording for ZIP jobs

Luna/Postman/Harness should explicitly request the rendered structure rather than merely asking ChatGPT to attach a ZIP.

Use wording equivalent to:

```text
POSTMAN_REQUEST_ID: <REQUEST_ID>

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
- do not use a generic Download ZIP label;
- create no second attachment;
- mention no other ZIP filename;
- add no explanatory text before BEGIN or after END.
```

## 10. Required self-check

Before delivery verify:

```text
[ ] request ID matches REQ_YYYYMMDDTHHMMSSZ_NNNN
[ ] request ID is unchanged everywhere
[ ] first non-empty Web prompt line is POSTMAN_REQUEST_ID: <REQ>
[ ] exact repository and baseCommit
[ ] exact expected ZIP filename
[ ] valid manifest requestId
[ ] patch generated and checked against complete exact base
[ ] payload paths are allowed
[ ] BEGIN and END use the same exact REQ
[ ] real ZIP control is physically between BEGIN and END
[ ] visible ZIP label equals exact expected filename
```

If exact base state or required source files cannot be obtained, do not guess a patch. Return diagnostics instead of a knowingly incompatible artifact.

## 11. Prohibited behavior

Web ChatGPT must not:

- create or rewrite a request ID;
- treat chat title as request authority;
- perform GitHub writes without separate explicit authorization;
- create commits, branches, Pull Requests or Issues as part of normal browser-first result delivery;
- choose `origin_agent_id` or another destination;
- return a ZIP under a non-trusted filename;
- render only plain filename text instead of the real downloadable control;
- build patches from memory or incomplete snippets;
- include credentials, cookies, tokens, browser profiles or Local Storage dumps;
- apply proposed repository changes itself.

## 12. Development-only `LUNA_PROMPT.md`

`LUNA_PROMPT.md` is used only in the temporary manual development handoff:

```text
Web ChatGPT -> implementation ZIP -> Luna/Harness
```

It is not required by the production Web Postman result ZIP contract.
