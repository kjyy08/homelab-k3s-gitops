# Homelab K3s GitOps Infrastructure

이 리포지토리는 홈랩 K3s 클러스터의 전체 인프라를 GitOps 방식으로 관리합니다.

## 접근 가능한 서비스
- **ArgoCD**: https://argocd.luigi99.cloud
- **Grafana**: https://grafana.luigi99.cloud  
- **n8n**: https://n8n.luigi99.cloud

## 아키텍처

### 프로젝트 구성
- **Amateurs**: AIBE 1기 최종 프로젝트 백엔드 (amateurs.co.kr)
- **Luigi Blog**: 개인 프로젝트 백엔드 (blog.luigi99.cloud) - 추가 예정

### 인프라 구성요소
- **Traefik**: Ingress 컨트롤러 및 자동 Let's Encrypt 인증서 관리
- **ArgoCD**: GitOps 배포
- **Prometheus**: 메트릭 수집
- **Grafana**: 모니터링 대시보드
- **Loki**: 로그 수집
- **Tempo**: 분산 추적
- **OpenTelemetry Collector**: 오픈 텔레메트리 데이터 수집
- **n8n**: 워크플로우 자동화

## 디렉토리 구조
```
homelab-k3s-gitops/
├── bootstrap/           # ArgoCD
├── core/               # 핵심 인프라 (Traefik, Storage)
├── observability/      # 모니터링 스택 (Prometheus, Grafana, Loki, Tempo)
├── automation/         # n8n 워크플로우 자동화
├── applications/       # ArgoCD 애플리케이션 정의
└── docs/              # 문서
```