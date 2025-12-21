# AWS High Availability & Scalability 정리

## 1️⃣ Vertical Scaling (수직 확장)

### 개념
- **하나의 EC2 인스턴스 성능을 키우는 방식**
- CPU, RAM, 네트워크 성능 증가

### 방식
- EC2 인스턴스 타입 변경
  - 예: `t2.micro → m5.large`

### 장점
- 구조가 단순함
- 빠르게 성능 향상 가능

### 단점 (중요)
- 물리적 한계 존재
- 보통 인스턴스 재부팅 필요
- **Single Point of Failure**
- High Availability 제공 ❌

### 시험 키워드
- **Scale Up / Scale Down**

---

## 2️⃣ Horizontal Scaling (수평 확장)

### 개념
- **EC2 인스턴스 개수를 늘리거나 줄이는 방식**

### 방식
- 여러 EC2 인스턴스 사용
- 보통 다음과 함께 사용됨:
  - Auto Scaling Group (ASG)
  - Load Balancer (ELB)

### 장점
- 트래픽 변화에 유연
- 인스턴스 장애 시 서비스 유지 가능
- High Availability 구현 가능

### 시험 포인트
- 트래픽 증가 / 사용자 급증 → **Horizontal Scaling**

### 시험 키워드
- **Scale Out / Scale In**

---

## 3️⃣ High Availability (HA)

### 개념
- **장애가 발생해도 서비스가 중단되지 않도록 하는 것**
- 성능이 아니라 **가용성**이 목적

### AWS에서의 핵심
- **Multi-AZ (여러 Availability Zone)**
- 동일 애플리케이션을 여러 AZ에 배포

### 대표 구성
- EC2 + Auto Scaling Group (Multi-AZ)
- Load Balancer (Multi-AZ)

### 주의할 점
- ❌ HA = 성능 향상
- ✅ HA = 장애 대응 / 무중단 서비스

---

## 🧠 시험용 핵심 공식

> **Horizontal Scaling + Multi-AZ = High Availability**

---

## ✍️ 한 줄 요약
- Vertical Scaling → 서버를 **더 크게**
- Horizontal Scaling → 서버를 **더 많이**
- High Availability → **장애 나도 안 멈춤**

---
# AWS High Availability & Scalability – 시험 문제 패턴 정리

## 1️⃣ Vertical vs Horizontal Scaling 구분 문제

### 문제 패턴
- “EC2 인스턴스의 CPU / RAM을 증가시켜야 한다”
- “인스턴스 타입을 변경해야 한다”

### 정답 포인트
- 👉 **Vertical Scaling**
- 키워드:
  - instance size 변경
  - scale up / scale down
  - 재부팅 언급

---

### 문제 패턴
- “트래픽이 증가함에 따라 서버 수를 늘려야 한다”
- “수요에 따라 자동으로 확장/축소해야 한다”

### 정답 포인트
- 👉 **Horizontal Scaling**
- 키워드:
  - scale out / scale in
  - Auto Scaling Group
  - Load Balancer

---

## 2️⃣ High Availability (HA) 문제 패턴

### 문제 패턴
- “하나의 AZ 장애가 발생해도 서비스는 계속 제공되어야 한다”
- “Single point of failure를 제거하라”
- “무중단 서비스가 필요하다”

### 정답 포인트
- 👉 **Multi-AZ**
- 보통 함께 나오는 구성:
  - EC2 in multiple AZ
  - Load Balancer (Multi-AZ)
  - Auto Scaling Group (Multi-AZ)

⚠️ 주의  
- 인스턴스 수만 많고 **같은 AZ**면 HA ❌

---

## 3️⃣ Scaling + HA 조합 문제 (가장 자주 출제됨)

### 문제 패턴
- “트래픽이 예측 불가능하다”
- “장애에도 자동으로 복구되어야 한다”
- “확장성과 가용성이 모두 필요하다”

### 정답 공식
> **ELB + Auto Scaling Group + Multi-AZ**

👉 시험에서 거의 **정답 템플릿**처럼 사용됨

---

## 4️⃣ 오답 유도 패턴 (함정!)

### 함정 ①
- “성능 문제 해결” → Multi-AZ 선택 ❌  
→ 성능 문제는 **Scalability**, AZ는 **HA**

### 함정 ②
- “HA 필요” → 인스턴스 사이즈 업 ❌  
→ Vertical Scaling은 **HA 제공 안 함**

### 함정 ③
- “트래픽 증가” → EC2 하나 더 큰 타입 ❌  
→ 보통 **Horizontal Scaling**이 정답

---

## 5️⃣ 한 줄로 문제 해석하는 법

- **성능 부족** → Scaling
- **장애 대응** → High Availability
- **트래픽 예측 불가 + 장애 대응**  
  → **Horizontal Scaling + Multi-AZ**

---

## 🧠 시험 직전 암기 문장
> “AWS에서 HA는 Multi-AZ,  
> 확장은 Horizontal Scaling이 기본이다.”
