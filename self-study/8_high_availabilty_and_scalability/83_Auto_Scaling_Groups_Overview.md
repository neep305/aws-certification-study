# Auto Scaling Groups (ASG) - 완벽 가이드

## 개요
- **정의**: Auto Scaling Groups(ASG)는 사용자가 정의한 정책에 따라 EC2 인스턴스를 자동으로 추가/제거하는 AWS 기능이다.
- **목적**: 트래픽 변동에 맞춰 비용을 최적화하면서 애플리케이션의 가용성을 유지.
- **적용 범위**: EC2, ECS, RDS, DynamoDB 등 여러 서비스와 통합 가능.

## 핵심 개념

### ASG의 작동 원리
```
1. 정책 정의: 최소/최대/원하는 용량 설정
2. 트래픽 증가: CPU 사용률 상승 감지
3. Scale-out: 인스턴스 자동 추가
4. 트래픽 감소: CPU 사용률 하강 감지
5. Scale-in: 인스턴스 자동 제거
6. 비용 최적화: 필요한만큼만 실행
```

### ASG의 3가지 용량 설정
```
최소 용량 (Min Capacity): 
- 항상 실행되는 인스턴스 수 (최소한)
- 예: 2 (장애 발생 시 최소 2개 유지)

최대 용량 (Max Capacity):
- 최대로 실행할 수 있는 인스턴스 수
- 예: 6 (비용 상한선)

원하는 용량 (Desired Capacity):
- 현재 목표하는 인스턴스 수
- 범위: Min ≤ Desired ≤ Max
- 예: 4 (현재 필요한 수)
```

---

## ASG 구성 요소

### 1. Launch Template (권장) 또는 Launch Configuration

| 특성 | Launch Template | Launch Configuration |
|------|-----------------|----------------------|
| 현재 상태 | 최신 표준 (권장) | 레거시 (지원 중단 예정) |
| 버전 관리 | 여러 버전 지원 | 단일 버전만 가능 |
| 유연성 | 높음 (여러 옵션) | 낮음 |
| 다른 서비스와 호환 | EC2, ECS 등 | EC2만 |

### 2. Scaling Policy (확장/축소 정책)

**Target Tracking Scaling (권장):**
- CPU 사용률 70% 유지
- 목표 값을 정하면 ASG가 자동 조정
- 간단하고 효율적

**Step Scaling:**
- CPU 70~80%: 1개 추가
- CPU 80~90%: 2개 추가
- 더 세밀한 제어 가능

**Simple Scaling:**
- 임계값 초과 시 정해진 수만큼 추가
- 가장 기본적 (권장하지 않음)

### 3. Health Check

```
ELB Health Check (권장):
- 로드밸런서가 헬스 체크 수행
- 응답 없는 인스턴스 자동 제거 후 교체

EC2 Health Check:
- EC2 메타데이터 기반 (간단)
- 실제 애플리케이션 상태 감시 X

Custom Health Check:
- 사용자 정의 로직으로 헬스 체크
- 예: 데이터베이스 연결 확인
```

---

## ASG 설정 방법

### AWS 콘솔에서 ASG 생성

```
1. EC2 → Auto Scaling Groups → Create
2. Launch Template 선택 (또는 생성)
3. 용량 설정:
   - Min: 2
   - Max: 6
   - Desired: 4
4. 로드밸런서 연결 (선택)
5. Scaling Policy 설정:
   - Target Tracking CPU 70%
6. 태그 설정 (인스턴스에 자동 적용)
7. Create
```

### AWS CLI 명령어

**Launch Template 생성:**
```bash
aws ec2 create-launch-template \
  --launch-template-name my-template \
  --launch-template-data '{
    "ImageId":"ami-xxxxx",
    "InstanceType":"t2.micro",
    "KeyName":"my-key",
    "SecurityGroupIds":["sg-xxxxx"]
  }'
```

**ASG 생성:**
```bash
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --launch-template LaunchTemplateName=my-template,Version='$Latest' \
  --min-size 2 \
  --max-size 6 \
  --desired-capacity 4 \
  --availability-zones us-east-1a us-east-1b us-east-1c
```

**Target Tracking Scaling Policy:**
```bash
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name cpu-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "TargetValue":70.0,
    "PredefinedMetricSpecification":{
      "PredefinedMetricType":"ASGAverageCPUUtilization"
    },
    "ScaleOutCooldown":60,
    "ScaleInCooldown":300
  }'
```

**ASG 상태 확인:**
```bash
aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names my-asg
```

### CloudFormation에서 ASG 설정

```yaml
Resources:
  MyASG:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      AutoScalingGroupName: my-asg
      LaunchTemplate:
        LaunchTemplateId: !Ref LaunchTemplate
        Version: !GetAtt LaunchTemplate.LatestVersionNumber
      MinSize: 2
      MaxSize: 6
      DesiredCapacity: 4
      AvailabilityZones:
        - us-east-1a
        - us-east-1b
        - us-east-1c
      HealthCheckType: ELB
      HealthCheckGracePeriod: 300

  TargetTrackingScaling:
    Type: AWS::AutoScaling::ScalingPolicy
    Properties:
      AutoScalingGroupName: !Ref MyASG
      PolicyType: TargetTrackingScaling
      TargetTrackingConfiguration:
        TargetValue: 70.0
        PredefinedMetricSpecification:
          PredefinedMetricType: ASGAverageCPUUtilization
```

