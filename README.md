# Release Guardian — demo

A minimal public demonstration that **GENESIS Release Guardian** installs and runs
inside a real GitHub Actions workflow.

Release Guardian is a pay-per-call x402 API compatibility gate: it detects breaking
API / OpenAPI / GraphQL / schema changes between two versions. No account, no API
key — payment is x402 USDC on Base (`0.005` quick / `0.019` deep).

## What this repo proves (without spending money)

The workflow runs the published action
`genesiscode2026/genesis-release-guardian@v1` in `dry-run` mode against two fixtures:

| Fixture | Change | Expected verdict |
|---|---|---|
| `spec/safe.json` | adds `POST /users` (additive) | `SAFE` |
| `spec/breaking.json` | removes `GET /users/{id}` | `BREAKING` |

Dry-run mode reads the fixture files, calls the live production endpoint, receives
the HTTP `402` payment challenge, decodes it, and reports the exact quoted price —
without signing or spending anything. This proves the orchestration layer
(checkout → read spec → call endpoint → parse challenge) end to end.

The `SAFE` vs `BREAKING` verdicts above are produced by the deterministic Release
Guardian engine and are covered by 8 value-fixture tests in the project test suite
(0 false-SAFE, 0 false-BREAKING). The live paid verdict is the final E2E step.

## Run it paid (your own workflow)

Add a repository secret `X402_PRIVATE_KEY` (a Base USDC-funded EIP-3009 signing key)
and drop `dry-run`:

```yaml
- uses: genesiscode2026/genesis-release-guardian@v1
  with:
    previous-spec: spec/baseline.json
    current-spec: spec/breaking.json
    mode: quick
    fail-on: breaking
    max-spend-usd: '0.01'
    private-key: ${{ secrets.X402_PRIVATE_KEY }}
```

## Privacy

Only the two explicitly supplied spec files leave the repository. The action does not
clone/upload your repository server-side, and it does not persist IP/geolocation or
use analytics trackers. Wallet addresses are necessarily public/on-chain.
