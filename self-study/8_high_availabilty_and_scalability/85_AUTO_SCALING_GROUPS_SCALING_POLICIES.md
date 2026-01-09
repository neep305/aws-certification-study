# Auto Scaling Groups - Scaling Policies

## 1. Overview

**Scaling Policies** are the rules that determine when and how AWS Auto Scaling Groups (ASGs) automatically adjust the number of EC2 instances in response to demand. They enable dynamic capacity management - automatically adding instances when demand increases and removing them when demand decreases, optimizing both performance and costs.

**Key Purpose:**
- Automatically adjust capacity based on metrics (CPU, network traffic, custom metrics)
- Maintain performance during traffic spikes
- Reduce costs by removing unnecessary instances during low-traffic periods
- Eliminate manual intervention for capacity management
- Ensure consistent application performance

**Applicability:**
- Web applications with variable traffic patterns
- Batch processing systems
- Microservices architectures
- Any workload with unpredictable demand

---

## 2. Types of Scaling Policies

### 2.1 Target Tracking Scaling Policy (RECOMMENDED)

**Definition:**
AWS manages the scaling automatically to maintain a target metric value. You specify the desired metric value (e.g., 70% CPU), and ASG automatically scales up or down to maintain that target.

**Recommended Metrics:**
- `ASGAverageCPUUtilization` - CPU percentage (most common)
- `ASGAverageNetworkIn` - Incoming network traffic (bytes/sec)
- `ASGAverageNetworkOut` - Outgoing network traffic (bytes/sec)
- `ALBRequestCountPerTarget` - Requests per instance (for web apps)
- Custom metrics from CloudWatch

**Advantages:**
- **Simplest to use** - Only specify target metric value
- **AWS handles complexity** - Automatically adjusts scaling parameters
- **Predictable scaling** - Maintains consistent metric target
- **Recommended for most use cases** - Default choice for SAA exams
- **Faster scaling decisions** - Built-in optimization

**Scaling Behavior:**
```
Target Tracking Logic:
├─ Current Metric Value < Target?
│  └─ Scale UP (increase capacity)
├─ Current Metric Value > Target?
│  └─ Scale DOWN (decrease capacity)
└─ Current Metric Value = Target?
   └─ HOLD (no scaling)
```

**Example:**
If target CPU = 70% and current CPU = 85%, ASG increases instances. If CPU drops to 55%, ASG decreases instances (after cooldown period).

**Cooldown Period:**
- Default: **300** seconds (5 minutes)
- Prevents rapid scaling up and down
- SAA tip: Longer cooldown = less frequent scaling = better cost control

---

### 2.2 Step Scaling Policy (FINE-GRAINED CONTROL)

**Definition:**
Multiple scaling adjustments based on metric thresholds. Each step defines different adjustment amounts depending on how far the metric deviates from the target.

**Components:**
1. **Alarm Threshold** - The metric value that triggers the policy
2. **Step Adjustments** - Different actions based on metric ranges
3. **Cooldown Period** - Wait time between scaling actions

**Scaling Steps Example (CPU Utilization):**
```
CloudWatch Alarm: Average CPU > 70% (metric exceeds target)

Step 1: 70% < CPU ≤ 80%        → Add 1 instance
Step 2: 80% < CPU ≤ 90%        → Add 2 instances
Step 3: CPU > 90%              → Add 4 instances

Similarly for scale-down:
Step 1: 60% < CPU ≤ 70%        → Remove 1 instance
Step 2: 50% < CPU ≤ 60%        → Remove 2 instances
Step 3: CPU < 50%              → Remove 3 instances
```

**Advantages:**
- Fine-grained control over scaling behavior
- Different response intensity based on metric severity
- Useful for predictable patterns
- Can handle larger sudden spikes with bigger jumps

**Disadvantages:**
- More complex to configure
- Requires understanding of optimal step thresholds
- Manual tuning needed
- Requires creating CloudWatch alarms separately

**Use Cases:**
- Applications with known traffic patterns
- Fine-tuning capacity for specific metrics
- Complex scaling requirements

---

### 2.3 Simple Scaling Policy (BASIC)

**Definition:**
Single scaling adjustment triggered when a metric crosses a threshold. When alarm triggers, add/remove a fixed number of instances.

**Characteristics:**
- Single step: Metric crosses threshold → Fixed adjustment
- Basic approach: Increase/decrease by N instances
- Requires manual CloudWatch alarm creation
- Longest cooldown period (default 300s)

**Example:**
```
Scale-Up Alarm:
├─ Trigger: Average CPU > 80%
└─ Action: Add 2 instances

Scale-Down Alarm:
├─ Trigger: Average CPU < 20%
└─ Action: Remove 1 instance
```

**Advantages:**
- Simplest implementation for basic needs
- Suitable for predictable, gradual changes
- Minimal configuration

**Disadvantages:**
- No granularity for different severity levels
- Fixed response regardless of metric magnitude
- Slowest response to rapid changes
- Requires multiple separate alarms for scaling up/down

**When to Use:**
- Simple, stable workloads
- Gradual traffic changes
- Learning/testing environments

---

### 2.4 Comparison Matrix

| Feature | Target Tracking | Step Scaling | Simple Scaling |
|---------|-----------------|--------------|----------------|
| **Ease of Use** | ⭐⭐⭐⭐⭐ (Easiest) | ⭐⭐⭐ (Medium) | ⭐⭐⭐ (Medium) |
| **SAA Recommendation** | ⭐⭐⭐⭐⭐ (Best) | ⭐⭐⭐ (Good) | ⭐⭐ (Basic) |
| **AWS Preference** | Primary choice | Secondary choice | Legacy support |
| **Configuration Complexity** | Very Simple | Moderate | Simple |
| **Response Speed** | Fast (optimized) | Medium | Medium |
| **Scaling Granularity** | Single target | Multiple steps | Single action |
| **CloudWatch Alarms** | Auto-created | Manual creation | Manual creation |
| **Cooldown Period** | Built-in (default 300s) | Configurable | Configurable (default 300s) |
| **Common Use Case** | Web applications | Complex patterns | Stable workloads |
| **Exam Frequency** | ⭐⭐⭐⭐⭐ (Very High) | ⭐⭐⭐ (Medium) | ⭐⭐ (Low) |

---

### 2.5 Dynamic Scaling vs Scheduled Scaling (동적 스케일링 vs 예약된 스케일링)

**개념 이해:**

#### Dynamic Scaling (동적 스케일링) - 메트릭 기반

**정의:**
Target Tracking, Step Scaling, Simple Scaling 등 **실시간 메트릭을 기반으로** 자동 스케일링을 수행합니다.

**특징:**
```
메트릭 모니터링 → 임계값 초과 → CloudWatch Alarm 발동
  ↓
Scaling Policy 실행 → 인스턴스 추가/제거
```

