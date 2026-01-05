# Elastic Load Balancer - SSL/TLS Certificates

## 개요
- **정의**: SSL/TLS 인증서는 로드밸런서와 클라이언트 간의 HTTPS 통신을 보호하고 서버의 신원을 증명하는 디지털 인증서다.
- **목적**: 네트워크 트래픽 암호화, 데이터 무결성 확보, 서버 인증을 통한 신뢰성 구축.
- **적용 범위**: ALB, CLB, NLB 모두 HTTPS(포트 443) 리스너에 SSL/TLS 인증서 필수.

## ELB 유형별 SSL/TLS 지원 요약

| 특성 | Classic LB | ALB | NLB |
|------|-----------|-----|-----|
| HTTPS 지원 | 지원 (포트 443) | 지원 (포트 443) | 지원 (포트 443) |
| 인증서 출처 | ACM, IAM (자체 관리) | ACM 권장, IAM 가능 | ACM 권장, IAM 가능 |
| SNI (Server Name Indication) | 비지원 (한 리스너에 1개만) | 지원 (여러 도메인 가능) | 지원 (여러 도메인 가능) |
| TLS 버전 제어 | 제한적 | 우수한 제어 | 우수한 제어 |
| 암호화 스위트 선택 | 고정 | 정책으로 제어 가능 | 정책으로 제어 가능 |
| 설정 난이도 | 수동 관리 | ACM 자동 갱신 | ACM 자동 갱신 |

## 인증서 출처별 특징

### 1. AWS Certificate Manager (ACM) — 권장
**장점:**
- **무료**: AWS 소유 리소스(로드밸런서, CloudFront 등)에서 사용 시 완전 무료
- **자동 갱신**: 만료 60일 전부터 자동 갱신 시도
- **쉬운 배포**: 콘솔에서 인증서 선택 후 적용
- **와일드카드 지원**: `*.example.com` 패턴으로 다중 서브도메인 커버
- **SANs 지원**: Subject Alternative Names로 여러 도메인 하나의 인증서로 처리

**단점:**
- AWS 리소스에서만 사용 가능 (외부 웹서버에 다운로드 불가)
- Public 인증서만 자동 갱신 (Private CA는 별도 처리)

**비용:**
- EC2/로드밸런서에 사용: $0 (완전 무료)
- Private CA 사용: 월 $400 + 발급당 $0.75

### 2. IAM에서 자체 관리 인증서
**특징:**
- 외부에서 구입한 인증서(예: Comodo, DigiCert)를 IAM에 수동 import
- 갱신은 사용자가 수동으로 처리해야 함
- 어디든 사용 가능 (EC2, 온프레미스 등)

**단점:**
- 수동 갱신으로 인한 만료 위험
- 관리 복잡성 증가

### 3. 자체 서명 인증서 (Self-Signed)
**용도:**
- 개발/테스트 환경만 사용
- 공개 인증서로는 사용 불가 (브라우저 경고)

---

## SSL/TLS 버전과 보안 정책

### TLS 버전 (최신 권장)
```
레거시 (비권장):
- SSLv3: 더 이상 지원 안 함 (2023년 이후)
- TLSv1.0: 2018년 주요 브라우저에서 지원 중단 → 사용 금지
- TLSv1.1: 향후 지원 중단 예정

현재 표준:
- TLSv1.2: 일반적 대부분의 클라이언트 지원
- TLSv1.3: 최신 기술, 더 빠른 핸드셰이크, AWS 권장

AWS 기본 정책:
- ALB/NLB: 기본적으로 TLSv1.2 이상만 허용 (권장)
- CLB: 설정에 따라 TLSv1.0부터 지원 가능 (레거시 대응용)
```

### 암호화 스위트 (Cipher Suite)
- **강력한 암호화**: AES-256, ChaCha20 등 권장
- **약한 암호화**: DES, RC4 등은 비활성화
- AWS에서 제공하는 보안 정책 선택으로 자동 관리 권장

