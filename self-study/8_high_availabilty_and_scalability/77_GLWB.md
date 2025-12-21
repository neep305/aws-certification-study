# Gateway Load Balancer (GWLB) 쉽게 이해하기  
*(AWS SAA 시험 대비)*

GWLB는 **처음 보면 가장 헷갈리는 Load Balancer**입니다.  
이유는 👉 **“웹 트래픽용도 아니고, 서버 앞단도 아니다”** 이기 때문이에요.

---

## 1️⃣ GWLB를 한 문장으로 요약하면

> **GWLB는 트래픽을 보안 장비(방화벽, IDS/IPS 등)로 투명하게 보내기 위한 로드 밸런서다**

핵심 키워드:
- **보안(Security)**
- **투명(Transparent)**
- **네트워크 레벨**

---

## 2️⃣ 왜 GWLB가 필요할까? (문제 상황부터)

### ❌ 기존 방식의 문제
예를 들어 VPC 안으로 들어오는 모든 트래픽을  
**방화벽 → IDS → IPS** 같은 보안 장비를 거쳐야 한다고 가정해봅시다.

문제점:
- 트래픽 경로가 복잡해짐
- 보안 장비를 수동으로 연결
- 장비 확장/축소가 어려움
- 장애 시 우회가 힘듦

👉 **이걸 AWS가 “서비스 형태”로 해결한 게 GWLB**

---

## 3️⃣ GWLB의 핵심 개념 (이게 제일 중요)

### 🔑 GWLB = 두 가지 역할을 동시에 수행

#### 1. **Gateway (입구 역할)**
- 모든 트래픽의 **단일 진입점**
- 라우팅 테이블에 연결됨

#### 2. **Load Balancer**
- 트래픽을 여러 보안 장비(가상 어플라이언스)로 분산

---

## 4️⃣ GWLB는 OSI 어느 계층에서 동작할까?

| Load Balancer | OSI Layer |
|--------------|-----------|
| ALB | Layer 7 (HTTP) |
| NLB | Layer 4 (TCP/UDP) |
| **GWLB** | **Layer 3 (IP)** |

👉 GWLB는 **IP 패킷 자체**를 다룸  
👉 HTTP, TCP 같은 개념이 없음

---

## 5️⃣ “투명하다(Transparent)”는 말의 의미

### 🔍 투명하다는 뜻
- 클라이언트와 서버는 **GWLB 존재를 모름**
- IP 주소가 **변하지 않음**
- 트래픽이 **몰래(?) 검사만 받고 돌아옴**

📌 그래서 이름이 **Gateway Load Balancer**

---

## 6️⃣ GWLB 트래픽 흐름 (그림 대신 말로 설명)

1. 사용자가 애플리케이션으로 트래픽 전송
2. 라우팅 테이블이 트래픽을 **GWLB로 보냄**
3. GWLB가 트래픽을 **보안 어플라이언스(방화벽 등)** 로 전달
4. 보안 검사 수행
5. 트래픽이 **원래 목적지로 다시 돌아감**

👉 **중간에 검사만 하고 경로는 그대로**

---

## 7️⃣ GWLB에서 자주 나오는 구성 요소

### 1. Target Group
- 대상: **보안 가상 어플라이언스**
  - Firewall
  - IDS / IPS
  - Deep Packet Inspection 장비

### 2. GENEVE 프로토콜
- GWLB 전용 터널링 프로토콜
- **UDP 6081 포트 사용**
- 시험에서 가끔 키워드로 등장

---

## 8️⃣ ALB / NLB / GWLB 차이 한 번에 정리

| 항목 | ALB | NLB | GWLB |
|----|----|----|----|
| 주 목적 | 웹/API | 고성능 네트워크 | **보안 트래픽 검사** |
| OSI Layer | 7 | 4 | **3** |
| HTTP 이해 | O | X | X |
| 대상 | EC2, ECS, Lambda | EC2, IP | **보안 어플라이언스** |
| 특징 | Path/Host 라우팅 | 초저지연 | **투명 보안** |

---

## 9️⃣ SAA 시험에서 나오는 대표 문장 🚨

아래 문장이 보이면 **GWLB를 떠올리세요**

- “Deploy third-party virtual appliances”
- “Centralized security inspection”
- “Transparent traffic inspection”
- “Firewall, IDS, IPS”
- “Traffic must pass through security appliances”

👉 **정답: GWLB**

---

## 🔟 한 줄 암기 문장 (시험 직전용)

> **GWLB는 모든 트래픽을 보안 장비로 투명하게 보내는 Layer 3 로드 밸런서다**

---

## 다음으로 추천하는 학습 흐름
- GWLB 시험 문제 2~3개 풀어보기
- ALB / NLB / GWLB **선택 문제 비교 연습**
- VPC 라우팅 테이블과 GWLB 연결 구조 이해

원하면 다음 단계 바로 이어서 도와줄게요 👍

---

## SAA Practice Questions (GWLB)

### Q1. Centralized Security Inspection
**Problem (EN)**: A company must route all inbound and outbound VPC traffic through third-party firewall appliances without changing source or destination IPs. Which AWS service best fits this requirement?  
**Choices (EN)**:  
A. Application Load Balancer  
B. Network Load Balancer  
C. Gateway Load Balancer  
D. AWS Network Firewall  
**Answer**: C  
**해설 (KR)**: GWLB는 IP를 유지한 채 트래픽을 보안 어플라이언스로 투명하게 보내는 L3 로드 밸런서로, 중앙 집중 보안 검사 요구에 적합하다.

### Q2. Protocol and Port
**Problem (EN)**: Which protocol and port are used by Gateway Load Balancer to encapsulate traffic toward virtual appliances?  
**Choices (EN)**:  
A. VXLAN over UDP 4789  
B. GENEVE over UDP 6081  
C. GRE over TCP 443  
D. IPsec over UDP 500  
**Answer**: B  
**해설 (KR)**: GWLB는 GENEVE 터널(UDP 6081)을 사용해 패킷을 보안 어플라이언스로 전달한다는 것이 시험 포인트다.

### Q3. Traffic Flow Behavior
**Problem (EN)**: When traffic passes through a Gateway Load Balancer, which statement is correct about how the client and application view the path?  
**Choices (EN)**:  
A. Client IP is replaced by the appliance IP.  
B. Application must add X-Forwarded-For headers to see client IP.  
C. Traffic is transparently inspected; client/server are unaware of GWLB.  
D. Requests are routed based on HTTP Host headers.  
**Answer**: C  
**해설 (KR)**: GWLB는 투명 모드로 동작해 클라이언트와 서버가 로드 밸런서를 인지하지 못하고, IP 변경 없이 보안 검사를 통과한다.
