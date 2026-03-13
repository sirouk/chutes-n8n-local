# chutes-n8n-embed

Self-hosted n8n with:

- native `Sign in with Chutes` on Community n8n
- baked-in `n8n-nodes-chutes`
- local mode via the embedded `e2ee-proxy` certificate path
- public-domain mode via Caddy and ACME
- repeatable end-to-end coverage using a local test IdP

## Layout

- `bootstrap.sh`: primary install/reconfigure entry point
- `docker-compose*.yml`: base, local, domain, and test-IdP stacks
- `Dockerfile.n8n`: pinned n8n build with overlays and bundled Chutes nodes
- `Dockerfile.local-proxy`: local edge image for `e2ee-local-proxy.chutes.dev`
- `scripts/smoke-test.sh`: syntax/runtime smoke checks
- `scripts/e2e-test.sh`: destructive local E2E against the test Chutes IdP
- `n8n-overlays/`: native n8n SSO and UI overlays
- `tests/fake-chutes-idp/`: local test IdP used by CI and local E2E

## Local Development

This repo expects `n8n-nodes-chutes` to exist as a sibling checkout:

```text
workspace/
├── chutes-n8n-embed/
└── n8n-nodes-chutes/
```

`bootstrap.sh` syncs that sibling repo into `build/n8n-nodes-chutes/` before building the custom n8n image. On reruns, bootstrap also attempts a safe fast-forward refresh of the sibling `n8n-nodes-chutes` checkout when the branch is clean and has an upstream.

## Bootstrap

```bash
./bootstrap.sh
./bootstrap.sh --force
./bootstrap.sh --reset-owner-password
./bootstrap.sh --down
```

The script prompts for:

- install mode: `local` or `domain`
- rerun action on an existing install: `update` or `wipe` with `update` as the default
- Chutes OAuth client ID / secret
- ACME email when using a public domain

`update` rebuilds and reapplies configuration while preserving the existing Postgres and n8n data volumes. `wipe` recreates the stack from scratch, including data volumes and encrypted n8n state.

## Quality Gates

```bash
./scripts/smoke-test.sh --syntax
./scripts/smoke-test.sh
./scripts/e2e-test.sh
```

- `smoke-test.sh --syntax` is safe everywhere
- `e2e-test.sh` is destructive and recreates the local stack from scratch

## CI

The GitHub Actions workflow runs:

- syntax smoke checks
- local test-IdP E2E

The CI job checks out `n8n-nodes-chutes` as a sibling repository so the build path matches local development.
