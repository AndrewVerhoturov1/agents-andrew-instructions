# agents-andrew-instructions

Public, version-controlled instructions for Andrew's agents and Web ChatGPT workflows.

The purpose of this repository is to keep reusable agent policies in one public location so models can read them through ordinary public web access. Authenticated connectors should be used only when the requested operation actually requires them.

## Canonical policies

- [`policies/github-public-first.md`](policies/github-public-first.md) — general GitHub least-privilege policy for all chats and agents.
- [`policies/postman-webchat-result-artifact.md`](policies/postman-webchat-result-artifact.md) — mandatory result/ZIP format and delivery policy for browser-first Postman → Web ChatGPT jobs.
- [`templates/postman-github-bootstrap.md`](templates/postman-github-bootstrap.md) — short trusted-request block Postman can attach to Web ChatGPT requests.
- [`templates/chatgpt-custom-instructions.md`](templates/chatgpt-custom-instructions.md) — recommended small global Custom Instructions block for Web ChatGPT.

## Canonical public URLs

Postman Web Chat result/artifact policy:

`https://raw.githubusercontent.com/AndrewVerhoturov1/agents-andrew-instructions/main/policies/postman-webchat-result-artifact.md`

General GitHub policy:

`https://raw.githubusercontent.com/AndrewVerhoturov1/agents-andrew-instructions/main/policies/github-public-first.md`

Postman bootstrap template:

`https://raw.githubusercontent.com/AndrewVerhoturov1/agents-andrew-instructions/main/templates/postman-github-bootstrap.md`

## Policy layering

Use the following order:

1. Platform/system safety requirements always remain in force.
2. Explicit task-specific authorization defines what the current task is allowed to change.
3. Postman trusted request bootstrap defines the exact request metadata supplied for the job.
4. The Postman Web Chat result/artifact policy defines normal TEXT/ZIP result delivery.
5. The general GitHub public-first policy applies to GitHub access unless a task explicitly authorizes a different write workflow.
6. Global Custom Instructions should contain only broad reusable principles.

## Browser-first Postman design principle

Normal Postman result delivery does not require GitHub writes.

The expected production path is:

```text
Postman trusted request
-> Web ChatGPT
-> assistant TEXT and/or exact result ZIP
-> correlated assistant turn
-> browser download by Web Postman
-> validation
-> durable local result
-> Postman Runtime READY
-> originating Harness Agent
```

GitHub is normally a READ source for Web ChatGPT. The visible assistant response and its correlated attachment are the browser transport result channel.

For ZIP results, Web ChatGPT must use the exact Runtime-supplied request metadata and filename, validate patches against the exact base, and emit the canonical BEGIN / POSTMAN_ARTIFACT / END envelope in the same assistant turn as the attachment.
