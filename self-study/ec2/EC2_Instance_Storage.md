# EC2 Instance Storage

## EBS Overview
- EBS는 AZ 내에서만 접근 가능한 네트워크 블록 스토리지로 내구성(99.999%)을 제공하며, 한 볼륨은 기본적으로 한 인스턴스에만 연결된다.
- **루트 볼륨은 기본적으로 종료 시 삭제(Delete on termination)**되며, 데이터 유지가 필요하면 속성을 해제하거나 추가 데이터 볼륨을 사용한다.
- 볼륨 크기/타입/IOPS는 온라인으로 조정 가능하며, 변경 후 성능 극대화를 위해 파일시스템 레벨 최적화가 필요할 수 있다.
- gp3는 기본 3,000 IOPS/125 MBps를 제공하고 IOPS와 처리량을 독립적으로 증설할 수 있어 범용 워크로드의 기본 선택지다.
- 고성능·저지연 워크로드는 io1/io2(프로비저닝 IOPS) 또는 io2 Block Express를 사용하고, 대역폭 중심 워크로드는 st1/sc1을 고려한다.

#### Multiple Choice
You need shared file storage that supports concurrent read/write across multiple AZs. What should you choose?

- A. Attach a gp3 EBS volume to two instances with Multi-Attach
- B. Amazon EFS file system
- C. RAID 0 on instance store
- D. Replicate a gp2 EBS snapshot to S3
Answer: B
해설: 멀티 AZ 동시 읽기/쓰기는 블록 스토리지로 불가하고, 관리형 다중 AZ NFS는 EFS만 제공한다.

## EBS Snapshots
- 스냅샷은 S3에 증분 저장되며, 생성 중에도 볼륨을 사용할 수 있지만 애플리케이션 정합성은 스톱/플러시 또는 멀티 볼륨 스냅샷으로 확보한다.
- 스냅샷을 복사(copy)하여 다른 리전/계정으로 이동할 수 있으며, 암호화 상태는 복사 시점에 재정의 가능하다.
- Fast Snapshot Restore(FSR)는 AZ 단위로 사전 워밍하여 첫 블록 액세스 지연을 없앤다.
- Recycle Bin을 통해 스냅샷 보존 기간을 정책화하고 실수 삭제를 방지할 수 있다.
- DLM(Data Lifecycle Manager) 또는 AWS Backup으로 스냅샷 생성/보존을 자동화한다.

#### Multiple Choice
You must create weekly snapshots with a 30-day retention policy and guard against accidental deletion. What is the best approach?

- A. Use only multi-volume snapshots
- B. Schedule with DLM and apply a Recycle Bin policy
- C. Manually copy each snapshot to another Region every time
- D. EBS automatic backups are impossible
Answer: B
해설: DLM이 주기 생성·보존을 자동화하고 Recycle Bin이 실수 삭제를 막아 요구사항을 모두 충족한다.

## AMI
- AMI는 루트 볼륨 스냅샷, 블록 디바이스 매핑, 런치 권한, 사용자 메타데이터로 구성된다.
- 퍼블릭/프라이빗/계정 공유 형태로 배포할 수 있으며, 커스텀 KMS 키로 암호화된 스냅샷을 포함할 수 있다.
- AMI 복사를 통해 리전 간 마이그레이션이 가능하며, 복사 시 KMS 키 재암호화 및 퍼미션 재설정이 필요하다.
- 최신 패치나 에이전트가 적용된 골든 이미지를 주기적으로 재생성하고, 오래된 AMI는 정리한다.

#### Multiple Choice
You need to distribute an encrypted custom AMI to a different account in another Region. What is the correct sequence?

- A. Make the AMI public, then copy it
- B. Copy the snapshot while re-encrypting with the target Region KMS key, create an AMI there, and share that AMI with the account
- C. Copy only the root volume and it shares automatically
- D. Sharing works only within the same Region
Answer: B
해설: 교차 리전 암호화 스냅샷 복사 후 대상 리전 KMS 키로 재암호화하고, 그 스냅샷으로 만든 AMI를 공유해야 한다.

