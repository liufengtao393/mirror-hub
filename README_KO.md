<p align="center">
  <a href="../../">简体中文</a> | <a href="./README_TW.md">繁體中文</a> | <a href="./README_EN.md">English</a> | <a href="./README_JA.md">日本語</a> | <b>한국어</b>
</p>

<div align="center">

# 🚀 OmniMirror Sync Engine
### 완전 자동 · 무제한 · 듀얼 트랙 오픈소스 자산 및 컨테이너 재해 복구 허브

<p align="center">
  <a href="../../actions"><code>⚡ Engine: GitHub Actions</code></a> &nbsp;•&nbsp;
  <a href="./.github/workflows/batch-mirror.yml"><code>🔄 Mode: Dual-Track Incremental</code></a> &nbsp;•&nbsp;
  <a href="https://ghcr.io"><code>🐳 Registry: GHCR.io</code></a> &nbsp;•&nbsp;
  <a href="./sync_state.json"><code>📒 Ledger: sync_state.json</code></a>
</p>

**상태 원장 기반(State-Driven) 분산형 자동 미러링 및 재해 복구 시스템**  
*Git 소스 코드, Wiki, 전체 Release 바이너리 자산 및 멀티 아키텍처 Docker 이미지의 영구 무손실 아카이빙 구현*

</div>

---

## 📖 프로젝트 개요 (Overview)

**OmniMirror Sync Engine**은 고가용성 셀프 호스팅 및 오픈소스 자산의 영구 보존을 위해 설계된 자동화 오케스트레이션 허브입니다. 기존 동기화 스크립트의 고질적인 문제인 **실행 시간 초과, 중복 풀(Pull), 버전 누락, 원본 저장소 삭제(HTTP 404)로 인한 자산 영구 손실**을 해결하기 위해 구조화된 상태 원장(`sync_state.json`)과 듀얼 트랙 스케줄링 알고리즘을 도입하여 밀리초 단위의 차분 필터링과 전 자동 네트워크 복구(Salvage)를 제공합니다.

---

## ✨ 핵심 아키텍처 및 기능 (Core Architecture)

### 1. 🧭 기준 시간 듀얼 트랙 분배 알고리즘 (Dual-Track Scheduling)
* **실시간 최신 추적 채널 (Zero-Quota Fast Track)**：프로젝트 최초 등록 시점의 `baseline_time` 이후에 게시된 신규 버전은 과거 백필 할당량을 소모하지 않고 **무조건 최우선 동기화**됩니다.
* **점진적 과거 이력 백필 (Progressive History Backfill)**：수천 개의 과거 버전을 보유한 대형 저장소에 대해 무제한 페이지네이션(`--paginate`)으로 전체 목록을 매핑하고 배치 단위로 안전하게 보완합니다. 완료 시 자동으로 `history_completed: true`가 기록되어 경량 모니터링 모드로 전환됩니다.

### 2. 🔐 다이제스트 기반 차분 중복 제거 (Digest-Level Deduplication)
* **코드 및 문서 온디맨드 복제**：원본의 `pushed_at` 타임스탬프를 실시간 비교하여 소스 변경이 없으면 `git clone --mirror`를 건너뛰어 네트워크 및 I/O 부하를 90% 이상 절감합니다.
* **컨테이너 이미지 SHA256 검증**：`buildx imagetools`를 통해 원격 Manifest Digest 지문을 추출하여 이미지 해시가 실제로 변경되거나 새 태그가 추가된 경우에만 `ghcr.io`로 푸시합니다.

### 3. 🚑 최종 폴백 및 404 네트워크 구조 (Fallback & 404 Rescue)
* **큐 레벨 자동 재시도**：모든 일반 작업 완료 후 최종 폴백 단계가 자동 실행되어 일시적 실패 항목을 여러 차례 재시도하여 완전히 해소합니다.
* **원본 삭제 감지 및 전체 네트워크 구조**：원본의 `HTTP 404` 상태를 정확히 감지합니다. 원본 저장소나 릴리스가 삭제된 경우 GitHub 전체에서 인기 공개 포크/미러 저장소를 자동 검색하여 Release 자산을 구조하며, 완전히 절판된 항목은 `archived_dead`로 아카이빙하여 무한 루프를 방지합니다.

