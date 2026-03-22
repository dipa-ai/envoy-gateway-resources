# envoy-gateway-resources

A Helm chart for managing Envoy Gateway resources: GatewayClass, Gateway, and HTTP-to-HTTPS redirect.

> This chart does not install the Envoy Gateway controller itself. It only creates the resources that configure an already running Envoy Gateway installation.

## Prerequisites

- Kubernetes cluster with [Envoy Gateway](https://gateway.envoyproxy.io/) installed
- [cert-manager](https://cert-manager.io/) (optional, for automatic TLS certificates)

## Installation

```bash
helm install envoy-gateway . -n envoy-gateway --create-namespace
```

## Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `gatewayClass.enabled` | Create GatewayClass resource | `true` |
| `gatewayClass.controllerName` | Controller name for GatewayClass | `gateway.envoyproxy.io/gatewayclass-controller` |
| `gateway.enabled` | Create Gateway resource | `true` |
| `gateway.annotations` | Annotations for Gateway (e.g. cert-manager) | `{}` |
| `gateway.listeners` | Gateway listener configuration | HTTP 80, HTTPS 443 |
| `HTTPToHTTPSRedirectFilter.enabled` | Create HTTP-to-HTTPS redirect route | `true` |
| `HTTPToHTTPSRedirectFilter.redirectStatusCode` | Redirect status code | `301` |

## TLS Configuration

### With cert-manager

```yaml
gateway:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  listeners:
    - name: http
      protocol: HTTP
      port: 80
      allowedRoutes:
        namespaces:
          from: All
    - name: https
      protocol: HTTPS
      port: 443
      allowedRoutes:
        namespaces:
          from: All
      tls:
        certificateRefs:
          - group: ""
            kind: Secret
            name: my-tls-secret  # will be created automatically by cert-manager
        mode: Terminate
```

### With existing Secret

```yaml
gateway:
  listeners:
    - name: https
      protocol: HTTPS
      port: 443
      allowedRoutes:
        namespaces:
          from: All
      tls:
        certificateRefs:
          - group: ""
            kind: Secret
            name: my-existing-secret
        mode: Terminate
```

## Multiple Gateways

You can deploy multiple independent Gateways (e.g. external and internal) by installing the chart multiple times with different release names:

```bash
helm install external . -n envoy-gateway -f values-external.yaml
helm install internal . -n envoy-gateway -f values-internal.yaml
```

Each release will create its own GatewayClass and Gateway with a unique name derived from the release name.