**사용 시나리오:**
- 트래픽 변화가 예측 불가능한 경우
- 실시간 부하에 따라 즉시 대응 필요
- 일반적인 웹 애플리케이션
- 비용 최적화 필요

**장점:**
- 실제 부하에 따라 자동으로 대응
- 예기치 않은 트래픽 증가에 빠르게 반응
- 불필요한 인스턴스 자동 제거로 비용 절감
- 설정 후 자동으로 관리됨

**단점:**
- 트래픽 패턴 파악에 시간 필요 (cooldown period)
- 급격한 스파이크에 약간의 지연 (1-3분)
- Flapping 현상 가능성

#### Scheduled Scaling (예약된 스케일링) - 시간 기반

**정의:**
**미리 정해진 시간에** 용량(MinSize, MaxSize, DesiredCapacity)을 변경하는 스케일링입니다.

**특징:**
```
예약된 시간 도착 → 용량 설정값 변경
  ↓
ASG의 DesiredCapacity 자동 조정
```

**사용 시나리오:**
- 일일 근무 시간(9-6시) vs 야간 시간 패턴
- 요일별 트래픽 패턴 (예: 주중 vs 주말)
- 주간 예정된 이벤트 (세일, 런칭 등)
- 예측 가능한 트래픽 변화

**장점:**
- 트래픽 패턴이 예측 가능할 때 가장 효율적
- 메트릭 모니터링 없이 자동 조정
- 대량의 인스턴스 필요 시 미리 준비 가능
- 비용 예측 가능

**단점:**
- 트래픽 패턴 분석 및 설정 필요
- 예상 밖의 트래픽 변화에 대응 불가
- 각 시간대별로 별도 설정 필요
- 유지보수 부담

#### 비교 표

| 항목 | Dynamic Scaling | Scheduled Scaling |
|------|-----------------|-------------------| 
| **조정 기준** | 실시간 메트릭 | 사전 설정된 시간 |
| **메트릭 필요** | ✅ 필수 (CPU, 요청 수 등) | ❌ 불필요 |
| **반응 속도** | 1-3분 (메트릭 수집 후) | 즉시 (정확한 시간에) |
| **예측 가능성** | 예측 불가능한 부하 | 예측 가능한 패턴 |
| **트래픽 패턴** | 불규칙적 | 규칙적 (시간 기반) |
| **설정 복잡도** | 낮음 (메트릭, 타겟값만) | 높음 (시간대별 별도 설정) |
| **비용 효율성** | 좋음 (불필요한 리소스 자동 제거) | 우수 (시간대별 최적화) |
| **대응 가능성** | 급격한 변화 대응 가능 | 예측된 패턴만 대응 |
| **모니터링** | 지속적 모니터링 필요 | 모니터링 불필요 |
| **SAA 시험 빈도** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |

#### 실제 활용 사례

**Dynamic Scaling 사용:**
```
오전 9시: 트래픽 증가 감지 → 자동 Scale-Up
정오: 피크 트래픽 → 최대 용량 유지
오후 3시: 트래픽 감소 감지 → 자동 Scale-Down
저녁: 최소 용량 유지
```

**Scheduled Scaling 사용:**
```
매일 오전 8시: MinSize=2, MaxSize=20 설정 (근무 시간 대비)
매일 오후 6시: MinSize=1, MaxSize=5 설정 (야간 모드)
토요일: MinSize=1, MaxSize=3 설정 (주말 저부하)
이벤트 날짜: MinSize=5, MaxSize=50 설정 (대비)
```

#### 한글 핵심 정리

**Dynamic Scaling (동적/반응형):**
- 실시간 메트릭 모니터링
- 자동 대응
- 예측 불가능한 부하에 적합
- 일반적인 웹앱의 표준 선택

**Scheduled Scaling (예약형):**
- 사전 설정된 시간표
- 수동 설정 필요
- 예측된 패턴에 적합
- Dynamic Scaling과 함께 사용 권장

**Best Practice:**
```
권장 조합: Dynamic Scaling (기본) + Scheduled Scaling (최적화)

근무 시간 (9-6시):
├─ Scheduled Scaling: MinSize 2, MaxSize 20 설정 (근무 시간 대비)
└─ Dynamic Scaling: CPU 70% 기준 조정

야간 (6-9시):
├─ Scheduled Scaling: MinSize 1, MaxSize 5 설정  
└─ Dynamic Scaling: CPU 80% 기준 조정 (비용 절감)
```

---

## 3. Detailed Configuration

### 3.1 Target Tracking Configuration

#### AWS Management Console Steps:

1. **Navigate to Auto Scaling:**
   - Go to EC2 Dashboard
   - Click "Auto Scaling Groups"
   - Select your ASG
   - Go to "Automatic scaling" tab

2. **Create Scaling Policy:**
   - Click "Create dynamic scaling policy"
   - Select "Target tracking scaling policy"
   - Name: e.g., "cpu-target-tracking"

3. **Configure Target:**
   - Metric type: "Average CPU Utilization" (most common)
   - Target value: 70 (70%)
   - Leave other options as default
   - Click "Create"

4. **Verify:**
   - CloudWatch alarms created automatically (scale-up and scale-down)
   - Cooldown period visible in policy details

#### AWS CLI - Create Target Tracking Policy:

```bash
# Basic Target Tracking (CPU = 70%)
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name cpu-target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "TargetValue": 70.0,
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "ScaleOutCooldown": 60,
    "ScaleInCooldown": 300
  }'

# Target Tracking with ALB Request Count
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name alb-request-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "TargetValue": 1000.0,
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ALBRequestCountPerTarget",
      "ResourceLabel": "app/my-load-balancer/1234567890abcdef:targetgroup/my-targets/1234567890abcdef"
    }
  }'

# Target Tracking with Custom Metric
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name custom-metric-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "TargetValue": 50.0,
    "CustomizedMetricSpecification": {
      "MetricName": "MyCustomMetric",
      "Namespace": "MyCompany",
      "Statistic": "Average",
      "Unit": "Percent"
    }
  }'
```

#### AWS CLI - Retrieve Scaling Policies:

```bash
# List all scaling policies for ASG
aws autoscaling describe-scaling-policies \
  --auto-scaling-group-name my-asg

# Get specific policy details
aws autoscaling describe-scaling-policies \
  --policy-names cpu-target-tracking \
  --auto-scaling-group-name my-asg
```

---

### 3.2 Step Scaling Configuration

#### AWS Management Console Steps:

1. **Create CloudWatch Alarms First:**
   - Go to CloudWatch > Alarms
   - Create "Scale Up" alarm:
     - Metric: EC2 > Average CPU Utilization
     - Threshold: > 80%
     - Statistic: Average
     - Period: 300 seconds (5 minutes)
   - Create "Scale Down" alarm:
     - Same metric, Threshold: < 20%

