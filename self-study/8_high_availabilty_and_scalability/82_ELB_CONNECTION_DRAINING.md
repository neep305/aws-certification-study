# ELB - Connection Draining (및 Target Deregistration Delay)

## 개요
- **정의**: Connection Draining(클래식 LB) 또는 Deregistration Delay(ALB/NLB)는 로드밸런서가 인스턴스 등록 해제 시 기존 연결을 우아하게 종료하는 기능이다.
- **목적**: 진행 중인 요청을 강제 중단하지 않고 완료될 때까지 대기한 후 인스턴스를 안전하게 제거.
- **적용 범위**: CLB (Connection Draining), ALB/NLB (Deregistration Delay로 명칭 변경).

## 핵심 개념

### 활성화되지 않을 때의 문제
```
시나리오: 로드밸런서에서 인스턴스 갑자기 제거
1. 진행 중인 요청이 중단됨 (HTTP 500 에러 등)
2. 사용자 경험 저하 (응답 실패)
3. 데이터 손실 가능성 (트랜잭션 미완료)
4. 로그 미기록 (요청 처리 과정 손실)

결과: 
- 가용성 저하
- 사용자 불만
- 데이터 무결성 문제
```

### 활성화되었을 때의 동작
```
시나리오: Connection Draining 활성화 (예: 300초)
1. 관리자: 인스턴스 등록 해제 요청
2. 로드밸런서: 즉시 새로운 요청 받지 않음
3. 로드밸런서: 기존 연결은 유지
4. 인스턴스: 진행 중인 요청 계속 처리
5. 300초 후: 남은 연결 모두 종료
6. 인스턴스: 안전하게 제거됨

결과:
- 기존 요청 정상 완료
- 데이터 무결성 보장
- 사용자 경험 보호
```

## ELB 유형별 특징

| 특성 | Classic LB | ALB | NLB |
|------|-----------|-----|-----|
| 기능명 | Connection Draining | Deregistration Delay | Deregistration Delay |
| 기본값 | 300초 (활성화) | 300초 (활성화) | 300초 (활성화) |
| 최대값 | 3600초 (1시간) | 3600초 (1시간) | 3600초 (1시간) |
| 최소값 | 1초 | 1초 | 0초 (비활성화) |

## 설정 값 결정 기준

```
권장 설정값:
- 일반 웹 애플리케이션: 300초 (기본값)
- 빠른 응답 API: 30~60초
- 장시간 처리: 600초~3600초
- 마이크로서비스: 10~30초

결정 요소:
1. 평균 요청 처리 시간
2. 최대 처리 시간
3. 배포/갱신 주기
4. 가용성 vs 신속성 우선순위
```

---

## 설정 방법

### AWS 콘솔에서 설정

**Classic Load Balancer:**
```
1. EC2 → Load Balancers → CLB 선택
2. Instances → Edit Deregistration
3. "Connection Draining" 체크
4. 타임아웃 값 설정
5. Save
```

**Application/Network Load Balancer:**
```
1. EC2 → Target Groups → 대상 그룹
2. Attributes → Edit
3. "Deregistration delay" 설정
4. Save
```

### AWS CLI 명령어

**CLB Connection Draining:**
```bash
aws elb modify-load-balancer-attributes \
	--load-balancer-name my-clb \
	--load-balancer-attributes ConnectionDrainingPolicy={Enabled=true,Timeout=300}
```

**ALB/NLB Deregistration Delay:**
```bash
aws elbv2 modify-target-group-attributes \
	--target-group-arn <arn> \
	--attributes Key=deregistration_delay.timeout_seconds,Value=300
```

### CloudFormation 설정

```yaml
TargetGroup:
	Type: AWS::ElasticLoadBalancingV2::TargetGroup
	Properties:
		Name: my-targets
		Port: 80
		Protocol: HTTP
		VpcId: vpc-xxxxx
		TargetGroupAttributes:
			- Key: deregistration_delay.timeout_seconds
				Value: '300'
```

---

## 동작 메커니즘

### Drain 타임라인 (300초)

```
00:00 - 인스턴스 등록 해제 명령
00:00 - 로드밸런서: "Draining" 상태
			 • 새 요청: 라우팅 안 함
			 • 기존 요청: 계속 처리
03:00 - Delay 만료
			 • 남은 연결 강제 종료
			 • 인스턴스 제거 완료
```

---

## SAA 시험 대비 예제 문제

### 문제 1
**상황:** REST API, 평균 2초, 최대 10초, 주 1회 배포

**Deregistration Delay:** 30초 (권장)
- 최대 처리 시간 + 여유
- 배포 효율성 고려

### 문제 2
**상황:** 금융 거래 시스템, 최대 처리 3분

**Deregistration Delay:** 300~600초
- 데이터 무결성 최우선
- 여유 있게 설정

### 문제 3
**상황:** WebSocket 기반 채팅, 300초 설정하니 배포 후 연결 끊김

**해결책:** 3600초로 증가
- WebSocket은 장시간 연결 필요
- 배포 중에도 연결 보호

### 문제 4
**상황:** 10개 인스턴스 scale-down, 각 60초

**예상 시간:** 600초 (순차 처리)
- 배포 단축: 30초 설정 → 300초

---

## 비용 영향

```
직접 비용: 없음 (로드밸런서 요금 포함)

간접 비용:
1. 인스턴스 추가 실행 시간
	 - 300초 동안 계속 실행
	 - 소형 인스턴스: ~$0.03

2. Auto Scaling 지연
	 - 10개 × 300초 = 50분 소요

비용 최적화:
- 빠른 배포: 시간 단축
- 안정성 우선: 기본값 유지
```

---

## 트러블슈팅

### "인스턴스가 Draining에서 멈춤"
**원인:** WebSocket, 롱 폴링, 응답 없는 요청

**해결:**
```bash
aws elbv2 modify-target-group-attributes \
	--target-group-arn <arn> \
	--attributes Key=deregistration_delay.timeout_seconds,Value=600
```

### "배포 시간이 너무 오래"
**해결:**
```bash
# 배포 중 임시 단축
aws elbv2 modify-target-group-attributes \
	--target-group-arn <arn> \
	--attributes Key=deregistration_delay.timeout_seconds,Value=30

# 배포 후 복구
aws elbv2 modify-target-group-attributes \
	--target-group-arn <arn> \
	--attributes Key=deregistration_delay.timeout_seconds,Value=300
```

---

## CLI 명령어 모음

```bash
# 변경
aws elbv2 modify-target-group-attributes \
	--target-group-arn <arn> \
	--attributes Key=deregistration_delay.timeout_seconds,Value=60

# 등록 해제
aws elbv2 deregister-targets \
	--target-group-arn <arn> \
	--targets Id=i-xxxxx

# 상태 확인
aws elbv2 describe-target-health \
	--target-group-arn <arn>

# 등록
aws elbv2 register-targets \
	--target-group-arn <arn> \
	--targets Id=i-xxxxx
```

---

## 핵심 요약

- **CLB**: Connection Draining
- **ALB/NLB**: Deregistration Delay
- **기본값**: 300초
- **목적**: 기존 요청 완료 → 안전한 인스턴스 제거
- **설정**: 요청 처리 시간 + 여유
- **배포**: 필요시 임시 단축 후 복구

**SAA 시험:** 개요와 예제 문제 정독 필수!