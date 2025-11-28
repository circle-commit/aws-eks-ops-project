# EFK Stack 구축 및 로그 수집기 마이그레이션 (Fluentd -> Fluent Bit)

## 1. 개요
Kubernetes 환경의 Pod 로그를 영구적으로 저장하고 분석하기 위해 EFK (Elasticsearch + Fluent Bit + Kibana) 스택을 구축

## 2. 기술적 의사결정: 왜 Fluent Bit인가?
초기에는 Fluentd를 고려했으나, 다음과 같은 이유로 Fluent Bit을 최종 채택

1. EKS 호환성 이슈 해결: AWS EKS 1.24 버전부터 컨테이너 런타임이 Docker에서 Containerd로 변경되면서, 기존 Dockershim 의존성이 있는 Fluentd 설정에 호환성 문제가 발생할 수 있음을 확인
2. 경량화 (Lightweight): Fluent Bit은 C언어로 작성되어 메모리 사용량이 매우 적음(약 650KB). 사이드카나 데몬셋으로 모든 노드에 배포해야 하는 로그 수집기의 특성상, 리소스 효율성이 더 뛰어난 Fluent Bit이 적합하다고 판단

## 3. 구축 상세
* DaemonSet 배포: 모든 워커 노드에서 로그를 수집하기 위해 `DaemonSet` 형태로 배포.
* ServiceAccount & RBAC: 클러스터 레벨의 로그 파일에 접근할 수 있도록 적절한 권한(ClusterRole) 부여.
* Elasticsearch 연동: 수집된 로그를 ES 인덱스로 전송하여 중앙 집중화된 로그 저장소 구축.