# ELB Cross-Zone Load Balancing

## 개요
- **정의**: Cross-zone load balancing은 로드밸런서가 여러 가용영역(AZ)에 걸쳐 있는 대상들로 트래픽을 인스턴스 단위로 균등 분배하는 기능이다.
- **목적**: AZ별 인스턴스 수가 다를 때에도 각 인스턴스에 고르게 트래픽을 할당하여 성능과 가용성 최적화.

## ELB 유형별 동작 요약
- **Classic Load Balancer (CLB)**: Cross-zone 설정을 수동으로 활성화/비활성화 가능. 비활성화 시 각 AZ로 들어온 트래픽은 그 AZ의 인스턴스에만 분배되어 AZ별 인스턴스 수 차이에 따라 부하 편중 발생.
- **Application Load Balancer (ALB)**: 기본적으로 인스턴스 단위 균등 분배를 수행(사용자가 별도 설정 불필요).
- **Network Load Balancer (NLB)**: 리전/시점별 기본값이 달라질 수 있으므로 문서 확인 필요. 최신 NLB는 cross-zone 설정을 활성화할 수 있음.

## 장점
- AZ 간 인스턴스 수 불균형 시에도 인스턴스 단위로 균등 분배되어 전반적 성능 안정화.
- 특정 AZ 장애 시 다른 AZ의 인스턴스가 더 많은 요청을 받아 가용성 보장.
- 스케일링 시 트래픽 분배가 더 예측 가능해짐.

## 단점 및 비용 고려사항
- **인터-AZ 데이터 전송 비용**: 로드밸런서가 요청을 다른 AZ의 인스턴스로 라우팅하면 AZ 간 데이터 전송이 발생할 수 있으며 비용이 발생할 수 있다. (요금은 리전·서비스별 상이)
- 요청 경로가 복잡해져 네트워크 디버깅이 다소 어려워질 수 있음.

## 운영 및 구성 포인트
- **헬스체크 연동**: 헬스체크 실패로 특정 AZ의 가용 인스턴스 수가 줄어들면 Cross-zone이 유리.
- **세션 유지(스티키 세션)**: 세션 지속성이 필요한 경우 로드밸런서 타입별 세션 유지 방식과 함께 동작을 확인해야 함.
- **설정 방법**: 콘솔/CLI/CloudFormation에서 해당 로드밸런서의 CrossZoneLoadBalancing 또는 동등한 속성으로 제어.
- **검증**: 부하 테스트를 통해 각 인스턴스의 요청 수 분포 확인, CloudWatch의 per-instance metrics(요청 수) 확인.

## 트러블슈팅 포인트
- 인스턴스별 요청 분포가 예상과 다를 경우 헬스체크 상태, 인스턴스 등록 상태, 로드밸런서 설정(Zone 설정 포함)을 먼저 확인.
- 비용 이슈가 의심되면 CloudWatch 및 VPC Flow Logs로 AZ 간 트래픽 패턴을 분석.

## SAA 시험 대비 핵심 포인트
- CLB는 cross-zone 설정 여부에 따라 동작이 달라짐, ALB는 기본적으로 인스턴스 단위 균등 분배.
- Cross-zone 활성화는 인터-AZ 트래픽(비용) 증가 가능성을 유발할 수 있음 — 답변에 비용 영향 언급 필요.
- 실무 판단: 가용성과 균등 분배가 우선이면 활성화, 비용 민감하면 트래픽·요금 영향 분석 후 결정.

## 예제 문제
- 문제: "AZ A에는 4대 인스턴스, B에는 1대가 있을 때 CLB에서 Cross-zone이 비활성화되어 있다면 예상되는 현상은?"
  - 정답 요약: A로 유입된 트래픽은 A의 4대에만 분배되어 A의 인스턴스에 과부하가 발생하고, B로 유입된 트래픽은 B의 1대만 처리하여 전체적으로 불균형이 발생한다.

---

## ELB 유형별 상세 비교표

