# External Secrets Operator

AWS Secrets Manager의 시크릿을 Kubernetes Secret으로 동기화합니다.

## 사용 방법

### 1. ExternalSecret 생성

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: my-app-secrets
  namespace: <YOUR_NAMESPACE>
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: skkuding-aws-secrets-manager   # 또는 proseed-aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: my-app-secrets                  # 생성될 K8s Secret 이름
    creationPolicy: Owner
  data:
    - secretKey: DATABASE_URL             # K8s Secret의 키
      remoteRef:
        key: my-app/database              # AWS Secrets Manager의 시크릿 이름
        property: url                     # JSON 시크릿의 특정 필드 (선택)
```

### 2. 전체 JSON 시크릿 가져오기

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: my-app-secrets
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: skkuding-aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: my-app-secrets
  dataFrom:
    - extract:
        key: my-app/config                # 전체 JSON을 키-값으로 변환
```

### 3. ClusterSecretStore

| 이름 | AWS 계정 | 리전 |
|------|----------|------|
| `skkuding-aws-secrets-manager` | skkuding | ap-northeast-2 |
| `proseed-aws-secrets-manager` | proseed | ap-northeast-2 |

### 4. 상태 확인

```bash
# ExternalSecret 상태
kubectl get externalsecret -n <namespace>

# 동기화 상태 상세
kubectl describe externalsecret <name> -n <namespace>

# 생성된 Secret 확인
kubectl get secret <target-name> -n <namespace> -o yaml
```

### 5. 트러블슈팅

```bash
# ClusterSecretStore 상태 확인
kubectl get clustersecretstore

# External Secrets Operator 로그
kubectl logs -n external-secrets -l app.kubernetes.io/name=external-secrets
```

## AWS Credentials

AWS Secrets Manager 접근을 위해 `aws-credentials` namespace의 Secret을 Reflector로 복사해서 사용합니다.

- `skkuding-aws-credentials` → skkuding-aws-secrets-manager
- `proseed-aws-credentials` → proseed-aws-secrets-manager