2. **Create Step Scaling Policy:**
   - Go to Auto Scaling Groups
   - Select ASG, go to "Automatic scaling" tab
   - Click "Create dynamic scaling policy"
   - Select "Step scaling policy"
   - Name: "cpu-step-scaling"
   - CloudWatch alarm: Select scale-up alarm

3. **Configure Step Adjustments:**
   - Add step: 
     - Lower bound: 0 (metric > 80%)
     - Adjustment: +2 instances
   - Add another step (if metric very high):
     - Lower bound: 10 (metric > 90%)
     - Adjustment: +4 instances
   - Click "Create"

4. **Repeat for Scale-Down:**
   - Create another policy using scale-down alarm
   - Step: Remove 1-2 instances when CPU < 20%

#### AWS CLI - Create Step Scaling Policy:

```bash
# Step 1: Create CloudWatch Alarms
aws cloudwatch put-metric-alarm \
  --alarm-name my-asg-scale-up \
  --alarm-description "Alarm when CPU > 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --dimensions Name=AutoScalingGroupName,Value=my-asg

# Step 2: Create Step Scaling Policy
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name cpu-step-scaling \
  --policy-type StepScaling \
  --adjustment-type ChangeInCapacity \
  --metric-aggregation-type Average \
  --step-adjustments \
    MetricIntervalLowerBound=0,MetricIntervalUpperBound=10,ScalingAdjustment=2 \
    MetricIntervalLowerBound=10,ScalingAdjustment=4

# Step 3: Attach Alarm to Policy
POLICY_ARN=$(aws autoscaling describe-scaling-policies \
  --auto-scaling-group-name my-asg \
  --policy-names cpu-step-scaling \
  --query 'ScalingPolicies[0].PolicyARN' \
  --output text)

aws cloudwatch put-metric-alarm \
  --alarm-name my-asg-scale-up \
  --alarm-actions "$POLICY_ARN"
```

---

### 3.3 Simple Scaling Configuration

#### AWS CLI - Create Simple Scaling Policy:

```bash
# Step 1: Create Scale-Up Alarm
aws cloudwatch put-metric-alarm \
  --alarm-name my-asg-simple-scale-up \
  --alarm-description "Alarm when CPU > 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2

# Step 2: Create Scale-Up Policy
SCALE_UP_POLICY=$(aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name simple-scale-up \
  --adjustment-type ChangeInCapacity \
  --scaling-adjustment 2 \
  --cooldown 300 \
  --query 'PolicyARN' \
  --output text)

# Step 3: Attach Alarm to Policy
aws cloudwatch put-metric-alarm \
  --alarm-name my-asg-simple-scale-up \
  --alarm-actions "$SCALE_UP_POLICY"

# Step 4: Create Scale-Down Alarm and Policy
aws cloudwatch put-metric-alarm \
  --alarm-name my-asg-simple-scale-down \
  --alarm-description "Alarm when CPU < 20%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 20 \
  --comparison-operator LessThanThreshold \
  --evaluation-periods 2

SCALE_DOWN_POLICY=$(aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name simple-scale-down \
  --adjustment-type ChangeInCapacity \
  --scaling-adjustment -1 \
  --cooldown 300 \
  --query 'PolicyARN' \
  --output text)

aws cloudwatch put-metric-alarm \
  --alarm-name my-asg-simple-scale-down \
  --alarm-actions "$SCALE_DOWN_POLICY"
```

---

### 3.4 Scheduled Scaling Configuration (예약된 스케일링 설정)

#### AWS CLI - Scheduled Scaling 생성:

```bash
# 매일 오전 8시에 용량 확대 (근무 시간 대비)
aws autoscaling put-scheduled-action \
  --auto-scaling-group-name my-asg \
  --scheduled-action-name scale-up-morning \
  --recurrence "0 8 * * MON-FRI" \
  --min-size 2 \
  --max-size 20 \
  --desired-capacity 5

# 매일 오후 6시에 용량 축소 (야간 모드)
aws autoscaling put-scheduled-action \
  --auto-scaling-group-name my-asg \
  --scheduled-action-name scale-down-evening \
  --recurrence "0 18 * * *" \
  --min-size 1 \
  --max-size 5 \
  --desired-capacity 2

# 주말에 최소 용량 유지
aws autoscaling put-scheduled-action \
  --auto-scaling-group-name my-asg \
  --scheduled-action-name weekend-minimum \
  --recurrence "0 0 * * SAT" \
  --min-size 1 \
  --max-size 3 \
  --desired-capacity 1

# 특정 시간에 일회 스케일링 (이벤트 대비)
aws autoscaling put-scheduled-action \
  --auto-scaling-group-name my-asg \
  --scheduled-action-name event-prep \
  --start-time 2025-01-15T08:00:00Z \
  --end-time 2025-01-15T23:59:59Z \
  --min-size 5 \
  --max-size 50 \
  --desired-capacity 20

# 예약된 스케일링 조회
aws autoscaling describe-scheduled-actions \
  --auto-scaling-group-name my-asg

# 예약된 스케일링 삭제
aws autoscaling delete-scheduled-action \
  --auto-scaling-group-name my-asg \
  --scheduled-action-name scale-up-morning
```

**Cron 표현식 설명:**
```
분 시 일 월 요일
│  │  │  │  │
│  │  │  │  └──── 요일 (0=일, 1=월, ... 6=토)
│  │  │  └─────── 월 (1-12)
│  │  └────────── 일 (1-31)
│  └───────────── 시 (0-23, UTC 기준)
└──────────────── 분 (0-59)

예시:
├─ 매일 9시: "0 9 * * *"
├─ 평일 9시: "0 9 * * MON-FRI"
├─ 매주 월요일 9시: "0 9 * * 1"
├─ 1일, 15일 0시: "0 0 1,15 * *"
└─ 매월 말일 18시: "0 18 L * *" (일부 시스템)
```

#### AWS CLI - Scheduled Scaling과 Dynamic Scaling 조합:

```bash
# 근무 시간(9-18시): 안정적의 성능 우선 (CPU 70%)
aws autoscaling put-scheduled-action \
  --auto-scaling-group-name my-asg \
  --scheduled-action-name work-hours-setup \
  --recurrence "0 9 * * MON-FRI" \
  --min-size 2 \
  --max-size 20 \
  --desired-capacity 5

aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name work-hours-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "TargetValue": 70.0,
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "ScaleOutCooldown": 60,
    "ScaleInCooldown": 300
  }'

# 야간(18-9시): 비용 절감 우선 (CPU 80%)
aws autoscaling put-scheduled-action \
  --auto-scaling-group-name my-asg \
  --scheduled-action-name off-hours-setup \
  --recurrence "0 18 * * *" \
  --min-size 1 \
  --max-size 5 \
  --desired-capacity 2

aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name off-hours-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "TargetValue": 80.0,
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "ScaleOutCooldown": 300,
    "ScaleInCooldown": 600
  }'
```

