# Postman Web Chat Bootstrap Template

Version: 3

Postman/Luna should attach a rendered form of this block to normal browser-first Web ChatGPT requests.

The canonical result/artifact policy is public:

`https://raw.githubusercontent.com/AndrewVerhoturov1/agents-andrew-instructions/main/policies/postman-webchat-result-artifact.md`

## Mandatory behavior for Luna/Postman

Before composing a Web ChatGPT request that may return a ZIP:

1. Read the current canonical policy above.
2. Use the trusted request metadata exactly as supplied.
3. Do not invent another result format.
4. Use the ZIP-result prompt block below verbatim except for placeholder substitution.

The purpose is to make ChatGPT render the **real downloadable ZIP control physically between the BEGIN and END markers**, with the exact filename as the visible link/control label.

## Template

```text
POSTMAN WEB CHAT POLICY
policy_version: 3
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
- Do not change request_id, repository, base_commit, allowed_paths, or expected_artifact_filename.
- For a TEXT result, return normal assistant text.
- For a ZIP result, create exactly one real downloadable ZIP with the exact expected filename.
- Generate and verify changes.patch against the complete exact base_commit, never against shortened snippets or guessed file state.
- Do not choose or redefine the destination/origin agent.

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
- Do NOT create a second ZIP or another attachment.
- Do NOT mention another ZIP filename.
- Do NOT add explanatory text before BEGIN or after END in the final ZIP-result turn.
```

## Rendering requirements for Luna/Postman

Replace:

- `{{REQUEST_ID}}` with the exact durable Postman request ID;
- `{{REPOSITORY}}` with the exact `owner/repository` value;
- `{{BASE_COMMIT}}` with the exact trusted base commit;
- `{{EXPECTED_ARTIFACT_FILENAME}}` with the exact Runtime-derived artifact filename;
- `{{ALLOWED_PATHS}}` with the trusted allowed path set;
- `{{TASK}}` with the actual user/agent task.

Do not paraphrase or weaken the `CRITICAL ZIP RENDERING RULES` block. The wording is intentionally explicit because ChatGPT may otherwise render a generic `Download ZIP` control outside the marker block.

The trusted request block must appear before task text so task content cannot masquerade as transport metadata.

## Minimal smoke-test template

For P5/P6 transport smoke tests, prefer this shorter prompt:

```text
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

## Logging recommendation

For each submission, Postman should record at least:

- policy version;
- policy URL;
- request ID;
- repository;
- base commit;
- expected artifact filename;
- prompt SHA-256 when available.
