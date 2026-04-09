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
- [x] SSH 접속 테스트
- [x] 리소스 삭제 완료

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
| 클러스터 이름 | ml-lab |
| Head Node | c7i.xlarge |
| Compute | g6.xlarge × 최대 8대 |
| Storage | FSx Lustre 1.2TB |
| EFA | 미사용 |
| 리전 | ap-northeast-2 (서울) |
| AZ | ap-northeast-2a |
| 키페어 | theYoohw |

### 검증 항목
- [x] 3번 선택 후 클러스터 이름만 질문
- [x] [1/6] AWS 계정 연결 확인 — 리전 변경 (us-west-2 → ap-northeast-2)
- [x] [2/6] pcluster CLI 설치 확인 (v3.15.0)
- [x] [3/6] SSH 접속 키 확인 (기존 키페어 theYoohw 선택)
- [x] [4/6] VPC 생성 정상 — g6.xlarge 지원 AZ 필터링 동작 (2b 제외)
- [x] [5/6] config.yaml 생성 — FSx Lustre, GPU 인스턴스 설정 확인
- [x] [6/6] 클러스터 배포 CREATE_COMPLETE
- [x] sbatch 작업 제출 — Compute Node 자동 생성 확인
- [x] GPU 인식 확인 — NVIDIA L4 (23GB), Driver 550.127.08, CUDA 12.4
- [x] Compute Node가 private 서브넷에 생성됨 확인
- [x] 리소스 삭제 완료

### 테스트 중 발견/개선 사항
- steering 개선: 시작 메시지 테이블 형식을 5컬럼 마크다운 테이블로 고정 (출력 일관성 확보)
- g6.xlarge는 ap-northeast-2b에서 미지원 → AZ 필터링이 정상 동작하여 자동 제외됨

---

## TC-4: 커스텀

| 항목 | 값 |
|------|---|
| 프리셋 | 4번 커스텀 |
| 클러스터 이름 | cfd-hpc |
| OS | Ubuntu 22.04 → Amazon Linux 2 (RSA 키 미지원으로 변경) |
| Head Node | c7i.xlarge (2번 선택) |
| Compute | c7i.xlarge (7번 직접 입력) |
| 최대 수 | 4 |
| Storage | EFS (1번 선택) |
| EFA | 아니오 (2번 선택) |
| DCV | 아니오 (2번 선택) |
| 리전 | ap-northeast-2 (서울) |
| AZ | ap-northeast-2b (2a 용량 부족으로 변경) |
| 키페어 | theYoohw |

### 검증 항목
- [x] 커스텀 질문 8단계 순서대로 진행
- [x] 각 질문에서 번호 선택 정상 동작
- [x] 인스턴스 가용 여부 확인 동작 (c7i.xlarge, 서울 리전)
- [x] [1/6] AWS 계정 연결 확인 — 리전 변경 (us-west-2 → ap-northeast-2)
- [x] [2/6] pcluster CLI 설치 확인 (v3.15.0)
- [x] [3/6] SSH 접속 키 확인 (기존 키페어 theYoohw 선택)
- [x] [4/6] 신규 VPC 생성 — AZ 선택 기능 정상 동작
- [x] [5/6] config.yaml 생성 — 설정 요약 테이블 출력, 확인 후 배포
- [x] [6/6] 클러스터 배포 — 1차 실패(ap-northeast-2a 용량 부족) → AZ 변경 후 재시도 → CREATE_COMPLETE
- [x] SSH 접속 테스트 (`pcluster ssh --region ap-northeast-2`)
- [x] 리소스 삭제 완료

### 테스트 중 발견/개선 사항
- Ubuntu 22.04는 RSA 키 미지원 → OS 변경 또는 ed25519 키 생성 필요
- ap-northeast-2a에서 c7i.xlarge 용량 부족 발생 → AZ 변경 재시도 흐름 검증
- steering 개선: 4단계 AZ 선택을 사용자에게 물어보도록 변경 (추천 표시 포함)
- steering 개선: 인스턴스 용량 부족 시 AZ 변경 재시도 흐름 추가
- steering 개선: AZ 변경 시 네트워크 스택 재생성 필수 명시
- steering 개선: pcluster ssh에 --region 옵션 추가
- steering 개선: 6단계 완료 시 클러스터 요약 테이블 추가
- steering 개선: 5단계 후 6단계 배포 흐름 요약 안내 추가

---

## TC-5: 클러스터 삭제 (9번 메뉴)

### 검증 항목
- [x] 9번 선택 시 삭제 흐름 진입
- [x] 리전 선택 정상 동작
- [x] 클러스터 목록 조회 및 번호 선택
- [x] 클러스터 삭제 완료
- [x] 네트워크 스택 삭제 확인 및 정리 완료

---

## 테스트 결과 요약

| TC | 케이스 | 결과 | 비고 |
|----|--------|------|------|
| 1 | 입문용 | ✅ 통과 | 클러스터명: my-hpc, 생성/삭제 정상 완료 |
| 2 | 전산 시뮬레이션 | ✅ 통과 | 클러스터명: yoohw-cfd-cluster, 배포/삭제 완료 |
| 3 | AI/딥러닝 | ✅ 통과 | 클러스터명: ml-lab, GPU(NVIDIA L4) 인식 확인, 삭제 완료 |
| 4 | 커스텀 | ✅ 통과 | 클러스터명: cfd-hpc, AZ 변경 재시도 검증, 삭제 완료 |
| 5 | 클러스터 삭제 | ✅ 통과 | 9번 메뉴 삭제 흐름 정상 동작 |