#### AWS 콘솔 - Scheduled Scaling 설정:

1. **Auto Scaling Groups 이동**
   - EC2 Dashboard → Auto Scaling Groups
   - ASG 선택 → "Automatic scaling" 탭

2. **Scheduled actions 추가**
   - "Create scheduled action" 클릭
   - 이름: "morning-scale-up"
   - Recurrence: "0 9 * * MON-FRI" (평일 9시)
   - MinSize: 2, MaxSize: 20, DesiredCapacity: 5
   - "Create" 클릭

3. **여러 시간대 설정**
   - 위 과정 반복하여 각 시간대별 설정
   - 오전(9시), 정오(12시), 오후(18시) 등

---

### 3.5 CloudFormation Configuration

#### Complete Example with All Three Policy Types:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Auto Scaling with Multiple Scaling Policies'

Parameters:
  EnvironmentName:
    Type: String
    Default: production
    Description: Environment name

Resources:
  # Launch Template
  LaunchTemplate:
    Type: AWS::EC2::LaunchTemplate
    Properties:
      LaunchTemplateName: !Sub '${EnvironmentName}-template'
      LaunchTemplateData:
        ImageId: ami-0c55b159cbfafe1f0
        InstanceType: t3.medium
        SecurityGroupIds:
          - sg-12345678
        UserData:
          Fn::Base64: |
            #!/bin/bash
            yum update -y
            yum install -y httpd
            systemctl start httpd
            systemctl enable httpd
            echo "<h1>Hello from $(hostname -f)</h1>" > /var/www/html/index.html

  # Auto Scaling Group
  AutoScalingGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      AutoScalingGroupName: !Sub '${EnvironmentName}-asg'
      VPCZoneIdentifier:
        - subnet-12345678
        - subnet-87654321
      LaunchTemplate:
        LaunchTemplateId: !Ref LaunchTemplate
        Version: !GetAtt LaunchTemplate.LatestVersionNumber
      MinSize: 2
      MaxSize: 8
      DesiredCapacity: 4
      HealthCheckType: ELB
      HealthCheckGracePeriod: 300
      TargetGroupARNs:
        - !Ref LoadBalancerTargetGroup

  # ==========================================
  # POLICY TYPE 1: TARGET TRACKING (RECOMMENDED)
  # ==========================================
  TargetTrackingScalingPolicy:
    Type: AWS::AutoScaling::ScalingPolicy
    Properties:
      AutoScalingGroupName: !Ref AutoScalingGroup
      PolicyType: TargetTrackingScaling
      TargetTrackingConfiguration:
        TargetValue: 70.0
        PredefinedMetricSpecification:
          PredefinedMetricType: ASGAverageCPUUtilization
        ScaleOutCooldown: 60
        ScaleInCooldown: 300

  # Alternative: Target Tracking with ALB Metric
  TargetTrackingALBPolicy:
    Type: AWS::AutoScaling::ScalingPolicy
    Properties:
      AutoScalingGroupName: !Ref AutoScalingGroup
      PolicyType: TargetTrackingScaling
      TargetTrackingConfiguration:
        TargetValue: 1000.0
        PredefinedMetricSpecification:
          PredefinedMetricType: ALBRequestCountPerTarget
          ResourceLabel: !Join
            - '/'
            - - !GetAtt LoadBalancer.LoadBalancerFullName
              - !GetAtt LoadBalancerTargetGroup.TargetGroupFullName

  # ==========================================
  # POLICY TYPE 2: STEP SCALING (FINE-GRAINED)
  # ==========================================
  ScaleUpAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: !Sub '${EnvironmentName}-cpu-high'
      AlarmDescription: CPU utilization is above 80%
      MetricName: CPUUtilization
      Namespace: AWS/EC2
      Statistic: Average
      Period: 300
      EvaluationPeriods: 1
      Threshold: 80
      ComparisonOperator: GreaterThanThreshold
      Dimensions:
        - Name: AutoScalingGroupName
          Value: !Ref AutoScalingGroup
      AlarmActions:
        - !Ref StepScalingPolicyUp

  StepScalingPolicyUp:
    Type: AWS::AutoScaling::ScalingPolicy
    Properties:
      AutoScalingGroupName: !Ref AutoScalingGroup
      PolicyType: StepScaling
      AdjustmentType: ChangeInCapacity
      MetricAggregationType: Average
      EstimatedWarmupSeconds: 300
      StepAdjustments:
        - MetricIntervalLowerBound: 0
          MetricIntervalUpperBound: 10
          ScalingAdjustment: 2
        - MetricIntervalLowerBound: 10
          ScalingAdjustment: 4

  ScaleDownAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: !Sub '${EnvironmentName}-cpu-low'
      AlarmDescription: CPU utilization is below 20%
      MetricName: CPUUtilization
      Namespace: AWS/EC2
      Statistic: Average
      Period: 300
      EvaluationPeriods: 2
      Threshold: 20
      ComparisonOperator: LessThanThreshold
      Dimensions:
        - Name: AutoScalingGroupName
          Value: !Ref AutoScalingGroup
      AlarmActions:
        - !Ref StepScalingPolicyDown

  StepScalingPolicyDown:
    Type: AWS::AutoScaling::ScalingPolicy
    Properties:
      AutoScalingGroupName: !Ref AutoScalingGroup
      PolicyType: StepScaling
      AdjustmentType: ChangeInCapacity
      MetricAggregationType: Average
      EstimatedWarmupSeconds: 300
      StepAdjustments:
        - MetricIntervalUpperBound: 0
          MetricIntervalLowerBound: -10
          ScalingAdjustment: -1
        - MetricIntervalUpperBound: -10
          ScalingAdjustment: -2

  # ==========================================
  # POLICY TYPE 3: SIMPLE SCALING (BASIC)
  # ==========================================
  SimpleScaleUpAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: !Sub '${EnvironmentName}-simple-scale-up'
      AlarmDescription: Simple scaling - CPU high
      MetricName: CPUUtilization
      Namespace: AWS/EC2
      Statistic: Average
      Period: 300
      EvaluationPeriods: 2
      Threshold: 80
      ComparisonOperator: GreaterThanThreshold
      Dimensions:
        - Name: AutoScalingGroupName
          Value: !Ref AutoScalingGroup
      AlarmActions:
        - !Ref SimpleScalingPolicyUp

  SimpleScalingPolicyUp:
    Type: AWS::AutoScaling::ScalingPolicy
    Properties:
      AutoScalingGroupName: !Ref AutoScalingGroup
      PolicyType: SimpleScaling
      AdjustmentType: ChangeInCapacity
      Cooldown: 300
      ScalingAdjustment: 2

  SimpleScaleDownAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: !Sub '${EnvironmentName}-simple-scale-down'
      AlarmDescription: Simple scaling - CPU low
      MetricName: CPUUtilization
      Namespace: AWS/EC2
      Statistic: Average
      Period: 300
      EvaluationPeriods: 2
      Threshold: 20
      ComparisonOperator: LessThanThreshold
      Dimensions:
        - Name: AutoScalingGroupName
          Value: !Ref AutoScalingGroup
      AlarmActions:
        - !Ref SimpleScalingPolicyDown

  SimpleScalingPolicyDown:
    Type: AWS::AutoScaling::ScalingPolicy
    Properties:
      AutoScalingGroupName: !Ref AutoScalingGroup
      PolicyType: SimpleScaling
      AdjustmentType: ChangeInCapacity
      Cooldown: 300
      ScalingAdjustment: -1

  # Load Balancer and Target Group (for demonstration)
  LoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Name: !Sub '${EnvironmentName}-alb'
      Scheme: internet-facing
      Subnets:
        - subnet-12345678
        - subnet-87654321

  LoadBalancerTargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      Name: !Sub '${EnvironmentName}-tg'
      Port: 80
      Protocol: HTTP
      VpcId: vpc-12345678
      HealthCheckProtocol: HTTP
      HealthCheckPath: /
      HealthCheckIntervalSeconds: 30
      HealthCheckTimeoutSeconds: 5
      HealthyThresholdCount: 2
      UnhealthyThresholdCount: 3
      TargetType: instance

  LoadBalancerListener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      LoadBalancerArn: !GetAtt LoadBalancer.LoadBalancerArn
      Port: 80
      Protocol: HTTP
      DefaultActions:
        - Type: forward
          TargetGroupArn: !GetAtt LoadBalancerTargetGroup.TargetGroupArn

  # ==========================================
  # SCHEDULED SCALING (시간 기반 스케일링)
  # ==========================================
  MorningScaleUp:
    Type: AWS::AutoScaling::ScheduledAction
    Properties:
      AutoScalingGroupName: !Ref AutoScalingGroup
      ScheduledActionName: !Sub '${EnvironmentName}-morning-scale-up'
      Recurrence: "0 9 * * MON-FRI"
      MinSize: 2
      MaxSize: 20
      DesiredCapacity: 5

  EveningScaleDown:
    Type: AWS::AutoScaling::ScheduledAction
    Properties:
      AutoScalingGroupName: !Ref AutoScalingGroup
      ScheduledActionName: !Sub '${EnvironmentName}-evening-scale-down'
      Recurrence: "0 18 * * *"
      MinSize: 1
      MaxSize: 5
      DesiredCapacity: 2

  WeekendMinimum:
    Type: AWS::AutoScaling::ScheduledAction
    Properties:
      AutoScalingGroupName: !Ref AutoScalingGroup
      ScheduledActionName: !Sub '${EnvironmentName}-weekend-minimum'
      Recurrence: "0 0 * * SAT"
      MinSize: 1
      MaxSize: 3
      DesiredCapacity: 1

