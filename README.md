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

아직 Istio Ingress가 없어서 외부 도메인(`argo.wlstmd.cloud`)으로는 못 들어가고, 포트포워드로 접속:

```bash
export KUBECONFIG=~/.kube/lab-node.yaml
kubectl -n argocd port-forward svc/argocd-server 8080:443
```

브라우저에서 `https://localhost:8080` (자체서명 인증서라 경고 뜸, 무시하고 진행)

- ID: `admin`
- 초기 비밀번호 확인:
  ```bash
  kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d
  ```
- 로그인 후 `argocd account update-password`로 바꿔두는 거 권장 (초기 비밀번호 그대로 두지 말 것)

### 현재 상태 / 막혀있는 것

`apps/`에 정의된 Application들(`root`, `web`, `spotify`, `terminal`, `linux`, `tree`, `wave`, `istio-ingress`, `argocd` 자신)은 전부 **매니페스트가 Istio(`networking.istio.io`)와 cert-manager CRD를 전제로 작성돼 있는데 아직 설치 안 함** → `OutOfSync` / `Missing` 상태로 멈춰있음. 이건 버그가 아니라 의도적으로 범위를 ArgoCD 설치까지만 잡은 것.

다음에 필요한 것 (아직 안 함):
1. Istio(ambient mode) + cert-manager 설치 → `networking.istio.io`, `cert-manager.io` CRD 생성
2. `spotify` 등 각 앱의 컨테이너 이미지 빌드 & 레지스트리 푸시 (지금은 이미지 자체가 없음)
3. 그 다음 ArgoCD가 자동으로 Sync해서 실제 배포됨 (`syncPolicy.automated` 이미 켜져있음, 수동 sync 불필요)

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
