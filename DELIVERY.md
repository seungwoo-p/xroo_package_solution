# DELIVERY — XROO 사용자 데스크탑 앱 (UI/UX + 프로토타입)

> CTO 전달용 한 장 요약. 상세 맥락은 [`HANDOFF.md`](HANDOFF.md), 디자인 명세는 [`DESIGN_SYSTEM.md`](DESIGN_SYSTEM.md) 참조.

## 무엇인가
XROO **사용자(End-user) 데스크탑 앱**의 UI/UX 디자인 + **동작하는 인터랙티브 프로토타입**.
어드민 콘솔과 **동일 디자인 시스템**(브랜드 그린 `#178551`)을 공유합니다.
- 플로우: **로그인 → 프로젝트 리스트 → (상세) → 다운로드/업데이트 → 실행**
- 라이트/다크/시스템 테마, 반응형 처리 포함.

## 받는 법
```bash
git clone https://github.com/seungwoo-p/xroo_package_solution.git
cd xroo_package_solution
```
**동작 확인** — 빌드 불필요, 단일 파일:
```bash
open _repro/user-app.html        # macOS (Windows: 파일 더블클릭)
```

## 먼저 읽을 순서
1. **`HANDOFF.md`** — 작업 내용 / 파일맵 / 디자인 결정·근거 / 남은 작업
2. **`DESIGN_SYSTEM.md`** — 토큰·타이포·컴포넌트 SSOT (§11에 globals.css + tailwind.config 스니펫)
3. **`_repro/user-app.html`** — 시각·인터랙션 ground-truth (이 파일이 "정답")
4. **`docs/context/claude-memory.md`** — AI 작업 맥락 (Codex/Claude로 이어갈 때 유용)

## 권장 스택
**Electron + React + Tailwind** — 어드민이 이미 같은 스택이라 **재작업 0**.
디자인 토큰이 Tailwind/CSS-var 호환이므로 `DESIGN_SYSTEM.md §11` 스니펫을 그대로 적용하면 됩니다.

> ⚠️ `user-app.html`은 **디자인/인터랙션 명세**입니다. 바닐라 HTML/JS이므로 프로덕션엔 그대로 복붙하지 말고 React로 재구현하세요. (값·상태·플로우는 이 파일이 기준)

## 이미 구현 vs 백엔드 연동 필요 (Mock ↔ Real)
프로토타입엔 **데모용 mock**이 섞여 있습니다 — 실제 구현 시 백엔드와 연결하세요:

| UI | 현재(프로토타입) | 실제로 연결할 것 |
|---|---|---|
| 로그인 | 아무 비번이나 통과 | 인증 API (아이디/비번 → 토큰) |
| 프로젝트 리스트 | 샘플 3개 하드코딩(`projects[]`) | 프로젝트 목록 API (이름·버전·크기·설명·실행파일경로) |
| 상태 뱃지(미설치/설치됨/업데이트) | `state` 필드 수동 | 로컬 설치 버전 vs 서버 최신 버전 비교 |
| 다운로드/압축해제 진행률 | 가짜 `setInterval` 애니메이션 | 실제 다운로드/해제 진행 이벤트 스트림 |
| 카드 썸네일 | 그라데이션 placeholder | 실제 썸네일 이미지 URL |
| 폴더 선택/열기 | Electron API 배선만 됨 (브라우저는 토스트) | Electron `dialog`/`shell.openPath` + `ipcMain` 핸들러 |
| 실행하기 | 토스트만 | 실행파일 spawn |
| 버전 기록 테이블 | 샘플 `history[]` | 버전 이력 API |

## 컨펌 필요한 디자인 항목
- **에러 토큰**(`--fg-error #E5484D`/`#FF6B6E`)과 **업데이트 amber 토큰**(`--fg-warning`)은 어드민 디자인시스템에 없어 신규 추가한 값 → 디자인팀 정식 확정 필요.

## 문의 시 참고
- 프레임워크 확정 / 백엔드 스펙 공유 시 `docs/`에 연동 맵(`INTEGRATION.md`)·AI 빌드 프롬프트 추가 가능.
