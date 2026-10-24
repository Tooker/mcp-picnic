# Docker image builds

The `Docker images` workflow tests the server and builds both container variants.
It publishes multi-platform images for `linux/amd64` and `linux/arm64` on pushes
to `main`, version tags matching `v*`, and manual runs on `main`. Pull requests
and manual runs on other branches validate without publishing. The tunnel image
also runs the existing smoke test with fictitious credentials and a local
control plane before publication.

Images are published to `ghcr.io/<owner>/<repository>` with the repository's
`GITHUB_TOKEN`. No Picnic credentials, OpenAI API key, or additional registry
secret is required for builds. The npm release workflow only runs in the
upstream repository; this fork's Docker publishing is independent of npm.

For this fork, use `ghcr.io/tooker/mcp-picnic`:

| Standalone tag | Tunnel tag | Contents |
| --- | --- | --- |
| `latest` | `tunnel` | Latest successful build of `main` |
| `v1.2.3` | `v1.2.3-tunnel` | Build of a matching version tag |
| `sha-<full-commit-sha>` | `sha-<full-commit-sha>-tunnel` | Build of a specific commit |

## Pull and update

After configuring `.env`, pull the desired variant and recreate its container.
For the tunnel deployment:

```bash
docker compose -f docker-compose.tunnel.yml pull
docker compose -f docker-compose.tunnel.yml up -d --no-build
```

For standalone HTTP, run `docker compose pull` and
`docker compose up -d --no-build` instead. Use the same commands for updates.

GitHub may initially create container packages as private. If pulling requires
authentication, log Docker in to `ghcr.io` with an account that has package read
access, or change the package's visibility in its GitHub settings to allow
anonymous pulls. Registry credentials stay on the Docker host.

## Local builds

The tunnel Compose file retains its build definition for development:

```bash
docker compose -f docker-compose.tunnel.yml build
docker compose -f docker-compose.tunnel.yml up -d --no-build --pull never
```

The next registry pull replaces the locally built image tag. The setup guides
also show how to build a separate local image for the offline smoke test.
