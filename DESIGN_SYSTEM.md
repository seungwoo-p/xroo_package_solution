# XROO Package Solution — Design System (v3.3)

> 어드민 콘솔(`http://192.168.1.70:5173`)에서 라이브 추출한 디자인 시스템 명세.
> 사용자(End-user) **데스크탑 앱**을 동일 시스템 위에서 일관되게 구축하기 위한 단일 소스(SSOT).
>
> - 추출일: 2026-06-04 (v3.3 — 전 페이지 × 양 테마 round-trip 완료)
> - **검증 상태: ✅ 토큰/컴포넌트 실측 100% 대조 + 4개 페이지(login·projects·users·projects-new) × (dark+light) round-trip 재현 검증** (§부록 C~C-5)
> - 누적 정정: 토큰·컴포넌트 18건(D) + 로그인 6건(E) + 사이드바 4건(G) + 구조 4건(P) + 테이블/풀폼(C-5)
> - 폰트: **Pretendard JP Variable / Pretendard Variable**
> - 모드: `light` / `dark` / `system` (`html.dark` 토글)
> - 브랜드 컬러: **Primary Green `#178551`**
> - 아이콘: **lucide-react** (24px 베이스 그리드, stroke-width 2)
> - 스타일링: **Tailwind CSS + CSS Variables**
> - Typography utility: **`t-*`** 시스템 (`t-title-3`, `t-label-1`, `t-caption-1` 등)

---

## 0. 목차
1. 핵심 디자인 원칙
2. 테마 모드 시스템
3. 컬러 토큰 (Light / Dark) — 전체
4. 타이포그래피 (`t-*` utility + 클래스)
5. Spacing · Radius · Elevation
6. Density 토큰
7. 아이콘 시스템 (lucide-react)
8. 컴포넌트 명세 — **모든 상태** (default / hover / focus / active / disabled / error)
   - 8.1 Button (Primary / Secondary / Ghost / Icon / Disabled)
   - 8.2 Input
   - 8.3 Textarea
   - 8.4 Select / Dropdown
   - 8.5 DatePicker
   - 8.6 Checkbox
   - 8.7 Radio (가이드)
   - 8.8 Switch / Toggle (가이드)
   - 8.9 Segmented Toggle (테마 토글 패턴)
   - 8.10 Card
   - 8.11 Section Card
   - 8.12 Table
   - 8.13 Modal (Dialog)
   - 8.14 Popover (드롭다운 메뉴 / DatePicker 컨테이너)
   - 8.15 Filter Chip (pill 형태)
   - 8.16 Status Badge
   - 8.17 Tag / Count Badge
   - 8.18 Sidebar Navigation
   - 8.19 Pagination
   - 8.20 File Drop Zone
   - 8.21 Tooltip (가이드 — 어드민에 미사용)
   - 8.22 Tab (가이드 — 어드민에 미사용)
   - 8.23 Toast / Notification (가이드)
   - 8.24 Empty State (가이드)
   - 8.25 Loading / Skeleton (가이드)
   - 8.26 Header Bar / Back Button / Version Badge
9. 페이지 레이아웃 패턴
10. **사용자 데스크탑 앱(exe) 적용 가이드** — 프레임워크 중립
11. `globals.css` + `tailwind.config` 복붙 스니펫
12. 부록: 검증된 토큰·utility 매핑표

---

## 1. 핵심 디자인 원칙

1. **Token-first**: 모든 색은 CSS 변수에서. 절대 하드코딩 금지.
2. **Tailwind utility 매핑 = 디자인 토큰**: `bg-bg-brand`, `text-fg-strong`, `border-line-strong` 등 utility 이름이 곧 토큰 이름.
3. **Typography는 `t-*` 클래스로**: 인라인 `text-[16px] font-bold` 대신 `t-body-1` 식으로 사용.
4. **다크는 빛바램이 아닌 재매핑**: `.dark` 시 토큰의 의미는 같고 값만 바뀜.
5. **사이드바는 라이트/다크 무관 항상 다크 톤**: 어드민의 의도된 정체성. 사용자 페이지에선 override 권장.
6. **에러색은 별도 정의 없음**: 어드민은 `--destructive`로 매핑되어 있으나 실제 값은 다크 컬러. → 사용자 페이지에선 **에러용 빨강 토큰을 새로 추가** 권장(섹션 10).
7. **Status는 색이 아닌 텍스트로 구분**: 노출중/예정/만료 뱃지의 dot 색이 전부 동일(`--fg-assistive`). 시각적 구분이 약함 → 사용자 페이지에선 **상태별 컬러 권장**(섹션 10).
8. **아이콘 = lucide-react 16px stroke 2**: 일관성 핵심.
9. **상호작용 = transition-all/colors 200ms**: hover/focus 부드러운 전환.
10. **모달은 backdrop이 투명**: 어드민의 의도 — 깊이감을 그림자만으로 표현. 데스크탑 앱에선 backdrop 디밍 권장(섹션 10).

---

## 2. 테마 모드 시스템

| 항목 | 값 |
|---|---|
| 저장 위치 | `localStorage` |
| 키 | `xroo-theme-mode` |
| 가능 값 | `"light"` · `"dark"` · `"system"` |
| 적용 메커니즘 | `document.documentElement.classList.toggle('dark')` |
| 라이트 클래스 | 없음 (기본 `:root`) |
| 다크 클래스 | `html.dark` |
| 시스템 모드 | `matchMedia('(prefers-color-scheme: dark)')` 결과 반영 |

```ts
type Mode = 'light' | 'dark' | 'system'
function applyTheme(mode: Mode) {
  const isDark = mode === 'dark' || (mode === 'system' && matchMedia('(prefers-color-scheme: dark)').matches)
  document.documentElement.classList.toggle('dark', isDark)
  localStorage.setItem('xroo-theme-mode', mode)
}
// system 모드일 땐 prefers-color-scheme media change 구독 필요
matchMedia('(prefers-color-scheme: dark)').addEventListener('change', () => {
  if (localStorage.getItem('xroo-theme-mode') === 'system') applyTheme('system')
})
```

**Flash 방지 (FOUC)**: `index.html` `<head>` 최상단에 inline 스크립트로 즉시 토글.

---

## 3. 컬러 토큰

### 3.1 Primary Green Scale
| 토큰 | HEX | 용도 |
|---|---|---|
| `--color-primary-50` | `#EAF7F0` | Light brand-subtle bg |
| `--color-primary-100` | `#D6F0E2` | |
| `--color-primary-200` | `#A9E4C5` | |
| `--color-primary-300` | `#6FD3A0` | Dark `--fg-brand-bright` |
| `--color-primary-400` | `#34C381` | Dark `--fg-brand`, Light `--fg-brand-bright` |
| `--color-primary-500` | `#25AC6E` | Dark press, Light hover-step |
| `--color-primary-600` | `#1C9A61` | Dark hover, mid step |
| `--color-primary-700` | `#178551` | **Light primary** (CTA bg, `--bg-brand`) |
| `--color-primary-800` | `#136E44` | Light hover (`--bg-brand-hover`) |
| `--color-primary-900` | `#0F5C39` | Light press (`--bg-brand-press`) |
| `--color-primary-1000` | `#0A4429` | |

### 3.2 CoolGrey Scale
| 토큰 | HEX | 용도 |
|---|---|---|
| `--color-coolgrey-50` | `#F7F7F8` | Light `--bg-secondary` |
| `--color-coolgrey-100` | `#F4F4F5` | Light `--bg-tertiary` |
| `--color-coolgrey-200` | `#EAEBEC` | Light hover bg (secondary btn) |
| `--color-coolgrey-400` | `#C2C4C8` | `--switch-background` |
| `--color-coolgrey-500` | `#AEB0B6` | Light `--fg-disabled` |
| `--color-coolgrey-600` | `#8B8E97` | Sidebar inactive (dark `--fg-assistive` 근사) |
| `--color-coolgrey-700` | `#70737C` | Light `--fg-assistive` |
| `--color-coolgrey-1350` | `#171719` | Light `--bg-sidebar`, `--fg-strong` |

### 3.3 Semantic 토큰 — Light vs Dark 완전 매핑

| 토큰 | Light `:root` | Dark `.dark` | Tailwind utility |
|---|---|---|---|
| `--background` | `#ffffff` | `#111213` | — |
| `--foreground` | `#171719` | `#F0F1F3` | — |
| `--bg-primary` | `#FFFFFF` | `#1C1D20` | `bg-bg-primary` |
| `--bg-secondary` | `#F7F7F8` | `#111213` | `bg-bg-secondary` |
| `--bg-tertiary` | `#F4F4F5` | `#28292E` | `bg-bg-tertiary` |
| `--bg-sidebar` | `#171719` | `#0D0E10` | `bg-bg-sidebar` |
| `--bg-inverse` | `#171719` | `#F0F1F3` | `bg-bg-inverse` |
| `--bg-brand` | `#178551` | `#178551` | `bg-bg-brand` |
| `--bg-brand-hover` | `#136E44` | `#1C9A61` | `bg-bg-brand-hover` |
| `--bg-brand-press` | `#0F5C39` | `#25AC6E` | `bg-bg-brand-press` |
| `--bg-brand-subtle` | `#EAF7F0` | `#0C2B1A` | `bg-bg-brand-subtle` |
| `--fg-strong` | `#171719` | `#F0F1F3` | `text-fg-strong` |
| `--fg-default` | `#2E2F33` | `#C4C6CC` | `text-fg-default` |
| `--fg-subtle` | `#46474C` | `#9B9EA8` | `text-fg-subtle` |
| `--fg-assistive` | `#70737C` | `#82858F` | `text-fg-assistive` |
| `--fg-disabled` | `#AEB0B6` | `#52555F` | `text-fg-disabled` |
| `--fg-inverse` | `#FFFFFF` | — | `text-fg-inverse` |
| `--fg-brand` | `#178551` | `#34C381` | `text-fg-brand`, `border-fg-brand`, `ring-fg-brand` |
| `--fg-brand-bright` | `#34C381` | `#6FD3A0` | `text-fg-brand-bright` |
| `--line-default` | `rgba(112,115,124,0.22)` | `rgba(255,255,255,0.09)` | `border-line-default` |
| `--line-subtle` | `rgba(112,115,124,0.10)` | `rgba(255,255,255,0.05)` | `border-line-subtle` |
| `--line-strong` | `rgba(112,115,124,0.40)` | `rgba(255,255,255,0.18)` | `border-line-strong` |
| `--line-accent` | `#178551` | `#34C381` | `border-line-accent` |
| `--input` | `rgba(112,115,124,0.40)` | `rgba(255,255,255,0.18)` | — |
| `--input-background` | `#ffffff` | `#28292E` | — |
| `--ring` | `#178551` | `#34C381` | — |
| `--primary` | `#178551` | `#34C381` | shadcn alias |
| `--secondary` | `#F4F4F5` | `#28292E` | |
| `--muted` | `#F4F4F5` | `#28292E` | |
| `--accent` | `#EAF7F0` | `#0C2B1A` | |
| `--card` | `#ffffff` | `#1C1D20` | |
| `--popover` | `#ffffff` | `#1C1D20` | |
| `--destructive` | `#171719` | (override 없음) | ⚠️ 실제 파괴 액션 색 아님 |
| `--switch-background` | `#C2C4C8` | (없음) | |

### 3.4 Sidebar 전용 (라이트/다크 동일 정의)
| 토큰 | 값 |
|---|---|
| `--sidebar` | `#171719` |
| `--sidebar-foreground` | `#ffffff` |
| `--sidebar-primary` | `#178551` |
| `--sidebar-primary-foreground` | `#ffffff` |
| `--sidebar-accent` | `rgba(255,255,255,0.06)` |
| `--sidebar-accent-foreground` | `#ffffff` |
| `--sidebar-border` | `rgba(255,255,255,0.08)` |
| `--sidebar-ring` | `#34C381` |

> 다크 모드에서는 sidebar bg가 `--bg-sidebar: #0D0E10`으로 더 어두워짐.

---

## 4. 타이포그래피

