# Oracle K8s Argo CD HomeLab

OCI Always Free (Ampere A1) 위에 올린 단일 노드 k3s 클러스터. GitOps 매니페스트는 이 레포(`apps/`, `root/`, `argocd/` 등)에서 관리.

## 서버 정보

| 항목 | 값 |
|---|---|
| 인스턴스 이름 | `lab-node` |
| 리전 | Japan East (Tokyo), `ap-tokyo-1` |
| Shape | VM.Standard.A1.Flex (Always Free) |
| 스펙 | 2 OCPU / 12GB RAM |
| OS | Ubuntu 24.04 LTS (aarch64) |
| Public IP | `[REDACTED_IP]` |
| k8s | k3s v1.37.2+k3s1 (단일 노드, control-plane) |

## SSH 접속

```bash
ssh -i ~/.ssh/lab-node ubuntu@[REDACTED_IP]
```

- Private key: `~/.ssh/lab-node`
- Public key: `~/.ssh/lab-node.pub`

## kubectl 접속 (로컬에서 바로)

```bash
export KUBECONFIG=~/.kube/lab-node.yaml
kubectl get nodes
```

- kubeconfig 파일: `~/.kube/lab-node.yaml`
- 서버의 `/etc/rancher/k3s/k3s.yaml`을 복사해온 뒤 `server` 주소를 `[REDACTED_IP]` → Public IP로 치환한 것
- k3s 인증서에 Public IP가 SAN으로 등록돼 있어서 (`/etc/rancher/k3s/config.yaml`에 `tls-san` 설정) 외부에서 TLS 검증 통과함

### kubeconfig를 다시 받아야 할 때

서버를 재설치하거나 k3s를 재설치하면 인증서가 바뀌므로 다시 받아야 함:

```bash
ssh -i ~/.ssh/lab-node ubuntu@[REDACTED_IP] "sudo cat /etc/rancher/k3s/k3s.yaml" > /tmp/k3s_raw.yaml
sed 's/[REDACTED_IP]/[REDACTED_IP]/' /tmp/k3s_raw.yaml > ~/.kube/lab-node.yaml
chmod 600 ~/.kube/lab-node.yaml
```

## ArgoCD

클러스터에 ArgoCD 설치 완료, 이 레포(`apps/*.yml`)의 Application들도 전부 등록된 상태.

```bash
export KUBECONFIG=~/.kube/lab-node.yaml
kubectl -n argocd get applications.argoproj.io
```

### UI 접속

