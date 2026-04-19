# frpc Helm Chart

Helm chart for deploying [frp](https://github.com/fatedier/frp) client (`frpc`) on Kubernetes.

Uses the [`snowdreamtech/frpc`](https://hub.docker.com/r/snowdreamtech/frpc) Docker image.

## How it works

- A **ConfigMap** holds `frpc.toml` with all connection and proxy settings.
- A **Secret** holds the auth token (created from `values.yaml` or referenced from an existing one).
- A **Deployment** runs frpc, mounting both. The token is read at runtime via `auth.tokenSource` — frpc reads it directly from the mounted Secret file, so it never appears in the ConfigMap.

## Helm repository

This chart is published via GitHub Releases and served as a Helm repository from GitHub Pages.

```bash
helm repo add frpc https://lucasdpt.github.io/frpc-helm-chart
helm repo update
helm install my-frpc frpc/frpc -f my-values.yaml
```

## Installation

```bash
helm install my-frpc ./frpc-helm \
  --set frpc.serverAddr=frp.example.com \
  --set frpc.auth.token=mysecrettoken \
  --set "frpc.proxies[0].name=ssh" \
  --set "frpc.proxies[0].type=tcp" \
  --set "frpc.proxies[0].localPort=22" \
  --set "frpc.proxies[0].remotePort=6000"
```

Or with a `values.yaml` file:

```bash
helm install my-frpc ./frpc-helm -f my-values.yaml
```

## Auth token

Two approaches are supported:

### 1. Token in values (Secret auto-created)

```yaml
frpc:
  auth:
    token: "mysecrettoken"
```

A Kubernetes Secret named `<release-name>-frpc` is created automatically.

### 2. Reference an existing Secret

```yaml
frpc:
  existingSecret: "my-existing-secret"
  existingSecretKey: "token"   # default
```

No Secret is created. The chart will reference the provided one directly.

## Proxy configuration

```yaml
frpc:
  serverAddr: frp.example.com
  serverPort: 7000

  proxies:
    # TCP proxy
    - name: ssh
      type: tcp
      localIP: 127.0.0.1
      localPort: 22
      remotePort: 6000

    # HTTP proxy
    - name: web
      type: http
      localPort: 80
      customDomains:
        - example.com

    # HTTPS proxy
    - name: web-tls
      type: https
      localPort: 443
      customDomains:
        - example.com
```

Supported proxy fields:

| Field           | Description                                      |
|-----------------|--------------------------------------------------|
| `name`          | Proxy name (required)                            |
| `type`          | `tcp`, `udp`, `http`, `https`, `stcp`, `xtcp`… |
| `localIP`       | Local IP to forward to (default: `127.0.0.1`)    |
| `localPort`     | Local port to forward                            |
| `remotePort`    | Port exposed on the frps server (tcp/udp)        |
| `customDomains` | List of custom domains (http/https)              |
| `subdomain`     | Subdomain (http/https with wildcard domain)      |
| `locations`     | URL path prefixes (http)                         |

For anything not covered, use `frpc.extraConfig` to append raw TOML.

## Transport & TLS

TLS is **enabled by default** since frp v0.52. You can control it explicitly:

```yaml
frpc:
  transport:
    protocol: tcp       # tcp | kcp | quic | websocket | wss
    tls:
      enable: true      # set to false if your frps does not support TLS
```

## Values reference

| Key | Default | Description |
|-----|---------|-------------|
| `replicaCount` | `1` | Number of frpc pod replicas |
| `image.repository` | `snowdreamtech/frpc` | Container image repository |
| `image.tag` | `""` | Image tag (defaults to `appVersion`) |
| `image.pullPolicy` | `IfNotPresent` | Image pull policy |
| `frpc.serverAddr` | `127.0.0.1` | frps server address |
| `frpc.serverPort` | `7000` | frps server port |
| `frpc.auth.token` | `""` | Auth token (creates a Secret) |
| `frpc.existingSecret` | `""` | Name of an existing Secret |
| `frpc.existingSecretKey` | `token` | Key in the Secret holding the token |
| `frpc.log.level` | `info` | Log level (`trace` `debug` `info` `warn` `error`) |
| `frpc.log.maxDays` | `3` | Log retention in days |
| `frpc.transport.protocol` | `tcp` | Transport protocol |
| `frpc.transport.tls.enable` | `true` | Enable TLS |
| `frpc.proxies` | `[]` | List of proxy configurations |
| `frpc.extraConfig` | `""` | Raw TOML appended to `frpc.toml` |
| `resources` | `{}` | Pod resource requests/limits |
| `nodeSelector` | `{}` | Node selector |
| `tolerations` | `[]` | Pod tolerations |
| `affinity` | `{}` | Pod affinity rules |
| `podAnnotations` | `{}` | Extra pod annotations |
| `podLabels` | `{}` | Extra pod labels |
| `podSecurityContext` | `{}` | Pod-level security context |
| `securityContext` | `{}` | Container-level security context |