### AWS의 미리 정의된 보안 정책
```
정책 레벨:
- ELBSecurityPolicy-FS-1-2-Res-2019-08 (강력함)
  → TLSv1.2, 전방향 비밀성(Forward Secrecy) 강제
  
- ELBSecurityPolicy-TLS-1-2-2017-01 (표준)
  → TLSv1.2 최소, 광범위 클라이언트 호환성
  
- ELBSecurityPolicy-TLS13-1-2-2021-06 (최신)
  → TLSv1.3 지원, 최고 보안 수준
  
레거시:
- ELBSecurityPolicy-2016-08 (구형)
  → TLSv1.0 허용 (현대 브라우저와 호환성 저하)
```

권장: `ELBSecurityPolicy-TLS13-1-2-2021-06` 또는 `ELBSecurityPolicy-FS-1-2-Res-2019-08`

---

## 인증서 설정 방법

### AWS 콘솔에서 ACM 인증서 적용

**ALB 생성 시:**
```
1. EC2 → Load Balancers → Create Application Load Balancer
2. Listeners and routing → Add listener
3. Protocol: HTTPS, Port: 443
4. "Choose a certificate from ACM"
5. 원하는 인증서 선택 (자동 갱신됨)
```

**기존 ALB에 인증서 추가:**
```
1. Listeners → 443 리스너 선택
2. Edit → Manage certificates
3. ACM 인증서 추가 (SNI로 여러 인증서 가능)
```

### AWS CLI를 통한 설정

**ACM 인증서 생성 요청:**
```bash
aws acm request-certificate \
  --domain-name example.com \
  --subject-alternative-names www.example.com \
  --validation-method DNS \
  --region us-east-1
```

**인증서 검증 (DNS 검증):**
```bash
# Route53에서 자동으로 레코드가 생성되면 검증됨
# 또는 이메일 검증 선택 시 이메일로 링크 클릭
```

**ALB에 인증서 적용:**
```bash
aws elbv2 modify-listener \
  --listener-arn arn:aws:elasticloadbalancing:region:account-id:listener/app/my-alb/... \
  --protocol HTTPS \
  --port 443 \
  --certificates CertificateArn=arn:aws:acm:region:account-id:certificate/xxxxx \
  --ssl-policy ELBSecurityPolicy-TLS13-1-2-2021-06
```

**인증서 상태 확인:**
```bash
aws acm describe-certificate \
  --certificate-arn arn:aws:acm:region:account-id:certificate/xxxxx
```

**로드밸런서의 HTTPS 리스너 확인:**
```bash
aws elbv2 describe-listeners \
  --load-balancer-arn arn:aws:elasticloadbalancing:region:account-id:loadbalancer/app/my-alb/... \
  --query 'Listeners[?Protocol==`HTTPS`]'
```

### CloudFormation에서 설정

**ACM 인증서를 사용하는 ALB:**
```yaml
Resources:
  MyALB:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      LoadBalancerName: my-alb
      Subnets:
        - subnet-xxxxx (us-east-1a)
        - subnet-yyyyy (us-east-1b)
      Scheme: internet-facing
      SecurityGroups:
        - sg-xxxxx

  HTTPSListener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      DefaultActions:
        - Type: forward
          TargetGroupArn: !Ref TargetGroup
      LoadBalancerArn: !Ref MyALB
      Port: 443
      Protocol: HTTPS
      Certificates:
        - CertificateArn: 'arn:aws:acm:us-east-1:123456789012:certificate/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'
      SslPolicy: ELBSecurityPolicy-TLS13-1-2-2021-06

  # HTTP를 HTTPS로 리다이렉트
  HTTPListener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      DefaultActions:
        - Type: redirect
          RedirectConfig:
            Protocol: HTTPS
            Port: '443'
            StatusCode: HTTP_301
      LoadBalancerArn: !Ref MyALB
      Port: 80
      Protocol: HTTP

  TargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      Port: 80
      Protocol: HTTP
      VpcId: vpc-xxxxx
      HealthCheckEnabled: true
      HealthCheckPath: /
```