| 특성 | Classic LB | ALB | NLB |
|------|-----------|-----|-----|
| Cross-Zone 기본값 | 비활성화 (활성화 가능) | 활성화 | 비활성화 (활성화 가능, 최근 변경) |
| 트래픽 분배 단위 | AZ 단위 (비활성화 시) → 인스턴스 단위 (활성화 시) | 인스턴스 단위 (항상) | 흐름 해시(포트 기반) |
| 설정 난이도 | 수동 설정 필요 | 설정 불필요 | 수동 설정 필요 |
| 주 사용 케이스 | 레거시 애플리케이션 | HTTP/HTTPS 기반 앱 | 고성능/극저지연 앱 |

---

## 상세 비용·장단점 분석

### Cross-Zone 활성화 시 비용 영향
```
비용 구조:
- ALB/CLB/NLB는 로드밸런서 시간 요금 + 데이터 처리 요금
- Cross-Zone 활성화 → 인터-AZ 트래픽 발생 가능
- 인터-AZ 데이터 전송 비용: 보통 리전 내 데이터 이동의 약 50% 수준
  (예: us-east-1에서 일반적으로 GB당 $0.01 ~ $0.02)

계산 예시:
- 월 1TB의 트래픽이 AZ 간 이동 시: 1000GB × $0.01 = $10/월
- 대규모 트래픽(100TB): 100,000GB × $0.01 = $1,000/월
```

### 언제 비용이 증가하는가?
- 로드밸런서가 매번 다른 AZ로 요청을 라우팅할 때
- 클라이언트의 지리적 분포가 특정 AZ에 편중되었을 때
- 인스턴스 수가 AZ별로 크게 불균형할 때

### 언제 비용 증가가 무시할 만한가?
- 트래픽이 적거나 균등하게 분산되어 있을 때
- 가용성이 매우 중요한 업무용 애플리케이션
- 비용보다 성능·신뢰성이 우선인 경우

---

## 구성 및 검증 방법

### AWS 콘솔에서 설정
```
Classic Load Balancer:
1. EC2 → Load Balancers → CLB 선택
2. Actions → Edit Attributes
3. "Enable Cross-Zone Load Balancing" 체크

Application Load Balancer:
1. EC2 → Load Balancers → ALB 선택
2. Actions → Edit Attributes
3. Cross-Zone Load Balancing이 기본 활성화됨 (필요 시 비활성화 가능)

Network Load Balancer:
1. EC2 → Load Balancers → NLB 선택
2. Actions → Edit Attributes
3. "Enable Cross-Zone Load Balancing" 체크 (리전별로 기본값 상이)
```

### AWS CLI를 통한 설정

**CLB의 Cross-Zone 활성화:**
```bash
aws elb modify-load-balancer-attributes \
  --load-balancer-name my-classic-lb \
  --load-balancer-attributes CrossZoneLoadBalancing={Enabled=true}
```

**CLB의 Cross-Zone 상태 확인:**
```bash
aws elb describe-load-balancer-attributes \
  --load-balancer-name my-classic-lb \
  --query 'LoadBalancerAttributes.CrossZoneLoadBalancing'
```

**ALB/NLB의 Cross-Zone 활성화:**
```bash
aws elbv2 modify-load-balancer-attributes \
  --load-balancer-arn arn:aws:elasticloadbalancing:region:account-id:loadbalancer/app/my-alb/... \
  --attributes Key=load_balancing.cross_zone.enabled,Value=true
```

### CloudFormation 설정

**CLB with Cross-Zone:**
```yaml
Resources:
  MyClassicLB:
    Type: AWS::ElasticLoadBalancing::LoadBalancer
    Properties:
      LoadBalancerName: my-classic-lb
      Listeners:
        - InstancePort: 80
          LoadBalancerPort: 80
          Protocol: HTTP
      AvailabilityZones:
        - us-east-1a
        - us-east-1b
      CrossZoneLoadBalancing: true
      HealthCheck:
        HealthyThreshold: 3
        Interval: 30
        Target: HTTP:80/
        Timeout: 5
        UnhealthyThreshold: 5
```

