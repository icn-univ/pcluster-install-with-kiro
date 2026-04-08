# pcluster-agentic-setup 프로젝트 계획

## 목표
연구자들이 Kiro와 한국어로 대화하면서 AWS ParallelCluster를 쉽게 배포할 수 있도록 한다.
연구자는 AWS 용어나 Shell 명령어를 몰라도 되며, Kiro가 AWS 리소스 조회, config.yaml 생성, 실제 배포까지 대신 처리한다.

## 구현 방식
Kiro Steering 파일 (`.kiro/steering/`) 에 지침을 작성하여,
연구자가 Kiro를 열면 자동으로 이 맥락을 가지고 동작하도록 한다.

## 대상 사용자
- 대학 연구실 연구자 (각자 본인 AWS 계정 사용)
- AWS/Shell 경험이 적은 비개발자

## Kiro 대화 흐름

### 시작 메시지
```
Kiro: AWS Parallel Cluster를 생성할 준비가 되었습니다. 어떤 HPC 클러스터가 필요하신가요?

1. 🚀 입문용 클러스터 (어느 과든 부담 없이 테스트)
   - 용도: 코드 테스트, 소규모 병렬 계산, 수업용
   - 관리 서버(Head Node): t3.medium
   - 연산 서버(Compute): c7i.xlarge × 최대 4대
   - 공유 저장소: EFS (용량 자동 확장)
   - 비용: 가장 저렴

2. 🔧 전산 시뮬레이션 클러스터 (기계공학, 재료공학, 화학공학, 토목 등)
   - 용도: FEM, CFD (ANSYS, OpenFOAM, LAMMPS 등)
   - 관리 서버(Head Node): c7i.xlarge
   - 연산 서버(Compute): c8i.48xlarge × 최대 8대
   - 공유 저장소: FSx for Lustre 1.2TB (고성능 파일시스템)
   - 비용: 중~고

3. 🤖 AI/딥러닝 클러스터 (반도체공학, 전기전자, 컴퓨터공학, 바이오 등)
   - 용도: 모델 학습, 데이터 분석
   - 관리 서버(Head Node): c7i.xlarge
   - 연산 서버(Compute): g6.xlarge × 최대 4대
   - 공유 저장소: FSx for Lustre 1.2TB (고성능 파일시스템)
   - 비용: 중~고

4. 🛠️ 커스텀 클러스터 — 직접 설정
```

### 프리셋 클러스터 (1~3번 선택 시)
클러스터 이름만 입력받고 바로 생성.

| # | 이름 | 용도 | Head Node | Compute | Storage | 비용 |
|---|------|------|-----------|---------|---------|------|
| 1 | 입문용 | 코드 테스트, 소규모 병렬 계산, 수업용 | t3.medium | c7i.xlarge × 최대 4노드 | EFS (용량 자동 확장) | 가장 저렴 |
| 2 | 전산 시뮬레이션 | FEM, CFD (ANSYS, OpenFOAM, LAMMPS 등) | c7i.xlarge | c8i.48xlarge × 최대 8노드 | FSx for Lustre 1.2TB | 중~고 |
| 3 | AI/딥러닝 | 모델 학습, 데이터 분석 | c7i.xlarge | g6.xlarge × 최대 4노드 | FSx for Lustre 1.2TB | 중~고 |

### 커스텀 클러스터 (4번 선택 시)
아래 질문을 순서대로 한국어로 진행:
1. 클러스터 이름
2. Head Node 인스턴스 타입:
   ```
   번호  Head Node
   1.   t3.medium (가벼운 작업, 저렴)
   2.   c7i.xlarge (중간 규모) — 기본값
   3.   직접 입력
   ```
