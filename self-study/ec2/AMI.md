# AMI (Amazon Machine Image)

## 정의
- AMI는 EC2 인스턴스를 시작하기 위한 템플릿입니다. 일반적으로 루트 볼륨의 스냅샷, 블록 디바이스 매핑, 런치 권한 등이 포함됩니다.
- AMI 유형:
  - 퍼블릭 AMI(공개적으로 사용 가능한 AMI)
  - AWS Marketplace AMI(서드파티/라이선스가 포함된 AMI)
  - 프라이빗(계정 소유) AMI
- 저장 방식 차이: EBS-backed AMI(EBS 볼륨을 루트로 사용, 스냅샷 기반) vs 인스턴스 스토어-backed AMI(임시 스토리지 사용, stop/start 지원 제한).

## 생성 및 자동화
- EC2 콘솔이나 CLI에서 기존 인스턴스로부터 AMI를 생성할 수 있습니다(인스턴스 -> Create Image).
- 이미지 생성 자동화 도구: Packer, EC2 Image Builder 등을 사용하여 일관된 Golden AMI를 만들고 배포합니다.
- 생성 시 유의사항: 불필요한 비밀번호/자격증명 제거, 최신 보안 패치 적용, SSM/CloudInit 설정, 필요 시 KMS로 스냅샷 암호화.

## 공유와 암호화
- 암호화된 AMI를 다른 계정과 공유하려면 두 가지가 필요합니다:
  1) 대상 계정에 대해 AMI Launch Permission을 부여
  2) AMI에 사용된 KMS CMK에 대해 대상 계정이 사용할 수 있도록 키 정책 또는 Grant를 설정
- AWS Organizations를 통해 OU(조직 단위)에 AMI를 공유하거나 런치 권한을 관리할 수 있습니다.
- Marketplace AMI는 라이선스 및 배포 정책 때문에 별도 절차가 필요합니다.

## 스냅샷과 보안
- AMI는 보통 EBS 스냅샷을 포함하며, 스냅샷이 KMS로 암호화되어 있으면 해당 키의 접근 권한을 관리해야 합니다.
- Golden AMI 전략: Packer/EC2 Image Builder로 빌드하고 AWS Inspector, SSM Run Command 등으로 보안 검사 및 설정을 적용한 뒤 배포합니다.

## 제약 및 주의사항
- 인스턴스 스토어-backed AMI로 시작된 인스턴스는 `stop` 및 `start`가 지원되지 않습니다(terminate 후 재실행 필요). 이는 인스턴스 스토어가 휘발성(storage)이기 때문입니다.
- Marketplace AMI는 추가 비용이나 라이선스 제약이 있을 수 있습니다.
- 암호화된 AMI를 공유할 때는 AMI 런치 권한뿐 아니라 KMS 키 권한도 반드시 확인해야 합니다.

## 운영 및 DR(재해복구)
- 리전 간 복제: AWS 콘솔/CLI의 AMI Copy 기능을 사용해 다른 리전으로 AMI를 복사하고 정기적으로 업데이트합니다.
- 백업 전략: AMI 버전 관리(태그/네이밍), 스냅샷 보관 정책, KMS 키 회전 및 접근 제어를 포함합니다.

## 권한 및 런치
- Launch Permissions: AMI를 특정 계정에 공유하거나 퍼블릭으로 설정할 수 있습니다.
- KMS: 암호화된 AMI를 다른 계정에서 사용하려면 해당 계정에 CMK 사용 권한(Decrypt/GenerateDataKey 등)을 부여해야 합니다.

### 연습 문제 (Practice MCQs)
1) An EC2 instance uses an instance-store-backed AMI. Which action is NOT supported?
A. Reboot the instance
B. Terminate the instance and relaunch from the same AMI
C. Stop the instance and later start it
Answer: C. Stop the instance and later start it
Explanation: 인스턴스 스토어-backed 인스턴스는 `stop`/`start`를 지원하지 않습니다. 인스턴스 스토어는 휘발성(storage)이므로 인스턴스를 중지하면 데이터가 유지되지 않거나 동작이 제한됩니다.

2) You need to share an encrypted AMI with another AWS account. Which two permissions are required for the target account to launch it?
A. S3 object permission and KMS key permission
B. AMI launch permission and KMS key permission
C. AMI launch permission only
Answer: B. AMI launch permission and KMS key permission
Explanation: 암호화된 AMI를 다른 계정에서 런치하려면 대상 계정에 AMI Launch Permission을 부여해야 하며, 동시에 AMI(또는 스냅샷)에 사용된 KMS CMK에 대한 사용 권한도 부여해야 합니다(키 정책이나 Grant를 통해).

3) A company wants a disaster recovery copy of its hardened "golden" AMI in another Region. What is the simplest AWS-native approach?
A. Copy the AMI to the target Region and keep it updated on a schedule
B. Export the AMI to S3, replicate the bucket, and import in the DR Region
C. Rebuild the AMI from scratch in the DR Region each time using EC2 Image Builder
Answer: A. Copy the AMI to the target Region and keep it updated on a schedule
Explanation: AWS는 AMI 복사 기능을 제공하므로 리전 간 AMI 복사를 사용해 간단히 DR용 이미지를 준비하고 주기적으로 최신화하는 것이 가장 간단하고 관리하기 쉬운 방법입니다.
