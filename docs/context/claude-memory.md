# Claude 작업 메모리 (이전 세션 인수인계)

> 이 문서는 이전 Claude Code 세션이 이 PC의 `~/.claude/.../memory/`에 쌓아둔 메모리 노트를
> **레포에 복사한 것**입니다 (git으로는 전달되지 않으므로). 맥북/새 세션에서 맥락 복원용.
> 메인 인수인계는 루트의 [`HANDOFF.md`](../../HANDOFF.md) 참조.

---

## project-overview

XROO Package Solution은 패키지 빌드·배포·권한·계정 운영을 위한 콘솔이다.

- **어드민 콘솔**: `http://192.168.1.70:5173` 에서 동작 중. 기능 완성된 상태.
- **사용자(End-user) 페이지**: 신규로 디자인/구현 중. 어드민의 디자인 시스템 위에서 일관되게 작업.
- 브랜드 컬러: Primary Green `#178551`.
- 디자인 SSOT: `DESIGN_SYSTEM.md`.

**Why:** 어드민과 사용자 페이지가 시각적·인터랙션 일관성을 갖기를 원함. 디자인 토큰을 어드민에서 라이브 추출해 사용자 페이지에 그대로 적용.

**How:** 사용자 페이지 작업 시 항상 DESIGN_SYSTEM.md를 SSOT로 참조. 새 컴포넌트는 기존 토큰 재사용.

---

## desktop-app-plan

사용자 페이지는 **웹이 아닌 데스크탑 실행파일(exe)** 로 빌드된다 (사용자 컨펌). 프레임워크는 아직 미정.

영향:
- 폰트를 앱 번들에 임베딩 (Pretendard JP Variable, OFL OK)
- OS 다크모드 감지는 프레임워크별 API
- frameless 윈도우 + 자체 타이틀바 가능
- 오프라인 동작 필요

프레임워크별:
- **Electron**: 어드민 React/Tailwind/lucide-react 그대로 재사용 (재작업 0). `DESIGN_SYSTEM.md §11` 스니펫 즉시 사용. ← 가장 유력
- **Tauri**: 토큰 동일, WebView2 의존성 주의. 같은 React 스택.
- **Flutter**: 토큰을 `ThemeData`/`ColorScheme`로 변환 필요. `lucide_icons` 패키지.

프레임워크 정해지기 전엔 **프레임워크 중립**으로 토큰만 정확히 유지. 향후 `tokens.json`(Style Dictionary)로 정리 권장.

---

## design-system-sot

디자인 시스템 SSOT: **`DESIGN_SYSTEM.md`** (v3.3, 2026-06-04 — 전수 검증 완료).

검증 이력 요약:
- **v3**: 실제 DOM 클래스를 ground-truth로 v2 전 값 1:1 대조. 토큰 오차 0, 컴포넌트 18건 정정. (`--radius-20` 미존재, 사이드바 `w-60`=240px, 폼 라벨 `t-caption-1`(12px), 콘솔 타이틀 `t-title-1`(32px) 등)
- **v3.1**: `/login` round-trip 재현으로 6건 정정 — 우측 폼 배경은 테마종속(검정 하드코딩 아님), 좌우=44/56, 그라데이션 3-stop `#0A4429→#0F5C39→#178551`, 좌측 글로우 2개+그리드, 로그인 인풋은 표준 `bg-bg-primary`.
- **v3.2**: `/projects` 재현 — 카드 그리드 반응형 `grid-cols-2 md:grid-cols-3 xl:grid-cols-4`, 카드 `<article>`+`aspect-video`, 설명 `truncate`, 메타 `<dl grid-cols-3>`.
- **v3.3**: 4개 페이지(login·projects·users·projects-new) dark+light round-trip → 모두 일치.

포함 내용: 핵심원칙 10 / 테마모드 / 컬러토큰(Primary 12단·CoolGrey 8단·Semantic 50+) / **타이포 `t-*` 10개** / Spacing·Radius·Elevation / Density / **아이콘 lucide-react** / **컴포넌트 26종 전 상태 실측** / 페이지 레이아웃 3종 / **데스크탑 앱 가이드** / globals.css+tailwind.config 스니펫.

**규칙:** 색·사이즈·라운드 임의 지정 금지(토큰/utility 사용). 타이포는 `t-*`. 아이콘은 lucide-react만. 다크 모드 검증 필수.

---

## design-tokens-core

**1. 브랜드/폰트/아이콘**
- Primary: `#178551`(라이트 CTA, 그린700) / `#34C381`(다크 fg, 그린400)
- 폰트: **Pretendard JP Variable** → Pretendard Variable → system fallback
- 아이콘: **lucide-react**. 16px 베이스, viewBox `0 0 24 24`, stroke 2, fill none, currentColor.

**2. 테마 토글**
- localStorage 키: `xroo-theme-mode`, 값 `light|dark|system`
- 적용: `document.documentElement.classList.toggle('dark', isDark)` (라이트 클래스 없음, 다크만 `html.dark`)
- `system`은 `matchMedia('(prefers-color-scheme: dark)')` 따름
- Flash 방지: `<head>` 최상단 inline 스크립트

**3. 스타일링 = Tailwind + CSS Variables** — Utility 이름이 곧 토큰 이름(`bg-bg-brand`, `text-fg-strong`…). 색 하드코딩 금지.

**4. Typography `t-*` (10개)**
- `t-title-1` 32/700, `t-title-3` 24/700(모달)
- `t-heading-1` 22/700, `t-heading-2` 20/700(섹션·카드 타이틀)
- `t-body-1` 16/500, `t-body-2` 15/500
- `t-label-1` 14/600(사이드바·폼라벨), `t-label-2` 13/600
- `t-caption-1` 12/600(헬퍼), `t-caption-2` 11/600