### 인증서 갱신

**ACM 자동 갱신 (AWS 권장):**
- 만료 60일 전부터 자동 갱신 시도
- DNS 검증: 자동으로 진행 (Route53 통합)
- 사용자 개입 없음

**CLI로 갱신 상태 모니터링:**
```bash
aws acm describe-certificate \
  --certificate-arn arn:aws:acm:region:account-id:certificate/xxxxx \
  --query 'Certificate.[Status,RenewalEligibility,Serial]'
```

**IAM에 import한 인증서 갱신 (수동):**
```bash
# 새로운 인증서 발급받음 (외부 CA에서)
aws iam upload-server-certificate \
  --server-certificate-name my-cert-2025 \
  --certificate-body file://certificate.pem \
  --certificate-chain file://chain.pem \
  --private-key file://private-key.pem

# 로드밸런서 리스너에서 사용하는 인증서 변경
aws elbv2 modify-listener \
  --listener-arn arn:aws:elasticloadbalancing:... \
  --certificates CertificateArn=arn:aws:iam::account-id:server-certificate/my-cert-2025
```

---

## SNI (Server Name Indication) 활용

**개념:** 단일 리스너에서 여러 도메인의 인증서를 호스팅

**지원 현황:**
- ALB: 지원 ✓
- NLB: 지원 ✓
- CLB: 미지원 (한 리스너에 1개 인증서만)

**SNI 설정 (ALB):**
```bash
# 첫 번째 인증서 (기본)
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:... \
  --protocol HTTPS \
  --port 443 \
  --certificates CertificateArn=arn:aws:acm:...:certificate/cert-1

# 추가 인증서 (SNI)
aws elbv2 add-listener-certificates \
  --listener-arn arn:aws:elasticloadbalancing:...:listener/app/my-alb/... \
  --certificates CertificateArn=arn:aws:acm:...:certificate/cert-2 \
               CertificateArn=arn:aws:acm:...:certificate/cert-3
```

**CloudFormation에서 SNI:**
```yaml
HTTPSListener:
  Type: AWS::ElasticLoadBalancingV2::Listener
  Properties:
    DefaultActions:
      - Type: forward
        TargetGroupArn: !Ref TargetGroup
    LoadBalancerArn: !Ref MyALB
    Port: 443
    Protocol: HTTPS
    Certificates:
      - CertificateArn: 'arn:aws:acm:...:certificate/primary-cert'
    SslPolicy: ELBSecurityPolicy-TLS13-1-2-2021-06

ListenerCertificates:
  Type: AWS::ElasticLoadBalancingV2::ListenerCertificate
  Properties:
    ListenerArn: !Ref HTTPSListener
    Certificates:
      - CertificateArn: 'arn:aws:acm:...:certificate/alt-cert-1'
      - CertificateArn: 'arn:aws:acm:...:certificate/alt-cert-2'
```

---

## 비용 분석

### ACM 비용
```
Public 인증서 (권장):
- 무료 (로드밸런서/CloudFront 사용 시)

Private CA:
- 월 비용: $400
- 인증서 발급당: $0.75
- 해지/갱신 요청당: $0.75
```

### 로드밸런서 비용 (인증서 추가로 인한 변화)
```
일반적으로:
- HTTPS 리스너 추가 시 추가 비용 없음
- 트래픽에 따른 데이터 처리 비용만 발생
- ALB: 시간당 $0.0225 (기본 로드밸런서 요금)
```

### 비용 최적화
```
권장:
1. ACM Public 인증서 사용 (완전 무료)
2. 여러 도메인은 SNI로 통합 (인증서 수 최소화)
3. 와일드카드 인증서 활용 (*.example.com)
4. 자동 갱신 설정 (수동 갱신 오류 비용 감소)
```

