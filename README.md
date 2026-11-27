![](./docs/banner.svg)

# wlstmd/lab

**lab** is the single source of truth repository containing Kubernetes resource manifest files for my Homelab GitOps environment powered by ArgoCD.

Covered domains include, but are not limited to, apex and subdomains of https://wlstmd.cloud.

An ArgoCD instance continuously monitors the `apps` folder, which contains manifest files that reference each application's manifest directory. When updates occur, these manifests trigger full resource synchronization. More information on this pattern is here: [App of Apps](https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/#app-of-apps-pattern-alternative).

## Contents

This repository defines many components of the Kubernetes cluster, such as:

* HTTP Routing (Traefik, Istio Ambient Gateway)
* TLS Certificate Management/Ordering (cert-manager)
* Observability (Grafana, Prometheus)
* GitOps Delivery (ArgoCD)
* and Actual Service Components.

## Sync Status

<!-- SYNC_TABLE_START -->

<table>
  
  <tr>
  <td><img src="https://argo.wlstmd.cloud/api/badge?name=apps&revision=false&showAppName=true" width="200px" /><br /></td>
  
  
  
  <td><img src="https://argo.wlstmd.cloud/api/badge?name=argocd&revision=false&showAppName=true" width="200px" /><br /></td>
  
  
  
  <td><img src="https://argo.wlstmd.cloud/api/badge?name=argocd-image-updater&revision=false&showAppName=true" width="200px" /><br /></td>
  
  
  
  <td><img src="https://argo.wlstmd.cloud/api/badge?name=authentik&revision=false&showAppName=true" width="200px" /><br /></td>
  </tr>
  
  <tr>
  <td><img src="https://argo.wlstmd.cloud/api/badge?name=authentik-ingress&revision=false&showAppName=true" width="200px" /><br /></td>
  
  
  
  <td><img src="https://argo.wlstmd.cloud/api/badge?name=daily-ai&revision=false&showAppName=true" width="200px" /><br /></td>
  
  
  
  <td><img src="https://argo.wlstmd.cloud/api/badge?name=dormitory&revision=false&showAppName=true" width="200px" /><br /></td>
  
  
  
  <td><img src="https://argo.wlstmd.cloud/api/badge?name=grafana&revision=false&showAppName=true" width="200px" /><br /></td>
  </tr>
  
  <tr>
  <td><img src="https://argo.wlstmd.cloud/api/badge?name=istio-ingress&revision=false&showAppName=true" width="200px" /><br /></td>
  
  
  
  <td><img src="https://argo.wlstmd.cloud/api/badge?name=keycap&revision=false&showAppName=true" width="200px" /><br /></td>
  
  
  
  <td><img src="https://argo.wlstmd.cloud/api/badge?name=kmu-room-finder&revision=false&showAppName=true" width="200px" /><br /></td>
  
  
  
  <td><img src="https://argo.wlstmd.cloud/api/badge?name=monitoring&revision=false&showAppName=true" width="200px" /><br /></td>
  </tr>
  
  <tr>
  <td><img src="https://argo.wlstmd.cloud/api/badge?name=prometheus&revision=false&showAppName=true" width="200px" /><br /></td>
  
  
  
  <td><img src="https://argo.wlstmd.cloud/api/badge?name=root&revision=false&showAppName=true" width="200px" /><br /></td>
  
  
  
  <td><img src="https://argo.wlstmd.cloud/api/badge?name=spotify&revision=false&showAppName=true" width="200px" /><br /></td>
  
  
  
  <td><img src="https://argo.wlstmd.cloud/api/badge?name=terminal&revision=false&showAppName=true" width="200px" /><br /></td>
  </tr>
  
  <tr>
  <td><img src="https://argo.wlstmd.cloud/api/badge?name=tree&revision=false&showAppName=true" width="200px" /><br /></td>
  
  
  
  <td><img src="https://argo.wlstmd.cloud/api/badge?name=vault&revision=false&showAppName=true" width="200px" /><br /></td>
  
  
  
  <td><img src="https://argo.wlstmd.cloud/api/badge?name=vault-ingress&revision=false&showAppName=true" width="200px" /><br /></td>
  
  
  
  <td><img src="https://argo.wlstmd.cloud/api/badge?name=velero&revision=false&showAppName=true" width="200px" /><br /></td>
  </tr>
  
  <tr>
  <td><img src="https://argo.wlstmd.cloud/api/badge?name=visualized&revision=false&showAppName=true" width="200px" /><br /></td>
  
  
</table>

<!-- SYNC_TABLE_END -->

## Rights

&copy; 2026. wlstmd. MIT Licensed.
