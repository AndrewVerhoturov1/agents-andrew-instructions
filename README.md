# agents-andrew-instructions

Public, version-controlled instructions for Andrew's agents and Web ChatGPT workflows.

The purpose of this repository is to keep reusable agent policies in one public location so models can read them through ordinary public web access. Authenticated connectors should be used only when the requested operation actually requires them.

## Canonical policies

- [`policies/github-public-first.md`](policies/github-public-first.md) — general GitHub least-privilege policy for all chats and agents.
- [`policies/postman-webchat-github.md`](policies/postman-webchat-github.md) — mandatory GitHub policy for normal Postman → Web ChatGPT jobs.
- [`templates/postman-github-bootstrap.md`](templates/postman-github-bootstrap.md) — short block Postman should attach to every Web ChatGPT request.
- [`templates/chatgpt-custom-instructions.md`](templates/chatgpt-custom-instructions.md) — recommended small global Custom Instructions block for Web ChatGPT.

## Canonical public URLs

Postman policy:

`https://raw.githubusercontent.com/AndrewVerhoturov1/agents-andrew-instructions/main/policies/postman-webchat-github.md`

General GitHub policy:

`https://raw.githubusercontent.com/AndrewVerhoturov1/agents-andrew-instructions/main/policies/github-public-first.md`

Postman bootstrap template:

`https://raw.githubusercontent.com/AndrewVerhoturov1/agents-andrew-instructions/main/templates/postman-github-bootstrap.md`

## Policy layering

Use the following order:

1. Platform/system safety requirements always remain in force.
2. Explicit task-specific authorization defines what the current task is allowed to change.
3. Postman task bootstrap defines the exact request, repository, Issue, and request ID.
4. The Postman GitHub policy defines connector use for normal Postman jobs.
5. The general GitHub public-first policy applies as the default for other GitHub work.
6. Global Custom Instructions should contain only broad reusable principles, not Postman-specific Issue routing.

## Design principle

Public information should be read through public access whenever practical. Authenticated GitHub connector operations are privileged and should be minimized. A normal Postman result-delivery job should generally require zero connector reads and one connector write: updating the exact preassigned Postman Issue from `WAITING` to `READY` with the final response.
