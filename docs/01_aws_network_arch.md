# AWS 네트워크 아키텍처 설계 및 NAT 인스턴스 구축 보고서

## 1. 아키텍처 설계 의도
* VPC 구성: `10.0.0.0/16` 대역을 사용하여 확장성을 고려한 사설 네트워크망 구축
* Subnet 이중화: 가용성 확보를 위해 `ap-northeast-2a`, `ap-northeast-2c` 두 개의 가용 영역(AZ)에 Public/Private Subnet 분산 배치
* 보안: 외부에서 접근 불가능한 Private Subnet에 Worker Node를 배치하여 보안성 강화

## 2. NAT Gateway 대신 NAT Instance를 선택한 이유 (Cost Optimization)
본 프로젝트에서는 AWS Managed Service인 NAT Gateway 대신 EC2 기반의 NAT Instance를 직접 구축

### 비교 분석
* NAT Gateway: 시간당 비용($0.059) + 데이터 처리 비용 발생. 설정은 간편하나 트래픽이 적은 개발/테스트 환경에서는 고정 비용이 부담됨.
* NAT Instance (t2.micro): 프리티어 사용 가능 또는 매우 저렴한 비용으로 운영 가능.

### 구축 과정 (Technical Detail)
1. Source/Destination Check 비활성화: EC2가 자신이 목적지가 아닌 트래픽도 라우팅할 수 있도록 AWS 콘솔에서 설정 변경.
2. IP Forwarding 설정: 리눅스 커널 파라미터(`net.ipv4.ip_forward=1`) 수정.
3. iptables 마스커레이딩: 사설 IP 대역에서 나가는 패킷의 소스 IP를 NAT 인스턴스의 공인 IP로 변환하는 SNAT 규칙 적용.