---

## 보안 모범 사례

### 1. 강력한 TLS 설정
```
하지 말것:
- TLSv1.0, TLSv1.1 허용 금지
- 약한 암호화 스위트(DES, MD5) 비활성화
- 자체 서명 인증서를 프로덕션에 사용

권장:
- TLSv1.2 이상만 허용
- ELBSecurityPolicy-TLS13-1-2-2021-06 정책 사용
- 90일 이상의 만료 기간
```

### 2. 인증서 검증
```bash
# 인증서 정보 확인
openssl x509 -in certificate.pem -text -noout

# 인증서 유효성 테스트
openssl s_client -connect example.com:443 -servername example.com
```

### 3. 만료 모니터링
```
ACM: 자동 모니터링 (SNS 알림 옵션)
IAM import 인증서: CloudWatch Events로 알림 설정

AWS Chatbot으로 Slack 알림:
- 인증서 만료 90일 전 알림
- 갱신 실패 시 알림
```

### 4. HTTP → HTTPS 강제 리다이렉트
```bash
# 모든 HTTP 요청을 HTTPS로 리다이렉트
aws elbv2 modify-listener \
  --listener-arn arn:aws:elasticloadbalancing:...:listener/app/my-alb/xxx/xxx \
  --default-actions Type=redirect,RedirectConfig={Protocol=HTTPS,Port=443,StatusCode=HTTP_301}
```

---

## 트러블슈팅 가이드

### 상황 1: "Certificate not found" 에러
**원인:**
- ACM 인증서가 잘못된 리전에 있는 경우
- IAM에 인증서가 정상 import되지 않음
- 인증서 ARN이 잘못된 경우

**해결:**
```bash
# 현재 리전의 인증서 목록 확인
aws acm list-certificates --region us-east-1

# 인증서 상세 정보 확인
aws acm describe-certificate \
  --certificate-arn arn:aws:acm:region:account-id:certificate/xxxxx
```

### 상황 2: SSL/TLS 핸드셰이크 실패
**원인:**
- 클라이언트가 지원하는 TLS 버전과 로드밸런서 정책 불일치
- 약한 암호화 스위트만 지원하는 구형 클라이언트

**확인:**
```bash
# 로드밸런서의 SSL 정책 확인
aws elbv2 describe-listeners \
  --load-balancer-arn arn:aws:elasticloadbalancing:... \
  --query 'Listeners[?Protocol==`HTTPS`].SslPolicy'

# TLSv1.0 지원 필요한 경우 레거시 정책으로 변경
aws elbv2 modify-listener \
  --listener-arn arn:aws:elasticloadbalancing:... \
  --ssl-policy ELBSecurityPolicy-2016-08
```

### 상황 3: 인증서 만료 임박
**ACM의 경우:**
- 자동 갱신이 활성화되어 있는지 확인
- 만료 60일 전부터 갱신 프로세스 시작

**확인:**
```bash
aws acm describe-certificate \
  --certificate-arn arn:aws:acm:region:account-id:certificate/xxxxx \
  --query 'Certificate.[Status,NotAfter]'
```

**IAM import 인증서의 경우:**
```bash
# 인증서 목록 확인 (만료 날짜 표시)
aws iam list-server-certificates

# 만료된 인증서 삭제
aws iam delete-server-certificate \
  --server-certificate-name old-cert-2024
```

### 상황 4: SNI 관련 이슈 (여러 도메인)
**문제:** 일부 도메인에서만 인증서 오류 발생

**확인:**
```bash
# 리스너에 등록된 모든 인증서 확인
aws elbv2 describe-listener-certificates \
  --listener-arn arn:aws:elasticloadbalancing:...

# 클라이언트 테스트
openssl s_client -connect example1.com:443 -servername example1.com
openssl s_client -connect example2.com:443 -servername example2.com
```

---

## SAA 시험 대비: 핵심 포인트 & 예제 문제