3. Compute Node 인스턴스 타입 — 아래 표를 보여주고 번호 선택 or 직접 입력:
   ```
   번호  용도                              추천 인스턴스     특징
   1.   분자동역학, 유체역학, CFD, FEM     c8i.48xlarge    고성능 CPU, HPC 최적화
   2.   딥러닝, AI, 모델 학습              g6.xlarge       GPU
   3.   유전체, 바이오인포매틱스            r7i.48xlarge    대용량 메모리
   4.   기후/날씨 시뮬레이션, 천문          hpc7a.48xlarge  AMD HPC 최적화, EFA (서울 리전 불가)
   5.   반도체 시뮬레이션 (TCAD 등)        c8i.48xlarge    고성능 CPU, 대용량 메모리
   6.   일반/기타                          c7i.xlarge      범용

   번호를 선택하거나 인스턴스 타입을 직접 입력하세요. (예: p4d.24xlarge)
   ```
   - 선택 후 현재 리전에서 해당 인스턴스 가용 여부 확인
   - 미지원 시:
   ```
   ⚠️  {인스턴스}는 {현재 리전} 리전에서 지원되지 않습니다.
       이 인스턴스는 {지원 리전} 리전에서 사용 가능합니다.

       1. 다른 인스턴스 타입 선택
       2. 리전을 {지원 리전}으로 변경 후 다시 시작
   ```
4. Compute Node 최대 수:
   ```
   연산 서버 최대 수를 입력하세요:
   (동시에 사용할 연산 서버의 최대 수입니다. 사용하지 않을 때는 자동으로 꺼집니다.)
   [4] (Enter로 기본값 사용)
   ```
5. 공유 스토리지:
   ```
   번호  스토리지
   1.   EFS (용량 자동 확장, 저렴) — 기본값
   2.   FSx for Lustre (고성능 I/O)
   ```
   - FSx 선택 시 용량 질문:
   ```
   스토리지 용량을 입력하세요 (GB 단위, 최소 1200):
   [1200] (Enter로 기본값 사용)
   ```
6. 고속 네트워크(EFA):
   ```
   고속 네트워크(EFA)를 사용하시겠습니까?
   (노드 간 병렬 통신이 필요한 경우 권장: CFD, FEM, 분자동역학 등)
   1. 예 (EFA + PlacementGroup 활성화)
   2. 아니오 — 기본값
   ```
7. 원격 데스크톱(DCV):
   ```
   웹 브라우저로 GUI 접속이 필요하신가요?
   (시각화, 그래픽 작업 등에 유용합니다)
   1. 예 (DCV 활성화)
   2. 아니오 — 기본값
   ```
8. OS 선택:
   ```
   번호  OS
   1.   Amazon Linux 2 (기본값)
   2.   Amazon Linux 2023
   3.   Ubuntu 22.04
   4.   Ubuntu 24.04
   5.   RHEL 8
   6.   Rocky 8
   7.   RHEL 9
   8.   Rocky 9
   ```

### 배포 흐름
클러스터 번호와 이름을 입력받으면 아래 메시지와 함께 단계별로 진행:

```
Kiro: [클러스터명] "[이름]" 설치를 시작합니다. 🚀

아래 단계로 진행됩니다:
[1/6] AWS 계정 연결 확인
[2/6] pcluster CLI 설치 확인
[3/6] SSH 접속 키 확인
[4/6] 네트워크 생성
[5/6] 클러스터 설정 파일 생성
[6/6] 클러스터 배포
```

각 단계 상세:

- **[1/6] AWS 계정 연결 확인** — `aws sts get-caller-identity` 로 계정 ID, 사용자, 리전 확인 후 아래 형식으로 안내. 실패 시 자격증명 설정 방법 안내 후 중단.
  ```
  ✅ 완료
     - 계정 ID: 790932119007
     - 사용자:   layzner
     - 리전:     ap-northeast-2 (서울)

  위 계정의 서울 리전에 클러스터를 생성합니다. 계속할까요? (Y/n)
  ```
  - N 선택 시 — Kiro가 리전 변경을 대신 처리:
  ```
  어떤 리전을 사용하시겠습니까?

  번호  리전
  1.   서울 (ap-northeast-2)
  2.   미국 동부 - 버지니아 (us-east-1)
  3.   미국 동부 - 오하이오 (us-east-2)
  4.   미국 서부 - 오레곤 (us-west-2)
  5.   직접 입력
  ```
  선택 후 `export AWS_DEFAULT_REGION=<리전코드>` 를 Kiro가 자동 실행하여 현재 세션의 리전 변경 (`~/.aws/config` 영구 변경 방지).