### 4. 🛡️ 우아한 서킷 브레이커 및 실시간 트랜잭션 저장 (Graceful Circuit Breaker)
* **프로젝트별 원자적 커밋**：각 프로젝트 동기화 완료 즉시 `rebase`를 통해 `sync_state.json`을 푸시 저장하여 진행 상황 손실을 원천 차단합니다.
* **안전 실행 시간 윈도우**：내장 타이머가 실행 시간 5시간 20분 도달 시 메인 루프를 안전하게 중단하고 폴백 및 상태 저장 시간을 확보하여 모든 CI 워크플로우가 정상(Green) 종료되도록 보장합니다.

---

## 📂 디렉토리 구조 (Directory Structure)

```text
.
├── .github/
│   ├── workflows/
│   │   └── batch-mirror.yml   # 핵심 스케줄링 워크플로우 (정기 실행 + 변경 트리거 + 수동 실행)
│   └── .last_run              # 자동 활성 유지 타임스탬프 (60일 미활동 비활성화 방지)
├── repos.txt                  # 동기화 작업 선언형 매니페스트
├── sync_state.json            # 글로벌 상태 원장 (기준 시간, 동기화된 태그, 다이제스트 및 구조 기록)
├── README.md                  # 简体中文 문서
├── README_TW.md               # 繁體中文 문서
├── README_EN.md               # English documentation
├── README_JA.md               # 日本語ドキュメント
└── README_KO.md               # 한국어 문서
```

---

## ⚙️ 매니페스트 규격 (`repos.txt`)

파이프 구분자(`|`) 형식으로 작성합니다. `repos.txt`를 수정하여 푸시하면 자동으로 증분 동기화가 시작됩니다：

```text
# 형식: GitHub 원본 | Docker 원본(없으면 none) | 대상 저장소명 | 동적 추적 태그 수
n8n-io/n8n               | docker.n8n.io/n8nio/n8n         | n8n          | 10
teableio/teable          | ghcr.io/teableio/teable         | teable       | 10
langgenius/dify          | langgenius/dify-api             | dify-api     | 10
none                     | postgres                        | postgres     | 10
Panniantong/Agent-Reach  | none                            | agent-reach  | 10
```

| 열 위치 | 파라미터명 | 설명 |
| :--- | :--- | :--- |
| **1열** | `GitHub Source` | 원본 `owner/repo` 경로 (이미지만 있는 경우 `none`) |
| **2열** | `Docker Source` | Docker Hub, GHCR 및 사설 레지스트리 지원 (없으면 `none`) |
| **3열** | `Target Name` | 현재 계정 하위의 대상 저장소명 및 `ghcr.io` 이미지명 |
| **4열** | `Tag Limit` | 실행당 동적으로 추적할 최신 이미지 태그 수 (기본값: `5`) |

---

## 🐳 이미지 풀(Pull) 사용 가이드 (Usage)

동기화된 모든 컨테이너 이미지는 GitHub Container Registry(GHCR)에 호스팅되며 멀티 아키텍처(`linux/amd64` / `linux/arm64`)를 기본 지원합니다：

```bash
# 최신 버전 이미지 다운로드
docker pull ghcr.io/liufengtao393/<대상 저장소명>:latest

# 특정 버전 태그 다운로드
docker pull ghcr.io/liufengtao393/<대상 저장소명>:<버전 태그>
```

---

## 📊 상태 원장 스키마 (`sync_state.json`)

자동 유지 관리되는 JSON 상태 머신을 통해 모든 프로젝트의 동기화 상태를 언제든 감사할 수 있습니다：

* `baseline_time`：프로젝트가 처음 등록된 UTC 기준 타임스탬프.
* `history_completed`：기준 시간 이전의 모든 과거 자산이 100% 동기화 완료되었음을 나타내는 불리언 플래그.
* `synced_releases`：동기화가 완료된 전체 Release 태그 목록.
* `synced_images`：각 이미지 태그에 대응하는 최신 `sha256` 다이제스트 지문 및 동기화 시간.
* `failed_releases`：최종 폴백 단계에서 자동 재시도될 예외 대기열.
* `archived_dead`：원본이 `HTTP 404`이며 공개 미러도 존재하지 않는 영구 절판 자산 감사 기록.

---

<div align="center">

**Powered by GitHub Actions & Buildx Imagetools**  
*Automated · Resilient · Self-Healing*

</div>