### 시험 출제 패턴
1. **"어느 로드밸런서가 SNI를 지원하는가?"** 유형 → ALB/NLB (CLB 제외)
2. **"인증서 갱신을 자동화하려면?"** 유형 → ACM 사용
3. **"프로덕션 환경에서 권장하는 TLS 버전은?"** 유형 → TLSv1.2 이상
4. **"여러 도메인의 HTTPS를 한 로드밸런서에서 처리하려면?"** 유형 → SNI + ACM
5. **"비용을 최소화하면서 보안을 유지하려면?"** 유형 → ACM + 강력한 정책

### 시험 대비 체크리스트
- [ ] CLB는 SNI 미지원 (한 리스너에 1개만)
- [ ] ACM은 AWS 리소스에서만 무료
- [ ] 자동 갱신은 ACM의 큰 장점
- [ ] TLSv1.0/1.1 사용 금지
- [ ] HTTP → HTTPS 리다이렉트는 별도 리스너 필요

---

## 예제 문제 풀이

### 문제 1
**상황:** 회사가 example.com과 api.example.com 두 도메인을 하나의 로드밸런서로 관리하려고 한다. 비용을 최소화하면서 HTTPS를 지원해야 한다.

**문제:** 가장 적절한 구성은?
- A) CLB에 2개의 HTTPS 리스너 생성 (도메인별)
- B) ALB에 1개의 HTTPS 리스너와 SNI로 2개 인증서 추가
- C) ALB에 2개의 HTTPS 리스너 생성 (도메인별)
- D) NLB만 여러 도메인 지원

**정답:** B
- ALB는 SNI를 지원하므로 단일 리스너(포트 443)에 여러 인증서 등록 가능
- 와일드카드 인증서 `*.example.com` 사용으로 더 간단하게 가능
- CLB는 SNI 미지원이므로 A, C는 불가능
- 비용 최소화: ACM 무료 인증서 사용

---

### 문제 2
**상황:** 기업의 자체 관리 인증서(3년 유효)를 로드밸런서에 적용했다. 최근 인증서 만료를 관리하기 어렵다는 우려가 제기됐다.

**문제:** 권장하는 개선 방안은?
- A) 인증서 유효 기간을 5년으로 연장 (갱신 빈도 감소)
- B) ACM으로 마이그레이션하고 자동 갱신 활성화
- C) 갱신 알림을 위해 Lambda + SNS 구성
- D) 6개월마다 수동 갱신 일정 수립

**정답:** B
- ACM은 자동 갱신으로 수동 개입 최소화
- 추가 비용 없음 (로드밸런서 사용 시)
- 갱신 실패 위험이 거의 없음
- A는 보안 모범 사례 위반 (짧은 주기 권장)

---

### 문제 3
**상황:** 조직에서 레거시 시스템(TLSv1.0 지원만)과 최신 시스템(TLSv1.3)을 동일 로드밸런서에서 제공해야 한다.

**문제:** 가장 안전한 방안은?
- A) ELBSecurityPolicy-2016-08 (TLSv1.0 허용)를 모든 리스너에 적용
- B) 레거시 시스템용 별도 로드밸런서 구성 (TLSv1.0 정책)
- C) API Gateway 또는 WAF를 통한 트래픽 분리
- D) 클라이언트 업그레이드 대기 후 TLSv1.2 이상으로 강제

**정답:** B (또는 C)
- TLSv1.0은 보안 취약점 많음 → 프로덕션 환경에서 금지
- 레거시와 최신을 분리하는 것이 최선 (보안 + 성능)
- D도 장기 전략으로 고려할 수 있음
- A는 절대 금지 (보안 위험)

---

### 문제 4
**상황:** 조직에서 프라이빗 도메인(내부용) HTTPS 인증서를 로드밸런서에 적용하려고 한다.

