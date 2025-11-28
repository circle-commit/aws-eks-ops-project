# AWS EKS 기반 웹 서비스 구축 및 EFK/CloudWatch 관제 시스템

## 1. 프로젝트 개요
* 목표: AWS 클라우드 환경 내 고가용성 네트워크 설계 및 EKS(Kubernetes) 기반 웹 서비스 배포
* 핵심 내용: 운영 안정성 확보를 위한 EFK Stack(Logging) 및 CloudWatch/Lambda(Monitoring) 자동화 파이프라인 구축
* 역할: AWS 인프라 설계, EKS 클러스터 구축, 로그 수집기(Fluent Bit) 구성, 장애 알람 자동화 구현

## 2. 시스템 아키텍처
* Infrastructure: AWS (VPC, Public/Private Subnet, NAT Instance, Bastion Host)
* Compute: EKS (Amazon Linux 2, t3.medium/m7i-flex.large)
* Logging: Elasticsearch, Fluent Bit, Kibana (EFK Stack)
* Monitoring: CloudWatch, SNS, Lambda(Python), Slack

## 3. 주차별 핵심 수행 내용

### Week 1: 보안을 고려한 AWS 네트워크 환경 구성
* VPC 설계: `10.0.0.0/16` CIDR 기반의 독립된 네트워크 환경 및 Subnet 이중화 구성
* NAT Instance 구축: 비용 절감을 위해 NAT Gateway 대신 NAT Instance 직접 구축 및 `iptables` 마스커레이딩 설정
* 접근 제어 강화: Bastion Host를 통한 Private Subnet 접근 제어 및 IAM 계정 MFA 적용

### Week 2: EKS 클러스터 구축 및 웹 서비스 배포
* EKS 프로비저닝: `eksctl`을 활용한 Managed Node Group 및 클러스터 배포
* 워크로드 배포: Kubernetes `Deployment` 객체를 활용한 Nginx 웹 서버 구성 및 `ReplicaSet` 관리
* 외부 접속 구성: `LoadBalancer` 타입의 Service 생성을 통한 외부 트래픽 연결 및 통신 확인

### Week 3: EFK Stack 기반의 로그 모니터링 체계 구현
* 로그 파이프라인 구축: `Fluent Bit` -> `Elasticsearch` -> `Kibana` 로 이어지는 로그 수집/저장/시각화 아키텍처 구현
* Collector 최적화: EKS 1.24+ 버전 호환성 및 경량화를 위해 Fluentd 대신 **Fluent Bit**을 DaemonSet으로 배포
* 데이터 시각화: Kibana Index Pattern 설정 및 대시보드 구성을 통한 Pod별 실시간 로그 분석

### Week 4: 장애 대응을 위한 CloudWatch & Slack 알람 자동화
* 모니터링 설정: CPU 사용률 임계치(50%) 초과 시 즉시 반응하는 CloudWatch Alarm 트리거 설정
* 알람 자동화: SNS 토픽과 **AWS Lambda(Python)**를 연동하여 Slack 채널로 장애 알림 실시간 전송 구현
* 보안 강화: AWS KMS(Key Management Service)를 활용한 Slack Webhook URL 암호화 및 Lambda 내 복호화 로직 구현

## 4. 트러블 슈팅 (Troubleshooting)
* NAT 비용 이슈: 프리티어 활용 및 비용 절감을 위해 Managed NAT Gateway 대신 EC2 기반 NAT Instance 구축 및 라우팅 테이블 수정으로 해결
* 로그 수집기 호환성: EKS 버전 업그레이드(1.24+)로 인한 Dockershim 중단 이슈 확인 후, Containerd 런타임에 최적화된 Fluent Bit으로 마이그레이션 수행
* Webhook 보안: 소스 코드 내 Slack URL 평문 노출 방지를 위해 AWS KMS 비대칭 키 암호화 방식 적용