**ALB with Target Group (Cross-Zone 자동 활성화):**
```yaml
Resources:
  MyALB:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      LoadBalancerAttributes:
        - Key: load_balancing.cross_zone.enabled
          Value: 'true'
      Subnets:
        - subnet-xxxxx (us-east-1a)
        - subnet-yyyyy (us-east-1b)

  TargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      TargetGroupAttributes:
        - Key: deregistration_delay.timeout_seconds
          Value: 30
      VpcId: vpc-xxxxx
      Port: 80
      Protocol: HTTP
```

### 검증 방법: CloudWatch 메트릭 확인

**Per-Instance Request Count 확인:**
```bash
aws cloudwatch get-metric-statistics \
  --namespace AWS/ELB \
  --metric-name RequestCount \
  --dimensions Name=LoadBalancerName,Value=my-clb \
  --statistics Sum \
  --start-time 2025-01-06T00:00:00Z \
  --end-time 2025-01-06T23:59:59Z \
  --period 3600
```

**인스턴스별 처리 요청 수 추적 (활성 인스턴스 모니터링):**
```bash
# EC2 인스턴스별로 웹서버 로그 요청 수 확인
# (ALB/CLB 로그 분석으로도 가능)
```

### 간단한 부하 테스트 절차

```bash
# 1. 부하 생성 도구 설치 (예: Apache Bench)
# Windows: choco install ab
# macOS: brew install httpd
# Linux: apt-get install apache2-utils

# 2. 로드밸런서 DNS로 동시 요청 전송
ab -n 10000 -c 100 http://my-load-balancer-dns.com/

# 3. 결과 분석
# - 각 인스턴스의 로그 확인 (요청 수, 응답 시간)
# - CloudWatch 그래프로 분산 상태 시각화
```

---

## 트러블슈팅 가이드

### 상황 1: 특정 인스턴스에만 트래픽이 집중됨
**원인 분석:**
- Cross-Zone 설정이 제대로 활성화되지 않았나?
- 헬스체크 실패로 다른 AZ의 인스턴스가 제외되었나?
- 세션 유지(스티키) 설정으로 인한 것은 아닌가?

**확인 순서:**
```bash
# 1. 로드밸런서 Cross-Zone 상태 확인
aws elb describe-load-balancer-attributes --load-balancer-name my-clb

# 2. 등록된 인스턴스 및 헬스 상태 확인
aws elb describe-instance-health --load-balancer-name my-clb

# 3. 스티키 세션 설정 확인
aws elb describe-load-balancer-attributes --load-balancer-name my-clb \
  --query 'LoadBalancerAttributes.AppCookieStickinessPolicy'
```

### 상황 2: 인터-AZ 비용이 예상보다 높음
**분석 방법:**
- VPC Flow Logs로 인터-AZ 트래픽 패턴 분석
- CloudWatch 상세 로그 수집 및 분석
- 트래픽 분포가 균등한지 확인

**대응 방안:**
```
- Cross-Zone 비활성화 검토 (가용성 영향도 함께 평가)
- 각 AZ에 인스턴스 수를 균등하게 배치 (Cross-Zone 필요성 감소)
- NAT Gateway, VPC Endpoint 등으로 트래픽 경로 최적화
```

---

## SAA 시험 대비: 핵심 포인트 & 예제 문제

### 시험 출제 패턴
1. **"3개 AZ에 걸쳐 로드밸런서를 배치했을 때 Cross-Zone의 영향은?"** 유형
2. **"비용을 최소화하면서도 고가용성을 유지하려면?"** 유형
3. **"특정 로드밸런서 유형에서 기본 동작은?"** 유형
4. **"장애 시나리오에서 어떤 구성이 안전한가?"** 유형

### 출제 시 주의할 포인트
- **CLB vs ALB**: CLB는 설정 필수, ALB는 기본 활성화
- **비용**: 인터-AZ 전송 비용 언급 여부로 점수 차이
- **운영**: 헬스체크와 인스턴스 상태 함께 고려
- **실무성**: 비용과 가용성의 트레이드오프 이해

---

## 예제 문제 풀이

### 문제 1
**상황:** 회사에서 3개 AZ(a, b, c)에 로드밸런서를 배치했으며, AZ-a에는 8대, AZ-b에는 2대, AZ-c에는 2대의 EC2 인스턴스가 있다. 현재 CLB를 사용 중이고 Cross-Zone이 비활성화되어 있다.