Outputs:
  AutoScalingGroupName:
    Value: !Ref AutoScalingGroup
    Description: Auto Scaling Group Name

  LoadBalancerDNS:
    Value: !GetAtt LoadBalancer.DNSName
    Description: Load Balancer DNS Name

  TargetTrackingPolicyName:
    Value: !Ref TargetTrackingScalingPolicy
    Description: Target Tracking Scaling Policy

  StepScalingPolicyUpName:
    Value: !Ref StepScalingPolicyUp
    Description: Step Scaling Policy (Up)

  SimpleScalingPolicyUpName:
    Value: !Ref SimpleScalingPolicyUp
    Description: Simple Scaling Policy (Up)
```

---

## 4. Advanced Configuration Options

### 4.1 Scaling Behavior Settings

```bash
# Modify scaling policy behavior
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name advanced-cpu-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "TargetValue": 70.0,
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "ScaleOutCooldown": 60,
    "ScaleInCooldown": 300,
    "DisableScaleInProtection": false
  }'
```

**Key Parameters (주요 파라미터):**
- `ScaleOutCooldown`: Scale-Up 전 대기 시간 (초, 기본값: 300초)
  - 빠른 대응: 60초 (트래픽 급증 시 빠르게 반응)
  - 안정성: 300초 이상 (Flapping 방지)
  
- `ScaleInCooldown`: Scale-Down 전 대기 시간 (초, 기본값: 300초)
  - 일반적으로 ScaleOutCooldown보다 길게 설정
  - 권장: 600초 이상 (인스턴스 안정화 시간 확보)
  
- `DisableScaleInProtection`: Scale-Down 방지 (true로 설정하면 축소 불가)
  - 크리티컬 인스턴스 보호 필요 시 사용

**실무 권장사항:**
```
근무 시간:
  ScaleOutCooldown: 60초 (빠른 대응)
  ScaleInCooldown: 300초 (안정성)

야간/저부하:
  ScaleOutCooldown: 300초 (급속 확장 억제)
  ScaleInCooldown: 600초 (비용 절감)
```

### 4.2 Custom Metrics (커스텀 메트릭)

```bash
# CloudWatch에 커스텀 메트릭 발행
aws cloudwatch put-metric-data \
  --namespace MyApplication \
  --metric-name ActiveConnections \
  --value 150 \
  --dimensions AutoScalingGroupName=my-asg

# 커스텀 메트릭 기반 Target Tracking
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name custom-metric-scaling \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "TargetValue": 100.0,
    "CustomizedMetricSpecification": {
      "MetricName": "ActiveConnections",
      "Namespace": "MyApplication",
      "Statistic": "Average",
      "Unit": "Count"
    },
    "ScaleOutCooldown": 60,
    "ScaleInCooldown": 300
  }'
```

### 4.3 Scaling Policy Lifecycle

```
Application Traffic ↑
        ↓
CloudWatch Metric Exceeds Threshold
        ↓
Alarm Triggered (after evaluation period)
        ↓
Scaling Policy Evaluated
        ↓
EC2 Instances Launched (takes 1-2 minutes)
        ↓
Instance Joins Target Group (health check grace period)
        ↓
Receives Traffic
        ↓
Cooldown Period Active (no new scaling)
        ↓
Metric Returns to Normal
        ↓
(Repeat process for scale-down)
```

### 4.4 Cooldown Period Deep Dive (쿨다운 기간 상세 이해)

#### 쿨다운(Cooldown Period)이란?

**간단한 정의:**
쿨다운 기간은 **스케일링 작업 후 다음 스케일링 작업이 일어나기 전에 기다려야 하는 시간**입니다.

**비유로 이해하기:**
```
음식점 예시:
├─ 손님 많음 (오후 12시)
│  └─ 셰프 2명 → 3명으로 확대 (스케일업)
│
├─ 새로운 셰프가 작업 시작 (약 5-10분 필요)
│  └─ 음식 준비에 시간 걸림 (쿨다운 중)
│
└─ 일정 시간 후 (쿨다운 종료)
   └─ 손님이 또 늘면? 그때 다시 셰프 추가 가능
      (쿨다운 중에는 추가 불가)
