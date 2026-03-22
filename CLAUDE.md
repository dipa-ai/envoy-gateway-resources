# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Helm chart that creates Envoy Gateway **resources** (GatewayClass, Gateway, HTTPRoute for HTTP-to-HTTPS redirect). It does **not** install the Envoy Gateway controller itself — it assumes Envoy Gateway is already running in the cluster.

## Common Commands

```bash
# Install the chart
helm install envoy-gateway . -n envoy-gateway --create-namespace

# Lint the chart
helm lint .

# Render templates locally (dry-run)
helm template my-release .

# Render with custom values
helm template my-release . -f values-custom.yaml
```

## Architecture

This is a minimal Helm chart with three conditional resources, all controlled by `values.yaml` toggles:

- **GatewayClass** (`templates/gateway-class.yaml`) — references the Envoy Gateway controller. Enabled via `gatewayClass.enabled`.
- **Gateway** (`templates/gateway.yaml`) — creates listeners (HTTP/HTTPS). Supports cert-manager annotations. Enabled via `gateway.enabled`.
- **HTTPRoute redirect** (`templates/http-route-http-redirect-filter.yaml`) — attaches to the Gateway's HTTP listener and redirects to HTTPS. Enabled via `HTTPToHTTPSRedirectFilter.enabled`.

Resource names are derived from the Helm release name via the `envoy-gateway.fullname` helper in `templates/_helpers.tpl`. This allows multiple independent Gateway deployments (e.g., external/internal) by installing the chart with different release names.

## CI

GitHub Actions workflow (`.github/workflows/docker-build.yml`) triggers on `v*` tags. It builds/pushes a Docker image and packages/pushes a Helm chart to an OCI registry. Note: this workflow references a different chart path (`install/kubernetes/chart`) and a different app name — it appears to be copied from a parent project and may need updating for this repo.