## EC2 Instance Store
- 인스턴스 스토어는 호스트에 직접 연결된 NVMe/SATA 일시적 스토리지로, 스톱/테미네이트/호스트 장애 시 데이터가 사라진다.
- 고성능 임시 캐시, 버퍼, 스크래치 공간, 분산 스토리지 레이어(예: 캐시 샤드)에 적합하다.
- 재부팅에서는 데이터가 유지될 수 있으나 보장되지 않으므로 중요한 데이터는 EBS/EFS/S3로 내보내 백업한다.
- 스냅샷이나 멀티 AZ 복제 기능이 없으므로 애플리케이션 레벨 복제가 필요하다.

#### Multiple Choice
You must keep data after stopping an instance. What should you choose?

- A. Instance store
- B. EBS volume
- C. In-memory cache
- D. Single-AZ EFS One Zone-IA
> Answer: B

해설: Instance Store는 스톱/테미네이트 시 데이터가 사라지므로, **종료 후 보존하려면 EBS가 필요**하다.

## [EBS Volume Types](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html)
- gp3: 기본 3,000 IOPS/125 MBps, 최대 16,000 IOPS/1,000 MBps, IOPS와 처리량 독립 구성.
- gp2: 용량에 비례(3 IOPS/GB)하여 최대 16,000 IOPS, 버스트 크레딧 모델.
- io1/io2: 프로비저닝 IOPS 최대 64,000(16 KiB I/O 기준), 저지연·일관된 성능, 멀티 어태치 지원.
- io2 Block Express: 최대 256,000 IOPS/4,000 MBps, 99.999% 내구성, Nitro 전용, 초고성능 OLTP/HPC용.
- st1/sc1: HDD 기반 처리량 최적화(최대 500 MBps) / 콜드 HDD(최대 250 MBps), 부팅 볼륨 불가.

#### Multiple Choice
**Which volume type best fits a transactional database that requires 64,000 steady IOPS?**
- A. gp3
- B. st1
- C. io2
- D. sc1
> Answer: C

해설: 64K IOPS를 일관되게 제공하려면 프로비저닝 IOPS SSD가 필요하며 io2가 내구성과 지연 측면에서 가장 적합하다.

## EBS IOPS 성능
- IOPS는 볼륨 타입·크기·프로비저닝 값(io1/io2)·I/O 크기(16 KiB 기준)·큐 깊이·EBS-optimized 네트워킹 여부에 의해 결정된다.
- gp3는 크기와 무관하게 기본 3,000 IOPS/125 MBps를 제공하고, IOPS/처리량을 별도로 확장한다.
- gp2는 용량에 따라 IOPS가 증가(3 IOPS/GB)하며 버스트 크레딧 소진 시 성능이 떨어질 수 있다.
- io1/io2는 프로비저닝한 IOPS를 일관되게 제공하며, 처리량 한도(볼륨당 최대 1,000 MBps 또는 Block Express 4,000 MBps)를 고려해야 한다.
- 워크로드는 초당 요청 수와 I/O 크기를 기준으로 필요한 IOPS·처리량을 계산하고, 인스턴스가 지원하는 최대 EBS 대역폭을 확인한다.

#### Multiple Choice
An application issues 16 KiB reads at 40,000 IOPS and needs consistent latency. Which option is the best fit?

- A. gp3 volume at 1 TB with default settings
- B. gp2 volume at 5 TB relying on burst credits
- C. io2 volume provisioned for 40,000 IOPS on a Nitro instance
- D. st1 throughput-optimized HDD volume
> Answer: C

해설: 버스트 의존(gp2)이나 HDD(st1)는 일관 지연을 못 주며, 프로비저닝 IOPS SSD(io2)+Nitro만 40K IOPS를 안정 제공한다.

