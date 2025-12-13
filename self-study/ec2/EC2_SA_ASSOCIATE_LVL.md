# EC2 - Solutions Architect Associate Level
> Networking essentials (IP types, Placement Groups, Hibernate, ENI) with exam-oriented notes.

## Table of Contents
1. [Private vs. Public vs. Elastic IP](#private-vs-public-vs-elastic-ip)
2. [EC2 Placement Groups](#ec2-placement-groups)
3. [EC2 Hibernate](#ec2-hibernate)
4. [Elastic Network Interfaces (ENI)](#elastic-network-interfaces-eni)
   - [Overview](#overview)
   - [Hands On](#hands-on)
   - [Extra Reading](#extra-reading)

---

## Private vs. Public vs. Elastic IP
- **Private IP**: Lives inside VPC/subnet CIDR. One primary (sticks across stop/start) plus optional secondary addresses.
- **Public IP**: Internet-facing. Only useful if the subnet routes to an Internet Gateway. Assigned on launch; changes on stop/start unless you use an EIP.
- **Elastic IP (EIP)**: Static public IPv4 that you can remap to another ENI/instance for failover. Charged when allocated but not attached.
- **Exam tips**
  - Need fixed IP for DNS or partners -> use EIP and map with an A record.
  - Preserve address across stop/start -> private IP persists; public IP does not (use EIP).
  - Outbound-only from private subnets -> NAT (instance or gateway) with one EIP serving many instances.
  - No internet but need S3/DynamoDB -> VPC Endpoints; no EIP needed and cheaper traffic.
> **Callout - When to skip EIP**: Route53 + managed endpoints (ALB/NLB/CloudFront) are better when you just need a stable name and higher availability. Use EIP only when a fixed public IP is required (partner whitelists, legacy clients, VPN/firewall rules). Keep DNS TTL short if you rely on DNS-based failover without EIP.

## EC2 Placement Groups
- **Purpose**: Control latency/bandwidth and failure domains between instances.
- **Types**
  - **Cluster**: Packs instances in one AZ/rack set. Ultra-low latency/high throughput (HPC, distributed DB). Blast radius is larger.
  - **Spread**: Places instances on distinct racks (limit 7 per AZ). Use for a small set of critical nodes (masters, key services).
  - **Partition**: Splits into partitions with isolated racks per partition. Suits tens/hundreds of nodes (Hadoop, HBase, Cassandra). You distribute instances across partitions.
- **Constraints and tips**
  - Cluster is single-AZ; capacity failures possible. Mitigate with multiple instance types or mixed-policy ASG.
  - Spread: remember the 7-per-AZ limit.
  - Partition: up to 7 partitions per AZ; know your partition-to-node mapping.
  - For HPC, add EFA (Elastic Fabric Adapter) to cut latency further.
- **선택 사례 (언제 어떤 타입?)**
  - Cluster: 노드 간 초저지연이 최우선일 때 (HPC, 인메모리 분산 캐시, 분산 DB 리더-팔로워, 실시간 게임 매치 서버). AZ 단일 배치라 장애 반경 커짐을 감수.
  - Spread: 소수(<=7대/AZ)의 중요 노드가 동시에 죽으면 안 될 때 (마스터 노드, 컨트롤 플레인, 로그 수집 허브, Jump/Bastion). 서로 다른 랙에 흩어져 장애 격리 극대화. (Spread Placement Group places your EC2 instances on different physical hardware across different AZs.)
  - Partition: 다수 노드를 사용하지만 랙/전력 도메인별 격리가 필요한 분산 시스템 (Hadoop/HBase/Cassandra/ElasticSearch, 대규모 Kafka 브로커). 파티션 수는 AZ당 최대 7, 노드를 파티션에 직접 배분.
  - 용량 확보가 우선일 때: Mixed Instances ASG + Partition/Spread로 다중 타입을 섞어 용량 부족 시 실패를 줄이고, Cluster는 용량 부족 실패에 대비해 롤백/재시도 로직 필요.

## EC2 Hibernate
- **개념**: 인스턴스 메모리(RAM) 상태를 루트 EBS에 저장하고 전원을 끄는 모드. 다시 켤 때 OS 부팅을 건너뛰고 메모리 상태를 복원해 빠르게 재개(노트북 절전 모드와 유사).
- **동작 흐름**
  1) Hibernate 요청 -> 메모리 덤프를 암호화된 루트 EBS에 저장
  2) 인스턴스가 stopped 상태로 전환(루트/데이터 EBS는 유지, 인스턴스 스토어는 삭제)
  3) Start 시 메모리 복원 후 애플리케이션 상태 그대로 재개
- **필수 조건**
  - 지원 인스턴스/AMI: Nitro 기반 최신 인스턴스와 Amazon Linux 2/AL2023/Ubuntu/일부 Windows. 지원되지 않는 AMI면 Hibernate 불가.
  - 루트 EBS 크기: RAM보다 충분히 커야 메모리 덤프 저장 가능.
  - 루트 EBS는 암호화 필수(KMS 기본키 가능).
  - 인스턴스 스토어 데이터는 보존되지 않으므로 캐시/임시 용도로만 사용.
- **유지/변경되는 것**
  - 유지: 메모리 상태, 루트/데이터 EBS 내용, 인스턴스 ID, Private IP.
  - 변경 가능: Public IP(Elastic IP면 유지).
- **비용**: 중지 동안 EC2 요금은 없고, EBS 스토리지 비용만 청구.
- **언제 유용한가**
  - 애플리케이션 웜업 시간이 길어 Stop/Start보다 더 빠른 복귀가 필요할 때
  - 세션/캐시를 메모리에 들고 있어 즉시 처리 재개가 필요할 때
  - 야간/주말에 내려뒀다가 빠르게 올려야 하는 개발/배치 워크로드
- **시험 포인트**
  - 메모리 상태 복원이 필요하면 Hibernate, 단순 Stop/Start는 메모리 상태를 잃음
  - IP 유지가 필요하면 EIP 또는 고정 Private IP 설계
  - 인스턴스 스토어 데이터는 보존되지 않음을 기억
> **콜아웃 - 지원 요금제**: Hibernate는 On-Demand, Reserved(또는 Savings Plans로 과금되는 온디맨드 실행)에서만 지원됩니다. Spot 인스턴스는 Hibernate를 지원하지 않습니다.
> **콜아웃 - 최대 유지 기간**: Hibernate 상태는 최대 60일까지만 유지됩니다. 60일을 넘기면 다시 시작할 수 없으므로 주기적으로 기동/정지하거나 다른 방식으로 상태를 저장하세요.

## Elastic Network Interfaces (ENI)
### Overview
- **What**: Virtual NIC in a VPC. Holds private IPs (primary plus secondary), optional public/EIP, security groups, MAC.
- **Attach model**: Primary ENI is bound to the instance; secondary ENIs can be detached and moved (same AZ only).
- **Use cases**
  - **Failover**: Move a secondary ENI to a standby instance to keep the same IP/SG and recover quickly.
  - **Traffic separation**: Split management vs data, or front/back of a virtual appliance on different ENIs.
  - **Multiple SG sets**: Apply different security-group combinations per ENI.
> **콜아웃 - ENI 장애 조치**: 같은 AZ에서 보조 ENI를 떼어 새 EC2에 붙이면 그 ENI의 사설 IP/EIP/보안그룹이 그대로 따라옵니다. 기본 ENI는 인스턴스에 묶여 있으니, 서비스용 IP/SG는 보조 ENI에 두고 장애 시 그 보조 ENI를 옮기는 전략을 쓰세요. ENI를 옮긴 뒤 OS에서 추가 IP를 인터페이스에 올려야 트래픽이 흐릅니다.
- **Performance**: Nitro plus ENA provides high bandwidth/low latency. Check per-instance-type limits for ENIs and IPs.

### Hands On
1) Create ENI: choose VPC/subnet, security groups, primary/secondary private IPs.
2) (Optional) Associate EIP to a chosen private IP on the ENI for a static public address.
3) Attach to instance: pick an instance in the same AZ; it becomes a secondary ENI.
4) Failover drill: detach ENI from the failed node -> attach to a healthy node -> service resumes with identical IP/SG.
5) Add secondary IPs: assign in the console/CLI, then configure inside the OS (for example, ip addr add 10.0.1.50/24 dev eth0).

### Extra Reading
- **Quotas**: ENIs, private IPs per ENI, and SG associations vary by instance type; request limit increases if needed.
- **Trunk/branch ENI**: Nitro-only advanced pattern (VPC sharing/outbound gateway); rarely tested at Associate level.
- **IMDSv2**: Enable token-based metadata to prevent SSRF/credential theft.
- **Auditing**: ENI moves show in CloudTrail (AttachNetworkInterface, DetachNetworkInterface).