**5. 어드민 약점 (사용자 페이지에서 보강)**
- 에러 컬러 토큰 없음 → `--fg-error:#E5484D` 추가 권장
- Status badge 모든 상태 동일색 → 상태별 컬러 권장
- Modal backdrop 투명 → `bg-black/40` 권장
- 자세히는 `DESIGN_SYSTEM.md §10.4`.

---

## admin-routes

어드민 콘솔 라우트:
- `/login` — 좌(다크 그린 그라데이션)/우(거의 검정 폼) 2단 스플릿.
- `/projects` — 프로젝트 리스트(카드 그리드). 검색·필터칩·정렬·페이지네이션.
- `/projects/new` — 풀폼 페이지(모달 아님). 섹션 카드 5개.
- `/users` — 사용자 관리(테이블). 사용자 추가는 **모달**.

세션: 토큰은 sessionStorage(탭 격리). Claude가 동작하는 정확한 탭에서 로그인 필요.

---

## sidebar-dark-only

어드민 사이드바는 **라이트/다크 무관 항상 다크 톤**:
- 라이트: `--bg-sidebar:#171719` / 다크: `#0D0E10` / 둘 다 fg `#FFFFFF`

**Why:** 콘솔/IDE 느낌 대비, 어드민 정체성.
**How:** 어드민 사이드바 색 변경 금지. 사용자 앱에서도 사이드바는 다크 유지(현 user-app.html도 항상 다크 사이드바).

---

## user-app-draft (사용자 앱 프로토타입 사양) — 최신

사용자 앱 UI/UX 초안: **`_repro/user-app.html`** (단일 파일 인터랙티브 프로토타입).

**기반:** `Screenshot/` 8장 + DESIGN_SYSTEM v3.3. 어드민과 통신해 언리얼 패키지를 다운로드·실행.

**플로우:** 로그인 → 프로젝트 리스트 → (상세) → 다운로드(진행 모달) → 실행하기. 버전 업데이트 시 재다운로드. 상세에 버전 기록 테이블.

**구현된 화면/상태:**
- **로그인**: 어드민 `/login` 실측 1:1. 좌 44%/우 56%, gradient `to-br oklab #0A4429→#0F5C39→#178551`. 그리드(.07)+글로우 2개(우상단 그린.5/좌하단 .35, 은은한 자동 모션). 컨텐츠 max-w420 가로·세로 중앙: 로고 28px → **언더라인 #6FD3A0(그린)** → 헤드라인 "Package Solution / User Console" 36px/45 ls-0.9 → 서브카피 15px white/.7. lg(1024) 미만 좌패널 숨김. 우측 폼=캡션 없이 "사용자 로그인"+서브+아이디/비번. 에러=인풋 아래 헬퍼 텍스트.
- **테마 첫 페인트**: `<head>` 인라인 스크립트로 OS 다크 즉시 반영(플래시 없음).
- **사이드바**(항상 다크): 로고+"사용자 앱". 푸터=테마 세그먼트(시스템/라이트/다크)+계정칩(ID+"로그인 계정" 좌, 로그아웃 아이콘 우). nav=프로젝트 리스트·환경설정.
- **리스트**: 반응형 카드 그리드(등높이). 상태 뱃지=타이틀 위(미설치 회색/설치됨 그린+체크/업데이트필요 amber). 스펙=버전·크기 레이블+값(업데이트면 `v1→v2` amber 화살표). **카드 전체 클릭→상세**. 헤더 우측 새로고침. 개수뱃지 brand-subtle 30px/15px. 카드 타이틀 `t-heading-2` 20px(어드민 카드와 동일).
- 푸터 버튼: 다운로드/업데이트=solid 그린(업데이트 아이콘 circle-arrow-up) / 실행하기=tonal 그린(▶).
- **상세**: 히어로(썸네일+제목+뱃지+실행파일위치 그린 mono) + 버전 기록 테이블(헤더 구분선 `line-subtle`).
- **다운로드/업데이트 모달**(`.modal-dl`): 아이콘+이름+용량 헤더 / 글로우 디바이더 / 진행 pill(그린 스피너+라벨+퍼센트 / 그린 진행바). 자동 종료. (레퍼런스 다크 카드 레이아웃 참고, 그린 액센트)
- **로그아웃 다이얼로그**(`.modal-confirm`): 라운드 스퀘어 아이콘 + 2줄 메시지 + 균형 풀폭 버튼(취소/로그아웃).
- **환경설정**: 그룹형(화면/연결/다운로드/계정/정보). 폴더 선택/열기=Electron `shell.openPath`/`dialog`(브라우저는 폴백 토스트).
- `.btn-sec`(새로고침·목록·폴더)=bg-primary+`line-default` 테두리(라이트 가독성).
- **테마**: localStorage `xroo-theme-mode` + matchMedia, 사이드바·환경설정 두 토글 동기화. 양테마 검증 완료.

**결정 사항(사용자 확정):**
- 형태 = 인터랙티브 HTML 프로토타입(프레임워크 미정, Electron 렌더러로 이식 가능).
- 실행하기 색 = 브랜드 그린 통일(스크린샷 파란색 대신). 상태는 solid/tonal로 구분.
- 상세 페이지 라이트로 통일(나머지와 일관).

**추가 토큰(컨펌 필요):** `--fg-error:#E5484D`(light)/`#FF6B6E`(dark), 업데이트 amber `--fg-warning`/`--bg-warning-subtle`.

**미구현:** 실제 백엔드 연동, 카드 실제 썸네일, drag&drop, Electron 골격.