## 32,000+ IOPS와 Nitro 필요성 이해
- io1/io2 볼륨은 프로비저닝 IOPS로 일관된 성능을 제공하지만, **32,000 IOPS를 초과하려면 Nitro 기반 인스턴스가 필수**다. Nitro는 전용 하드웨어 오프로드(EBS-optimized 네트워크 포함)를 제공해 높은 IOPS/처리량을 뒷받침한다.
- Nitro가 아닌 구세대 인스턴스는 EBS 전용 대역폭과 I/O 경로가 제한적이라 32K IOPS 이상 성능을 내지 못하거나 지연이 커질 수 있다.
- io2 Block Express는 Nitro 전용이며 최대 256K IOPS까지 확장 가능하다. Nitro 상에서도 인스턴스 타입별 EBS 대역폭 한도를 확인해야 한다.
- 실무 팁: 필요한 IOPS 계산(초당 요청 수 × 16 KiB 기준) 후, io1/io2 프로비저닝 값과 인스턴스의 EBS 대역폭/멀티큐 지원 여부를 함께 맞춘다.

#### Multiple Choice
You need more than 32,000 consistent IOPS from an EBS volume. What is required?

- A. Any gp3 volume with burst credits
- B. io1/io2 on a non-Nitro instance
- C. io1/io2 on a Nitro-based instance to exceed 32K IOPS
- D. st1 volume with provisioned throughput
> Answer: C

해설: 32K IOPS 초과는 Nitro 하드웨어 대역폭이 필수이고, io1/io2 프로비저닝으로만 달성할 수 있다.

## EBS Multi-Attach
- 실제 활용 사례:
  - Windows Failover Cluster에서 공유 디스크(quorum/witness)로 사용해 노드 장애 시 즉시 페일오버
  - RHEL Pacemaker/Corosync + GFS2로 다중 노드가 동일 블록 디바이스에 동시 접근하는 HA 파일시스템
  - 컨테이너 오케스트레이션(예: Kubernetes DaemonSet)에서 노드별 동일 블록 데이터를 동시에 읽기/쓰기
  - 대용량 로그/큐를 공유해야 하는 액티브-액티브 애플리케이션의 저지연 공용 저널 디스크
**io1/io2(일반 및 Block Express 제외) 볼륨**을 **동일 AZ 내 최대 16개** Nitro 기반 EC2에 동시에 연결할 수 있다.
- 루트 볼륨은 멀티 어태치를 지원하지 않으며, 클러스터 인식 파일시스템(GFS2, OCFS2 등) 또는 애플리케이션 레벨 락 관리가 필요하다.
- 장애 조치나 분산 데이터베이스(예: Windows Failover Cluster)에서 공유 블록 스토리지로 활용한다.
- 각 인스턴스는 동일 AZ에 있어야 하며, I/O 충돌을 막기 위한 락·펜싱 전략이 필수다.

#### Multiple Choice
Several instances must read/write concurrently to the same block device. What is the correct option?

- A. gp3 volume with Multi-Attach
- B. io2 volume with Multi-Attach plus a cluster-aware file system
- C. st1 volume with a self-managed NFS server
- D. Instance store RAID
> Answer: B

해설: 멀티 어태치는 io1/io2만 지원하며, 동시 쓰기엔 클러스터 파일시스템 같은 락 관리가 필요하다.

## EBS Encryption
- 언제 사용하나: 기본값으로 켜서 at-rest 보호, 규제·컴플라이언스 충족, 교차 계정·리전 공유 시 키 정책 제어, 내부자 위협 대비 키 로테이션, 스냅샷/AMI를 안전하게 보관·이동할 때.
- EBS 암호화는 KMS(AES-256)를 사용하며, 데이터 플레인(디스크·스냅샷·복제)과 전송(TLS) 모두 암호화된다.
- 기본 EBS 암호화를 **리전 단위로 설정**할 수 있으며, 스냅샷/AMI/볼륨 생성 시 자동 적용된다.
- 암호화된 스냅샷을 다른 계정과 공유하려면 **CMK 키 정책과 스냅샷 퍼미션 모두를 허용**해야 한다.
- 비암호화 스냅샷을 복사하면서 암호화하거나, 암호화 스냅샷을 비암호화로 되돌릴 수는 없다.
- CloudTrail로 KMS API 호출을 감사하고, 데이터 접근 로깅은 CloudWatch Logs/OS 레벨에서 별도 관리한다.

