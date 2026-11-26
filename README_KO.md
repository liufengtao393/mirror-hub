# 🚀 Mirror Hub — 무제한 점진적 통합 미러링 시스템

[简体中文](./README.md) | [繁體中文](./README.zh-TW.md) | [English](./README.en.md) | [日本語](./README.ja.md) | [한국어](./README.ko.md)

GitHub Actions 기반의 자동화 오픈소스 자산 미러링 및 재해 복구 아카이빙 시스템입니다. **Git 소스 코드, Wiki, 전체 과거 Releases(2GB 단일 파일 제한 돌파) 및 멀티 아키텍처 Docker 이미지**의 자동 동기화와 전체 네트워크 구조를 지원합니다.

## ✨ 핵심 기능

- **듀얼 트랙 동기화 엔진**: 최초 등록 시 `baseline_time`을 고정합니다. 기준 시간 이후의 새 버전은 **즉시 추적**하며, 기준 시간 이전의 방대한 과거 버전은 배치 단위(기본 15개/회)로 **점진적 보완**을 수행합니다.
- **GitHub 2GB 단일 파일 제한 돌파**: `1900MB`를 초과하는 릴리스 자산을 자동 감지하여 `split`으로 `.part_000` 분할 업로드하며, 릴리스 노트에 복원 명령어를 자동으로 추가합니다.
- **동적 타임아웃 및 체크포인트 저장**: 실행 시간을 실시간으로 감시하여 GitHub의 6시간 강제 종료 한도에 도달하기 전(5시간 시점)에 `sync_state.json` 상태 장부를 안전하게 저장하고 종료합니다.
- **404 삭제 저장소 전체 네트워크 구조**: 실패한 항목을 3회 자동 재시도합니다. 원본 저장소나 태그가 삭제(HTTP 404)된 경우 공개 미러 저장소를 검색해 복구하며, 완전히 유실된 경우 `archived_dead`에 기록합니다.
- **자동 활성 유지(Keep-Alive)**: 실행 시마다 `.github/.last_run`을 업데이트하여 60일 비활성으로 인한 워크플로우 자동 중지를 방지합니다.

## ⚙️ 빠른 설정

### 1. 시크릿 설정 (`MIRROR_TOKEN`)
저장소의 `Settings -> Secrets and variables -> Actions`에서 `MIRROR_TOKEN`이라는 이름으로 시크릿을 생성하고, `repo`, `workflow`, `write:packages`, `read:packages` 권한이 있는 Personal Access Token (Classic)을 입력합니다.

### 2. 미러링 목록 편집 (`repos.txt`)
한 줄에 하나의 프로젝트를 `|`로 구분하여 작성합니다(`#` 주석 지원):
```text
# 형식: 원본GitHub저장소 | 원본Docker이미지 | 대상저장소이름 | 동기화할Docker태그수
# 해당 사항이 없으면 none 입력
dify-ai/dify | langgenius/dify-api | dify | 5
n8n-io/n8n | n8nio/n8n | n8n | 5
torvalds/linux | none | linux-kernel | 0
```

## 🧩 분할 파일(>2GB) 병합 가이드
2GB를 초과하는 단일 파일은 `파일명.part_000`, `파일명.part_001` 등으로 자동 분할됩니다. 모든 분할 파일을 같은 폴더에 다운로드한 후 아래 명령어를 실행하세요:
- **Linux / macOS**: `cat 파일명.part_* > 파일명`
- **Windows (CMD)**: `copy /b 파일명.part_* 파일명`
