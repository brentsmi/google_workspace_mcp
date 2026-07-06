# JIT Fork Notes

This is JIT's fork of [taylorwilsdon/google_workspace_mcp](https://github.com/taylorwilsdon/google_workspace_mcp)
(MIT). It exists to carry **one focused feature** until it is merged upstream.

## The patch: GCS attachment backend (`feat/gcs-attachment-backend`)

Upstream stages downloaded files on **instance-local disk** and serves them via
`/attachments/{id}` — unreliable on horizontally scaled / scale-to-zero
deployments (Cloud Run), and fully disabled in stateless mode.

This fork adds a **GCS-backed attachment storage** (`core/gcs_attachment_storage.py`):
files are staged in a GCS bucket and returned as **V4 signed URLs**. Works in
stateless mode (`WORKSPACE_MCP_STATELESS_MODE=true`) — credential/session
statelessness is unaffected.

### Configuration

| Env var | Meaning |
|---|---|
| `WORKSPACE_MCP_FILES_GCS_BUCKET` | enables the backend (bucket name) |
| `WORKSPACE_MCP_FILES_GCS_PREFIX` | object prefix (default `attachments`) |
| `WORKSPACE_MCP_FILES_SIGNED_URL_SECONDS` | URL validity (default 3600) |

### Deployment prerequisites

- Runtime service account needs `roles/storage.objectAdmin` on the bucket.
- Keyless signing (Cloud Run): runtime SA needs
  `roles/iam.serviceAccountTokenCreator` **on itself** (IAM signBlob).
- Configure a bucket **lifecycle rule** (auto-delete) — GCS lifecycle
  granularity is days; signed URLs expire after 1 h regardless.

## Versioning & exit strategy

- Fork tags: `v<upstream>-jit.<n>` (e.g. `v1.22.0-jit.1`).
- Consumed by the `mcp-workspace` deploy repo via git pin.
- An upstream PR for this feature is intended. **Once merged upstream, pin back
  to PyPI and archive this fork** — it is a bridge, not a permanent divergence.

## Syncing with upstream

```bash
git fetch upstream
git checkout main && git merge --ff-only upstream/main && git push origin main
git checkout feat/gcs-attachment-backend && git rebase <new-upstream-tag>
# retag: v<new-version>-jit.1
```