- **[2/6] pcluster CLI 확인** — `pcluster version` 으로 설치 여부 확인.
  - 설치되어 있으면 → 버전 확인 후 ✅ 완료
  - 미설치 시 → 버전 선택 후 설치:
    ```
    pcluster가 설치되어 있지 않습니다. 어떤 버전을 설치할까요?

    1. 3.14.0
    2. 3.15.0
    3. 최신 버전
    ```
    선택한 버전으로 `pip3 install aws-parallelcluster==<version>` 실행.

- **[3/6] SSH 접속 키 확인** — 해당 리전의 키페어 목록 조회 후 번호로 선택.
  - 키페어 있으면 → 목록 보여주고 선택
  - 키페어 없으면 → 자동 생성:
    - 키페어 이름 입력 (기본값: `{클러스터명}-key`)
    - ⚠️ 안내: "생성된 .pem 파일은 ./{이름}.pem 에 저장됩니다. 이 파일을 잃어버리면 클러스터에 접속할 수 없으니 안전한 곳에 보관하세요."
    - `chmod 400` 자동 적용

- **[4/6] 네트워크 생성**
  - 💡 모범 사례: "Head Node는 public 서브넷, Compute Node는 private 서브넷에 배치합니다."
  - **프리셋(1~3번)**: HPC 전용 VPC와 서브넷을 자동 생성
    - VPC: `10.0.0.0/16` (`hpc-{클러스터명}-vpc`)
    - public 서브넷: `10.0.0.0/24` (Head Node용)
    - private 서브넷: `10.0.16.0/20` (Compute Node용)
    - CloudFormation 템플릿(`sample-vpc-subnet.json`)을 사용하여 배포:
      ```bash
      aws cloudformation create-stack \
        --stack-name {클러스터명}-network \
        --template-body file://sample-vpc-subnet.json \
        --parameters \
          ParameterKey=AvailabilityZone,ParameterValue={AZ} \
          ParameterKey=VpcName,ParameterValue=hpc-{클러스터명}-vpc
      ```
    - 스택 완료 후 Outputs에서 VpcId, PublicSubnetId, PrivateSubnetId 추출
  - **커스텀(4번)**: 기존 VPC 목록 제공 후 선택 or 새로 생성
    - 기본 VPC 있으면 → 자동 선택 후 확인 (Y/n)
    - private 서브넷 없으면 → ⚠️ 경고 후 선택:
      1. private 서브넷 자동 생성 후 진행 (권장)
      2. public 서브넷에 배치하고 계속 진행

  - **CloudFormation 템플릿** (`sample-vpc-subnet.json`):
    - 새 VPC 생성 또는 기존 VPC/IGW 재사용 가능 (Condition 로직)
    - Parameters: `AvailabilityZone`, `VpcId`(선택), `InternetGatewayId`(선택), `VpcCIDR`, `PublicCIDR`, `PrivateCIDR`
    - Conditions:
      - `CreateVpc` — VpcId가 비어있으면 새 VPC 생성
      - `CreateInternetGateway` — InternetGatewayId가 비어있으면 새 IGW 생성
      - `ExistingInternetGateway` — 기존 IGW 사용
    - Resources: VPC, IGW, Public/Private 서브넷, NAT Gateway, 라우트 테이블, 라우트 연결
    - 모든 리소스에 `Name`/`Stack` 태그 부착
    - Outputs: `VpcId`, `InternetGatewayId`, `PublicSubnetId`, `PrivateSubnetId`
    - 프리셋(1~3번): VpcId/InternetGatewayId를 비워서 전체 새로 생성
    - 커스텀(4번) 기존 VPC 선택 시: VpcId/InternetGatewayId를 전달하여 서브넷만 생성

