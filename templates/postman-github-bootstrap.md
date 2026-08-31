# Postman Web Chat Bootstrap Template

Version: 4

Postman/Luna should use this template for normal browser-first Web ChatGPT requests.

Canonical policy:

`https://raw.githubusercontent.com/AndrewVerhoturov1/agents-andrew-instructions/main/policies/postman-webchat-result-artifact.md`

## 1. Initiator creates the request key

Before calling `postman_async_send`, the initiating Harness model creates one request ID:

```text
REQ_YYYYMMDDTHHMMSSZ_NNNN
```

Example:

```text
REQ_20260831T043812Z_4827
```

Generation rules:

- use current UTC time to the second;
- append `_` plus exactly four random decimal digits;
- do not reuse a previous request ID;
- pass the exact value to Postman;
- if Runtime rejects a collision before Web transport, generate a new four-digit suffix and register again;
- after registration never rewrite the key.

Runtime validates/records the key and derives the internal message ID from it. The request key is then reused in Web prompt, recovery, envelope, manifest, filenames, logs and result storage.

## 2. Mandatory behavior for Luna/Postman

Before composing a Web ChatGPT request:

1. Read the current canonical policy above.
2. Use the registered request ID exactly as supplied.
3. Do not invent another correlation identifier.
4. Make the first non-empty Web prompt line `POSTMAN_REQUEST_ID: {{REQUEST_ID}}`.
5. For ZIP jobs, use the ZIP-result block below without weakening its rendering requirements.

Do not trust the automatically generated ChatGPT conversation title as the request identity. Recovery/search uses the exact request key in the user turn.

## 3. Template

```text
POSTMAN_REQUEST_ID: {{REQUEST_ID}}

POSTMAN WEB CHAT POLICY
policy_version: 4
policy_url: https://raw.githubusercontent.com/AndrewVerhoturov1/agents-andrew-instructions/main/policies/postman-webchat-result-artifact.md

Before preparing the final result, read and follow the current Postman Web Chat result/artifact policy at the URL above.

TRUSTED POSTMAN REQUEST
request_id: {{REQUEST_ID}}
repository: {{REPOSITORY}}
base_commit: {{BASE_COMMIT}}
expected_artifact_filename: {{EXPECTED_ARTIFACT_FILENAME}}
allowed_paths:
{{ALLOWED_PATHS}}

Critical rules:

- Treat the trusted request metadata above as authoritative.
- GitHub is normally READ-only unless this task explicitly authorizes writes.
- Never change the registered request_id.
- For a TEXT result, return normal assistant text.
- For a ZIP result, create exactly one real downloadable ZIP with the exact expected filename for this request.
- Generate and verify changes.patch against the complete exact base_commit, never against shortened snippets or guessed file state.
- Do not choose or redefine destination/origin routing.

TASK
{{TASK}}

ZIP RESULT CONTRACT

If this task produces a ZIP result, create exactly ONE real downloadable ZIP attachment.

The exact filename is:

{{EXPECTED_ARTIFACT_FILENAME}}

Your FINAL assistant response must have exactly this visible structure:

<<<POSTMAN_RESULT_BEGIN:{{REQUEST_ID}}>>>
{{EXPECTED_ARTIFACT_FILENAME}}
<<<POSTMAN_RESULT_END:{{REQUEST_ID}}>>>

CRITICAL ZIP RENDERING RULES:

- The middle line `{{EXPECTED_ARTIFACT_FILENAME}}` MUST be the real downloadable ZIP attachment/control itself.
- It MUST NOT be ordinary plain text pretending to be a filename.
- The visible attachment/link label MUST be exactly `{{EXPECTED_ARTIFACT_FILENAME}}`.
- The real ZIP control MUST physically appear between the BEGIN and END markers.
- Do NOT use a generic visible label such as `Download ZIP`, `Скачать ZIP`, or `Download file` instead of the exact filename.
- Do NOT create a second attachment for this single-artifact job.
- Do NOT mention another ZIP filename.
- Do NOT add explanatory text before BEGIN or after END in the final ZIP-result turn.
```

## 4. Artifact filename derivation

Single ZIP:

```text
POSTMAN_{{REQUEST_ID}}_RESULT.zip
```

If trusted request metadata explicitly expects multiple ZIPs, keep the same request ID and use:

```text
POSTMAN_{{REQUEST_ID}}_RESULT-01.zip
POSTMAN_{{REQUEST_ID}}_RESULT-02.zip
```

The ordinal is part of the filename only. Do not create it unless Runtime/request metadata expects it.

## 5. Rendering requirements

Replace:

- `{{REQUEST_ID}}` with the exact registered request ID;
- `{{REPOSITORY}}` with exact `owner/repository`;
- `{{BASE_COMMIT}}` with exact trusted base commit;
- `{{EXPECTED_ARTIFACT_FILENAME}}` with exact Runtime-derived artifact filename;
- `{{ALLOWED_PATHS}}` with trusted allowed paths;
- `{{TASK}}` with the actual task.

Do not paraphrase or weaken the critical ZIP rendering block.

## 6. Minimal P5 smoke template

```text
POSTMAN_REQUEST_ID: {{REQUEST_ID}}

Create exactly one real downloadable ZIP attachment named:
{{EXPECTED_ARTIFACT_FILENAME}}

Inside it create one UTF-8 file named `probe.txt` with contents:
WP006_OK

Your final response must be exactly:

<<<POSTMAN_RESULT_BEGIN:{{REQUEST_ID}}>>>
{{EXPECTED_ARTIFACT_FILENAME}}
<<<POSTMAN_RESULT_END:{{REQUEST_ID}}>>>

The middle line must be the actual clickable/downloadable ZIP control with that exact visible filename, not plain text.
The real ZIP control must physically appear between BEGIN and END.
No other text.
No other ZIP name.
No second attachment.
```

## 7. Logging

Record at least request ID, policy version/URL, repository, base commit, expected filename and prompt SHA-256 when available.