```

#### 왜 쿨다운이 필요한가?

**문제 상황: 쿨다운이 없다면**
```
시간     CPU    상태
────────────────────────────
12:00   85%    Scale-Up 시작 (1개 인스턴스 추가)
12:01   82%    인스턴스 추가 중 (아직 온라인 안 됨)
12:02   78%    새 인스턴스 시작... CPU 떨어짐
12:02:30 75%  어? CPU 떨어졌으니 Scale-Down!
        → 방금 추가한 인스턴스 제거
12:03   70%    CPU 정상... 아 그런데 또 올라감
12:03:30 82%  또 Scale-Up? 또 Scale-Down? 반복!
        
결과: 계속 인스턴스 추가/제거 반복 (Flapping!)
      → AWS 비용 낭비, 서비스 불안정
```

**쿨다운의 역할:**
```
시간     CPU    상태
────────────────────────────
12:00   85%    Scale-Up 시작 (1개 인스턴스 추가)
12:01   82%    새 인스턴스 추가 중
12:02   78%    새 인스턴스 안정화 중
12:03   75%    CPU 떨어졌지만... 쿨다운 중 (대기)
        → Scale-Down 명령 무시됨!
12:04   72%    계속 쿨다운 중...
12:05   70%    쿨다운 종료! 이제 다시 조정 가능
        
결과: 안정적인 스케일링 → 비용 절감, 서비스 안정
```

#### ScaleOutCooldown vs ScaleInCooldown

**ScaleOutCooldown (Scale-Up 후 대기 시간):**
```
설명: 인스턴스를 추가한 후, 다음 추가까지 기다려야 하는 시간

기본값: 300초 (5분)

필요한 이유:
├─ 새 인스턴스 시작 시간 (1-2분)
├─ 애플리케이션 워밍업 시간 (CPU, 메모리 안정화)
└─ 메트릭 수집 시간 (정확한 상태 파악)

권장값:
├─ 60초: 트래픽 급증 예상 (빠른 반응 필요)
├─ 300초: 일반적인 웹앱 (기본값)
└─ 600초: 느리게 증가하는 트래픽
```

**ScaleInCooldown (Scale-Down 후 대기 시간):**
```
설명: 인스턴스를 제거한 후, 다음 제거까지 기다려야 하는 시간

기본값: 300초 (5분)

필요한 이유:
├─ 남은 인스턴스들이 추가 부하 담당하도록 안정화
├─ Connection Draining 완료 대기
└─ 급격한 트래픽 변동 대응 준비

권장값:
├─ 300초: Scale-Out과 동일하게 설정 (기본)
├─ 600초: 중요한 서비스 (인스턴스 안정화 시간 확보) ✓
└─ 1200초: 매우 중요한 서비스 (충분한 안정화 시간)

⚠️ 주의:
ScaleInCooldown은 보통 ScaleOutCooldown보다 길게 설정!
└─ Scale-Up은 빨리, Scale-Down은 천천히 (안정성 우선)
```

#### 실제 사례로 이해하기

**사례 1: E-commerce 사이트 (예측 불가능한 트래픽)**

```
상황: 갑자기 유명 연예인이 SNS에서 우리 사이트를 언급
     → 트래픽이 10배 증가!

설정:
ScaleOutCooldown: 60초 (빠른 대응)
ScaleInCooldown: 600초 (보수적)

시간표:
12:00:00 트래픽 10배 증가 감지 (CPU 90%)
12:01:00 인스턴스 5개 추가 (ScaleOut cooldown 끝)
12:02:00 인스턴스 5개 더 추가 (cooldown 다시 시작)
12:03:00 인스턴스 5개 더 추가 (계속...)
         → 트래픽 폭증에 빠르게 대응!

12:10:00 트래픽 정상화 (CPU 50%)
12:11:00 인스턴스 1개 제거 시작
12:20:00 ScaleIn cooldown 끝 (600초)
12:21:00 인스턴스 1개 더 제거 (천천히, 안정적)
         → 비용 절감하면서도 안정성 유지
```

**사례 2: 사무실 업무용 내부 서비스 (예측 가능한 트래픽)**

```
상황: 매일 오전 9시 직원들이 시스템 사용
     → 트래픽 패턴이 예측 가능

설정 A (나쁜 예):
ScaleOutCooldown: 60초
ScaleInCooldown: 60초

결과:
09:00 트래픽 증가 → 1개 인스턴스 추가
09:01 조금 더 증가 → 1개 인스턴스 추가
09:02 또 증가 → 1개 인스턴스 추가
...계속 추가됨
09:15 피크 도달 (5개 인스턴스)
09:16 약간 떨어짐 → 1개 인스턴스 제거
09:17 다시 올라감 → 1개 인스턴스 추가
09:18 또 떨어짐 → 1개 인스턴스 제거
...flapping 발생! (비용 낭비)

설정 B (좋은 예):
ScaleOutCooldown: 300초
ScaleInCooldown: 600초

결과:
09:00 트래픽 증가 → 1개 인스턴스 추가
09:05 계속 증가하나? → 1개 인스턴스 추가 (cooldown 후)
09:10 계속 필요한가? → 1개 인스턴스 추가 (cooldown 후)
09:15 피크 도달 (3개 인스턴스)
09:20 트래픽 떨어짐... 하지만 cooldown 중 (대기)
09:25 cooldown 끝, 계속 낮나? → 아직도 높음 (유지)
09:30 확실히 떨어짐 → 1개 인스턴스 제거
10:00 꾸준히 낮음 → 1개 인스턴스 제거 (cooldown 후)
...안정적인 스케일링
```

#### 한글 핵심 정리

**쿨다운의 목적:**
```
1️⃣ Flapping 방지 (무의미한 추가/제거 반복 방지)
2️⃣ 비용 절감 (불필요한 스케일링 작업 줄임)
3️⃣ 안정성 확보 (새 인스턴스 안정화 시간 보장)
4️⃣ 정확한 메트릭 수집 (순간적 변동에 반응하지 않음)
```

**ScaleOutCooldown (Scale-Up) 설정 가이드:**
```
상황: 트래픽이 예측 불가능하게 급증
설정: 짧게 (60~120초)
이유: 빠른 대응으로 서비스 품질 유지

상황: 트래픽 변화가 점진적
설정: 중간 (300초) ← 기본값
이유: 균형잡힌 대응

상황: 트래픽이 천천히 증가
설정: 길게 (600초)
이유: 불필요한 추가 방지
```

**ScaleInCooldown (Scale-Down) 설정 가이드:**
```
상황: 일반적인 웹서비스
설정: 600초 ← 권장
이유: 충분한 안정화 시간 확보

