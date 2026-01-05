# ELB Overview for AWS SAA

## 1. Elastic Load Balancing Basics
- 역할: 다수의 AZ에 걸쳐 트래픽을 분산하고, 헬스 체크를 통해 비정상 대상 제외
- 핵심 구성: Listener(Protocol/Port) → Target Group(대상/포트/헬스체크) → 대상 인스턴스·IP·Lambda
- 건강 상태: `healthy_threshold`, `unhealthy_threshold`, `timeout`, `interval`, `matcher`(HTTP 코드)

**Question (Multiple Choice)**  
Which ELB component is responsible for performing health checks?  
A. Listener  
B. Target Group  
C. Security Group  
D. Route 53  
- Answer: B  
- 해설: 헬스 체크 설정은 Target Group 단위이며 Listener는 포트/프로토콜 매핑만 담당한다.

## 2. Load Balancer Types
- ALB: L7, HTTP/HTTPS/WebSocket, 경로·호스트 기반 라우팅, WAF 연계
- NLB: L4, TCP/UDP/TLS, 초저지연/고성능, 고정 IP·EIP 지원
- CLB: 레거시 L4/L7, 신규 워크로드에는 비권장
- GWLB: 3rd-party 보안/네트워킹 가시성(VPC 미러링 유사)

**Question (Multiple Choice)**  
You need HTTP header-based routing and WebSocket support. Which LB fits best?  
A. Network Load Balancer  
B. Gateway Load Balancer  
C. Application Load Balancer  
D. Classic Load Balancer  
- Answer: C  
- 해설: 헤더 기반 라우팅과 WebSocket은 L7 기능으로 ALB가 제공한다.

## 3. Target Types & Modes
- 대상 유형: `instance`, `ip`(프라이빗 IP), `lambda`
- 등록 모드: `instance mode`(ALB/CLB) vs `IP mode`(ALB/NLB) 선택 시, 컨테이너/프라이빗 엔드포인트 분산에 유용
- ALB는 동적 포트 매핑(EC2 + ECS) 지원 → 다중 태스크 포트 자동 등록

**Question (Multiple Choice)**  
Which target type enables distributing traffic to Lambda functions via ALB?  
A. Instance  
B. IP  
C. Lambda  
D. Container  
- Answer: C  
- 해설: ALB는 Lambda를 대상 유형으로 직접 등록해 L7 트리거로 사용할 수 있다.

## 4. Listeners, Certificates, and TLS
- Listener별 프로토콜/포트 지정, 규칙 순서 기반 매칭(우선순위 숫자 낮을수록 우선)
- TLS 종료는 LB 단에서 수행(서버 인증서 ACM에 저장), 백엔드와는 HTTP 또는 재암호화(TLS) 선택
- SNI: 단일 ALB/NLB TLS 리스너에 여러 인증서 바인딩 가능(도메인별 제공)

**Question (Multiple Choice)**  
How do you serve multiple HTTPS domains on one ALB listener?  
A. Use multiple listeners on the same port  
B. Attach multiple security groups  
C. Configure SNI with multiple certificates  
D. Create separate ALBs  
- Answer: C  
- 해설: 하나의 TLS 리스너에서 다수의 인증서를 SNI로 선택 제공할 수 있다.

## 5. Cross-Zone Load Balancing & IPs
- Cross-Zone: ALB 기본 활성화(요금 없음), NLB/GLB는 선택적(데이터 전송 요금 고려)
- NLB는 가용영역별 고정 IP를 제공하고, 필요 시 EIP 할당 가능 → 방화벽 화이트리스트에 유리

**Question (Multiple Choice)**  
For predictable source IPs from the load balancer, which choice is best?  
A. ALB with default settings  
B. NLB with Elastic IPs  
C. GWLB with VPC endpoints  
D. CLB with stickiness  
- Answer: B  
- 해설: NLB는 AZ별 고정 IP 또는 EIP를 제공해 소스 IP 예측이 가능하다.

## 6. Stickiness, Sessions, and Headers
- ALB Stickiness: Target Group 쿠키 기반(`AWSALB`), 지속 시간 설정 가능
- NLB Stickiness: 소스 IP 기반(연결 지향 워크로드에 유용)
- X-Forwarded-* 헤더(ALB/CLB): `X-Forwarded-For`, `X-Forwarded-Port`, `X-Forwarded-Proto`

**Question (Multiple Choice)**  
Which header allows backend apps to know the original client IP when using ALB?  
A. X-Forwarded-Proto  
B. X-Forwarded-Port  
C. X-Forwarded-For  
D. X-Client-IP  
- Answer: C  
- 해설: ALB는 클라이언트 원본 IP를 `X-Forwarded-For` 헤더에 담아 전달한다.

## 7. Health Checks & High Availability
- 헬스 체크 엔드포인트는 가볍고 빠르게 응답하도록 설계(200/301/302 등 허용 코드)
- AZ 다중 배포 + 최소 2개 대상의 헬시 상태 권장(ASG와 연계 시 자동 복구)
- Deregistration delay: LB가 기존 연결을 안전 종료하도록 대기 시간 설정

**Question (Multiple Choice)**  
Why configure a deregistration delay on a Target Group?  
A. To speed up new connections  
B. To allow in-flight requests to complete before removing a target  
C. To reduce TLS handshake time  
D. To force immediate scale-in  
- Answer: B  
- 해설: 대상 제거 시 진행 중 요청이 정상 완료되도록 대기 시간을 두어 장애를 방지한다.

## 8. Security & Integration
- ALB는 Security Group 적용, NLB는 SG 미적용(백엔드 ENI/인스턴스 SG로 제어)
- WAF는 ALB/CloudFront와 통합, NLB에는 직접 적용 불가
- ALB Access Logs(S3), NLB Flow Logs(VPC), CloudWatch Metrics/Alarms으로 관측성 확보

**Question (Multiple Choice)**  
Which combination lets you add managed layer-7 filtering to an internet-facing app?  
A. NLB + VPC Flow Logs  
B. ALB + WAF  
C. NLB + Security Groups  
D. GWLB + VPC endpoints only  
- Answer: B  
- 해설: L7 보안 관리형 필터링은 WAF가 제공하며 ALB나 CloudFront와 연계해 사용한다.
