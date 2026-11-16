# Filter Management

Manage Open WebUI Filter Functions from chat: inspect source, create message-processing filters and configure activation and settings.

## What you can do

Create/update filter source, name and description; enable/disable; inspect and update scalar non-secret global valves.

## Try asking

- “List the installed filters and explain what the selected one does.”
- “Create an inactive filter that adds an agreed formatting instruction before the model receives a request.”
- “Show the selected filter’s global settings and update its non-secret scalar setting after review.”

Examples describe requests to your model, not guaranteed outcomes from every model. It discovers actual resource IDs and supported fields before writing. Cross-resource requests require the corresponding individual Tools or the combined Admin Toolkit.

## Install

1. Import this listing, or paste the complete `tool.py` into **Workspace → Tools → Create Tool**.
2. Keep the Tool private and attach it to an administrator-only model preset. Enable **Native** function calling.
3. Set the `OPEN_WEBUI_URL` valve to the trusted WebUI origin reachable from its backend. The default is `http://127.0.0.1:8080` for a backend calling itself. The caller’s token is sent to this origin; HTTPS uses the runtime trust store.
4. Start a fresh chat and begin with a list or get request.

No shared admin API key, custom harness or documentation knowledge base is required. Every operation verifies the caller’s current administrator identity through the Open WebUI API. Existing resource-access checks still apply.

## Changes and confirmations

Reads and ordinary requested writes run directly. Deletions, access and membership changes, role changes, executable source/activation changes, valve updates and connection writes require an interactive browser confirmation showing the proposed change. If confirmation is unavailable or cancelled, that write does not proceed.

Updates preserve unspecified settings; attachment arrays replace the selected array. Known credential fields are redacted from results. Inspect current state after a failed write before retrying. Saving executable extensions runs server-side Python through Open WebUI; validation does not sandbox that code.

## Scope and compatibility

A Filter processes the request/response lifecycle. Enabling one does not attach it to a preset; use Model Management for attachments. Source changes require confirmation. Global-application changes and per-user settings are outside scope.

The individual management operations were exercised against Open WebUI 0.13.3; API contracts were also inspected on 0.13.0. Other releases may need compatibility changes. Python 3.11+, httpx and Pydantic 2 are required. No runtime dependency installation is requested.

The standalone repository package includes the full operation reference, synthetic tests and MIT license. Open WebUI is a separate project with its own license and branding requirements.