#### Multiple Choice
You need to share an encrypted snapshot with another account. What must you do?

- A. Set the snapshot to public only
- B. Share the snapshot and add the target account to the CMK key policy
- C. First copy the snapshot unencrypted, then share it
- D. Convert to an AMI before sharing

> Answer: B

해설: 스냅샷 공유 권한과 함께 KMS 키 정책에 대상 계정을 추가해야 타 계정이 복호화해 쓸 수 있다.

## Amazon EFS
- 완전관리형 NFSv4 파일시스템으로 **다중 AZ에서 동시에 마운트 가능**하며, 필요 용량만큼 자동 확장/축소된다.
- 스토리지 클래스: Standard(다중 AZ), Standard-IA, One Zone, One Zone-IA. 라이프사이클 정책으로 IA 전환 자동화.
- 성능 모드(Performance Mode): 
    - 역할: "어떻게" I/O를 처리할 것인가
    - 영향: Latency, 동시연결 처리 방식
    - 모드:
        - General Purpose(지연 최적. web server, CMS)
        - Max I/O(대규모 병렬. big data, media processing). 
- 처리량 모드(Throughput Mode):
  - 역할: "얼마나 많은" 데이터를 처리할 것인가
  - 영향: 처리량(MB/s) 제공 방식
  - 모드: 
    - **Bursting(버스팅)**: 기본 모드. 파일 시스템 크기에 비례해 처리량이 정해지며, 크레딧을 사용해 트래픽 급증에 대응. 평소 사용량이 적은 워크로드(예: 웹 서버, 개발 환경)에 경제적.
    - **Provisioned(프로비저닝)**: 파일 시스템 크기와 무관하게 필요한 처리량(MB/s)을 직접 지정. 데이터 분석, 미디어 처리 등 지속적으로 높고 일관된 성능이 필요한 워크로드에 적합.
    - **Elastic** - 워크로드에 기반하여 자동으로 스케일 스루풋을 올리거나 낮춰줌
- EFS Access Points로 애플리케이션별 루트 디렉터리/권한을 강제할 수 있다.
- 백업은 AWS Backup으로 스케줄링하며, 전송 시 TLS를 사용한다.
- Linux based AMI만 호환(not Windows).
- **POSIX 호환 파일 시스템**: 표준 파일 API를 제공하여 Linux 애플리케이션과 완벽하게 호환된다.
### EFS Storage Class
- **EFS Standard vs. EFS Infrequent Access(IA)**: Standard는 자주 액세스하는 파일용, IA는 장기 보관용으로 스토리지 비용이 저렴하지만 액세스 시 비용이 발생합니다.
- **수명 주기 관리(Lifecycle Management)**: 설정된 기간(예: 30일) 동안 액세스하지 않은 파일을 Standard에서 IA로 자동 이전하여 비용을 최적화하는 정책입니다.
- **One Zone 클래스**: Standard(다중 AZ)와 달리 단일 AZ에만 데이터를 저장하여 비용이 더 저렴하지만, AZ 장애 시 데이터 유실 위험이 있어 백업이 중요합니다. (One Zone / One Zone-IA)

#### Multiple Choice
Multiple EC2 instances in different AZs must access the same files concurrently with a managed service. What should you choose?

- A. io2 Multi-Attach
- B. EFS Standard
- C. st1 volume directly shared over NFS
- D. Instance store RAID 0
> Answer: B

