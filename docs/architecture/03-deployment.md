# Deployment & CI/CD Flow

## Pipeline Overview

```mermaid
flowchart LR
    Dev[Developer] -->|git push| GH[GitHub]
    GH --> GHA[GitHub Actions]

    subgraph Pipeline["GitHub Actions Workflow"]
        Lint[Lint]
        Test[Test]
        Build[Build Container]
        Push[Push / Load Image]
        Deploy[Deploy via SSH over Tailscale]
        Verify[Health Check]
    end

    GHA --> Lint --> Test --> Build --> Push --> Deploy --> Verify
    Deploy -->|SSH over Tailscale| Apps[apps VM]
```

## Detailed Deployment Sequence (SSH-over-Tailscale – MVP path)

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant GH as GitHub
    participant GHA as GitHub Actions
    participant TS as Tailscale
    participant Apps as apps VM
    participant Caddy as Caddy
    participant App as Portfolio Website Container

    Dev->>GH: git push
    GH->>GHA: Trigger workflow
    GHA->>GHA: Lint + Test
    GHA->>GHA: Build image
    GHA->>TS: Authenticate / connect
    GHA->>Apps: SSH + docker pull/load + restart
    Apps->>App: New container starts
    App->>Caddy: Ready (or Caddy routes to new container)
    GHA->>Apps: Health check
    GHA-->>GH: Workflow success / failure
```

## Alternative: Self-Hosted Runner (Future / Optional)

```mermaid
flowchart TB
    GH[GitHub] -->|Job assigned| Runner[Self-hosted Runner<br/>on platform or apps VM]
    Runner -->|Local docker / compose| App[Portfolio Website]
```

## Notes

- **MVP decision:** GitHub-hosted runners + SSH over Tailscale.
- Secrets required by the pipeline (deploy keys, registry creds, etc.) should come from GitHub Secrets or, better, be fetched via OpenBao where practical.
- Rollback = redeploy previous known-good image tag + documented procedure in runbooks.
- The portfolio site migration from the current Cloudflare Worker will have its own cut-over and rollback plan.