- **[5/6] 클러스터 설정 파일 생성** — 이전 단계에서 수집한 정보와 프리셋 설정을 조합하여 `./{클러스터명}-config.yaml` 생성.
  - 참고 파일: `sample-hpc-cluster.yaml` (변수 치환 템플릿)
  - **템플릿 변수**:

    | 변수 | 설명 | 확보 단계 |
    |------|------|----------|
    | `REGION` | AWS 리전 | 1단계 |
    | `OS` | 운영체제 (기본: alinux2) | 프리셋: 고정 / 커스텀: 5번 질문 |
    | `HEAD_NODE_INSTANCE_TYPE` | Head Node 인스턴스 | 프리셋: 표 참조 / 커스텀: 2번 질문 |
    | `KEY_NAME` | SSH 키페어 이름 | 3단계 |
    | `PUBLIC_SUBNET_ID` | Head Node 서브넷 | 4단계 CloudFormation Outputs |
    | `COMPUTE_INSTANCE_TYPE` | Compute 인스턴스 | 프리셋: 표 참조 / 커스텀: 3번 질문 |
    | `MAX_COUNT` | Compute 최대 수 | 프리셋: 표 참조 / 커스텀: 4번 질문 |
    | `PRIVATE_SUBNET_ID` | Compute 서브넷 | 4단계 CloudFormation Outputs |
    | `STORAGE_TYPE` | Efs 또는 FsxLustre | 프리셋: 표 참조 / 커스텀: 5번 질문 |

  - **조건부 블록** (프리셋에 따라 포함 여부 결정):
    - `Efa` — 프리셋 2번(전산 시뮬레이션)만 활성화 / 커스텀: 6번 질문
    - `PlacementGroup` — 프리셋 2번(전산 시뮬레이션)만 활성화 / 커스텀: 6번 질문
    - `FsxLustreSettings` — StorageType이 FsxLustre일 때만 포함 (`StorageCapacity: 1200`, `DeploymentType: SCRATCH_2`)
  - **프리셋별 config 차이점**:

    | 항목 | 1. 입문용 | 2. 전산 시뮬레이션 | 3. AI/딥러닝 |
    |------|-----------|-------------------|-------------|
    | HeadNode 인스턴스 | t3.medium | c7i.xlarge | c7i.xlarge |
    | Compute 인스턴스 | c7i.xlarge | c8i.48xlarge | g6.xlarge |
    | MaxCount | 4 | 8 | 4 |
    | EFA | 미사용 | Enabled | 미사용 |
    | PlacementGroup | 미사용 | Enabled | 미사용 |
    | SharedStorage | EFS | FSx Lustre 1.2TB | FSx Lustre 1.2TB |

  - **SharedStorage 설정** (ParallelCluster가 자동 생성):
    - 1번 입문용: `StorageType: Efs` (추가 파라미터 불필요, 용량 자동 확장)
    - 2/3번: `StorageType: FsxLustre`, `StorageCapacity: 1200`, `DeploymentType: SCRATCH_2`

  - **공통 설정**:
    - Scheduler: `slurm`
    - Queue: 1개, 이름 `compute`
    - Compute Resource: Queue당 1개
    - MinCount: `0` (사용 시에만 노드 기동, 비용 절감)
    - HeadNode → PublicSubnetId, Compute → PrivateSubnetId
    - Image.Os: `alinux2`
    - SSM 정책 추가 (`AmazonSSMManagedInstanceCore`)
    - RootVolume: `gp3`
  - 생성 후 주요 설정 요약을 한국어로 출력하고 확인 (Y/n)

- **[6/6] 클러스터 배포** — `pcluster create-cluster --cluster-name <name> --cluster-configuration <name>-config.yaml` 실행 후 상태 자동 확인:
  - `pcluster describe-cluster` 로 1분마다 폴링
  - `CREATE_COMPLETE` 확인 시 아래 안내 출력:
  ```
  ✅ CREATE_COMPLETE! 클러스터가 준비되었습니다.

  클러스터에 접속하려면:
     pcluster ssh --cluster-name <name> -i ./<keypair>.pem

  ⚠️  사용이 끝나면 클러스터를 삭제하세요. (비용 발생 방지)
     pcluster delete-cluster --cluster-name <name>
  ```

각 단계에서 오류 발생 시 한국어로 원인과 해결 방법을 안내하고 중단.

