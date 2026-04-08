# 테스트 케이스

## 목적
steering 파일이 적용된 상태에서 Kiro가 4가지 케이스 모두 정상 동작하는지 검증한다.

## 사전 조건
- `pcluster-install-with-kiro` 폴더에서 `kiro-cli chat` 실행
- AWS 자격증명 설정 완료
- steering 파일 적용 확인 (시작 메시지 출력 여부)

## 테스트 후 정리
각 테스트 완료 후 반드시 리소스 삭제:
1. `pcluster delete-cluster --cluster-name <이름>`
2. `aws cloudformation delete-stack --stack-name <이름>-network`

---

## TC-1: 프리셋 1번 (입문용)

| 항목 | 값 |
|------|---|
| 프리셋 | 1번 입문용 |
| 클러스터 이름 | my-hpc |
| Head Node | t3.medium |
| Compute | c7i.xlarge × 최대 4대 |
| Storage | EFS |
| EFA | 미사용 |

### 검증 항목
- [x] 시작 메시지 정상 출력
- [x] 1번 선택 후 클러스터 이름만 질문
- [x] [1/6] AWS 계정 연결 확인 정상
- [x] [2/6] pcluster CLI 설치 확인 정상
- [x] [3/6] SSH 접속 키 확인/생성 정상
- [x] [4/6] CloudFormation VPC/서브넷 생성 정상
- [x] [5/6] config.yaml 생성 — EFS 설정 확인
- [x] [6/6] 클러스터 배포 CREATE_COMPLETE
- [x] SSH 접속 테스트
- [x] 리소스 삭제 완료

---

## TC-2: 프리셋 2번 (전산 시뮬레이션)

| 항목 | 값 |
|------|---|
| 프리셋 | 2번 전산 시뮬레이션 |
| 클러스터 이름 | test-simulation |
| Head Node | c7i.xlarge |
| Compute | c8i.48xlarge × 최대 8대 |
| Storage | FSx Lustre 1.2TB |
| EFA | Enabled |
| PlacementGroup | Enabled |

### 검증 항목
- [x] 2번 선택 후 클러스터 이름만 질문
- [x] [1/6] AWS 계정 연결 확인 정상
- [x] [2/6] pcluster CLI 설치 확인 정상
- [x] [3/6] SSH 접속 키 확인 정상 (기존 키페어 선택)
- [x] [4/6] 신규 VPC 자동 생성 정상 (CloudFormation)
- [x] [5/6] config.yaml 생성 — FSx Lustre, EFA, PlacementGroup 설정 확인
- [x] [6/6] 클러스터 배포 CREATE_COMPLETE (약 15분 소요)
- [ ] SSH 접속 테스트
- [ ] 리소스 삭제 완료

### 테스트 중 발견/개선 사항
- steering 개선: 프리셋(1~3번) 시 신규 VPC 자동 생성 흐름 명확화
- steering 개선: 6단계 배포 대기를 `cloudformation wait`으로 통일
- steering 개선: pcluster CLI 주의사항 추가 (`--output json` 금지, Warning 필터링)
- 클러스터명: yoohw-cfd-cluster, 키페어: my-cfd-cluster-key

---

## TC-3: 프리셋 3번 (AI/딥러닝)

| 항목 | 값 |
|------|---|
| 프리셋 | 3번 AI/딥러닝 |
| 클러스터 이름 | test-ai |
| Head Node | c7i.xlarge |
| Compute | g6.xlarge × 최대 4대 |
| Storage | FSx Lustre 1.2TB |
| EFA | 미사용 |

### 검증 항목
- [ ] 1번 선택 후 클러스터 이름만 질문
- [ ] [4/6] VPC 생성 정상
- [ ] [5/6] config.yaml 생성 — FSx Lustre, GPU 인스턴스 설정 확인
- [ ] [6/6] 클러스터 배포 CREATE_COMPLETE
- [ ] SSH 접속 테스트
- [ ] GPU 인식 확인 (`nvidia-smi`)
- [ ] 리소스 삭제 완료

---

## TC-4: 커스텀

| 항목 | 값 |
|------|---|
| 프리셋 | 4번 커스텀 |
| 클러스터 이름 | test-custom |
| Head Node | t3.medium (1번 선택) |
| Compute | c7i.xlarge (6번 일반/기타 선택) |
| 최대 수 | 2 |
| Storage | EFS (1번 선택) |
| EFA | 아니오 (2번 선택) |
| DCV | 예 (1번 선택) |
| OS | Amazon Linux 2 (1번 선택) |

### 검증 항목
- [ ] 커스텀 질문 8단계 순서대로 진행
- [ ] 각 질문에서 번호 선택 정상 동작
- [ ] 인스턴스 가용 여부 확인 동작
- [ ] [4/6] 기존 VPC 선택 or 새로 생성 선택지 제공
- [ ] [5/6] config.yaml 생성 — DCV 설정 포함 확인
- [ ] [6/6] 클러스터 배포 CREATE_COMPLETE
- [ ] SSH 접속 테스트
- [ ] DCV 웹 접속 테스트 (포트 8443)
- [ ] 리소스 삭제 완료

---

## 테스트 결과 요약

| TC | 케이스 | 결과 | 비고 |
|----|--------|------|------|
| 1 | 입문용 | ✅ 통과 | 클러스터명: my-hpc, 생성/삭제 정상 완료 |
| 2 | 전산 시뮬레이션 | 🔄 진행중 | 클러스터명: yoohw-cfd-cluster, 배포 완료. SSH 접속/삭제 미완 |
| 3 | AI/딥러닝 | | |
| 4 | 커스텀 | | |
