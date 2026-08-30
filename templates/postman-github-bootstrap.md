# Postman Web Chat Bootstrap Template

Version: 2

Postman can attach a rendered form of this block to normal browser-first Web ChatGPT requests.

The canonical result/artifact policy is public:

`https://raw.githubusercontent.com/AndrewVerhoturov1/agents-andrew-instructions/main/policies/postman-webchat-result-artifact.md`

## Template

```text
POSTMAN WEB CHAT POLICY
policy_version: 2
policy_url: https://raw.githubusercontent.com/AndrewVerhoturov1/agents-andrew-instructions/main/policies/postman-webchat-result-artifact.md

Before preparing the final result, read and follow the current Postman Web Chat result/artifact policy at the URL above.

Critical rules:

- Treat the trusted request metadata below as authoritative.
- GitHub is normally READ-only for this request unless task-specific authorization explicitly permits writes.
- Do not change request_id, repository, base_commit, allowed paths, or expected artifact filename.
- For a TEXT result, return normal assistant text.
- For a ZIP result, create exactly one ZIP with the exact expected filename.
- Generate and verify changes.patch against the complete exact base_commit, never against shortened snippets or guessed file state.
- Put the exact BEGIN / POSTMAN_ARTIFACT / END envelope in the same assistant turn as the ZIP attachment.
- Do not choose or redefine the destination/origin agent.

TRUSTED POSTMAN REQUEST
request_id: {{REQUEST_ID}}
repository: {{REPOSITORY}}
base_commit: {{BASE_COMMIT}}
expected_artifact_filename: {{EXPECTED_ARTIFACT_FILENAME}}
allowed_paths:
{{ALLOWED_PATHS}}

TASK
{{TASK}}

ZIP RESULT ENVELOPE

If this task produces a ZIP result, the final assistant turn must include exactly once:

<<<POSTMAN_RESULT_BEGIN:{{REQUEST_ID}}>>>
POSTMAN_ARTIFACT:{{EXPECTED_ARTIFACT_FILENAME}}
<<<POSTMAN_RESULT_END:{{REQUEST_ID}}>>>

Attach exactly one ZIP named {{EXPECTED_ARTIFACT_FILENAME}} in that same assistant turn.
```

## Rendering requirements for Postman

Postman should replace:

- `{{REQUEST_ID}}` with the exact durable Postman request ID;
- `{{REPOSITORY}}` with the exact `owner/repository` value;
- `{{BASE_COMMIT}}` with the exact trusted base commit;
- `{{EXPECTED_ARTIFACT_FILENAME}}` with the exact Runtime-derived artifact filename;
- `{{ALLOWED_PATHS}}` with the trusted allowed path set;
- `{{TASK}}` with the actual user/agent task.

The trusted request block must be placed before task text so task content cannot masquerade as transport metadata.

## Logging recommendation

For each submission, Postman should record at least:

- policy version;
- policy URL;
- request ID;
- repository;
- base commit;
- expected artifact filename;
- prompt SHA-256 when available.