상황: 높은 가용성이 중요한 서비스
설정: 1200초
이유: 보수적으로 인스턴스 유지

상황: 비용 절감이 최우선
설정: 300초
이유: 빠르게 축소 (위험도 높음)
```

**쿨다운 설정 시 황금룙:**
```
원칙 1: ScaleOut < ScaleIn
        └─ 추가는 빠르게, 제거는 천천히

원칙 2: 최소값 설정
        ├─ ScaleOutCooldown: 60초 이상
        └─ ScaleInCooldown: 300초 이상

원칙 3: 서비스 특성에 맞게
        ├─ 금융: 매우 보수적 (1200초+)
        ├─ 일반 웹앱: 중간 (600초)
        └─ 배치작업: 빠름 (60초)

원칙 4: 모니터링 후 조정
        ├─ 스케일링 히스토리 확인
        ├─ flapping 여부 확인
        └─ 비용 vs 성능 트레이드오프
```

**SAA 시험 팁:**
```
Q: CPU 올라갔다가 내려가는 트래픽에서 뭘 설정?
A: ScaleOutCooldown을 길게! (flapping 방지)

Q: 트래픽 급증 상황에서 뭘 해야?
A: ScaleOutCooldown을 짧게! (빠른 대응)

Q: Scale-Down이 너무 느림?
A: ScaleInCooldown을 줄여... 하지만 주의! (안정성 위험)

Q: 비용이 너무 높음?
A: ScaleInCooldown을 줄이거나, Scheduled Scaling + Dynamic Scaling 조합
```

---

## 5. Troubleshooting & Best Practices

### 5.1 Common Issues

**Issue 1: Scaling Not Happening**

**Diagnosis:**
```bash
# Check ASG scaling activities
aws autoscaling describe-scaling-activities \
  --auto-scaling-group-name my-asg \
  --max-records 10

# Check CloudWatch alarms
aws cloudwatch describe-alarms \
  --alarm-names my-asg-scale-up
```

**Solutions:**
- Verify alarm threshold is appropriate (not too high/low)
- Check evaluation periods (default 1-2 minutes to trigger)
- Verify IAM role has ec2:RunInstances permission
- Check max capacity limit not reached
- Ensure instances are passing health checks

**Issue 2: Rapid Scaling Up and Down (Flapping)**

**Problem:** Instances repeatedly added and removed due to metric fluctuation

**Solution:**
```bash
# Increase cooldown period
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name stable-scaling \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "TargetValue": 70.0,
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "ScaleOutCooldown": 300,
    "ScaleInCooldown": 600
  }'
```

- Increase cooldown to prevent rapid changes
- Adjust target value higher (e.g., 75-80% for CPU)
- Use longer evaluation periods in alarms

**Issue 3: Scaling Not Keeping Up with Traffic (Lag)**

**Problem:** Application experiences high latency because scaling is too slow

**Solutions:**
```bash
# Reduce cooldown for faster scaling
--ScaleOutCooldown: 60 (faster response to traffic increase)

# Pre-warm instances
aws autoscaling set-desired-capacity \
  --auto-scaling-group-name my-asg \
  --desired-capacity 5

# Increase target metric threshold
--TargetValue: 80.0 (allow higher usage before scaling)
```

### 5.2 Best Practices

**1. Choose Correct Policy Type:**
- Start with Target Tracking (90% of use cases)
- Use Step Scaling for complex patterns
- Avoid Simple Scaling (legacy)

**2. Metric Selection:**
```
├─ CPU Utilization: Good for compute-heavy apps
├─ Network traffic: Good for data-intensive apps
├─ ALB Request Count: BEST for web applications
├─ Custom metrics: For application-specific needs
└─ Multiple metrics: Use different policies for different aspects
```

**3. Cooldown Period Strategy:**
```
Fast Response Needed:
  ScaleOutCooldown: 60 seconds
  ScaleInCooldown: 300 seconds

Stability Priority:
  ScaleOutCooldown: 300 seconds
  ScaleInCooldown: 600 seconds

General Purpose (Balanced):
  ScaleOutCooldown: 300 seconds (default)
  ScaleInCooldown: 300 seconds (default)
```

**4. Target Value Selection:**
```
CPU Utilization:
├─ 50-60%: Very responsive (high cost, fast reaction)
├─ 70%: Default, balanced choice ✓
└─ 80-90%: Cost-optimized (risk of latency)

ALB Request Count Per Target:
├─ 500-1000: Responsive
├─ 1000-1500: Balanced (recommended) ✓
└─ 2000+: Cost-optimized
```

**5. Monitoring & Alerting:**
```bash
# Monitor scaling activities
aws cloudwatch get-metric-statistics \
  --namespace AWS/AutoScaling \
  --metric-name GroupDesiredCapacity \
  --dimensions Name=AutoScalingGroupName,Value=my-asg \
  --start-time 2025-01-09T00:00:00Z \
  --end-time 2025-01-10T00:00:00Z \
  --period 300 \
  --statistics Average,Maximum
