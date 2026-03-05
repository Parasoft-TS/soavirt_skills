# API Spec Cache

Purpose: keep a local, repo-scoped copy of each server's OpenAPI spec so skill development does not depend on repeatedly fetching docs from a live server.

## Layout
- `docs/reference/api-spec-cache/<server-key>/openapi_v6.yaml`

Example:
- `docs/reference/api-spec-cache/localhost-9080/openapi_v6.yaml`

## Server Key Convention
- Use `<host>-<port>` when available.
- Replace special characters with `-`.

## Refresh Command (PowerShell)
Use only the server base URL:

```powershell
$baseUrl = 'http://localhost:9080/soavirt/api/v6'
$serverKey = 'localhost-9080'
$out = "docs/reference/api-spec-cache/$serverKey/openapi_v6.yaml"
New-Item -ItemType Directory -Path (Split-Path $out) -Force | Out-Null
$resp = Invoke-WebRequest -UseBasicParsing $baseUrl
if ($resp.Content -is [byte[]]) {
  $text = [System.Text.Encoding]::UTF8.GetString($resp.Content)
} else {
  $text = [string]$resp.Content
}
Set-Content -Path $out -Value $text -Encoding utf8
```

## Usage Guidance
- Use cached spec for endpoint/schema lookups during skill authoring.
- Re-refresh when server version changes or endpoint behavior appears inconsistent.
- Keep run-specific payload captures under `work/runs/...` (not in this cache).
