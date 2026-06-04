# HANDOFF — XROO Package Solution (사용자 앱)

> 다른 컴퓨터(예: 맥북)나 새 Claude Code 세션에서 이 작업을 **이어서 진행**하기 위한 인수인계 문서.
> 새 세션이라면 먼저 이 파일과 [`docs/context/`](docs/context/)를 읽어주세요.

마지막 업데이트: 2026-06-04 (commit `532b5e6` 시점 기준)

---

## 1. 프로젝트 한 줄 요약

XROO Package Solution은 언리얼 패키지 빌드를 배포/운영하는 시스템.
- **어드민 콘솔**: `http://192.168.1.70:5173` — 이미 완성된 웹앱 (React/Tailwind/lucide-react).
- **사용자(End-user) 앱**: 사용자가 패키지를 **다운로드 → 압축해제 → 실행**하는 **데스크탑 앱**. ← **지금 우리가 디자인/구현 중인 대상.**
- 두 앱은 **동일 디자인 시스템**을 공유. 브랜드 그린 `#178551`.

## 2. 지금까지 한 일 (현재 상태)

사용자 앱의 **UI/UX 초안**을 단일 HTML 인터랙티브 프로토타입으로 완성:
👉 **[`_repro/user-app.html`](_repro/user-app.html)** ← 메인 산출물

- 플로우: **로그인 → 프로젝트 리스트 → (상세) → 다운로드/업데이트(진행 모달) → 실행하기** 전부 클릭 동작.
- 라이트/다크/시스템 **3 테마** 동작, 반응형 처리, OS 테마 자동 반영.
- 디자인은 어드민 콘솔을 **Chrome으로 직접 실측**해 1:1로 맞춤 (특히 로그인 화면).

## 3. 어떻게 보나

`user-app.html`은 **인라인 CSS/JS 단일 파일**이라 빌드 불필요:
```bash
# 그냥 브라우저로 열기 (가장 간단)
open _repro/user-app.html        # macOS
start _repro/user-app.html       # Windows

# 또는 로컬 정적 서버 (Claude_Preview 연동용)
node _repro/server.cjs           # → http://localhost:5180/user-app.html
```
- 데모 로그인: 아이디는 채워져 있고 **아무 비밀번호나 입력 후 로그인** (빈칸이면 인풋 아래 빨간 헬퍼 텍스트).
- 테마: 로그인 후 좌측 사이드바 하단 토글(시스템/라이트/다크) 또는 환경설정 > 화면.

## 4. 파일 지도

| 경로 | 내용 |
|---|---|
| `_repro/user-app.html` | **메인 산출물** — 사용자 앱 프로토타입 (단일 파일) |
| `DESIGN_SYSTEM.md` | **디자인 SSOT** (v3.3, 전수 검증). 토큰·타이포·26개 컴포넌트 명세·데스크탑 가이드 |
| `_repro/index.html` `users.html` `new.html` `login.html` | 어드민 페이지 **재현본**(검증용, 원본과 1:1 대조 완료) |
| `_repro/server.cjs` | node 정적 서버 (port 5180) |
| `_repro/logo.svg` `design_source/logo.svg` | XROO 워드마크 로고 (396×100, 흰색) |
| `Screenshot/` | 사용자 앱 기획 스크린샷 8장 (요구사항 출처) |
| `.claude/launch.json` | Claude_Preview용 서버 설정 ("repro") |
| `docs/context/` | **이전 Claude 세션의 작업 메모리 노트** (맥락 복원용) |

## 5. 핵심 디자인 결정 & 근거 (꼭 알아야 할 맥락)

