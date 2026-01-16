# Headscale Fleet Deployment

Headscale deployment for Kubernetes via Rancher Fleet.

## Components

- **Headscale** - Self-hosted Tailscale control plane
- **Headscale-UI** - Web management interface
- **Cloudflared** - Cloudflare Tunnel for secure ingress

## Prerequisites

1. Create Cloudflare Tunnel in Zero Trust dashboard
2. Configure tunnel routes:
   - `headscale.rkfleming.com` → `http://headscale.headscale.svc.cluster.local:8080`
   - `headscale-ui.rkfleming.com` → `http://headscale-ui.headscale.svc.cluster.local:80`

## Pre-deployment Secret

Create the Cloudflare tunnel token secret before Fleet deploys:

```bash
kubectl create namespace headscale
kubectl create secret generic cloudflared-token \
  --from-literal=token=<YOUR_TUNNEL_TOKEN> \
  -n headscale
```

## Post-deployment Setup

### Create API Key for UI

```bash
kubectl exec -it deployment/headscale -n headscale -- headscale apikeys create --expiration 365d
```

### Create a User

```bash
kubectl exec -it deployment/headscale -n headscale -- headscale users create myuser
```

### Generate Auth Key for Clients

```bash
kubectl exec -it deployment/headscale -n headscale -- headscale authkeys create --user myuser --reusable --expiration 24h
```

### Connect a Client

```bash
tailscale up --login-server https://headscale.rkfleming.com --authkey <AUTH_KEY>
```

## Files

| File | Purpose |
|------|---------|
| `01-namespace.yaml` | Namespace |
| `02-pvc.yaml` | Persistent storage |
| `03-configmap.yaml` | Headscale configuration |
| `04-headscale-deployment.yaml` | Headscale server |
| `05-headscale-ui-deployment.yaml` | Web UI |
| `06-cloudflared-deployment.yaml` | Cloudflare Tunnel |