---

## ASG의 배포 전략

### 1. 롤링 배포 (Rolling Deployment)
```
목표: 무중단 배포, 천천히 진행

프로세스:
1. 새 AMI 사용 (또는 LT 버전 업데이트)
2. 기존 인스턴스 1개씩 제거
3. 새 인스턴스 1개씩 추가
4. Draining 시간 동안 기존 요청 완료

장점:
- 무중단 배포
- 문제 발생 시 쉬운 롤백

단점:
- 배포 시간 오래 걸림
- 새/구 버전 동시 실행
```

### 2. Blue-Green 배포
```
목표: 빠르고 안전한 배포

프로세스:
1. 새 ASG 생성 (Green)
2. 로드밸런서를 Green으로 전환
3. 기존 ASG (Blue) 제거

장점:
- 빠른 배포
- 롤백 가능 (Blue 유지)
- 0 다운타임

단점:
- 인스턴스 2배 리소스 필요
- 수동 개입 필요
```

### 3. Canary 배포
```
목표: 안전한 점진적 배포

프로세스:
1. 신버전으로 1~5% 트래픽 라우팅
2. 메트릭 모니터링
3. 안전하면 100% 전환

도구:
- ALB weighted target groups
- AWS CodeDeploy
```

---

## ASG 라이프사이클

### 인스턴스 실행 순서
```
1. 라이프사이클 훅 (Launch)
   → Lambda/SNS로 사용자 정의 작업 수행
   
2. EC2 인스턴스 시작
   → User Data 스크립트 실행
   
3. 헬스 체크 (Grace Period 동안 대기)
   → 기본: 300초
   → 커스텀: 필요 시 연장
   
4. ASG에 등록
   → 로드밸런서에 추가
   → 트래픽 수신 시작
```

### 인스턴스 종료 순서
```
1. Scaling Policy 트리거
   → Scale-in 필요
   
2. Deregistration Delay 시작
   → 새 요청 거부 (Connection Draining)
   → 기존 요청 완료 대기 (기본 300초)
   
3. 라이프사이클 훅 (Terminate)
   → 로그 저장, 상태 정리 등
   
4. EC2 인스턴스 종료
```

---

## SAA 시험 대비: 핵심 포인트

### 자주 출제되는 질문
1. **"배포 중 기존 요청 안전하게?"** → Connection Draining + Lifecycle Hook
2. **"Min 2, Max 8, Desired 4. CPU 70% Threshold. 어떻게?"** → Desired 증가
3. **"여러 AZ에 분산?"** → VPCZoneIdentifier에 여러 Subnet
4. **"Scaling Policy 타입?"** → Target Tracking > Step > Simple

### 시험 대비 체크리스트
- [ ] Min ≤ Desired ≤ Max 관계 이해
- [ ] Launch Template > Launch Configuration (권장)
- [ ] Target Tracking Scaling (가장 권장)
- [ ] Health Check Grace Period (기본 300초)
- [ ] Connection Draining과 함께 무중단 배포

---

## 예제 문제 (3개)

### 문제 1
**상황:** CPU 70% Target Tracking. Min 2, Max 10, Desired 4. CPU 85%.

**Desired는?**
정답: Target Tracking은 천천히 조정 → 5~6개로 점진적 증가

### 문제 2
**상황:** 롤링 배포. Min 2, Max 6, Desired 4. 배포 시간?

**최소 시간:** 
Connection Draining 300초 × 4개 = 1200초 이상

### 문제 3
**상황:** Blue-Green 배포. 현재 5개.

**필요 인스턴스:** 10개 (Blue 5 + Green 5)

---

## CLI 명령어 모음

```bash
# Desired capacity 변경
aws autoscaling set-desired-capacity \
  --auto-scaling-group-name my-asg \
  --desired-capacity 5

# Scaling Policy 추가
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name my-policy \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{...}'

# ASG 일시 중단
aws autoscaling suspend-processes \
  --auto-scaling-group-name my-asg \
  --scaling-processes "Launch" "Terminate"

# ASG 재개
aws autoscaling resume-processes \
  --auto-scaling-group-name my-asg

# ASG 삭제
aws autoscaling delete-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --force-delete
```

---

## 핵심 요약

- **3가지 용량:** Min ≤ Desired ≤ Max
- **구성 요소:** Launch Template + Scaling Policy + Health Check
- **정책 순서:** Target Tracking > Step > Simple (권장도)
- **배포:** Rolling (점진적) / Blue-Green (빠름) / Canary (안전)
- **Draining:** 300초 (기본)

**SAA 시험:** 개요, 배포 전략, 예제 문제 정독 필수!