- **'실행하기' 버튼 = 브랜드 그린 통일.** 기획 스크린샷은 파란색이었지만 디자인 시스템 일관성을 우선. 상태는 색이 아니라 **채움**으로 구분: 다운로드/업데이트=solid 그린, 실행하기=tonal 그린(▶).
- **상태 뱃지**(카드 타이틀 위, `.pbadge`): 미설치=회색 / 설치됨=그린톤+체크 / 업데이트필요=**amber**.
- **사이드바는 항상 다크** (어드민 정체성). 사용자 앱 본문/카드/폼은 테마를 따름.
- **에러 토큰**(`--fg-error #E5484D`/`#FF6B6E`)·**업데이트 amber 토큰**은 어드민에 없어서 추가한 것 → **디자인팀 정식 컨펌 필요**.
- **로그인 화면은 어드민 `/login`을 Chrome으로 실측해 1:1 매칭** (아래 §6).
- 폴더 선택/열기 버튼은 **Electron `shell.openPath` / `dialog`** 로 배선됨 → 브라우저에선 폴백 토스트, **Electron 빌드에서 실제 동작**.

## 6. 로그인 화면 정밀 스펙 (어드민 실측값, 1:1)

가장 많이 반복 수정된 부분이라 값으로 기록:
- 좌/우 분할 = **44% / 56%** (좌 패널 `flex:0 0 44%`).
- 좌 패널 gradient = `linear-gradient(to bottom right in oklab, #0A4429 0%, #0F5C39 50%, #178551 100%)`, `padding:0 48px 48px`.
- 오버레이: 그리드(white 1px, opacity .07, 40px) + **글로우 2개**(우상단 `rgba(52,195,129,.5)` 420px, 좌하단 `.35` 360px) — 은은한 자동 모션(마우스 추적 아님), `prefers-reduced-motion` 정지.
- 컨텐츠: `max-width:420px` 가로 중앙 + 세로 중앙. 로고 28px → **언더라인 `#6FD3A0`(그린!)** w48 h2 → 헤드라인 **"Package Solution / User Console"** 36px/45 w700 ls-0.9px → 서브카피 15px/22 white/.7.
- 카피라이트 left48 bottom40 white/.4. **lg(1024) 미만 좌패널 숨김**.
- 우측 폼: 캡션 없음, "사용자 로그인" h1 + 서브 + 아이디/비밀번호만.
- **테마 첫 페인트**: `<head>` 인라인 스크립트가 `localStorage('xroo-theme-mode')||'system'`로 OS 다크 즉시 적용(플래시 방지).

## 7. 아직 안 한 것 (다음 후보)

- [ ] **프레임워크 확정 후 실제 앱 골격 생성** (Electron 가장 유력 — 어드민 React/Tailwind 그대로 재사용, 재작업 0). 정해지면 `DESIGN_SYSTEM.md §11` 스니펫 즉시 사용.
- [ ] 실제 백엔드 연동 (auth, 진짜 다운로드/압축해제/실행, 버전 체크).
- [ ] 폴더 선택/열기를 실제로 동작시키려면 Electron `ipcMain.handle('dialog:openDirectory')` 또는 `@electron/remote` 세팅 필요.
- [ ] 카드 썸네일을 실제 이미지로 (현재 그라데이션 placeholder).
- [ ] 추가 상태: 빈 목록 / 다운로드 실패 / 업데이트 상세 등.
- [ ] amber·error 추가 토큰 디자인팀 컨펌.

## 8. 맥북에서 이어가기

```bash
git clone https://github.com/seungwoo-p/xroo_package_solution.git
cd xroo_package_solution
open _repro/user-app.html        # 바로 확인
```
- **Claude Code로 이어갈 때**: 이 `HANDOFF.md` + `docs/context/`를 먼저 읽게 하면 맥락이 복원됨. (이전 세션의 메모리는 윈도우 PC의 `~/.claude`에만 있어 git으로 전달되지 않으므로, 그 내용을 `docs/context/`에 복사해 둠.)
- 새 작업 → 커밋 → push(GitHub Desktop의 `Push origin` 또는 `git push`)로 양 컴퓨터를 오감.

## 9. 도구/환경 메모

- 프리뷰: 파일 수정 후 **stop → start**로 캐시 버스트. 라이트/다크는 `documentElement.classList`로 토글하거나 localStorage `xroo-theme-mode` 설정.
- 어드민 로그인 세션은 자주 만료됨 — URL navigate보다 **사이드바 클릭 이동** 권장.
- 비밀번호 입력 등은 보안상 Claude가 대신 못 함 → 사용자가 직접.