**`https://argo.wlstmd.cloud`** — 동작 확인 완료 (DNS + Let's Encrypt 인증서 + 접속까지 테스트함)

- ID: `admin`
- 초기 비밀번호 확인:
  ```bash
  kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d
  ```
- 로그인 후 `argocd account update-password`로 바꿔두는 거 권장 (초기 비밀번호 그대로 두지 말 것)

### 현재 상태 / 막혀있는 것

`apps/`에 정의된 Application들(`root`, `web`, `spotify`, `terminal`, `linux`, `tree`, `wave`, `istio-ingress`, `argocd` 자신)은 전부 **매니페스트가 Istio(`networking.istio.io`)와 cert-manager CRD를 전제로 작성돼 있는데 아직 설치 안 함** → `OutOfSync` / `Missing` 상태로 멈춰있음. 이건 버그가 아니라 의도적으로 범위를 ArgoCD 설치까지만 잡은 것.

```
인터넷 → OCI 보안목록(80/443) → 서버 iptables → Traefik(k3s 기본, 80/443 소유)
       → IngressRoute/IngressRouteTCP(istio-ingress/traefik-bridge.yml)
       → Istio ingress gateway(ClusterIP) → Gateway/VirtualService → 각 서비스
```

Traefik는 그대로 두고(비활성화 안 함), Traefik가 받은 트래픽을 Istio ingress gateway로 넘겨주는 구조.
- HTTP(80): cert-manager의 Let's Encrypt HTTP-01 챌린지는 Traefik가 직접 처리, 나머지는 Istio로
- HTTPS(443): TLS passthrough — 인증서는 Istio gateway가 직접 종료 (Traefik는 그냥 통과만 시킴)

### DNS 설정 (가비아)

**주의: `@`(루트)나 `*`(와일드카드) A 레코드는 넣지 말 것.** `wlstmd.cloud`, `web`, `terminal`, `linux`, `tree`는 이미 GitHub Pages(`185.199.x.153`)로 가 있어서, 거기에 다른 걸 올리면 기존 사이트가 깨짐. 이 서버로 옮길 서브도메인만 **개별 A 레코드**로 추가:

| 호스트 | 타입 | 값 | 상태 |
|---|---|---|---|
| `argo` | A | `[REDACTED_IP]` | ✅ 완료, 인증서 발급됨 |
| `spotify` | A | `[REDACTED_IP]` | DNS 레코드 자체가 없음 — 필요할 때 추가 |
| `wave` | A | `[REDACTED_IP]` | DNS 레코드 자체가 없음 — 필요할 때 추가 |
| `wlstmd.cloud`(루트), `web`, `terminal`, `linux`, `tree` | - | - | 현재 GitHub Pages 사용 중 — **건드리지 말 것** (이 서버로 옮기려면 먼저 상의) |

새 서브도메인을 이 서버로 추가할 때마다:
1. 가비아에 A 레코드 추가 (`<서브도메인>` → `[REDACTED_IP]`)
2. `istio-ingress/certificate.yml`의 `dnsNames`에 해당 도메인 추가, 커밋 & 푸시
3. ArgoCD가 자동 반영 → cert-manager가 인증서 재발급 (기존 `argo.wlstmd.cloud`는 그대로 유지됨)

전파 확인:
```bash
dig +short spotify.wlstmd.cloud   # [REDACTED_IP] 뜨는지 확인
kubectl -n istio-ingress get certificate wlstmd-cloud-tls   # READY True 뜨면 완료
```

### 새 앱을 올릴 때 (예: spotify)

인프라는 전부 준비되어 있어서 **이미지만 빌드해서 레지스트리에 올리면 자동으로 뜸**:

1. `spotify/deployment.yml`에 이미지 경로 채워서 레지스트리에 푸시
2. 커밋 + 푸시만 하면 끝 — `syncPolicy.automated`가 켜져 있어서 ArgoCD가 3분 내로 알아서 감지하고 배포함
3. 수동으로 바로 반영하고 싶으면:
   ```bash
   kubectl -n argocd annotate application spotify argocd.argoproj.io/refresh=hard --overwrite
   ```

### 참고로 알아둘 것

- 이미지가 없는 앱들(`spotify`, `web`, `terminal`, `linux`, `tree`, `wave`, `root`)은 지금 전부 `ImagePullBackOff` 상태 — 정상, 이미지 올리면 바로 Running으로 바뀜
- Istio는 **ambient mode**로 설치 (사이드카 없음, `ztunnel` + `istio-cni`가 노드에서 트래픽 처리)
- Istio ingress gateway는 `ClusterIP`로 띄움 (Traefik가 이미 80/443을 소유하고 있어서 LoadBalancer로 또 열 필요 없음)
- `argocd-cmd-params-cm`에 `server.insecure: "true"` 설정함 — TLS는 Istio gateway가 이미 종단하는데 argocd-server가 또 https로 리다이렉트하면서 무한 리다이렉트 루프가 났던 걸 고친 것. 리버스 프록시 뒤에 두는 표준적인 설정
- 인증서(`wlstmd-cloud-tls`)는 지금 `argo.wlstmd.cloud` 하나만 포함 — wildcard 인증서는 HTTP-01로 발급 불가(DNS-01만 가능한데 가비아는 자동 연동 안 됨)라서 서브도메인 추가할 때마다 `certificate.yml`에 하나씩 추가해야 함

## 열려있는 포트

OCI는 **VCN 보안목록**과 **서버 내부 iptables** 둘 다 통과해야 접속이 됨 — 둘 다 아래 포트로 열어둔 상태.

| 포트 | 용도 |
|---|---|
| 22 | SSH |
| 80 | HTTP (Traefik Ingress) |
| 443 | HTTPS (Traefik Ingress) |
| 6443 | Kubernetes API |

- VCN: `lab-vpn` → `Default Security List for lab-vpn`
- LoadBalancer 타입 서비스는 k3s 기본 내장 ServiceLB(klipper-lb)가 노드 포트에 바로 바인딩함 — 별도 OCI Load Balancer 리소스 불필요

## OCI CLI / 인증

```bash
export PATH="$HOME/bin:$PATH"   # oci 바이너리 위치
oci compute instance list --compartment-id <tenancy-ocid> --display-name lab-node
```

- 설정 파일: `~/.oci/config`
- API 서명 키: `~/.oci/oci_api_key.pem` (콘솔 User Settings → API Keys에 공개키 등록됨)

## 참고

- 이 레포의 `istio-ingress/` 매니페스트들은 **Istio**를 전제로 작성돼 있는데, 지금 서버엔 k3s 기본 **Traefik**만 깔려있음 — Istio를 따로 설치하거나 매니페스트를 Traefik 리소스(IngressRoute 등)로 바꿔야 그대로 적용 가능
- 서버 생성/재시도에 쓴 스크립트: `~/.oci/lab-node-retry.sh` (현재는 성공해서 cron 제거된 상태, 코드는 그대로 남아있음)