### 4.1 폰트 패밀리 (CSS 변수)
```css
--font-sans:    "Pretendard JP Variable", "Pretendard Variable",
                "Pretendard JP", "Pretendard",
                -apple-system, BlinkMacSystemFont, "system-ui",
                "Apple SD Gothic Neo", "Noto Sans KR", sans-serif;
--font-display: var(--font-sans);
--font-mono:    "Pretendard JP Variable", "Pretendard Variable",
                "SF Mono", "JetBrains Mono", ui-monospace,
                Menlo, Consolas, monospace;
--font-size: 16px;
--font-weight-normal: 400;
--font-weight-medium: 500;
```

### 4.2 Typography Utility 클래스 (실측)
어드민은 인라인 `text-[N] font-bold` 대신 의미적 utility를 사용한다.

| 클래스 | font-size | line-height | font-weight | letter-spacing | 용도 (실측 확인) |
|---|---|---|---|---|---|
| `t-title-1` | 32px | 44px | 700 | -0.81px | **페이지 타이틀** (`프로젝트 리스트`, `사용자 관리`, `프로젝트 추가하기` 모두 사용) |
| `t-title-3` | 24px | 32px | 700 | -0.55px | **모달 타이틀** (`사용자 추가`) |
| `t-heading-1` | 22px | 30px | 700 | -0.43px | 페이지 타이틀(중) — 어드민 직접 사용 미확인 |
| `t-heading-2` | 20px | 28px | 700 | -0.24px | **섹션 타이틀**(`기본정보` 등), **카드 타이틀**(`송도 더샵…`) |
| `t-body-1` | 16px | 24px | 500 | 0 | 본문(기본) |
| `t-body-2` | 15px | 22px | 500 | 0 | 본문(소) |
| `t-label-1` | 14px | 20px | 600 | 0 | **사이드바 메뉴 전용** (active/inactive 모두) |
| `t-label-2` | 13px | 18px | 600 | 0 | **필터 칩**, **페이지네이션**, **테이블 ID 강조셀**, **푸터 링크**(오늘/지우기) |
| `t-caption-1` | 12px | 16px | 600 | 0 | **모든 폼 라벨**(`text-fg-default`), **헬퍼 텍스트**(`text-fg-assistive`), **테이블 헤더**, **카운트/버전 뱃지** |
| `t-caption-2` | 11px | 15px | 600 | 0 | **최소 라벨**, **테마 토글 라벨**, 사이드바 카테고리 헤더 |

> `t-title-2`, `t-title-4`, `t-heading-3/4`, `t-body-3`, `t-button-*` 는 어드민에 미정의 (실측 시 기본값 동일). 필요시 확장.

### 4.3 인라인 사이즈 (어드민 패턴)
일부 컴포넌트는 인라인으로 작성:
- 인풋 텍스트: `text-[13px] font-medium`
- 버튼 텍스트: `text-[13px] font-bold` (default), `text-[14px] font-bold` (large CTA), `text-[15px] font-bold` (로그인 등 hero)
- 카운트 뱃지: `text-[11px] font-semibold`

---

## 5. Spacing · Radius · Elevation

### 5.1 Spacing
```css
--space-1: 4px;   --space-2:  8px;   --space-3: 12px;
--space-4: 16px;  --space-5:  20px;  --space-6: 24px;
--space-8: 32px;  --space-10: 40px;  --space-16: 64px;
```
Tailwind 기본 스케일과 호환 (`p-1` = 4px, `p-2` = 8px, ...).

### 5.2 Radius
```css
/* ⚠️ 어드민에 실제 정의된 radius 토큰은 아래 7개뿐 (ground-truth 확인). */
--radius:      0.5rem;   /* 8px - shadcn base */
--radius-6:    6px;      /* 작은 버튼/칩/드롭다운 옵션/뱃지 */
--radius-8:    8px;      /* 일반 버튼, 인풋, 셀렉트 */
--radius-12:   12px;     /* 테이블 컨테이너, popover, datepicker, drop zone */
--radius-16:   16px;     /* 카드, 섹션 카드 (rounded-2xl) */
--radius-24:   24px;     /* 큰 컨테이너 */
--radius-full: 9999px;   /* pill(필터칩), 아바타, dot */
```
> ⚠️ **`--radius-20` 토큰은 존재하지 않음.** 모달 panel은 arbitrary 값 `rounded-[20px]`를 직접 사용 (토큰 미경유).
> Tailwind 매핑: `rounded-md` = 6, `rounded-lg` = 8, `rounded-xl` = 12, `rounded-2xl` = 16, `rounded-[20px]` = 20(arbitrary), `rounded-full` = 9999.

### 5.3 Elevation (Shadow) — Light only
```css
--elevation-1:       0 1px 2px rgba(23,23,23,0.06);
--elevation-2:       0 2px 8px rgba(23,23,23,0.07), 0 1px 2px rgba(0,0,0,0.04);
--elevation-overlay: 0 12px 32px rgba(0,0,0,0.08), 0 0 0 1px rgba(112,115,124,0.10);
```
Tailwind utility: `shadow-1`, `shadow-2`, `shadow-overlay`, `shadow-elevation-2`(카드 hover).

> 다크 모드에선 그림자 대신 **border + bg 대비** 로 깊이 표현.

---

## 6. Density 토큰
```css
--d-page-gutter: 40px;  /* 메인 콘텐츠 좌우 패딩 */
--d-card-pad:    20px;  /* 카드/섹션 내부 패딩 */
--d-form-gap:    16px;  /* 폼 필드 간 수직 간격 */
--d-row-height:  40px;  /* 인풋, 버튼, 테이블 row */
--d-cell-pad-x:  14px;
--d-cell-pad-y:  8px;
```

---

## 7. 아이콘 시스템 — **lucide-react** (확정)

### 7.1 아이콘 라이브러리
- 패키지: `lucide-react` (또는 동급 lucide 패키지 — flutter용은 lucide_icons)
- 모든 SVG: `viewBox="0 0 24 24"`, `fill="none"`, `stroke="currentColor"`, `stroke-width="2"`, `stroke-linecap="round"`, `stroke-linejoin="round"`
- 어드민 사용 예: `lucide-plus`, `lucide-layers`, `lucide-users`, `lucide-monitor`, `lucide-sun`, `lucide-search`, `lucide-calendar`, `lucide-eye`, `lucide-eye-off`, `lucide-chevron-left/right/down`, `lucide-x`, `lucide-edit`(pencil), `lucide-trash-2`, `lucide-refresh-cw`, `lucide-arrow-left`, `lucide-upload`

### 7.2 사이즈 스케일
| 사이즈 | px | 용도 |
|---|---|---|
| xs | 12 | 체크박스 안 마크(별도 인라인 SVG), 작은 인디케이터 |
| sm | 13 | 토글 아이콘(테마 토글: sun/moon/monitor) |
| **base** | **16** | **기본** — 버튼 안, 사이드바, 인풋 안, 칩 |
| md | 20 | 헤더 액션 |
| lg | 24 | 빈 상태(empty state) |
| xl | 32 | 업로드 아이콘 |

### 7.3 컬러 매핑
- 기본: `text-current` 또는 `currentColor` 상속
- 강조: `text-fg-brand-bright` (활성 사이드바 메뉴)
- 보조: `text-fg-assistive` (헬퍼/메타)
- 비활성: `text-fg-disabled`
- 인버스 (CTA 안): `text-white` 또는 `text-fg-inverse`

### 7.4 사용 패턴
```tsx
import { Plus, Users, Layers, Search, Calendar, Eye, EyeOff, ChevronDown, X, Pencil, Trash2, RefreshCw, ArrowLeft, Upload } from 'lucide-react'

<Plus className="w-4 h-4" strokeWidth={2} />           // 16px
<Layers className="w-4 h-4 text-fg-brand-bright" />    // 사이드바 active
<Search className="w-4 h-4 text-fg-assistive" />       // 인풋 prefix
```

> 데스크탑 앱(Tauri/Electron)이어도 React 사용 시 `lucide-react` 그대로 사용. Flutter 사용 시 `lucide_icons` 또는 `flutter_lucide` 패키지.

---

## 8. 컴포넌트 명세 — **모든 상태 실측**

> 표기 규약:
> - 다크/라이트 동일 값은 한 줄로 표기.
> - 다를 때만 `Light: X / Dark: Y` 로 표기.
> - 모든 측정값은 실제 DOM에서 추출. 추정 없음.

### 8.1 Button

| Variant | Tailwind 클래스 (실제 사용) | Default bg | Hover bg | Color |
|---|---|---|---|---|
| **Primary CTA** | `h-10 px-5 rounded-lg gap-2 font-bold text-[14px] bg-bg-brand text-white hover:bg-bg-brand-hover active:bg-bg-brand-press` (px는 18~20px 혼용: 사이드바 CTA `px-5`=20, 헤더 CTA `px-[18px]`=18) | `--bg-brand` | `--bg-brand-hover` | white |
| **Secondary** | `h-10 px-[14px] rounded-lg t-label-2 font-bold bg-bg-tertiary text-fg-strong hover:bg-grey-200 dark:hover:bg-white/10` | `--bg-tertiary` | L: `#EAEBEC` / D: `rgba(255,255,255,0.1)` | `--fg-strong` |
| **Ghost (icon)** | `w-9 h-9 rounded-md inline-flex items-center justify-center hover:bg-bg-tertiary` | transparent | `--bg-tertiary` | `--fg-strong` 또는 `--fg-assistive` |
| **Disabled** | `bg-bg-tertiary text-fg-disabled cursor-not-allowed` | `--bg-tertiary` | — | `--fg-disabled` |
| **Destructive (선택삭제 활성)** | `bg-transparent text-fg-strong border-transparent hover:bg-bg-tertiary` | transparent | `--bg-tertiary` | `--fg-strong` |

#### Primary CTA 전체 상태 (실측 다크):
| 상태 | bg | color | height | padding | radius | font |
|---|---|---|---|---|---|---|
| default | `#178551` | white | 40px | `0 20px` | 8px | 14px / 700 |
| hover | `#1C9A61` (Dark) / `#136E44` (Light) | white | — | — | — | — |
| press | `#25AC6E` (Dark) / `#0F5C39` (Light) | white | — | — | — | — |
| focus | + ring 3px `--fg-brand/20` | — | — | — | — | — |
| disabled | `--bg-tertiary` | `--fg-disabled` | — | — | — | cursor-not-allowed |

#### Secondary 전체 상태 (실측 다크):
| 상태 | bg | color |
|---|---|---|
| default | `#28292E` | `#F0F1F3` |
| hover (Light) | `#EAEBEC` | `#171719` |
| hover (Dark) | `rgba(255,255,255,0.10)` | `#F0F1F3` |
| disabled | `#28292E` | `#52555F` |

#### Ghost Icon Button (모달 X, 새로고침 등):
- 36×36 (w-9 h-9) 또는 32×32 (w-8 h-8)
- radius 6px (`rounded-md`)
- color `--fg-assistive` (보조) 또는 `--fg-strong` (강조)
- hover bg `--bg-tertiary`

#### 사이즈 스케일:
| 사이즈 | height | padding-x | font |
|---|---|---|---|
| sm | 32px | 12px | 13px / 700 |
| md (default) | 40px | 14–20px | 13–14px / 700 |
| lg (hero/CTA) | 44–48px | 20–24px | 15px / 700 |

### 8.2 Input

#### Default class (실측):
```
w-full h-10 rounded-lg outline-none transition-all
text-[13px] font-medium text-fg-strong
border border-line-strong
focus:border-fg-brand focus:ring-[3px] focus:ring-fg-brand/20
bg-bg-primary
px-[14px]
```