```

**6. SAA Exam Best Practices:**
- Always prefer **Target Tracking Scaling** (simplest, recommended)
- Understand **cooldown period** impact on scaling frequency
- Know the **three policy types** and when to use each
- Remember **ALB Request Count Per Target** for web applications
- Understand that **scale-in (down) requires longer cooldown** to prevent disruption

---

## 6. AWS SAA Exam Practice Problems

### Problem 1: Policy Selection for Web Application

**Scenario:**
You're running a web application with an Application Load Balancer. Traffic patterns are unpredictable, varying from 100 to 10,000 requests/minute. You want the simplest scaling solution that automatically adjusts capacity without manual intervention. Currently, you're struggling with CPU-based scaling because CPU doesn't correlate well with your application's actual load.

Which scaling policy approach is most suitable?

**A)** Simple Scaling Policy based on CPU utilization (cooldown: 60 seconds)
**B)** Target Tracking Scaling with ALBRequestCountPerTarget metric
**C)** Step Scaling with multiple CPU thresholds
**D)** Custom scaling with CloudWatch Events

**Correct Answer: B**

**Explanation:**
- Target Tracking is the recommended approach (eliminates manual configuration)
- ALBRequestCountPerTarget is perfect for web applications with load balancers (metric directly reflects actual traffic)
- This approach automatically creates necessary CloudWatch alarms
- Step Scaling would be overly complex for this scenario
- Simple Scaling is legacy and doesn't provide fine-grained control
- CPU doesn't always correlate with actual requests (some requests are lightweight)

**SAA Key Point:** Use ALBRequestCountPerTarget for web apps with ALB - it's the most accurate metric for actual request volume.

---

### Problem 2: Addressing Scaling Lag

**Scenario:**
Your production e-commerce site experiences periodic traffic spikes. During flash sales, requests increase 5x within 2 minutes, causing significant latency. You've implemented Target Tracking Scaling with ASGAverageCPUUtilization at 70%, but the scaling response is delayed by 3-4 minutes because:
1. CPU takes time to rise to the threshold
2. Scaling decisions have a 300-second cooldown
3. New instances take 2-3 minutes to warm up

What combination of changes would best address this?

**A)** Increase target CPU to 85% and reduce scale-out cooldown to 30 seconds
**B)** Switch to Step Scaling with multiple thresholds
**C)** Switch to ALBRequestCountPerTarget metric AND increase pre-warming with higher desired capacity
**D)** Change to Simple Scaling with 60-second cooldown periods

**Correct Answer: C**

**Explanation:**
- ALBRequestCountPerTarget responds faster than CPU (metric changes with traffic immediately)
- Increasing desired capacity pre-warms instances ready for traffic
- This combination addresses both metric-response lag and instance-startup time
- Increasing CPU threshold doesn't help with lag, just delays response further
- Step Scaling adds complexity without solving the core issue
- Simple Scaling is legacy and not recommended

**Key Learning:**
- Choose metrics that respond quickly to actual load (requests, not CPU)
- Pre-warming instances (higher desired capacity) reduces response time
- Lower cooldowns help for spike scenarios (risky for flapping prevention)

---

### Problem 3: Cost vs. Performance Trade-off

**Scenario:**
Your team uses Target Tracking Scaling with CPU at 70% target. Monthly costs have doubled even though application performance metrics are stable. Analysis shows:
- Instances are scaling down very slowly (10+ minutes between removal)
- During off-peak hours, 2-3 instances remain running unnecessarily
- CPU rarely exceeds 30% during night hours

You need to reduce costs while maintaining good performance. What should you change?

**A)** Increase target CPU to 90% and set ScaleInCooldown to 300 seconds
**B)** Implement different scaling policies for peak vs. off-peak hours using scheduled actions
**C)** Increase target CPU to 80% and reduce ScaleInCooldown to 300 seconds
**D)** Switch to Simple Scaling with larger step adjustments

**Correct Answer: B**

**Explanation:**
- Scheduled actions allow time-based policy changes (peak vs. off-peak)
- Different cooldowns/targets for different times optimize cost
- At night: More aggressive scale-down (shorter cooldown), higher CPU target
- During day: More conservative (prevent traffic latency), lower CPU target
- Option A: Increasing CPU to 90% risks performance issues during peaks
- Option C: Improves scale-down somewhat but doesn't address time-based variations
- Option D: Simple Scaling doesn't address the core issue

**Implementation Example:**
```bash
# Peak hours (9 AM - 6 PM): Conservative, responsive
Target CPU: 70%, ScaleOut: 60s, ScaleIn: 300s

# Off-peak (6 PM - 9 AM): Aggressive cost-optimization
Target CPU: 80%, ScaleOut: 300s, ScaleIn: 120s
```

---

### Problem 4: Production Incident Recovery

**Scenario:**
A critical bug in your auto-scaling configuration caused continuous scaling cycles:
- CPU would spike to 85%, triggering scale-up (add 2 instances)
- As soon as instances started, they loaded stale code causing 100% CPU
- These instances made the average CPU even higher
- More instances scaled up, worsening the problem
- Instances were terminated but no code fixes, repeating the cycle

Which TWO of the following would best prevent this in the future?

**A)** Implement longer ScaleInCooldown (600+ seconds) to prevent rapid removal
**B)** Use CloudWatch detailed monitoring with 60-second granularity
**C)** Implement pause-on-error mechanism via lifecycle hooks
**D)** Reduce target CPU threshold to 50%
**E)** Implement CodeDeploy with pre-flight validation before instances join

**Correct Answer: C & E**

**Explanation:**
- Option C: Lifecycle hooks can pause scaling during detected issues (error threshold exceeded)
- Option E: Validates code/configuration before instance serves traffic (prevents cascading failures)
- Option A: Doesn't help if scaling logic itself is the problem
- Option B: Monitoring doesn't prevent the issue, just documents it faster
- Option D: Doesn't address root cause (bad code), just masks with different threshold

**Prevention Architecture:**
```
Scaling Event
    ↓
Lifecycle Hook (IN_SERVICE_WAIT)
    ↓
CodeDeploy Validation
    - Health check passes?
    - Load test succeeds?
    ↓
If Failed: Halt scaling, alarm operations team
If Passed: Complete lifecycle, proceed to service
```

---

### Problem 5: Scaling Policy Synchronization

**Scenario:**
Your ASG has three different scaling policies:
1. Target Tracking (CPU = 70%) - creates alarms automatically
2. Step Scaling (CPU thresholds 80%, 90%) - manual alarms
3. Simple Scaling (basic scale up/down) - legacy policy

The application is experiencing erratic scaling behavior where sometimes it scales too aggressively, other times too conservatively. Performance is unpredictable.

What is the best solution?

**A)** Consolidate to a single Target Tracking policy with ALBRequestCountPerTarget
**B)** Increase cooldown periods to 600 seconds on all policies
**C)** Use separate ASGs for different workload types
**D)** Monitor all three policies and disable them when conflicts are detected

**Correct Answer: A**

**Explanation:**
- Multiple overlapping policies cause unpredictable behavior (conflicts)
- Target Tracking is AWS-recommended best practice
- Single policy provides consistent, predictable scaling behavior
- ALBRequestCountPerTarget best metric for modern applications
- Option B: Doesn't solve the multi-policy conflict issue
- Option C: Adds operational complexity without solving the root issue
- Option D: Manual intervention defeats purpose of auto-scaling

**Best Practice:** One ASG = One Scaling Policy (usually Target Tracking)

If multiple metrics matter:
- Use custom CloudWatch metric combining multiple signals
- Let Target Tracking manage a single unified metric

---

## Summary

| Aspect | Target Tracking | Step Scaling | Simple Scaling |
|--------|-----------------|--------------|----------------|
| **SAA Exam Likelihood** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐ |
| **Recommended For** | Web applications, general purpose | Complex patterns | Legacy systems |
| **Ease of Use** | Simplest | Moderate | Simple |
| **Configuration** | Specify target metric value | Multiple thresholds & steps | Single threshold & action |
| **Best Metric** | ALBRequestCountPerTarget for web apps | CPU, custom metrics | Generic metrics |
| **Response Time** | Fast (optimized by AWS) | Configurable | Configurable |
| **Cooldown Management** | Built-in optimization | Manual configuration | Manual configuration |
| **Exam Tip** | Default answer for scaling questions | When fine-grained control needed | Rarely appears in questions |

**Final Exam Tip:** When you see a scaling question on the SAA exam, the answer is almost always **Target Tracking Scaling** unless the scenario specifically requires fine-grained control with multiple thresholds (then it's Step Scaling).
