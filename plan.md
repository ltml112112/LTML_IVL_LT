# OLED IVL & LT Analyzer — 개발 플랜

> 마지막 업데이트: 2026-03-07
> 현재 브랜치: `claude/summary-copy-smart-select`

---

## 완료된 기능 (Done)

| 항목 | 커밋 | 비고 |
|------|------|------|
| IVL/LT CSV 파싱 + 자동 파일 매칭 | 초기 | REF/SAMPLE, IVL/LT, 1/2 슬롯 자동 인식 |
| IVL 분석 테이블 (Op.V, EL EFF, EQE, CIE, MWL) | 초기 | J=10mA/cm² 기준 |
| LT 수명 계산 (LT90~99) | 초기 | 영구 이탈 기준 |
| LT 차트 (Chart.js) + 슬라이더/토글 | 초기 | WHITE, SQUARE, Smooth, Axis 조절 |
| EL 스펙트럼 차트 | 초기 | |
| COPY 버튼 (raw CSV 값 기준) | `5af5bad` | Excel 붙여넣기 호환 |
| LT 결과 색상 (<95% 빨강, 95~105% 파랑, >105% 보라) | `6ca1fb9` | |
| LT 차트 Grid 토글 버튼 | `1252a35` | |
| 파일 매칭 레이아웃 개선 | `ec4e148` | |
| 요약 COPY (스마트 자동 추천 + 수동 선택) | `148dbf9` | IVL/LT 뱃지 클릭, 하이라이트 표시 |
| **Phase 1: IndexedDB 분석 이력 저장/복원** | 현재 | 저장, 검색, 불러오기, 삭제 |

---

## Phase 1 후속 개선 (단기 — index.html 단일 파일)

### 1-A. 이력 내보내기 / 가져오기 (JSON)
- **왜 필요**: IndexedDB는 같은 PC+브라우저에서만 접근 가능 → 다른 PC와 공유 불가
- **구현**: 이력 패널에 `[내보내기 JSON]` / `[가져오기 JSON]` 버튼 추가
  - 내보내기: 선택한 세션(또는 전체)을 `.json` 파일로 다운로드
  - 가져오기: JSON 파일 드래그 또는 클릭 업로드 → IndexedDB에 병합 저장
- **효과**: USB/네트워크 공유로 PC 간 이력 이동 가능

### 1-B. 이력에서 두 세션 비교
- **왜 필요**: 같은 샘플의 100hr vs 200hr 측정 결과 나란히 비교
- **구현**: 이력 목록에서 체크박스 2개 선택 → `[비교]` 버튼 → 결과 패널을 좌/우 분할 표시
- **난이도**: 중 (UI 레이아웃 변경 필요)

### 1-C. 저장 시 분석 조건 자동 태깅
- **왜 필요**: 라벨만으로는 나중에 조건 파악 어려움
- **구현**: 저장 모달에 파일명 기반 자동 추출 정보 표시
  - 온도(T50), 로트번호, 소자번호, 날짜 등을 파일명에서 파싱해 기본 라벨로 제안

### 1-D. 이력 항목 라벨 편집
- **왜 필요**: 저장 후 라벨 수정 불가
- **구현**: 이력 항목에 `[편집]` 버튼 → 인플레이스 입력으로 라벨 수정

---

## Phase 2 (중기 — 서버 환경 확보 시)

**조건**: 팀 내 PC 1대에 Node.js 설치 가능하거나 사내 서버 담당자 협의 완료 시

### 구성
```
[브라우저] <---> [Node.js + Express API] <---> [SQLite 파일]
                         |
                U드라이브 또는 허가된 서버 경로
```

### 핵심 작업
- [ ] `server.js` — Express REST API (`GET /sessions`, `POST /sessions`, `DELETE /sessions/:id`)
- [ ] `db.js` — better-sqlite3 기반 DB 초기화 + CRUD
- [ ] SQLite 스키마 생성 (sessions, session_files, ivl_data, ivl_spectrum, lt_data, lt_points, lt_levels)
- [ ] 프론트엔드: Phase 1 IndexedDB API를 서버 API로 교체 (또는 오프라인/온라인 이중 모드)
- [ ] Phase 1 → Phase 2 마이그레이션 스크립트 (IndexedDB 데이터를 서버로 업로드)

### SQLite 스키마 (메모)
```sql
sessions       -- id, saved_at, label, ivl_dp, lt_mode
session_files  -- session_id, slot_key, filename
ivl_data       -- session_id, slot_key, volt, eff, eqe, cx, cy, mwl
ivl_spectrum   -- ivl_id, wavelength_nm, intensity
lt_data        -- session_id, slot_key, max_hours
lt_points      -- lt_id, hours, luminance_pct
lt_levels      -- lt_id, level(90~99), breach_hours, breach_pct
```

---

## Phase 3 (장기 — MES 플랫폼)

전사 IT 인프라 프로젝트 수준. 별도 IT 부서 협의 필요.
현재 IVL/LT 모듈은 MES 서브모듈로 설계되어 있음 → 추후 통합 가능.

- [ ] 다중 사용자 인증 (로그인)
- [ ] 시료 트래킹 (Lot, 소자번호, 공정 조건 연결)
- [ ] HPLC/DSC/TGA/LC-MS 등 타 분석 장비 데이터 연동
- [ ] IoT 센서 실시간 데이터 수집
- [ ] 양산 재료 이력 역추적 / 순추적
- [ ] 대시보드 (트렌드 분석, 불량률 통계)

---

## 알려진 버그 / 개선 요청

| # | 내용 | 우선순위 |
|---|------|----------|
| - | (현재 없음) | - |

---

## 개발 환경 메모

- **단일 파일**: `index.html` (CSS + HTML + JS 모두 포함)
- **의존성**: Chart.js v4 (CDN)
- **브라우저 저장**: IndexedDB (`IVLAnalyzerDB`)
- **보안 제약**: C/D 드라이브 쓰기 불가 → 브라우저 기반 또는 U드라이브 배치
- **대상 브라우저**: Chrome/Edge (Clipboard API, IndexedDB 필요)
