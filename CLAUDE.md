# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

스꾸딩 Lab 클러스터(K3s) 인프라 관리 레포지토리. 개발/테스트 환경으로 ProSeed 등 실험적 프로젝트를 운영합니다.

## Architecture

```
k8s/
├── argocd/
│   ├── applications/
│   │   ├── infra/       # 인프라 앱 (argocd, cert-manager, external-secrets, reflector)
│   │   ├── observability/  # 모니터링 (prometheus, grafana, loki, tempo)
│   │   └── proseed/     # 프로젝트별 앱
│   └── values.yaml
├── aws-credentials/     # SealedSecret 소스 (Reflector로 배포)
├── cert-manager/        # Let's Encrypt DNS01 검증
├── external-secrets/    # AWS Secrets Manager 연동
├── reflector/           # Secret 복제 설정
└── observability/       # 모니터링 스택
```

## Key Patterns

### GitOps with ArgoCD
- 모든 앱은 `k8s/argocd/applications/`에 Application CR로 정의
- Helm 차트는 multi-source 패턴 사용 (chart + values ref)
- 자동 이미지 업데이트: `argocd-image-updater.argoproj.io` annotations

### Secret Management
- **SealedSecret**: 암호화된 시크릿을 Git에 저장
- **Reflector**: `aws-credentials` namespace에서 다른 namespace로 복제
- **External Secrets**: AWS Secrets Manager → K8s Secret 동기화

### AWS Credentials Flow
```
aws-credentials namespace (SealedSecret)
    │ Reflector (reflection-allowed: "true")
    ▼
cert-manager → Route53 DNS 검증 (*.skkuding.dev, *.proseednow.com)
external-secrets → AWS Secrets Manager 접근
```

## Common Commands

```bash
# SealedSecret 암호화
kubeseal --format yaml < secret.yaml > sealed-secret.yaml

# ArgoCD 앱 상태 확인
kubectl get applications -n argocd

# Image Updater 로그
kubectl logs -n argocd deployment/argocd-image-updater

# Secret 복제 상태
kubectl get secrets -A | grep -E "(skkuding|proseed)-aws-credentials"

# External Secrets 상태
kubectl get clustersecretstore
kubectl get externalsecret -A
```

## Testing Changes

ArgoCD auto-sync를 비활성화하고 수동 테스트:
```yaml
# applications/*.yaml
spec:
  syncPolicy: null  # auto-sync 비활성화
```

그 후 `kubectl apply -f`로 직접 적용.

## Conventions

- 프로젝트별 namespace 격리 (proseed → `proseed` namespace)
- Secret 키 이름: `aws_access_key_id`, `aws_secret_access_key`
- Helm values는 `k8s/<component>/values.yaml`에 저장