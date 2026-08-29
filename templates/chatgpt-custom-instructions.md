# Recommended Global ChatGPT Custom Instructions — GitHub

Version: 1

Use only this short, general principle in global Web ChatGPT Custom Instructions. Keep Postman-specific routing and Issue delivery rules out of global settings.

## Recommended text

```text
When working with public GitHub repositories, prefer ordinary public web/GitHub access for information that is already publicly available.

Treat authenticated GitHub connector operations as privileged actions. Use connector reads only when public access is insufficient for the specific task, and use connector writes only when the requested operation actually requires a GitHub mutation.

Do not use authenticated connector calls merely because they are available. Prefer the minimum necessary capability and the narrowest operation that completes the task.

When a task supplies a task-specific GitHub access policy, trusted delivery envelope, repository/Issue target, or other explicit connector policy, follow that task-specific policy for that task.
```

## Why the global text is intentionally short

Global Custom Instructions apply to unrelated chats as well as Postman jobs. They should therefore contain only reusable least-privilege principles.

Do not put Postman-specific rules such as:

- `WAITING → READY`;
- exact Postman Issue routing;
- `POSTMAN_SIGNAL_SENT`;
- restrictions intended only for normal Postman jobs;

into global Custom Instructions.

Those rules belong in the Postman task policy:

`https://raw.githubusercontent.com/AndrewVerhoturov1/agents-andrew-instructions/main/policies/postman-webchat-github.md`
