# Oracle K8s Argo CD HomeLab

Oracle Cloud Infrastructure Always Free 위에서 돌아가는 개인 Kubernetes 홈랩의 GitOps 매니페스트 저장소.

ArgoCD가 이 저장소를 계속 지켜보면서, 변경 사항이 생기면 자동으로 클러스터에 반영한다 (App of Apps 패턴).

## 구성 요소

- **k3s** — 단일 노드 Kubernetes
- **ArgoCD** — GitOps 배포
- **Istio (ambient mode)** — 서비스 메시 / 인그레스
- **cert-manager** — Let's Encrypt 인증서 자동 발급
- **Prometheus + Grafana** — 모니터링

## 구조

- `apps/` — ArgoCD Application 정의 (App of Apps)
- `argocd/`, `istio-ingress/`, `monitoring/` — 인프라 레이어
- 그 외 디렉터리 — 각 서비스별 매니페스트