#### 상태별 측정값 (Dark / Light 모두 적용):
| 상태 | border | bg | color | shadow (ring) |
|---|---|---|---|---|
| default | `--line-strong` | `--bg-primary` | `--fg-strong` | none |
| hover | (변화 없음) | (변화 없음) | (변화 없음) | none |
| focus | `--fg-brand` | `--bg-primary` | `--fg-strong` | 3px `--fg-brand/20` |
| filled (focused) | `--fg-brand` | `--bg-primary` | `--fg-strong` | 3px `--fg-brand/20` |
| placeholder color | `--fg-assistive` | — | — | — |
| disabled (가이드) | `--line-default` | `--bg-tertiary` | `--fg-disabled` | none, cursor-not-allowed |
| **error** ⚠️ | (어드민에 미정의) | — | — | — |

#### 검색 인풋 (좌측 아이콘):
```
pl-10 pr-[14px]    /* 좌측 40px 아이콘 공간 */
```
- 아이콘: `lucide-search` 16px, `text-fg-assistive`, 좌측 14px offset

#### 비밀번호 인풋 + 토글:
- `pr-14` (우측 56px 아이콘 공간)
- 토글 버튼: 36×36 ghost icon, `lucide-eye` / `lucide-eye-off`, `text-fg-assistive`

#### 필드 라벨 (실측 — 모달/풀폼 공통):
```
t-caption-1 text-fg-default     /* = 12px / 600, color --fg-default */
```
- ⚠️ **폼 라벨은 `t-label-1`(14px)이 아니라 `t-caption-1`(12px) `text-fg-default`.** (모달 ID/비밀번호/설명 + 풀폼 프로젝트명/노출시작 모두 동일하게 실측 확인)
- 필수 표시 `*`: 라벨 앞에 붙으며 색은 `text-fg-strong` (빨강/브랜드 아님)
- 라벨 ↔ 인풋 간격: 약 6~8px

#### ⚠️ **에러 상태 추가 필요** (어드민 미정의):
사용자 페이지에선 다음을 추가 권장:
```
data-error:border-fg-error data-error:ring-fg-error/20
text-fg-error (error message)
```
새 토큰: `--fg-error` = `#E5484D` (light) / `#FF6B6E` (dark) ← 디자인팀 컨펌 필요.

### 8.3 Textarea
```
w-full rounded-lg outline-none transition-all p-3 resize-none
text-[13px] font-medium text-fg-strong
bg-bg-primary border border-line-strong
focus:border-fg-brand focus:ring-[3px] focus:ring-fg-brand/20
```
- min-height ~100px (3행)
- padding 12px
- **카운터** (우측 하단): `t-caption-1 text-fg-assistive`, 위치는 textarea 외부 absolute `bottom-2 right-3`

### 8.4 Select / Dropdown

#### Trigger (실측 — 인풋보다 컴팩트):
```
h-8 pl-3 pr-8 rounded-lg border border-line-strong
bg-bg-primary hover:bg-bg-tertiary text-fg-strong
text-[13px] font-medium inline-flex items-center whitespace-nowrap
cursor-pointer transition-colors relative
```
- 32×N, radius 8px(`rounded-lg`), `pl-3 pr-8`(우측 8px chevron 공간)
- hover: `bg-bg-tertiary`
- 우측 `lucide-chevron-down` 16px, `text-fg-assistive`

#### Popover (펼친 메뉴):
```
absolute z-50 mt-1 bg-bg-primary
rounded-lg border border-line-default shadow-overlay
py-1 min-w-[trigger-width]
```
- bg `--bg-primary`, border `--line-default`, radius 8px
- padding `4px 0`
- shadow `--elevation-overlay`
- z-index 50

#### Option:
```
flex items-center justify-between px-3 py-[7px]
text-[13px] font-medium cursor-pointer select-none whitespace-nowrap
text-fg-default hover:bg-bg-tertiary
```
| 상태 | bg | color | 우측 아이콘 |
|---|---|---|---|
| default | transparent | `--fg-default` | — |
| hover | `--bg-tertiary` | `--fg-default` | — |
| selected | `--bg-brand-subtle` (`#0C2B1A` dark / `#EAF7F0` light) | `--fg-brand` | `lucide-check` 12px |
| disabled | transparent | `--fg-disabled` | cursor-not-allowed |

### 8.5 DatePicker

#### Trigger:
- Input + 우측 캘린더 아이콘 (`lucide-calendar` 16px, `text-fg-assistive`)
- `pr-10` (우측 40px 아이콘 공간)
- placeholder `YYYY-MM-DD`

#### Popover (캘린더):
```
absolute z-30 mt-2 w-[280px] p-3
bg-bg-primary rounded-xl border border-line-default shadow-overlay
```
- 280×308, radius 12px, padding 12px

#### Header (월/년 + 좌우 화살표):
- 가운데 라벨: `t-label-1 tabular text-fg-strong` ("2026년 6월")
- 화살표 버튼: `w-7 h-7 rounded-md hover:bg-bg-tertiary`
  - 아이콘 `lucide-chevron-left` / `right`, 16px

#### 요일 헤더:
```
text-center t-caption-1 py-1 text-fg-assistive
```
- 일/월/화/수/목/금/토 (한국어)

#### Day Cell:
```
h-8 rounded-md transition-colors tabular text-[13px] font-medium
text-fg-strong hover:bg-bg-brand-subtle
border border-transparent
```
| 상태 | border | bg | color |
|---|---|---|---|
| default | transparent | transparent | `--fg-strong` |
| hover | transparent | `--bg-brand-subtle` | `--fg-strong` |
| **today** | `--fg-brand` | transparent | `--fg-strong` |
| **selected** | `--fg-brand` | `--bg-brand` (가이드) | white |
| **other-month** (dimmed) | transparent | transparent | `--fg-disabled` |
| disabled | transparent | transparent | `--fg-disabled` cursor-not-allowed |

#### Footer 액션 ("오늘" / "지우기"):
```
t-label-2 text-fg-brand
```

### 8.6 Checkbox

```
w-[18px] h-[18px] rounded-[4px] inline-flex items-center justify-center
transition-all shrink-0
bg-bg-primary border-[1.5px] border-line-strong
```

