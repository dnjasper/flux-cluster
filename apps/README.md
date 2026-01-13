# SeaVibes Kubernetes Cluster

**Apps:** Linkding (`ldpi.seavibes.org`) & Audiobookshelf (`audio.seavibes.org`)  

---

## Overview

This repository manages the SeaVibes Kubernetes cluster on k3s (via Rancher) using GitOps with Flux.  
Applications are exposed externally via Raefix (Traefik) and a Cloudflare tunnel, while internal communication is handled with ClusterIP services.  
Secrets are encrypted with `sops` and `age` for security.  



- [Linkding](./apps/linkding/README.md)  
- [Audiobookshelf](./apps/audiobookshelf/README.md) 
---

## Architecture

       ┌─────────────────┐
       │  Internet Users │
       └────────┬────────┘
                │
                ▼
       ┌───────────────────┐
       │ Cloudflared Tunnel│
       └────────┬──────────┘
                │
                ▼
       ┌─────────────────────┐
       │  Raefix (Traefik)   │
       │  Ingress Controller │
       └───────┬─────────────┘
     ┌─────────┴─────────┐
     ▼                   ▼
┌──────────────────┐ ┌───────────────────┐
│ ldpi.seavibes.org│ │ audio.seavibes.org│
│ Linkding Service │ │ Audiobookshelf    │
│ (ClusterIP) │    │ Service (ClusterIP) │
└─────────┬────────┘ └─────────┬─────────┘
          │                    │
          ▼                    ▼
┌──────────────┐ ┌─────────────────────┐
│ Linkding Pods│ │ Audiobookshelf Pods │
└──────────────┘ └─────────────────────┘




---

## Prerequisites

- k3s cluster managed via Rancher  
- kubectl configured for your cluster  
- Flux installed for GitOps  
- Raefix (Traefik) ingress controller  
- Cloudflare tunnel credentials (Cloudflared)  
- sops + age for secrets management  

---

## Apps

### Linkding

- **Internal Service:** ClusterIP  
- **Ingress:** `ldpi.seavibes.org`  
- **Access URL:** `https://ldpi.seavibes.org`  

**Example Kubernetes manifests:**

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: linkding
spec:
  type: ClusterIP
  selector:
    app: linkding
  ports:
    - port: 80
      targetPort: 8080

---
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: linkding-ingress
  annotations:
    kubernetes.io/ingress.class: "traefik"
spec:
  rules:
    - host: ldpi.seavibes.org
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: linkding
                port:
                  number: 80










Audiobookshelf

Internal Service: ClusterIP

Ingress: audio.seavibes.org

Access URL: https://audio.seavibes.org

# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: audiobookshelf
spec:
  type: ClusterIP
  selector:
    app: audiobookshelf
  ports:
    - port: 80
      targetPort: 80

---
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: audiobookshelf-ingress
  annotations:
    kubernetes.io/ingress.class: "traefik"
spec:
  rules:
    - host: audio.seavibes.org
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: audiobookshelf
                port:
                  number: 80

Deployment (GitOps)

With Flux, deployment is automatic via your Git repository. Typical structure:

├─ apps/
│  ├─ linkding/
│  │  ├─ deployment.yaml
│  │  ├─ service.yaml
│  │  └─ ingress.yaml
│  └─ audiobookshelf/
│     ├─ deployment.yaml
│     ├─ service.yaml
│     └─ ingress.yaml
└─ secrets/
   └─ encrypted-secrets.yaml  # encrypted with sops/age















