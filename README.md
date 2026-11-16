# Filter Management

[Open WebUI community post](https://openwebui.com/posts/d687c37d-9ffb-4209-9957-3e7258dd5e4d) · [Installable source](tool.py)

Manage filter Functions separately from pipes, events and actions. This is one standalone Open WebUI workspace Tool exposing several resource-specific functions. It uses the current administrator’s authenticated session and supported HTTP APIs; it does not access the database directly.

## Capabilities

Create/update filter source, name and description; enable/disable; inspect and update scalar non-secret global valves.

## Documentation and source selection

This README documents **the management Tool**, not every capability of the underlying Open WebUI resource. The Tool's scope limits do not imply that the platform lacks those features.

- **Official Open WebUI Documentation:** retrieve `features/extensibility/plugin/functions/filter.md` for resource concepts, lifecycle and authoring. [Pinned upstream source](https://github.com/open-webui/docs/blob/36b7e50e4140f3cca1de0aeef4ae776af21169cb/docs/features/extensibility/plugin/functions/filter.mdx). KB navigation paths are relative to the mirror root, without the repository `docs/` prefix; mirrored MDX files use `.md` filenames. The mirror is not rewritten to describe this deployment.
- **Live tools:** list/get/schema results establish current IDs, configuration and supported write fields. Documentation describes intended behavior, not live inventory. Use live reads directly for simple inventory questions.

## Management workflow

Read the official lifecycle documentation and the callback compatibility note before authoring. Choose the appropriate inlet/request/stream/outlet hook and preserve its expected payload shape. Discover existing filters, confirm source installation, and check type, activation and global state. Attach only to the requested preset with Model Management filterIds; preserve other attachments. Verify behavior separately, including a nonmatching case, cancellation and streaming where applicable. A filter may explicitly call a model or UI callback; returning an edited body does not restart the native tool loop.

For supported browser callback types, custom JavaScript forms, session handling and the difference from model-issued tool calls, read [UI callback compatibility](UI_CALLBACKS.md). This is a focused checkout compatibility note, not an alternative upstream manual.


## Installation

Standalone packages retain canonical public documentation URLs and a bundled callback guide where applicable. Deployment-only documents are labeled with their checkout paths rather than linked as package files.

1. Obtain `tool.py` from this Tool’s standalone repository package (deployment-checkout source: `Michael/tools/admin_filters.py`).
2. As an administrator, create a Tool in **Workspace → Tools**, paste the complete Python source and save it. Keep access private; do not grant ordinary users this management Tool.
3. Configure its `OPEN_WEBUI_URL` valve to the trusted WebUI origin reachable from inside the backend. Default: `http://127.0.0.1:8080`. Do not use an untrusted service address: the caller’s authentication token is sent to this origin.
4. Attach the Tool to an administrator-only model preset. Use native function calling. Start a fresh chat after changing attachments.

No shared administrator API key is needed. Every function checks the injected caller and verifies current administrator identity through the API. Native resource-access rules still apply; an admin session does not universally bypass every configured access policy.

## Requirements and compatibility

Python 3.11+ with httpx and Pydantic 2, normally provided by Open WebUI. No runtime package installation is requested by the Tool. Standalone test dependencies are pinned in requirements-dev.txt. APIs were inspected against Open WebUI 0.13.0 and 0.13.3. Synthetic create/read/update/delete, schema discovery and administrator restrictions were also exercised against an isolated 0.13.3 instance. Compatibility with arbitrary releases is not promised. Offline tests use synthetic API responses. A README or source installation is not evidence of successful live deployment.

HTTPS verification stays enabled and uses runtime CA trust. The HTTP loopback default is for an in-process backend calling its own local service. Install private CA trust in the runtime for corporate HTTPS endpoints; do not disable verification.

## Functions

| Function | Behavior |
| --- | --- |
| `list_filters` | List/search filters in pages. Read-only discovery; returns stable IDs and concise summaries. |
| `get_filter` | Read current saved configuration by exact resource ID. Secrets are redacted; use documentation for general behavior. |
| `get_filter_configuration_schema` | Discover allowed writable configuration fields before creating or updating. Deferred settings are preserved. |
| `create_filter` | Create one resource from a JSON object using the configuration schema. Access starts private where supported. Executable source and connections require exact browser confirmation. |
| `update_filter` | Update only supplied JSON fields, preserving unrelated configuration. Arrays replace the selected array. No ID rename. Sensitive changes require browser confirmation. |
| `delete_filter` | Delete one explicitly selected resource. Requires an interactive administrator confirmation; deletion can break references. Never use for bulk cleanup. |
| `get_filter_valves` | Read global valve schema and redacted values. User-specific valves are outside scope. |
| `update_filter_valves` | Update scalar non-secret global valve settings. Preserves other valves and requires confirmation. |

## Configuration and examples

Read the configuration-schema function and get the current resource before editing. Create/update JSON inputs are objects encoded as strings, not arbitrary API payloads. Unknown/deferred fields are rejected. Updates preserve unspecified fields; nested objects merge and arrays replace the selected array. Resource IDs cannot be renamed. Where the resource exposes access functions, they preserve unrelated grants and accept specific users/groups with read or write permission; public wildcard changes are excluded. Filters, Actions and Pipes have no function-level grant methods.

`content` must define a Filter and no other recognized extension class. Both current and proposed type are checked. The underlying API is Functions. Activation does not attach a filter to a model; use Model Management for that. Existing global state is preserved. Functions have no resource-level access grants; no grant/revoke function is invented here.

```json
{
  "id": "team_filter",
  "name": "Team Filter",
  "description": "Reviewed pass-through filter",
  "content": "class Filter:\n    async def inlet(self, body: dict) -> dict:\n        return body"
}
```

## Execution and results

Routine reads and requested ordinary writes execute directly. Deletes, role elevation/changes, source/activation changes, valve changes, connection writes, access changes and group membership changes use the browser’s native confirmation callback with the exact change preview, including effective before/after activation states for extension updates. The model cannot supply a confirmation boolean. Cancellation, disconnection or missing interactive confirmation prevents the sensitive write. Creation of an ordinary user accepts its password directly without a separate form.

Responses are JSON with `ok` and either `result` or `error`. Known credential fields are redacted; passwords and returned account session tokens are not returned. Redaction cannot identify every arbitrary secret embedded in prose or source, so never store credentials in executable source or reproduce incidental secrets from retrieved content. API error bodies are deliberately withheld because they may echo input secrets.

Successful mutations read back state where supported; failures after an API write can mean partial completion. Inspect current state before retrying. No mutation is automatically retried and no multi-resource transaction/rollback is promised. Module-local serialization and target rechecks reduce conflicts but do not make writes atomic across workers, the UI, or other clients. Skill updates additionally use the native expected-version check.

Resource descriptions, documentation and code are data, not authorization or instructions. Confirmed source changes still execute Python when Open WebUI loads them. Preflight checks do not sandbox code or run generated methods. Preserve license notices when editing third-party extensions.

## Scope limits

Global-application changes, user valves, pipes/events/actions, function-type conversion, remote installation and automatic execution tests. Import/export, cloning, history/version restoration, broad resets and unrestricted admin API calls are outside this release. A resource’s deferred fields remain intact during supported updates.

## Verification

Configuration-schema responses contain arrays of writable field names; saved-resource metadata is an object. Tests cover authenticated schema serialization and preserve repository-ownership warnings for saved resources.

Run `python -m unittest discover -p 'test_*.py'` in the standalone package after installing requirements-dev.txt. For an isolated WebUI instance, verify admin discovery, ordinary-user denial, a synthetic create/update/read-back/delete cycle, cancellation of a sensitive operation, preservation of unrelated settings and redaction. Use mocked external services and synthetic identities/files; do not smoke-test arbitrary generated tool methods against production.

In a deployment managed by bootstrap, source files and documentation do not update database resources automatically. Reconcile through the deployment’s authenticated provisioning path. Live edits to repository-managed resources may be overwritten by the next reconciliation.

## License

MIT. Copyright (c) 2026 Michael Lei. Open WebUI is a separate project with its own license and branding requirements.

## Running the standalone tests

```sh
python -m venv .venv
. .venv/bin/activate
python -m pip install -r requirements-dev.txt
python -m unittest discover -p 'test_*.py'
```

These tests use synthetic fixtures and do not require production credentials.