해설: 다중 AZ 동시 파일 접근을 관리형으로 제공하는 서비스는 EFS뿐이며, 블록 멀티 어태치는 AZ를 넘지 못한다.

## EFS vs EBS
- EBS: AZ 한정, 블록 스토리지, 단일 인스턴스 연결(멀티 어태치 예외), 짧은 지연과 안정된 IOPS 제공.
- EFS: 리전/다중 AZ 가용성, 다수 인스턴스 동시 마운트, 공유 파일 락킹 제공, 용량 자동 확장.
- 비용: EFS는 GB당 비용이 높지만 관리형 다중 AZ를 제공하며, One Zone/IA로 최적화 가능. EBS는 용량 예약형이므로 미사용 공간도 과금.
- 사용 사례: EBS는 데이터베이스, 부팅, 로컬 스토리지에 적합; EFS는 웹/컨테이너 공유 파일, 홈 디렉터리, ML 데이터셋 공유에 적합.

#### Multiple Choice
Several ECS tasks must share the same dataset and remain available during an AZ failure. What should you choose?

- A. gp3 EBS with Multi-Attach
- B. EFS Standard with an Access Point
- C. st1 EBS with a self-managed NFS server
- D. Instance store with periodic sync to S3
> Answer: B

해설: EFS Standard는 다중 AZ 내구성과 공유 파일 접근을 제공하고, Access Point로 태스크별 경로/권한을 분리할 수 있다.

## EBS & EFS - Section Cleanup
- 비용/보안/보존 정책을 명확히: 기본 EBS 암호화, 스냅샷 Recycle Bin, DLM/Backup로 자동 생성·정리.
- 워크로드 특성별 볼륨 타입과 파일시스템/락 전략을 사전에 표준화하여 오용을 방지한다.
- 액세스 제어는 IAM(예: EFS Access Point POSIX 매핑, KMS 키 정책)과 네트워크(SG/NACL)로 이중화한다.
- 모니터링: CloudWatch 지표(볼륨 지연, Burst Balance, EFS 퍼포먼스 모드)와 FS-level 용량 알람을 설정한다.

#### Multiple Choice
How do you automatically create and clean up snapshots so they do not grow without limit?

- A. Manually delete old snapshots
- B. Use DLM or AWS Backup with retention policies
- C. Rely only on EFS lifecycle policies
- D. Change AMI sharing settings

> Answer: B

해설: DLM/AWS Backup이 생성·보존 정책을 자동 적용해 스냅샷 증가를 제어하며, 수동 삭제나 EFS 정책만으로는 부족하다.

## EC2 Data Management
- 데이터 보호: EBS 스냅샷/DLM, AWS Backup, AMI 주기화, 교차 리전 복사로 DR 구현.
- 구성 관리: User Data/클라우드이니트/SSM State Manager로 부팅 시 설정을 자동화하고, 애플리케이션 데이터를 외부 스토리지(EBS/EFS/S3)로 분리한다.
- 복구 전략: Instance Store 사용 시 종료 전 S3/EBS로 플러시, 멀티 AZ 아키텍처에서는 EFS/FSx 또는 애플리케이션 레벨 복제를 고려.
- 성능 튜닝: Nitro 기반 ENA, EBS-optimized 인스턴스, 파일시스템 옵션(noatime, 큐 깊이 관리)으로 지연을 최소화한다.

#### Multiple Choice
An instance-store-backed cache node fails and must be replaced quickly with data rebuilt. What is the best approach?

- A. Back up the instance store with snapshots
- B. Treat cache data as ephemeral and rebuild on boot via User Data/SSM from a source (S3/EBS)
- C. Stop the instance because data persists automatically
- D. Use Multi-Attach so two instances share the same cache
> Answer: B

해설: Instance Store는 백업이 불가하므로 캐시를 휘발성으로 보고 부팅 시 스크립트로 원본에서 재구성하는 것이 가장 빠르다.