**문제:** 권장하는 방법은?
- A) 자체 서명 인증서 생성 후 IAM에 import
- B) ACM Private CA로 인증서 발급
- C) 공개 CA에서 구매해 IAM에 import
- D) 인증서 없이 HTTP만 사용

**정답:** B
- ACM Private CA는 프라이빗 인증서 관리에 최적화
- 자동 갱신 지원
- 감사(Audit) 추적 가능
- 비용: 월 $400 (프라이빗 CA)
- A는 내부용이라도 권장하지 않음 (보안 취약)

---

### 문제 5
**상황:** ALB를 CloudFormation으로 배포 중이고, SSL 정책을 명시적으로 지정하고 싶다.

**문제:** 현재 가장 권장되는 SSL 정책은?
- A) ELBSecurityPolicy-2016-08
- B) ELBSecurityPolicy-TLS-1-2-2017-01
- C) ELBSecurityPolicy-FS-1-2-Res-2019-08
- D) ELBSecurityPolicy-TLS13-1-2-2021-06

**정답:** D
- TLSv1.3 지원 (최신 기술)
- 전방향 비밀성(Forward Secrecy) 강제
- 성능과 보안의 최적 균형
- C도 좋은 선택지 (TLSv1.3 미포함이지만 강력함)

---

## 의사결정 플로우

```
HTTPS 인증서 선택:

공개 도메인인가?
  ├─ YES → ACM Public 인증서 (무료, 자동 갱신)
  └─ NO  → 프라이빗 도메인인가?
           ├─ YES → ACM Private CA (월 $400)
           └─ NO  → 외부 구매 후 IAM import (수동 관리)

여러 도메인을 한 로드밸런서에서 관리?
  ├─ YES → ALB/NLB + SNI 사용
  │        (CLB는 불가)
  └─ NO  → 단일 도메인 인증서로 충분

자동 갱신이 가능한가?
  ├─ YES → ACM 강력 권장
  └─ NO  → 외부 CA로 수동 관리 (갱신 알림 필수)
```

---

## 빠른 참고: CLI 명령어 모음

```bash
# ACM 인증서 생성 요청
aws acm request-certificate \
  --domain-name example.com \
  --subject-alternative-names www.example.com api.example.com \
  --validation-method DNS

# ACM 인증서 목록
aws acm list-certificates

# 인증서 상세 정보
aws acm describe-certificate --certificate-arn <arn>

# ALB 리스너 생성 (HTTPS)
aws elbv2 create-listener \
  --load-balancer-arn <alb-arn> \
  --protocol HTTPS \
  --port 443 \
  --certificates CertificateArn=<cert-arn> \
  --ssl-policy ELBSecurityPolicy-TLS13-1-2-2021-06 \
  --default-actions Type=forward,TargetGroupArn=<tg-arn>

# HTTP → HTTPS 리다이렉트
aws elbv2 modify-listener \
  --listener-arn <http-listener-arn> \
  --default-actions Type=redirect,RedirectConfig={Protocol=HTTPS,Port=443,StatusCode=HTTP_301}

# SNI 인증서 추가
aws elbv2 add-listener-certificates \
  --listener-arn <https-listener-arn> \
  --certificates CertificateArn=<cert-arn-2> CertificateArn=<cert-arn-3>

# 로드밸런서의 HTTPS 리스너 확인
aws elbv2 describe-listeners \
  --load-balancer-arn <alb-arn> \
  --query 'Listeners[?Protocol==`HTTPS`]'
```

---

## 추가 학습 자료
- [AWS Certificate Manager 공식 문서](https://docs.aws.amazon.com/acm/)
- [ELB HTTPS 설정 가이드](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/listener-update-rules.html)
- [TLS 보안 정책 목록](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-https-listener.html)
- [openssl을 사용한 인증서 검증](https://www.ssl.com/article/how-to-read-an-x-509-certificate/)
- AWS SAA 시험 대비: 시험 전 이 문서의 "개요", "비교표", "보안 모범 사례" 섹션 정독
