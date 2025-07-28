# Homelab K3s GitOps Infrastructure

이 리포지토리는 홈랩 K3s 클러스터의 전체 인프라를 GitOps 방식으로 관리합니다. Kustomize를 활용한 선언적 배포와 구조화된 애플리케이션 관리를 제공합니다.

## 🏗️ 아키텍처

### 클러스터 구성
- **K3s**: 경량화된 Kubernetes 배포
- **GitOps**: 선언적 인프라 관리
- **Kustomize**: YAML 리소스 조합 및 관리

### 네임스페이스 구성
- `observability`: 모니터링 및 로깅 스택
- `automation`: 워크플로우 자동화 도구
- `wiki`: 문서화 및 지식 관리
- `cert-manager`: TLS 인증서 자동 관리

## 📁 디렉토리 구조

```
homelab-k3s-gitops/
├── kustomization.yaml          # 루트 Kustomize 설정
├── README.md                   # 프로젝트 문서
├── docs/                       # 추가 문서
├── core/                       # 핵심 인프라 구성요소
│   ├── cert-manager/          # TLS 인증서 관리
│   │   ├── kustomization.yaml
│   │   └── cluster-issuer.yaml
│   ├── ingress/               # Ingress 라우팅 설정
│   │   ├── kustomization.yaml
│   │   ├── grafana-ingress.yaml
│   │   ├── n8n-ingress.yaml
│   │   ├── bookstack-ingress.yaml
│   │   └── argocd-ingress.yaml
│   └── storage/               # 스토리지 클래스 설정
│       ├── kustomization.yaml
│       ├── local-path-config.yaml
│       └── storage-classes.yaml
└── applications/              # 애플리케이션별 배포 설정
    ├── observability/         # 모니터링 스택
    │   ├── namespace.yaml
    │   ├── kustomization.yaml
    │   ├── grafana/          # 메트릭 시각화
    │   │   ├── grafana.yaml
    │   │   ├── grafana-config.yaml
    │   │   └── secret.yaml
    │   ├── prometheus/       # 메트릭 수집
    │   │   ├── prometheus.yaml
    │   │   └── prometheus-config.yaml
    │   ├── loki/            # 로그 수집 및 저장
    │   │   ├── loki.yaml
    │   │   └── loki-config.yaml
    │   ├── tempo/           # 분산 추적
    │   │   ├── tempo.yaml
    │   │   └── tempo-config.yaml
    │   ├── otel-collector/  # OpenTelemetry 수집기
    │   │   ├── otel-collector.yaml
    │   │   └── otlp-collector-config.yaml
    │   └── node-exporter/   # 노드 메트릭 수집
    │       └── node-exporter.yaml
    ├── automation/            # 워크플로우 자동화
    │   ├── namespace.yaml
    │   ├── kustomization.yaml
    │   └── n8n/              # 워크플로우 엔진
    │       ├── n8n.yaml
    │       ├── n8n-config.yaml
    │       └── n8n-secret.yaml
    └── wiki/                  # 지식 관리 시스템
        ├── namespace.yaml
        ├── kustomization.yaml
        ├── bookstack/        # 위키 플랫폼
        │   ├── bookstack.yaml
        │   └── bookstack-config.yaml
        └── mariadb/          # 데이터베이스
            └── mariadb.yaml
```

## 🚀 배포된 애플리케이션

### 🔍 관측성 스택 (observability)
- **Grafana**: 메트릭 및 로그 시각화 대시보드
- **Prometheus**: 시계열 메트릭 수집 및 저장
- **Loki**: 로그 수집, 저장 및 쿼리
- **Tempo**: 분산 추적 백엔드
- **OpenTelemetry Collector**: 관측성 데이터 수집 및 전송
- **Node Exporter**: 시스템 및 하드웨어 메트릭 수집

### 🤖 자동화 (automation)
- **n8n**: 시각적 워크플로우 자동화 플랫폼

### 📚 지식 관리 (wiki)
- **BookStack**: 위키 및 문서 관리 플랫폼
- **MariaDB**: BookStack용 데이터베이스

### 🔧 핵심 인프라 (core)
- **Cert-Manager**: Let's Encrypt TLS 인증서 자동 발급 및 갱신
- **Ingress Controllers**: HTTP/HTTPS 라우팅 및 SSL 종료
- **Storage Classes**: 영구 볼륨 스토리지 관리

## 🛠️ 설치 및 배포

### 전제조건
- K3s 클러스터가 실행 중이어야 함
- `kubectl` 명령어 도구 설치
- `kustomize` 도구 설치 (또는 kubectl에 내장된 kustomize 사용)
- 각 애플리케이션마다 secret 생성 필요

### 전체 스택 배포
```bash
# 리포지토리 클론
git clone <repository-url>
cd homelab-k3s-gitops

# 전체 인프라 배포
kubectl apply -k .
```

### 개별 구성요소 배포
```bash
# 핵심 인프라만 배포
kubectl apply -k core/

# 관측성 스택만 배포
kubectl apply -k applications/observability/

# 자동화 도구만 배포
kubectl apply -k applications/automation/

# 위키 시스템만 배포
kubectl apply -k applications/wiki/
```

## 📊 관리 및 운영

### 상태 확인
```bash
# 전체 파드 상태 확인
kubectl get pods --all-namespaces

# 특정 네임스페이스 확인
kubectl get pods -n observability
kubectl get pods -n automation
kubectl get pods -n wiki
```

### 로그 확인
```bash
# 특정 애플리케이션 로그 확인
kubectl logs -n observability deployment/grafana
kubectl logs -n automation deployment/n8n
kubectl logs -n wiki deployment/bookstack
```

### 접속 정보
애플리케이션별 Ingress 설정을 통해 외부에서 접근 가능합니다:
- Grafana: 모니터링 대시보드
- n8n: 워크플로우 관리
- BookStack: 위키 및 문서

## 🔄 GitOps 워크플로우

1. **코드 변경**: YAML 파일 수정 및 커밋
2. **자동 배포**: Git 변경사항을 클러스터에 동기화
3. **상태 관리**: 선언적 설정을 통한 일관된 상태 유지
4. **버전 관리**: Git을 통한 인프라 변경 이력 추적

## 🚨 주의사항

- 민감한 정보(비밀번호, API 키 등)는 Kubernetes Secret으로 관리
- 프로덕션 환경에서는 적절한 리소스 제한 설정 필요
- 정기적인 백업 및 복구 전략 수립 권장
- 보안 업데이트 및 취약점 모니터링 실시