| 상태 | bg | border | 체크마크 |
|---|---|---|---|
| **unchecked** | `--bg-primary` | 1.5px `--line-strong` | — |
| **checked** | `--bg-brand` (#178551) | 1.5px `--bg-brand` | 인라인 SVG 12×12, viewBox `0 0 12 12`, path `M2 6l3 3 5-5`, stroke white 2.5px |
| **indeterminate** (가이드) | `--bg-brand` | 1.5px `--bg-brand` | 가로선 8×2px white center |
| **disabled** | `--bg-tertiary` | 1.5px `--line-default` | cursor-not-allowed, opacity 0.5 |
| focus (가이드) | + ring 3px `--fg-brand/20` | — | — |

### 8.7 Radio (어드민 미사용 — 가이드)
- 18×18, radius `rounded-full`
- bg `--bg-primary`, border 1.5px `--line-strong`
- checked: 내부 8×8 dot `bg-bg-brand`, border 1.5px `--bg-brand`

### 8.8 Switch / Toggle (어드민에 직접 사용 없음 — 가이드)
- track: 36×20, radius full
  - off bg: `--switch-background` (#C2C4C8)
  - on bg: `--bg-brand`
- thumb: 16×16 circle, white, shadow `--elevation-1`, transition `transform 150ms`
- disabled: opacity 0.5

### 8.9 Segmented Toggle (테마 토글 패턴)
어드민의 좌하단 "시스템/라이트/다크" 패턴.

#### Container:
```
inline-flex p-0.5 rounded-lg
bg-bg-tertiary  /* dark는 추가로 약간 어두운 톤 */
```

#### Segment:
- 64×36, radius 6px
- font 11px / 600 (`t-caption-2`)
- 아이콘 13px + 라벨 세로 정렬
- **inactive**: bg transparent, color `--fg-assistive`
- **active**: bg `rgba(255,255,255,0.15)` (dark) / white + `shadow-1` (light), color `--fg-strong`

라벨: 컨테이너 위에 "테마" — `t-caption-2 text-fg-assistive px-2`

### 8.10 Card (콘텐츠 카드) — 실측 정정 (v3.1)

루트 태그는 **`<article>`** (div 아님):
```
group relative rounded-2xl bg-bg-primary overflow-hidden
transition-all cursor-pointer border border-line-default
hover:shadow-elevation-2 [hover:-translate-y-0.5]
```
| 속성 | 값 |
|---|---|
| bg | `--bg-primary` |
| border | 1px `--line-default` |
| radius | 16px (`rounded-2xl`) |
| 크기 | 318×340 (xl 4-col 기준) |
| hover | `shadow-elevation-2` (+ 살짝 떠오름) |

#### 내부 구조 (실측 HTML 기준):
```html
<article class="group relative rounded-2xl bg-bg-primary overflow-hidden border border-line-default hover:shadow-elevation-2 ...">
  <!-- 1) 썸네일: aspect-video(16:9), 178px tall @318w, 실제 <img> 풀블리드 -->
  <div class="aspect-video overflow-hidden w-full"><img .../></div>

  <!-- 2) 메타 p-4 -->
  <div class="p-4">
    <div class="mb-2"><span class="badge(§8.16)">●노출중</span></div>   <!-- 뱃지 wrapper mb-2 -->
    <h3 class="t-heading-2 mb-1">제목</h3>                              <!-- 20/700, mb-1 -->
    <p class="t-caption-1 mb-3 truncate text-fg-assistive">설명</p>     <!-- ⚠️ truncate(한 줄), mb-3 -->
    <dl class="grid grid-cols-3 gap-2">                                <!-- 메타 3-col, gap-2(8px) -->
      <div class="flex flex-col">
        <dt class="t-caption-1 text-fg-assistive">버전</dt>
        <dd class="mt-0.5 tabular font-bold t-label-2 text-fg-strong">v2</dd>  <!-- 값: tabular, 13/700 -->
      </div>
      ... 크기(MB) / 권한 계정
    </dl>
  </div>

  <!-- 3) hover 액션 toolbar: 우하단 absolute, group-hover로 fade-in -->
  <div class="absolute bottom-2.5 right-2.5 flex items-center gap-1 bg-bg-primary
              rounded-lg p-1 border border-line-default shadow-elevation-1
              opacity-0 group-hover:opacity-100 pointer-events-none group-hover:pointer-events-auto transition-opacity">
    <button aria-label="수정" class="w-8 h-8 inline-flex items-center justify-center ...">✎</button>
  </div>
</article>
```
- ⚠️ **P2/P3 정정**: ① 썸네일은 `aspect-video`(16:9) + 실제 `<img>`, ② 설명은 **`truncate`**(한 줄 말줄임 — 줄바꿈 금지), ③ 메타는 `<dl class="grid grid-cols-3 gap-2">`, ④ **hover 시 우하단에 수정 액션 toolbar**(group-hover:opacity-100).

> 그리드(이 카드들의 컨테이너) 반응형은 §9.2 참조: `grid gap-4 grid-cols-2 md:grid-cols-3 xl:grid-cols-4`.

### 8.11 Section Card (풀폼 페이지)

```
/* 실측 클래스 (그대로) */
rounded-2xl bg-bg-primary border border-line-default p-[var(--d-card-pad)]
```
- bg `--bg-primary`, border `--line-default`, radius 16px(`rounded-2xl`), padding 20px (`p-[var(--d-card-pad)]`)
- 섹션 타이틀: `t-heading-2` (margin 없음 — 타이틀 자체엔 mb 없음), 타이틀↔본문 간격 16px
- 섹션 내부 레이아웃: `flex flex-col gap-4` (16px) / 2열 필드 그리드 `grid grid-cols-2 gap-4`
- ⚠️ **섹션 카드 간 간격**: 상위 래퍼 `flex flex-col gap-4` = **16px** (gap-6/24px 아님 — 실측 정정)
- ⚠️ **풀폼 푸터(취소/저장)**: `flex items-center justify-end gap-2 mt-6 pt-5 border-t border-line-default` (상단 구분선 + mt-6/pt-5)
- 필드 라벨: `t-caption-1 text-fg-default`(12px), 필수 `*`는 `text-fg-strong`
- 날짜 인풋: 표준 인풋 + `tabular`(tabular-nums) + `pr-10`, 우측 달력버튼 `absolute right-2 w-8 h-8 rounded-md text-fg-subtle hover:bg-bg-tertiary`

### 8.12 Table

#### Container:
```
bg-bg-primary border border-line-default rounded-xl overflow-hidden
```
- bg `--bg-primary`, border `--line-default`, radius 12px

#### Thead (column header):
- thead row 클래스: `border-b border-line-strong` (헤더 하단 보더 = `--line-strong`)
- height 44px
- bg transparent (실측 확인)
- 헤더 셀(정렬): `text-left px-3 py-2.5 t-caption-1 text-fg-assistive` (= 12px / 600, color `--fg-assistive`), padding `10px 12px`
- 체크박스 헤더 셀: `px-3 py-2.5 w-10`
- 정렬 가능: 우측에 `lucide-arrow-up-down` 또는 `↕` 12px, `text-fg-assistive`
- 정렬 활성: 아이콘만 `lucide-arrow-up` / `lucide-arrow-down`, color `--fg-strong`

#### Tbody row:
```
group transition-colors border-b border-line-subtle
h-[var(--d-row-height)] hover:bg-bg-tertiary
```
- height 40px (`h-[var(--d-row-height)]`)
- border-bottom `1px solid --line-subtle`
- ⚠️ hover bg: **`hover:bg-bg-tertiary` (라이트/다크 공통)** — 다크에서도 `--bg-tertiary`(#28292E), `rgba(255,255,255,0.04)` 아님
- 셀 padding `0 12px` (`px-3`)
- ⚠️ **모든 본문 셀 = `t-label-2`(13px)** (16px 아님). 컬럼별 색/변형 (실측):
  - ID(강조): `t-label-2 font-semibold text-fg-strong`
  - 설명: `t-label-2 text-fg-default`
  - 생성일(타임스탬프): `t-label-2 font-mono text-fg-subtle` — **모노스페이스**
  - 액션 셀: `px-3 text-right`, 수정/삭제 아이콘버튼은 행 hover 시 노출(`opacity-0 → group-hover`)
- /users 컬럼 폭(실측, 1320 컨테이너): 체크박스 `w-10`(40) / ID 240 / 설명 367 / 생성일 427 / 액션 242
- 정렬 가능 헤더(ID·생성일)엔 우측 정렬 아이콘 표시

#### Row Actions (hover 시 노출 — 어드민 패턴):
- 우측 끝에 아이콘 버튼 2개 (수정 `lucide-pencil` / 삭제 `lucide-trash-2`)
- 각 32×32 ghost, opacity 0 → row hover 시 opacity 1
- color `--fg-strong`

#### Selection Toolbar (어드민 패턴):
체크박스 선택 시 상단 액션 영역에 "N개 선택됨" 표시 + 활성화된 액션 버튼 노출.
```
text-fg-strong t-label-2  /* "N개 선택됨" */
```

### 8.13 Modal (Dialog)

#### Panel:
```
w-full bg-bg-primary rounded-[20px] shadow-overlay max-w-[480px] p-7
```
- bg `--bg-primary`
- radius **20px** — arbitrary `rounded-[20px]` 사용 (`--radius-20` 토큰은 없음). 카드(16px)보다 큰 라운드
- max-width 480px (`max-w-[480px]`, 기본), 가이드: sm 360 / md 480 / lg 640 / xl 800
- padding 28px (`p-7`)
- shadow `--elevation-overlay`
- 다크 모드: shadow 효과는 borderless하지만 `--elevation-overlay`의 1px ring이 보더 역할

#### Backdrop:
- ⚠️ 어드민 실측: `position: fixed; background: rgba(0,0,0,0); z-index: 2147483646;` — **투명**
- 가이드 (사용자 페이지 권장): `bg-black/40` 또는 `bg-black/50` + optional `backdrop-blur-sm`

#### Header:
- 타이틀: `t-title-3` (24px / 700)
- 우측 X 버튼: `w-9 h-9 rounded-md hover:bg-bg-tertiary`, `lucide-x` 16px
- 헤더와 본문 사이 간격: 16–20px

#### Body:
- 폼 필드 간격: `space-y-4` (16px = `--d-form-gap`)
- ⚠️ 라벨: `t-caption-1 text-fg-default` (**12px** / 600) — `t-label-1`(14px) 아님 (실측 확인)

#### Footer:
- 우측 정렬 (`justify-end`)
- 버튼 간격: `gap-2` (8px)
- 취소(Secondary) + 저장(Primary)

#### 사이즈 variants (가이드):
| 사이즈 | max-width | 용도 |
|---|---|---|
| sm | 360 | 확인/취소 다이얼로그 |
| md | 480 | **기본** — 폼 입력 |
| lg | 640 | 상세 폼, 다단 |
| xl | 800 | 미리보기, 큰 콘텐츠 |

### 8.14 Popover (드롭다운/캘린더 공통 컨테이너)

기본 패턴:
```
absolute z-{30|40|50} bg-bg-primary
rounded-{lg|xl} border border-line-default shadow-overlay
p-{1|3}
```
- bg `--bg-primary`
- border `--line-default`
- shadow `--elevation-overlay`
- radius 8 (small menu) ~ 12 (calendar/large)
- z-index 30 (date), 50 (select)

### 8.15 Filter Chip (pill 형태)

```
/* 실측 클래스 (그대로) */
h-8 px-4 rounded-full t-label-2 transition-colors border
/* inactive */ border-line-strong text-fg-default hover:bg-bg-tertiary
/* active   */ bg-bg-brand text-white border-bg-brand
```
- height 32px (`h-8`), padding `0 16px` (`px-4`), radius full (`rounded-full`)
- 폰트: **`t-label-2`** = 13px / 600

| 상태 | bg | border | color | 카운트 색 |
|---|---|---|---|---|
| **inactive** | transparent | 1px `--line-strong` | `--fg-default` | `--fg-assistive` |
| inactive hover | `--bg-tertiary` | 1px `--line-strong` | `--fg-default` | `--fg-assistive` |
| **active** | `--bg-brand` (#178551) | 1px `--bg-brand` | white | `white/70` (rgba 70%) |
| disabled (가이드) | transparent | 1px `--line-default` | `--fg-disabled` | `--fg-disabled` |

#### 카운트 뱃지 (칩 내):
- 11px / 600 (`text-[11px] font-semibold`)
- 텍스트 옆 4px gap
- 색: inactive `--fg-assistive` / active `white/70`

### 8.16 Status Badge (카드 안)

#### 어드민 실측 구조:
```html
<span class="inline-flex items-center rounded-md font-bold gap-1.5
             px-2 py-[3px] text-[11px] leading-[1.36]
             bg-bg-tertiary text-fg-subtle">
  <span class="inline-block w-1.5 h-1.5 rounded-full bg-fg-assistive"></span>
  노출중
</span>
```

| 속성 | 값 |
|---|---|
| bg | `--bg-tertiary` |
| color | `--fg-subtle` |
| radius | 6px (`rounded-md`) |
| padding | 8px 3px (`px-2 py-[3px]`) |
| font | 11px / 700 (`font-bold`) |
| line-height | 1.36 |
| gap | 6px |
| dot | 6×6 (`w-1.5 h-1.5`), radius full, bg `--fg-assistive` |

> ⚠️ **모든 상태(노출중/예정/만료)가 같은 색** — 어드민의 약점.

#### ✅ **사용자 페이지 권장 variants** (확장):
| status | bg | color | dot |
|---|---|---|---|
| `success` (노출중/진행중) | `--bg-brand-subtle` | `--fg-brand` (light) / `--fg-brand-bright` (dark) | `--fg-brand` |
| `warning` (예정) | `bg-amber-50` light / `bg-amber-900/30` dark | `text-amber-700` / `text-amber-300` | `bg-amber-500` |
| `neutral` (만료) | `--bg-tertiary` | `--fg-subtle` | `--fg-assistive` |
| `info` | `bg-blue-50` / `bg-blue-900/30` | `text-blue-700` / `text-blue-300` | `bg-blue-500` |
| `danger` | `bg-red-50` / `bg-red-900/30` | `text-red-700` / `text-red-300` | `bg-red-500` |

### 8.17 Tag / Count Badge (분류 라벨)

#### Tag (분류용 — 예: 카테고리 'VR', '교육'):
```
inline-flex items-center rounded-md
px-2 py-[2px] t-caption-1
bg-bg-tertiary text-fg-default
```

#### Count Badge — 페이지 타이틀 옆 "N개" (실측):
```
inline-flex items-center px-2 h-6 rounded-md bg-bg-tertiary text-fg-strong t-caption-1
```
- ⚠️ 단순 inline 텍스트가 아니라 **bg 있는 pill**: h-6(24px), px-2(8px), `rounded-md`(6px), `bg-bg-tertiary`, `text-fg-strong`, `t-caption-1`(12/600)
- **버전 뱃지(`v1`)도 완전히 동일한 컴포넌트** (§8.26 참조)

#### Count 숫자 (칩 안):
- `text-[11px] font-semibold`, inactive `text-fg-assistive` / active `text-white/70`

### 8.18 Sidebar Navigation

#### Container (실측):
```
w-60 shrink-0 h-screen sticky top-0 flex flex-col
bg-bg-sidebar text-fg-inverse
```
- ⚠️ 너비: **`w-60` = 240px** (216px 아님 — 216은 내부 nav item 폭 = 240 − 좌우 패딩)
- `shrink-0`, `h-screen sticky top-0` (스크롤 시 고정)
- bg: 항상 다크 (`--bg-sidebar`) — 라이트 모드도 동일
- 라이트: `#171719`, 다크: `#0D0E10`

#### ⚠️ 섹션별 패딩 구조 (실측 — P4): `aside` 자체 패딩은 **0**. 4개 섹션이 각자 패딩을 가짐:
1. **브랜드+유저 블록**: `px-5 pt-7 pb-6` (좌우 20 / 상 28 / 하 24)
2. **CTA 블록**: `px-3 pb-3` (좌우 12 / 하 12)
3. **`nav`**: `flex-1 px-3 flex flex-col gap-0.5 overflow-y-auto` (좌우 12, **항목 간격 gap-0.5 = 2px**, flex-1로 하단 밀어냄)
4. **하단 블록**: `p-3 border-t border-white/[0.08] flex flex-col gap-2` (패딩 12, **상단 보더** `rgba(255,255,255,0.08)`, gap 8)
> 브랜드는 px-5(20), 메뉴/CTA/하단은 px-3(12) — 브랜드가 살짝 더 들여쓰기됨 (의도된 디자인).

#### Brand 영역 (실측 정정 — v3.1):
- ⚠️ **워드마크**: SVG 로고 `shrink-0 text-white h-6 w-auto` (높이 24px, 폭 ~80px) — 텍스트 아님
- ⚠️ **브랜드 캡션 "XROO PACKAGE SOLUTION"**: `mt-3 text-[10px] font-bold tracking-[0.1em] text-fg-brand-bright`
  - = **10px / 700, letter-spacing 0.1em, color `--fg-brand-bright`(그린!)** — `t-caption-2 text-fg-assistive`(회색) 아님 (G3)
- ⚠️ **콘솔 타이틀 "관리자 콘솔"**: `mt-1.5 text-[22px] font-bold leading-[1.25] tracking-[-0.01em] text-white`
  - = **22px / 700 white** — `t-label-1`(14px) 아님 (G1)
- ⚠️ **서브 "admin"**: `mt-1 text-[12px] font-medium text-grey-600`
  - = **12px / 500, color `text-grey-600`(#8B8E97)** — `t-caption-1 text-fg-assistive` 아님 (G2)

#### 사이드바 내 CTA ("+ 프로젝트 추가") (실측):
```
inline-flex items-center justify-center gap-2 rounded-lg font-bold transition-colors
h-10 text-[14px] w-full px-5 bg-bg-brand text-white
hover:bg-bg-brand-hover active:bg-bg-brand-press
```
- 풀너비(`w-full`) 그린 버튼, h-10, `px-5`(20px), `text-[14px] font-bold`(700)
- 아이콘 `lucide-plus` 16px

#### 카테고리 헤더 (예: "테마"):
```
t-caption-2 text-grey-600 px-2
```
- 11px / 600, color **`text-grey-600`** (= coolgrey-600 `#8B8E97`)
- 좌측 패딩 8px

#### Nav Item (실측):
```
/* inactive */
relative flex items-center gap-3 px-3 h-10 rounded-lg transition-colors
t-label-1 text-left text-grey-600 hover:bg-white/[0.04]
/* active */
... bg-white/[0.06] text-white
```
> ⚠️ **사이드바는 항상 다크 톤이라 테마-의존 토큰(fg-*)이 아니라 고정 grey 스케일(`text-grey-600` = `--color-coolgrey-600` = `#8B8E97`)을 씀.** (다크 `--fg-assistive`는 `#82858F`로 다른 값 — 혼동 금지)

| 상태 | bg | color (텍스트) | 아이콘 color |
|---|---|---|---|
| inactive | transparent | `text-grey-600` (`#8B8E97`) | `text-current` (= `#8B8E97`) |
| hover | `bg-white/[0.04]` | `#8B8E97` (변화 없음) | — |
| **active** | `bg-white/[0.06]` (= `--sidebar-accent`) | `text-white` | **`text-fg-brand-bright`** — active 아이콘만 SVG에 직접 클래스 (`#6FD3A0` dark / `#34C381` light) |

> 어드민의 핵심 패턴: **active 시 텍스트는 흰색이지만 아이콘은 브랜드 그린 (밝은 톤).** 아이콘 색 강조가 active 표시의 핵심. (예: `<Layers className="... text-fg-brand-bright" />`)

#### 좌하단 영역:
- 테마 토글 (§8.9)
- 로그아웃 버튼 (Ghost, `text-fg-assistive hover:text-white`)

### 8.19 Pagination

#### Container 패턴:
```
flex items-center justify-between
```
- 좌측: 페이지 사이즈 Select (`h-8`)
- 우측: 화살표 + 번호 그룹 (`gap-1`)

#### Page Number Button (실측):
```
/* active */   min-w-8 h-8 px-2 rounded-md t-label-2 transition-colors bg-bg-brand text-white
/* inactive */ min-w-8 h-8 px-2 rounded-md t-label-2 transition-colors text-fg-subtle hover:bg-bg-tertiary
```
| 상태 | bg | color | size | radius |
|---|---|---|---|---|
| inactive | transparent | `--fg-subtle` | `min-w-8` 32px × h-8 32px | 6px |
| hover | `--bg-tertiary` | `--fg-strong` | — | — |
| **active** | `--bg-brand` | white | — | — |
- font: `t-label-2` = 13px / 600

#### Arrow Button (‹ ›) (실측):
```
h-8 px-3 rounded-md t-label-2 text-fg-subtle disabled:opacity-40
```
- h-8(32px), px-3(12px → 렌더 ~29px 폭), radius 6px, ghost
- color `--fg-subtle`
- ⚠️ 비활성 (첫/마지막 페이지): **`disabled:opacity-40` (0.4)**, 0.5 아님
- 아이콘 `lucide-chevron-left` / `right` 16px

### 8.20 File Drop Zone (실측)

```
flex flex-col items-center justify-center rounded-xl py-8 transition-colors
border-[1.5px] border-dashed border-line-strong cursor-pointer
```
- ⚠️ border: **`border-[1.5px]` dashed `--line-strong`** (1px도 2px도 아닌 1.5px)
- radius 12px(`rounded-xl`), padding `py-8`(상하 32px, 좌우 0 — 내용 중앙정렬)
- 중앙 정렬 (`items-center justify-center`)
- 아이콘: `lucide-upload`, `text-fg-assistive`
- 안내 텍스트 + 하위 보조 설명: 보조는 `t-caption-1 text-fg-assistive`
- ⚠️ "선택" 강조 링크: **`underline text-fg-brand`** (`text-fg-brand-bright` 아님 → 라이트에서 `#178551`, 다크에서 `#34C381`), 항상 underline

#### 상태:
| 상태 | border | bg | 비고 |
|---|---|---|---|
| default | `border-[1.5px]` dashed `--line-strong` | transparent | 실측 |
| hover/dragover (가이드) | dashed `--fg-brand` | `--bg-brand-subtle` | JS dragover 시 권장 — 정적 클래스엔 hover 보더 변화 없음 |
| disabled (가이드) | dashed `--line-default` | transparent, opacity 0.5 | |

### 8.21 Tooltip (어드민 미사용 — 권장 가이드)

```
absolute z-50 px-2 py-1
bg-bg-inverse text-fg-inverse  /* 라이트 모드에선 검정, 다크에선 화이트 */
rounded-md t-caption-1
shadow-1
pointer-events-none
```
- 라이트: bg `--bg-inverse` (`#171719`), color white
- 다크: bg `--bg-inverse` (`#F0F1F3`), color `#171719`
- 화살표: 4×4 회전된 사각 — 또는 생략
- 표시 지연: 500ms (hover delay)
- max-width: 240px

### 8.22 Tab (어드민 미사용 — 권장 가이드)

#### Bar:
```
flex border-b border-line-default
```

#### Tab Item:
```
px-4 h-10 inline-flex items-center
t-label-1 text-fg-subtle
border-b-2 border-transparent
hover:text-fg-strong transition-colors
```
| 상태 | color | border-bottom |
|---|---|---|
| inactive | `--fg-subtle` | transparent |
| hover | `--fg-strong` | transparent |
| **active** | `--fg-brand` | 2px `--fg-brand` |
| disabled | `--fg-disabled` | transparent |

### 8.23 Toast / Notification (어드민 미사용 — 권장 가이드)
권장 라이브러리: **sonner** (React) 또는 동급.

```
bg-bg-primary border border-line-default
rounded-xl shadow-overlay
p-4 min-w-[320px] max-w-[480px]
```
- Position: top-right 또는 bottom-right (fixed)
- 아이콘 + 타이틀(`t-label-1`) + 본문(`t-caption-1 text-fg-assistive`)
- 종류별 좌측 stripe 4px:
  - success: `bg-bg-brand`
  - warning: `bg-amber-500`
  - error: `bg-red-500`
  - info: `bg-blue-500`

### 8.24 Empty State (어드민 미사용 — 권장 가이드)

```
flex flex-col items-center justify-center
gap-3 py-16
```
- 아이콘: 32–48px, `text-fg-disabled` (또는 `text-fg-assistive`)
- 타이틀: `t-heading-2 text-fg-strong`
- 설명: `t-body-1 text-fg-subtle text-center max-w-xs`
- 액션 버튼 (선택적): Primary CTA

### 8.25 Loading / Skeleton

#### Spinner:
- `lucide-loader-2` with `animate-spin`
- 사이즈: 16/20/24px
- color: `text-fg-brand-bright` 또는 `text-fg-assistive`

#### Skeleton:
```
animate-pulse rounded-md
bg-bg-tertiary
```
- bg: `--bg-tertiary` (다크에선 더 어두운 톤)
- 카드/리스트 모양에 맞춰 사이즈 조정

#### Page Loading:
- 중앙 정렬 spinner 24px + 라벨(`t-label-2 text-fg-assistive`)

### 8.26 Header Bar / Back Button / Version Badge

#### Page Header:
```
flex items-center justify-between
```
- 좌측: (Back button) + 타이틀 + 카운트 뱃지
- ⚠️ **페이지 타이틀 = `t-title-1 truncate` (32px / 700)** — `t-title-3`/`t-heading-1` 아님 (콘솔 메인·풀폼 모두 t-title-1 실측 확인)
- 카운트 뱃지: §8.17 (pill `h-6 px-2 rounded-md bg-bg-tertiary text-fg-strong t-caption-1`)
- 우측: 액션 그룹 (새로고침 등)

#### Back Button (← "이전 화면으로") (실측):
```
w-9 h-9 inline-flex items-center justify-center rounded-lg shrink-0
border border-line-default bg-bg-primary text-fg-default
hover:bg-bg-tertiary transition-colors
```
- 36×36 (`w-9 h-9`), radius 8px(`rounded-lg`)
- bg `--bg-primary`, border 1px `--line-default`
- 아이콘 `lucide-arrow-left` 16px, `text-fg-default`
- hover: bg `--bg-tertiary`

#### Refresh Icon Button (실측):
```
w-10 h-10 inline-flex items-center justify-center rounded-lg
border border-line-strong bg-bg-primary text-fg-strong
hover:bg-bg-tertiary transition-colors
```
- 40×40 (`w-10 h-10`), radius 8px
- bg `--bg-primary`, border 1px `--line-strong`
- ⚠️ 아이콘/텍스트 color **`text-fg-strong`** (text-fg-default 아님)
- 아이콘 `lucide-refresh-cw` 16px
- hover: bg `--bg-tertiary`

#### Version Badge (우상단 "v1") (실측):
```
inline-flex items-center px-2 h-6 rounded-md bg-bg-tertiary text-fg-strong t-caption-1
```
- ⚠️ **pill(rounded-full) 아님 → `rounded-md`(6px)**, h-6(24px), px-2(8px)
- bg `--bg-tertiary`, color `--fg-strong`, `t-caption-1`(12/600)
- **카운트 뱃지(§8.17)와 완전히 동일한 컴포넌트**

---

## 9. 페이지 레이아웃 패턴

### 9.1 `/login` — 인증 (2단 스플릿) — **실측 전면 재작성 (v3.1)**

> ⚠️ v2/v3에서 이 섹션은 추론이 많아 오류가 있었음. 아래는 실제 DOM 클래스 기준 정정판.

#### 컨테이너
- 루트: `flex` (min-h-screen), 자식 2개
- 좌측 너비 **`w-[44%]`** (44%), 우측 **`flex-1`** (56%) — **50/50 아님**
- `lg` 미만에서 좌측은 `hidden` (좌측은 `hidden lg:flex`) → 모바일은 폼만 노출

#### 좌측 패널 (브랜드)
```
hidden lg:flex relative flex-col px-12 pb-12 w-[44%] overflow-hidden
bg-gradient-to-br from-[#0A4429] via-[#0F5C39] to-[#178551] text-white
```
- 패딩: `px-12 pb-12` (좌우 48px, 하단 48px, **상단 0** — 콘텐츠는 `mt-[calc(50vh-158px)]`로 수직 위치)
- ⚠️ **배경 그라데이션 (3-stop)**: `bg-gradient-to-br from-[#0A4429] via-[#0F5C39] to-[#178551]`
  - = primary-1000 `#0A4429` → primary-900 `#0F5C39` → primary-700 `#178551`, 우하향(to-br)
- **오버레이 3겹** (absolute):
  1. 그리드: `absolute inset-0 opacity-[0.07] bg-[linear-gradient(#fff_1px,transparent_1px),linear-gradient(90deg,#fff_1px,transparent_1px)]` (흰 1px 격자, opacity 0.07)
  2. 글로우 ①: `absolute -top-32 -right-32 w-[420px] h-[420px] rounded-full bg-[radial-gradient(circle,rgba(52,195,129,0.5)_0%,transparent_70%)]` (우상단 420px, fg-brand-bright/50)
  3. 글로우 ②: `absolute -bottom-40 -left-20 w-[360px] h-[360px] rounded-full bg-[radial-gradient(circle,rgba(52,195,129,0.35)_0%,transparent_70%)]` (좌하단 360px, /35)
- **콘텐츠 블록**: `relative z-10 mx-auto w-full max-w-[420px] mt-[calc(50vh-158px)]`
  - 워드마크: **SVG 로고**(XROO), `h-6 w-auto text-white`
  - 액센트 라인: `mt-4 h-[2px] w-12 rounded-full bg-fg-brand-bright` (2px×48px, `#6FD3A0`)
  - 헤드라인: `mt-6 text-[36px] font-bold leading-[1.25] tracking-tight text-white`
  - 서브카피: `mt-4 t-body-2 text-white/70` (15px)
  - 카피라이트: `absolute left-12 bottom-10 z-10 t-caption-1 text-white/40` (좌48/하40, 12px, white/40)

#### 우측 패널 (폼)
```
flex-1 flex items-center justify-center px-6 py-10 relative
```
- ⚠️ **배경 없음(투명)** → 페이지 배경(`--background`)이 그대로 보임 = **테마 종속**
  - 라이트: 흰색(`#FFFFFF`/`#111213` 아님) / 다크: `#111213`
  - **"#0D0E10 하드코딩 다크"는 오류였음** (E1)
- 패딩 `px-6 py-10` (24px / 40px), 폼 수직·수평 중앙 정렬
- **폼 블록** (max-width 측정상 ~360px 래퍼):
  - 헤딩 "로그인": bold (h-tag, 대략 t-title-3 또는 인라인 — 24px급)
  - 서브: `mt-2 t-body-2 text-fg-assistive` (15px, `--fg-assistive`)
  - 필드 라벨: `t-caption-1 text-fg-default` (12px) — ID / PW
  - ⚠️ **인풋: 표준 §8.2 인풋 그대로** (`bg-bg-primary border-line-strong`) — **글래스 톤 아님**
    - (v2의 `rgba(70,90,126,0.4)` 글래스는 **Chrome 자동완성 배경 아티팩트를 오독**한 것 — E5)
  - PW 인풋: 우측 eye 토글 (§8.2)
  - ⚠️ **로그인 버튼 (hero CTA)**: `h-13 text-[15px] w-full px-5 bg-bg-brand ...` = **52px 높이, 15px/700, 풀너비** (표준 h-10 아님 — E6)

### 9.2 콘솔 메인 (`/projects`, `/users`)
```
┌─────────┬─────────────────────────────────────┐
│ Sidebar │ Header: 타이틀 + count    [새로고침] │
│ (dark)  ├─────────────────────────────────────┤
│         │ [검색][조회]                  [+추가]│
│         │ [전체][노출중][예정][만료]   [정렬↓]│
│         ├─────────────────────────────────────┤
│         │ 〈콘텐츠: 카드 그리드 or 테이블〉    │
│         ├─────────────────────────────────────┤
│         │ [12개▼]              [‹][1][›]      │
│         │                                     │
│ Theme   │                                     │
│ Logout  │                                     │
└─────────┴─────────────────────────────────────┘
```
- 사이드바: **`w-60` = 240px** 고정 (`shrink-0 h-screen sticky top-0`)
- 메인 콘텐츠 영역 (실측): `main.flex-1 min-w-0 px-10 pt-8 pb-10`
  - ⚠️ padding = **좌우 40px(`px-10`), 상 32px(`pt-8`), 하 40px(`pb-10`)** — 상하 비대칭 (py-8 아님)
- 콘텐츠 내부 래퍼: **`max-w-[1320px] mx-auto`** (최대폭 1320px, 중앙 정렬)
- 최상위: `flex min-h-screen bg-bg-secondary`
- ⚠️ **카드 그리드는 반응형 (P1)**: `grid gap-4 grid-cols-2 md:grid-cols-3 xl:grid-cols-4`
  - 2열(기본) → 3열(`md` ≥768px) → 4열(`xl` ≥1280px), gap 16px
  - 4-col은 콘텐츠 1320px일 때 카드 318px. **하드코딩 4-col 금지** (좁은 창에서 카드 깨짐)
- 헤더↔툴바 24px, 툴바↔칩 16px, 칩↔그리드 16px, 그리드↔페이지네이션 20px (실측 마진)

### 9.3 풀폼 페이지 (`/projects/new`)
```
[←] 페이지 타이틀                            [v1]

┌── Section Card ────────────────────────────┐
│ 기본정보                                    │
│  필드들 (grid-cols-2 또는 1)                │
└────────────────────────────────────────────┘
┌── Section Card ────────────────────────────┐
│ 노출설정                                    │
│  ...                                        │
└────────────────────────────────────────────┘
... (반복)

                                  [취소] [저장]
```
- 섹션 간 `gap-6` (24px)
- 우하단 액션은 sticky 또는 페이지 끝

---

## 10. 사용자 데스크탑 앱(exe) 적용 가이드 (프레임워크 중립)

### 10.1 프레임워크 후보
| FW | 장점 | 단점 | 비고 |
|---|---|---|---|
| **Electron** | 어드민의 React/Tailwind/lucide-react 그대로 재사용 — **재작업 0** | 번들 크기 ~150MB | 가장 안전한 선택 |
| **Tauri (Rust + WebView2)** | 가벼움 (~10MB), Rust 백엔드 | Windows WebView2 의존, OS별 렌더 차이 | 디자인 토큰은 그대로 사용 가능 |
| **Flutter Desktop** | 단일 렌더링 엔진 — 픽셀퍼펙트 | 토큰을 `ThemeData`/`ColorScheme`로 옮겨 적어야 함 | Pretendard 임베딩 필요 |

### 10.2 프레임워크 중립 디자인 원칙
1. **모든 디자인 토큰을 `tokens.json` (Style Dictionary 호환)으로 정리**
   - Electron/Tauri → `tokens.css` 자동 생성
   - Flutter → `tokens.dart` 자동 생성
2. **폰트는 앱 번들에 임베딩** — 인터넷 없이 작동해야 함
3. **OS 다크모드 감지**:
   - Electron: `nativeTheme.shouldUseDarkColors` + `nativeTheme.on('updated')`
   - Tauri: `tauri::Theme` API
   - Flutter: `WidgetsBinding.instance.platformDispatcher.platformBrightness`
   - 웹/React: `matchMedia('(prefers-color-scheme: dark)')`
4. **localStorage 동등물 사용**:
   - Electron: `electron-store`
   - Tauri: `tauri-plugin-store`
   - Flutter: `shared_preferences`
5. **윈도우 크기**: 최소 너비 1024 (1280 권장), 최소 높이 720

### 10.3 Pretendard JP Variable 임베딩
- 파일: `Pretendard JP Variable.woff2` (가변 폰트, ~150KB)
- 라이선스: SIL OFL — 임베딩 가능
- 다운로드: https://github.com/orioncactus/pretendard
- Electron: `app://fonts/PretendardJPVariable.woff2`
- Tauri: `assets/fonts/PretendardJPVariable.woff2` + `tauri.conf.json`에서 protocol 허용
- Flutter: `pubspec.yaml`의 `fonts` 섹션에 등록

### 10.4 사용자 페이지 권장 조정 (어드민과의 차이)

| 항목 | 어드민 | 사용자 페이지(권장) |
|---|---|---|
| **사이드바** | 다크 고정 | **라이트 모드에선 라이트 사이드바** (또는 사이드바 없는 헤더 네비) |
| **Modal backdrop** | 투명 | **`bg-black/40` + 옵션 `backdrop-blur-sm`** (집중도 ↑) |
| **Status badge** | 모든 상태 동일 색 | **상태별 색 부여** (success/warning/danger 등) — 섹션 8.16 권장표 |
| **에러 컬러** | 없음 | **`--fg-error: #E5484D` 추가** + 인풋 에러 상태 토큰 |
| **Row height** | 40px | **48–56px** (콘솔 사용자가 아닌 일반 사용자 대상) |
| **Page gutter** | 40px | **24px** (좁은 윈도우) ~ **80px** (와이드) — 반응형 |
| **CTA 크기** | h-10 | **h-12** (CTA 강조) |
| **카드 라운드** | 16px | **20–24px** (소프트한 인상) |

#### 사용자 페이지 전용 CSS 예시:
```css
.user-app {
  /* 사이드바 라이트화 */
  --sidebar: #FFFFFF;
  --sidebar-foreground: #171719;
  --sidebar-border: rgba(112,115,124,0.22);
  --sidebar-accent: rgba(23,133,81,0.08);
  --sidebar-accent-foreground: #178551;

  /* 에러 토큰 추가 */
  --fg-error: #E5484D;
  --bg-error-subtle: #FFEBEC;
}
.dark .user-app {
  --sidebar: #1C1D20;
  --sidebar-foreground: #F0F1F3;
  --sidebar-border: rgba(255,255,255,0.09);
  --sidebar-accent: rgba(52,195,129,0.12);
  --sidebar-accent-foreground: #34C381;

  --fg-error: #FF6B6E;
  --bg-error-subtle: #2A1416;
}
```

### 10.5 데스크탑 앱 윈도우 chrome
- **Frameless 윈도우 권장**: 자체 타이틀바 구현으로 디자인 일관성
- 타이틀바: h-9, bg `--bg-sidebar` (사이드바 톤과 통합)
- 좌측: 앱 아이콘 + 타이틀(`t-label-2 text-fg-inverse`)
- 우측: 최소화/최대화/닫기 버튼 (16×16 ghost, hover bg)
- 드래그 영역: `-webkit-app-region: drag` (Electron) / `data-tauri-drag-region` (Tauri)

---

## 11. globals.css + tailwind.config 스니펫

### 11.1 `globals.css`
```css
/* === XROO Design Tokens v2 === */

:root {
  /* Primary */
  --color-primary-50:  #EAF7F0; --color-primary-100: #D6F0E2;
  --color-primary-200: #A9E4C5; --color-primary-300: #6FD3A0;
  --color-primary-400: #34C381; --color-primary-500: #25AC6E;
  --color-primary-600: #1C9A61; --color-primary-700: #178551;
  --color-primary-800: #136E44; --color-primary-900: #0F5C39;
  --color-primary-1000: #0A4429;

  /* CoolGrey */
  --color-coolgrey-50:  #F7F7F8; --color-coolgrey-100: #F4F4F5;
  --color-coolgrey-200: #EAEBEC; --color-coolgrey-400: #C2C4C8;
  --color-coolgrey-500: #AEB0B6; --color-coolgrey-600: #8B8E97;
  --color-coolgrey-700: #70737C; --color-coolgrey-1350: #171719;

  /* Semantic - Light */
  --background: #ffffff;       --foreground: #171719;
  --bg-primary: #FFFFFF;       --bg-secondary: #F7F7F8;
  --bg-tertiary: #F4F4F5;      --bg-sidebar: #171719;
  --bg-inverse: #171719;
  --bg-brand: #178551;         --bg-brand-hover: #136E44;
  --bg-brand-press: #0F5C39;   --bg-brand-subtle: #EAF7F0;

  --fg-primary: #000000;       --fg-strong: #171719;
  --fg-default: #2E2F33;       --fg-subtle: #46474C;
  --fg-assistive: #70737C;     --fg-disabled: #AEB0B6;
  --fg-inverse: #FFFFFF;
  --fg-brand: #178551;         --fg-brand-bright: #34C381;

  --line-default: rgba(112,115,124,0.22);
  --line-subtle:  rgba(112,115,124,0.10);
  --line-strong:  rgba(112,115,124,0.40);
  --line-accent:  #178551;

  --input: rgba(112,115,124,0.40);
  --input-background: #ffffff;
  --ring: #178551;

  /* shadcn aliases */
  --primary: #178551; --primary-foreground: #ffffff;
  --secondary: #F4F4F5; --secondary-foreground: #171719;
  --muted: #F4F4F5;    --muted-foreground: #70737C;
  --accent: #EAF7F0;   --accent-foreground: #178551;
  --card: #ffffff;     --card-foreground: #171719;
  --popover: #ffffff;  --popover-foreground: #171719;
  --border: rgba(112,115,124,0.22);
  --destructive: #171719; --destructive-foreground: #ffffff;
  --switch-background: #C2C4C8;

  /* Sidebar */
  --sidebar: #171719;
  --sidebar-foreground: #ffffff;
  --sidebar-primary: #178551;
  --sidebar-primary-foreground: #ffffff;
  --sidebar-accent: rgba(255,255,255,0.06);
  --sidebar-accent-foreground: #ffffff;
  --sidebar-border: rgba(255,255,255,0.08);
  --sidebar-ring: #34C381;

  /* Radius (어드민 ground-truth: 20px 토큰 없음. 모달은 rounded-[20px] arbitrary) */
  --radius: 0.5rem;
  --radius-6: 6px;   --radius-8: 8px;   --radius-12: 12px;
  --radius-16: 16px; --radius-24: 24px;
  --radius-full: 9999px;
  /* --radius-20: 20px;  ← 어드민엔 없음. 모달용으로 추가하고 싶으면 주석 해제(선택) */

  /* Spacing */
  --space-1: 4px;  --space-2: 8px;  --space-3: 12px;
  --space-4: 16px; --space-5: 20px; --space-6: 24px;
  --space-8: 32px; --space-10: 40px; --space-16: 64px;

  /* Density */
  --d-page-gutter: 40px; --d-card-pad: 20px;
  --d-form-gap: 16px;    --d-row-height: 40px;
  --d-cell-pad-x: 14px;  --d-cell-pad-y: 8px;

  /* Elevation */
  --elevation-1:       0 1px 2px rgba(23,23,23,0.06);
  --elevation-2:       0 2px 8px rgba(23,23,23,0.07), 0 1px 2px rgba(0,0,0,0.04);
  --elevation-overlay: 0 12px 32px rgba(0,0,0,0.08), 0 0 0 1px rgba(112,115,124,0.10);

  /* Typography */
  --font-sans:    "Pretendard JP Variable", "Pretendard Variable",
                  "Pretendard JP", "Pretendard",
                  -apple-system, BlinkMacSystemFont, "system-ui",
                  "Apple SD Gothic Neo", "Noto Sans KR", sans-serif;
  --font-display: var(--font-sans);
  --font-mono:    "Pretendard JP Variable", "Pretendard Variable",
                  "SF Mono", "JetBrains Mono", ui-monospace,
                  Menlo, Consolas, monospace;
  --font-size: 16px;
  --font-weight-normal: 400;
  --font-weight-medium: 500;
}

html.dark {
  --background: #111213; --foreground: #F0F1F3;
  --bg-primary: #1C1D20; --bg-secondary: #111213;
  --bg-tertiary: #28292E; --bg-sidebar: #0D0E10;
  --bg-inverse: #F0F1F3;
  --bg-brand: #178551; --bg-brand-hover: #1C9A61;
  --bg-brand-press: #25AC6E; --bg-brand-subtle: #0C2B1A;

  --fg-strong: #F0F1F3; --fg-default: #C4C6CC;
  --fg-subtle: #9B9EA8; --fg-assistive: #82858F;
  --fg-disabled: #52555F;
  --fg-brand: #34C381;  --fg-brand-bright: #6FD3A0;

  --line-default: rgba(255,255,255,0.09);
  --line-subtle:  rgba(255,255,255,0.05);
  --line-strong:  rgba(255,255,255,0.18);
  --line-accent:  #34C381;

  --input: rgba(255,255,255,0.18);
  --input-background: #28292E;
  --ring: #34C381;

  --primary: #34C381;  --primary-foreground: #0C2B1A;
  --secondary: #28292E; --secondary-foreground: #F0F1F3;
  --muted: #28292E;    --muted-foreground: #82858F;
  --accent: #0C2B1A;   --accent-foreground: #34C381;
  --card: #1C1D20;     --card-foreground: #F0F1F3;
  --popover: #1C1D20;  --popover-foreground: #F0F1F3;
  --border: rgba(255,255,255,0.09);
}

/* === Typography utilities === */
.t-title-1   { font-size: 32px; line-height: 44px;   font-weight: 700; letter-spacing: -0.81px; }
.t-title-3   { font-size: 24px; line-height: 32px;   font-weight: 700; letter-spacing: -0.55px; }
.t-heading-1 { font-size: 22px; line-height: 30px;   font-weight: 700; letter-spacing: -0.43px; }
.t-heading-2 { font-size: 20px; line-height: 28px;   font-weight: 700; letter-spacing: -0.24px; }
.t-body-1    { font-size: 16px; line-height: 24px;   font-weight: 500; }
.t-body-2    { font-size: 15px; line-height: 22px;   font-weight: 500; }
.t-label-1   { font-size: 14px; line-height: 20px;   font-weight: 600; }
.t-label-2   { font-size: 13px; line-height: 18px;   font-weight: 600; }
.t-caption-1 { font-size: 12px; line-height: 16px;   font-weight: 600; }
.t-caption-2 { font-size: 11px; line-height: 14.96px; font-weight: 600; }

html, body {
  font-family: var(--font-sans);
  background: var(--bg-secondary);
  color: var(--foreground);
  font-size: var(--font-size);
  font-feature-settings: 'tnum';   /* tabular numerals 옵션 */
}

/* tabular numerals utility */
.tabular { font-variant-numeric: tabular-nums; }
```

### 11.2 `tailwind.config.js` (Tailwind v3) — utility 매핑
```js
module.exports = {
  content: ['./src/**/*.{ts,tsx,html}'],
  darkMode: 'class',
  theme: {
    extend: {
      colors: {
        bg: {
          primary:      'var(--bg-primary)',
          secondary:    'var(--bg-secondary)',
          tertiary:     'var(--bg-tertiary)',
          sidebar:      'var(--bg-sidebar)',
          inverse:      'var(--bg-inverse)',
          brand:        'var(--bg-brand)',
          'brand-hover':  'var(--bg-brand-hover)',
          'brand-press':  'var(--bg-brand-press)',
          'brand-subtle': 'var(--bg-brand-subtle)',
        },
        fg: {
          primary:        'var(--fg-primary)',
          strong:         'var(--fg-strong)',
          default:        'var(--fg-default)',
          subtle:         'var(--fg-subtle)',
          assistive:      'var(--fg-assistive)',
          disabled:       'var(--fg-disabled)',
          inverse:        'var(--fg-inverse)',
          brand:          'var(--fg-brand)',
          'brand-bright': 'var(--fg-brand-bright)',
        },
        line: {
          default: 'var(--line-default)',
          subtle:  'var(--line-subtle)',
          strong:  'var(--line-strong)',
          accent:  'var(--line-accent)',
        },
        // CoolGrey 고정 스케일 (사이드바 text-grey-600 등 테마 비의존 회색)
        grey: {
          50:  'var(--color-coolgrey-50)',
          100: 'var(--color-coolgrey-100)',
          200: 'var(--color-coolgrey-200)',
          400: 'var(--color-coolgrey-400)',
          500: 'var(--color-coolgrey-500)',
          600: 'var(--color-coolgrey-600)',  // #8B8E97 — 사이드바 inactive
          700: 'var(--color-coolgrey-700)',
        },
        // shadcn aliases
        background: 'var(--background)',
        foreground: 'var(--foreground)',
        primary: { DEFAULT: 'var(--primary)', foreground: 'var(--primary-foreground)' },
        secondary: { DEFAULT: 'var(--secondary)', foreground: 'var(--secondary-foreground)' },
        muted: { DEFAULT: 'var(--muted)', foreground: 'var(--muted-foreground)' },
        accent: { DEFAULT: 'var(--accent)', foreground: 'var(--accent-foreground)' },
        card: { DEFAULT: 'var(--card)', foreground: 'var(--card-foreground)' },
        popover: { DEFAULT: 'var(--popover)', foreground: 'var(--popover-foreground)' },
        border: 'var(--border)',
        ring: 'var(--ring)',
      },
      borderRadius: {
        DEFAULT: 'var(--radius)',
        md: '6px', lg: '8px', xl: '12px', '2xl': '16px', '3xl': '20px',
      },
      boxShadow: {
        '1':         'var(--elevation-1)',
        '2':         'var(--elevation-2)',
        'overlay':   'var(--elevation-overlay)',
        'elevation-2': 'var(--elevation-2)',
      },
      fontFamily: {
        sans:    'var(--font-sans)',
        display: 'var(--font-display)',
        mono:    'var(--font-mono)',
      },
    },
  },
  plugins: [],
}
```

### 11.3 Theme bootstrap (Flash 방지)
```html
<!-- index.html <head> 최상단 -->
<script>
  (function() {
    var m = localStorage.getItem('xroo-theme-mode') || 'system';
    var isDark = m === 'dark' || (m === 'system' && matchMedia('(prefers-color-scheme: dark)').matches);
    if (isDark) document.documentElement.classList.add('dark');
  })();
</script>
```

---

## 12. 부록: 검증된 토큰 ↔ Utility 매핑표

| 토큰 | Utility | Light | Dark |
|---|---|---|---|
| `--bg-primary` | `bg-bg-primary` | `#FFFFFF` | `#1C1D20` |
| `--bg-secondary` | `bg-bg-secondary` | `#F7F7F8` | `#111213` |
| `--bg-tertiary` | `bg-bg-tertiary` | `#F4F4F5` | `#28292E` |
| `--bg-sidebar` | `bg-bg-sidebar` | `#171719` | `#0D0E10` |
| `--bg-brand` | `bg-bg-brand` | `#178551` | `#178551` |
| `--fg-strong` | `text-fg-strong` | `#171719` | `#F0F1F3` |
| `--fg-default` | `text-fg-default` | `#2E2F33` | `#C4C6CC` |
| `--fg-brand` | `text-fg-brand` / `border-fg-brand` / `ring-fg-brand` | `#178551` | `#34C381` |
| `--fg-brand-bright` | `text-fg-brand-bright` | `#34C381` | `#6FD3A0` |
| `--line-default` | `border-line-default` | `rgba(112,115,124,0.22)` | `rgba(255,255,255,0.09)` |
| `--line-strong` | `border-line-strong` | `rgba(112,115,124,0.40)` | `rgba(255,255,255,0.18)` |
| `--elevation-overlay` | `shadow-overlay` | (light shadow) | (dark — 효과 약함) |

---

## 부록 A: 추출 메타데이터 (v3)
- 추출 도구: Chrome DevTools (`element.className` + `getComputedStyle` + 인터랙션 시뮬레이션 + Tailwind class probe)
- 측정 viewport: 2560×1271 (devicePixelRatio ≈ 1.5)
- 측정한 경로: `/login`, `/projects`, `/projects/new`, `/users` (+ "사용자 추가" 모달, 드롭다운, DatePicker, 체크박스, 페이지네이션)
- 측정한 모드: dark + light + system 모두 검증
- 측정한 상태: default + hover + focus + filled + active + disabled + checked + selected
- **v3 검증 방식: 실제 DOM 클래스를 ground-truth로 잡고 v2 문서값 전수 대조** → 토큰 0 오차, 컴포넌트 18개 정정 (§부록 C)
- 식별한 아이콘 라이브러리: lucide-react (확정)
- 식별한 스타일 시스템: Tailwind CSS + CSS Variables (utility명 = 토큰명) + `t-*` typography 클래스

## 부록 B: 변경 이력
- 2026-06-04 (v1): 초기 작성 (정적 토큰만)
- 2026-06-04 (v2): 전체 보강 — 모든 컴포넌트 상태 측정, 아이콘 라이브러리 식별, Typography utility 매핑, Tailwind ↔ 토큰 검증, 데스크탑 앱 가이드, 사용자 페이지 권장 조정 (Status 색 부여 / 에러 토큰 추가 / 사이드바 라이트화)
- 2026-06-04 (v3): **전수 검증** — 실제 DOM 클래스를 ground-truth로 재추출해 v2의 모든 값 1:1 대조. 18개 불일치 발견·수정 (§부록 C). 컬러/Spacing/Elevation/Density/Font 토큰은 0 오차 확인. radius 토큰에서 v2가 추가했던 `--radius-20`(미존재) 제거. tailwind.config에 `grey` 스케일 추가.
- 2026-06-04 (v3.1): **재현 테스트(round-trip)** — MD만으로 `/login`을 직접 빌드→원본과 나란히 비교. 검수로는 못 잡은 §9.1 로그인 오류 6건(E1~E6) 발견·정정. §9.1 전면 재작성. (재현 산출물: `_repro/index.html`)
- 2026-06-04 (v3.2): **`/projects` 재현 + 실사용 검수** — 재현물을 실제 데스크탑 폭에서 열어 정렬/간격/드롭다운/컬러 점검. 사이드바 브랜드 4건(G1~G4) + 구조 4건(P1~P4: 반응형 그리드/카드구조/truncate/사이드바 섹션패딩) 발견·정정. 로고 SVG 반영. §8.10·§8.18·§9.2 보강.
- 2026-06-04 (v3.3): **라이트 테마 + 나머지 2페이지 round-trip** — 4개 페이지를 양 테마로 재현·대조(§C-5). 라이트 추가 오류 0(토큰 기반). `/users`(셀 t-label-2, 생성일 font-mono) §8.12, `/projects/new`(섹션 gap-4, 푸터 border-t, 날짜 tabular) §8.11 정정. 재현 산출물: `_repro/{index,users,new,login}.html`.

---

## 부록 C: 검증 로그 (v2 → v3 수정 내역)

> 검증 방법: 어드민의 실제 DOM에서 `element.className` + `getComputedStyle`을 추출해 v2 문서값과 1:1 대조. 인터랙션(클릭/포커스/체크)으로 상태 전이도 실측.
>
> **컬러 토큰 50+개 / Spacing 9개 / Elevation 3개 / Density 6개 / Font 6개 / Typography 10개 = 전부 일치 (오차 0).**
> 아래는 **컴포넌트 클래스 레벨**에서 발견된 18개 불일치와 수정 결과.

| # | 항목 | v2 (틀림) | v3 (실측 정정) |
|---|---|---|---|
| D1 | radius 토큰 | `--radius-20` 존재("실측 확인") | **미존재**. 모달은 arbitrary `rounded-[20px]` |
| D2 | 섹션카드 패딩 클래스 | `p-5` | `p-[var(--d-card-pad)]` (값 20px 동일) |
| D3 | 메인 콘텐츠 거터 | `py-8 px-10` (상하대칭) | `px-10 pt-8 pb-10` (상32/하40) + `max-w-[1320px] mx-auto` |
| D4 | `t-title-1` 용도 | "미사용 후보" | **페이지 타이틀로 실사용** |
| D5 | 사이드바 폭 | `w-[216px]` (240px) | **`w-60` = 240px** (216은 nav item 폭) |
| D6 | 사이드바 inactive 색 | `--fg-assistive` | **`text-grey-600`** = coolgrey-600 `#8B8E97` (다크 fg-assistive #82858F와 다름) |
| D7 | 타이틀 옆 카운트 뱃지 | inline `text-fg-assistive` | **pill** `h-6 px-2 rounded-md bg-bg-tertiary text-fg-strong t-caption-1` |
| D8 | Primary CTA 폰트 | `t-label-1` (600) | `text-[14px] font-bold` (**700**) |
| D9 | 콘솔 페이지 타이틀 | `t-title-3`/`t-heading-1` | **`t-title-1`** (32px) |
| D10 | 페이지네이션 화살표 비활성 | opacity 0.5 | **`disabled:opacity-40`** (0.4) |
| D11 | 새로고침 아이콘 색 | `text-fg-default` | **`text-fg-strong`** |
| D12 | 테이블 행 hover | 다크 `rgba(255,255,255,0.04)` | **`hover:bg-bg-tertiary`** (양 모드 공통) |
| D13 | 테이블 ID 강조셀 | 13px / **700** | `t-label-2` = 13px / **600** |
| D14 | 버전 뱃지 | pill `rounded-full` padding 10px 4px | `h-6 px-2 rounded-md` (카운트 뱃지와 동일) |
| D15 | 파일 "선택" 링크 | `text-fg-brand-bright` | **`underline text-fg-brand`** (라이트에서 값 달라짐) |
| D16 | Drop zone 보더 | 2px dashed(또는 1px) | **`border-[1.5px]` dashed** rounded-xl py-8 |
| D17 | **폼 필드 라벨** | `t-label-1` (14px) | **`t-caption-1 text-fg-default`** (12px) — 모달+풀폼 모두 |
| D18 | 섹션 타이틀 마진 | `mb-4` | margin 0 (간격은 부모 `flex flex-col gap-4`) |

추가 정정(표 외):
- 필수표시 `*` 색 = `text-fg-strong` (빨강/브랜드 아님)
- thead row 보더 = `border-b border-line-strong`
- 페이지네이션 활성 번호 = `min-w-8`(고정 w-8 아님)
- 셀렉트 트리거에 `hover:bg-bg-tertiary` 존재
- tailwind.config: `grey` 스케일 추가(사이드바 `text-grey-600` 동작용)

### C-2. 재현 테스트(round-trip) — `/login` 정정 (v3.1)

> MD §9.1만 보고 로그인 페이지를 빌드 → 원본과 나란히 비교. **DOM 검수로는 못 잡은 오류**들을 노출.
> (검수는 "값이 맞는가"를, 재현은 "이 문서로 똑같이 만들어지는가"를 검증한다.)

| # | 항목 | v2/v3 (틀림) | v3.1 (실측 정정) |
|---|---|---|---|
| E1 | **우측 폼 패널 배경** | "거의 검정 `#0D0E10` 하드코딩" | **`flex-1` + 배경 투명** → 페이지 배경 노출 = **테마 종속**(라이트 흰색 / 다크 #111213) |
| E2 | 좌우 분할 비율 | 50/50 (`flex-1`씩) | 좌 **`w-[44%]`** / 우 `flex-1`(56%) |
| E3 | 좌측 그라데이션 | `linear-gradient(135deg, #0F5C39, #178551)` 2-stop | **`bg-gradient-to-br from-[#0A4429] via-[#0F5C39] to-[#178551]`** 3-stop |
| E4 | 좌측 오버레이 | "SVG 그리드"만 | 그리드(opacity 0.07) + **방사형 글로우 2개**(우상단 420px/0.5, 좌하단 360px/0.35, fg-brand-bright) |
| E5 | **로그인 인풋** | 글래스 `rgba(70,90,126,0.4)` | **표준 `bg-bg-primary` 인풋** — "글래스"는 **Chrome 자동완성 배경 아티팩트 오독** |
| E6 | 로그인 버튼 | (미기재, h-10 가정) | **hero CTA `h-13`(52px) `text-[15px]`** 풀너비 |

추가(표 외): 액센트 라인 `bg-fg-brand-bright` 2px×48px / 서브카피 `t-body-2 text-white/70` / 로그인 서브 `t-body-2 text-fg-assistive` / 카피라이트 `t-caption-1 text-white/40` / 콘텐츠 `max-w-[420px] mt-[calc(50vh-158px)]` / 좌패딩 `px-12`(48) / 워드마크는 SVG 로고.

> **교훈**: 페이지 레이아웃(특히 배경의 테마 종속성·그라데이션·오버레이·hero 사이즈)은 토큰 검수만으로는 못 잡고 **재현해봐야** 드러난다.

### C-3. 재현 테스트 — `/projects` 콘솔 메인 (v3.1)

> MD만으로 `/projects`를 빌드 → 원본과 비교. **레이아웃·헤더·툴바·필터칩·카드그리드·페이지네이션·테마토글은 v3 명세로 정확히 재현됨**(추가 오류 0). 단 **사이드바 브랜드/유저 영역**(§8.18이 측정 아닌 눈대중으로 기술했던 부분)에서 4건 발견·정정.

| # | 항목 | v3 (틀림) | v3.1 (실측) |
|---|---|---|---|
| G1 | 사이드바 "관리자 콘솔" | `t-label-1`(14px) | **`text-[22px] font-bold`** (22px/700 white) |
| G2 | 사이드바 "admin" | `t-caption-1 text-fg-assistive`(12/600) | **`text-[12px] font-medium text-grey-600`** (12/500, #8B8E97) |
| G3 🟢 | 브랜드 캡션 "XROO PACKAGE SOLUTION" | `t-caption-2 text-fg-assistive`(회색) | **`text-[10px] font-bold tracking-[0.1em] text-fg-brand-bright`** (10/700, **그린**) |
| G4 | 워드마크 | 텍스트 | **SVG 로고 `h-6`(24px) w-auto text-white** |

> `/projects`의 나머지(헤더 t-title-1+카운트뱃지, 검색인풋·조회·CTA, 필터칩, 4-col 카드그리드(gap-4), status 뱃지, 페이지네이션, 세그먼트 테마토글, nav active 그린아이콘)는 **재현 일치**. → v3 컴포넌트 명세는 사이드바 브랜드 영역 외 정확.

> 미실시: `/users`(테이블·체크박스·모달), `/projects/new`(풀폼·날짜픽커·드롭존) 재현 테스트는 추후 권장.

### C-4. 실사용 검수 — `/projects` 재현물 직접 열람 (v3.2)

> 재현물을 데스크탑 폭에서 직접 열어보니 정렬 깨짐·간격 어색·드롭다운 부재·컬러 문제가 보였음. 원인을 분리하면:
> **(a) MD 문서 구멍** P1~P4 — 측정 안 했던 구조 / **(b) 재현 빌드 누락** — 드롭다운 미구현·로고 미반영.

| # | 증상 | 원인/구분 | 정정 |
|---|---|---|---|
| P1 | 카드 정렬 깨짐(좁은 창) | 문서구멍: 그리드 반응형 미기재 | `grid gap-4 grid-cols-2 md:grid-cols-3 xl:grid-cols-4` (§9.2) |
| P2 | 카드 구조 어색 | 문서구멍: 카드 내부 구조 부정확 | `<article>` + `aspect-video` 썸네일 + 우하단 hover 액션(§8.10) |
| P3 | 카드 메타 겹침/높이 들쭉 | 문서구멍: 설명 `truncate` 미기재, 메타 `<dl grid-cols-3 gap-2>` 구조 | §8.10 HTML로 명시 |
| P4 | 사이드바 정렬·간격 | 문서구멍: aside 패딩 0 + 섹션별 패딩(브랜드 px-5 / 나머지 px-3) + nav gap-0.5 + 하단 border-t | §8.18 |
| — | 드롭다운 부재 | 빌드누락: §8.4 popover 미구현 | 재현에 동작 드롭다운 추가(클릭 토글) |
| — | 로고 | 빌드누락: 텍스트로 대체했었음 | `design_source/logo.svg` 인라인(h-6 white) |
| — | "컬러 문제" | 위 레이아웃 깨짐의 부수효과 (토큰 색은 정확) | 레이아웃 수정으로 해소. 카드 썸네일은 실제 이미지(콘텐츠)→재현은 그라데이션 placeholder |

> 결과: 재현물이 원본 `/projects`와 정렬·간격·드롭다운까지 일치. **컬러 토큰 자체 오류는 없었음**(레이아웃 깨짐이 색 문제처럼 보였던 것).

### C-5. 라이트 테마 + 나머지 2페이지 재현 검증 (v3.3)

> **라이트 테마**: 4개 페이지(`/login`·`/projects`·`/users`·`/projects/new`)를 재현물에서 라이트로 렌더해 원본 라이트와 대조.
> 토큰 기반(utility=토큰)이라 **라이트 모드가 모든 페이지에서 충실히 따라옴** — 라이트 전용 추가 오류 0.
> 확인된 라이트 특이사항(모두 의도대로 동작):
> - **사이드바는 라이트에서도 다크 유지**(`--bg-sidebar` #171719) — 콘솔 정체성
> - 본문 배경은 테마 종속(`--bg-secondary`: 라이트 #F7F7F8 / 다크 #111213)
> - 카드/테이블/섹션 = 라이트에서 흰색(`--bg-primary` #FFFFFF) + `--line-default` 보더
> - 로그인 우측 패널 = 라이트에서 흰색(E1 그대로)

> **`/users` 재현**(dark+light 일치) — 측정 정정: 본문 셀 `t-label-2`(13px), 생성일 `font-mono`, 컬럼폭 확인 → §8.12 반영.
> **`/projects/new` 재현**(dark+light 일치) — 측정 정정: 섹션 간격 `gap-4`(16px, gap-6 아님), 푸터 `border-t pt-5 mt-6`, 날짜 인풋 `tabular`+달력버튼 → §8.11 반영.
> 잔여 미세차: /users 헤더 정렬아이콘(↕)은 재현 생략(사소).

> **종합**: login·projects·users·projects-new 4개 페이지 × (dark+light) round-trip 완료. 토큰/컴포넌트 명세로 재현 시 원본과 일치 확인.