**문제:** 이 구성에서 예상되는 문제는? (모두 선택)
- A) AZ-a의 인스턴스들이 AZ-b, c보다 상대적으로 낮은 부하를 받음
- B) AZ-b와 c의 인스턴스들이 과부하될 가능성이 높음
- C) 이 상황은 정상이며 로드밸런서가 자동으로 균등 분배함
- D) 인터-AZ 트래픽 비용이 발생함

**정답:** B
- CLB에서 Cross-Zone이 비활성화되면 각 AZ로 들어온 트래픽은 해당 AZ의 인스턴스에만 분배됨
- AZ-a는 8대가 있어 트래픽을 충분히 처리하지만, AZ-b와 c는 각각 2대씩만 있어 과부하 위험
- 해결: Cross-Zone을 활성화하거나 각 AZ의 인스턴스 수를 균등하게 조정

---

### 문제 2
**상황:** ALB를 사용하여 3개 AZ에 걸쳐 인스턴스를 배치했다. 트래픽은 비교적 균등하게 분산되고 있다.

**문제:** 이 경우 인터-AZ 데이터 전송 비용이 발생할 가능성은?

**정답:** "발생할 수 있음" - ALB는 기본적으로 인스턴스 단위로 균등 분배하므로 각 AZ의 인스턴스 수가 다르면 일부 요청이 다른 AZ로 라우팅되어 인터-AZ 비용 발생

---

### 문제 3
**상황:** 금융 서비스 회사에서 고가용성과 비용 효율성을 동시에 달성하고 싶다. 현재 NLB를 검토 중이다.

**문제:** NLB에서 Cross-Zone을 활성화할 때 가장 큰 고려사항은?

**정답:** 인터-AZ 데이터 전송 비용. NLB는 높은 처리량(throughput)을 위해 설계되었으므로 Cross-Zone 활성화 시 막대한 AZ 간 데이터 이동이 발생할 수 있으며, 비용 영향이 상당할 수 있다.

---

### 문제 4
**상황:** CloudFormation으로 ALB를 배포하는 중이다.

**문제:** ALB의 Cross-Zone 동작을 CloudFormation에서 명시적으로 제어하려면?

**정답:** LoadBalancerAttributes에서 `load_balancing.cross_zone.enabled` 키를 `true` 또는 `false`로 설정. ALB는 기본값이 true이지만, 명시적 설정으로 확실히 제어 가능.

---

### 문제 5
**상황:** 현재 CLB를 사용 중이며, 최근 한 AZ의 인스턴스에서 높은 CPU 사용률이 관찰되었다. 다른 AZ는 정상이다.

**문제:** 가장 먼저 확인해야 할 것은? (우선순위 높음)
- A) Cross-Zone 설정 상태
- B) 해당 AZ의 인스턴스 헬스 상태
- C) 로드밸런서의 요청 로그

**정답:** A → B 순서
1. CLB의 Cross-Zone이 비활성화되어 있으면 특정 AZ에만 트래픽이 집중될 수 있음
2. 헬스체크 실패로 다른 인스턴스가 제외되었을 가능성도 확인
3. 필요 시 로그 분석

---

## 빠른 참고: 의사결정 플로우

```
고가용성이 우선?
  ├─ YES → Cross-Zone 활성화 권장
  │        (비용은 감시하면서도 가용성 확보)
  └─ NO  → 비용 분석
           ├─ 트래픽 적음/AZ별 인스턴스 균등
           │  └─ Cross-Zone 비활성화 고려
           └─ 트래픽 많음/AZ별 불균형
              └─ 비용 vs 가용성 수량화 후 결정
```

---

## 추가 학습 자료
- [AWS ELB 공식 문서 - Cross-Zone Load Balancing](https://docs.aws.amazon.com/elasticloadbalancing/)
- [AWS 요금 - 데이터 전송 비용](https://aws.amazon.com/ec2/pricing/on-demand/#Data_Transfer)
- AWS SAA 시험 대비: 하루 전 위 3개 섹션(개요, 비교표, 트러블슈팅) 정독
