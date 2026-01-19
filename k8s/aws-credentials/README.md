# AWS Credentials

AWS 자격증명을 중앙에서 관리하고 Reflector를 통해 필요한 namespace로 복사합니다.

## 구조

```
aws-credentials namespace (소스)
├── skkuding-aws-credentials    # skkuding AWS 계정
└── proseed-aws-credentials     # proseed AWS 계정

        │ Reflector
        ▼

cert-manager namespace ─────► Route53 DNS 검증
external-secrets namespace ──► AWS Secrets Manager 접근
```

## 사용 방법

### 1. 대상 namespace에 reflected Secret 생성

사용하려는 namespace에 빈 Secret을 만들고 `reflects` annotation을 추가합니다:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: skkuding-aws-credentials
  namespace: <YOUR_NAMESPACE>
  annotations:
    reflector.v1.k8s.emberstack.com/reflects: "aws-credentials/skkuding-aws-credentials"
type: Opaque
```

### 2. Secret 데이터 키

| 키 | 설명 |
|----|------|
| `aws_access_key_id` | AWS Access Key ID |
| `aws_secret_access_key` | AWS Secret Access Key |

### 3. 새 AWS 계정 추가

1. Secret 파일 생성:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: <NAME>-aws-credentials
  namespace: aws-credentials
  annotations:
    reflector.v1.k8s.emberstack.com/reflection-allowed: "true"
type: Opaque
stringData:
  aws_access_key_id: "<ACCESS_KEY>"
  aws_secret_access_key: "<SECRET_KEY>"
```

2. SealedSecret으로 암호화:

```bash
cat <NAME>-aws-credentials.yaml | kubeseal --format yaml > <NAME>-aws-credentials-sealed.yaml
mv <NAME>-aws-credentials-sealed.yaml <NAME>-aws-credentials.yaml
```

3. Git에 커밋