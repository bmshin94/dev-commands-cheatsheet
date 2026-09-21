# 📊 dev-commands-cheatsheet 전수조사 & 활용 전략 리포트

> 작성: Claude Code (카리나 페르소나) · 정리일: 2026-09-21
> 이 문서는 저장소 전체(32개 파일)를 전수조사한 분석 대화를 정리한 결과물입니다.

## 🔗 GitHub 주소

| 구분 | 주소 |
|---|---|
| **내 저장소 (fork)** | https://github.com/bmshin94/dev-commands-cheatsheet |
| **원본 저장소 (upstream)** | https://github.com/abdosorour7/dev-commands-cheatsheet |
| **원본 라이브 데모** | https://abdosorour7.github.io/dev-commands-cheatsheet |
| **내 GitHub Pages (배포 시)** | https://bmshin94.github.io/dev-commands-cheatsheet |
| 라이선스 | MIT © abdosorour7 (상업적 이용 가능, 저작권 고지 필수) |

---

## 1. 한 줄 정의

> **Git / Docker 명령어를 검색해서 클릭 한 번으로 복사하는, 설치가 필요 없는 정적 웹사이트(치트시트).**

빌드 도구 없음 · 의존성 0개 · 백엔드 없음 · API 토큰 없음 · 비용 0원.

---

## 2. 저장소 구조 (전수조사 결과)

```
dev-commands-cheatsheet/          총 32개 파일
├─ index.html          (161줄)   페이지 뼈대 + SEO 메타 + JSON-LD 구조화 데이터
├─ manifest.json                 PWA 설치 설정
├─ sw.js               (115줄)   서비스워커 (오프라인 캐싱, CACHE_NAME=dev-cheatsheets-v7)
├─ robots.txt / sitemap.xml      검색엔진 노출용
├─ preview.png                   README 미리보기 이미지
├─ CLAUDE.md                     카리나 페르소나 지시문
├─ .github/ISSUE_TEMPLATE/       bug_report.md, feature_request.md
└─ assets/
   ├─ css/styles.css   (688줄)   다크 터미널 테마
   ├─ js/
   │  ├─ app.js        (241줄)   부트스트랩 / 오케스트레이션
   │  └─ modules/      (560줄)
   │     ├─ paths.js      ( 37줄) URL 경로 해석 (GitHub Pages 하위경로 대응)
   │     ├─ state.js      ( 43줄) 검색 / 카테고리 필터 상태
   │     ├─ interactions.js(47줄) 복사 · 단축키(/) · 맨 위로
   │     ├─ render.js     (160줄) 카드 / 탭 / 섹션 칩 렌더링
   │     ├─ data-loader.js(163줄) JSON 로딩 + 영어 폴백 병합
   │     └─ i18n.js       (110줄) 언어 전환 + localStorage
   ├─ brands/                    git.svg, docker.svg (Simple Icons, CC0)
   ├─ icons/                     PWA 아이콘 192 / 512
   └─ data/               ⭐ 실제 알맹이 (약 238KB)
      ├─ sections.json          섹션 매니페스트 (git, docker)
      ├─ git.en.json      13 카테고리 / 127 커맨드 (danger 6)
      ├─ git.it.json      12 카테고리 / 100 커맨드
      ├─ git.fr.json      12 카테고리 / 100 커맨드
      ├─ git.es.json      12 카테고리 / 115 커맨드
      ├─ docker.en.json   11 카테고리 / 142 커맨드 (danger 16)
      └─ ui.json          UI 문구 4개국어 (en / it / fr / es)
```

**총 커맨드 269개** (Git 127 + Docker 142), 그중 **파괴적(danger) 명령 22개**.

---

## 3. 동작 흐름

```
브라우저 진입
 → paths.js: 현재 URL 기준으로 asset 절대경로 계산
 → i18n.js: localStorage에서 언어 복원 (기본 en)
 → ui.json 로드 → data-i18n 속성 요소 일괄 치환
 → sections.json 로드 → 섹션 알약(pill) 버튼 생성
 → {section}.{lang}.json 로드 → 영어와 병합(미번역은 EN 뱃지) → 카드 렌더
 → 검색 입력 → state.js 필터 → display 토글 (재렌더 없이 빠름)
 → 복사 버튼 → navigator.clipboard.writeText → "✓" 1.5초 표시
 → sw.js 등록 → 재방문 시 오프라인 동작
```

### 핵심 설계 포인트

| 포인트 | 내용 | 이점 |
|---|---|---|
| 데이터 ↔ 코드 완전 분리 | 명령어는 전부 JSON, JS는 렌더링만 | 새 섹션 추가 시 **JS 수정 0줄** |
| classic script 사용 (`type="module"` 회피) | 로컬 서버가 `.js`를 `text/plain`으로 줘도 동작 | 윈도우 `python -m http.server` 환경 사고 방지 |
| 번역 자동 폴백 | 영어와 설명이 같으면 자동으로 `EN` 뱃지 | 부분 번역 파일을 올려도 앱이 깨지지 않음 |

---

## 4. 발견된 개선 포인트

1. **URL 불일치 (SEO 손해)** — README/데모는 `dev-commands-cheatsheet`인데
   `index.html`의 canonical·og:url·twitter:url, `robots.txt`, `sitemap.xml`은
   옛 이름 `git-commands-cheatsheet`로 남아 있음. 저장소 rename 후 미수정 흔적.
2. **XSS 여지** — `codeHtml` / `descriptionHtml`이 이스케이프 없이 `innerHTML`로 삽입됨.
   현재는 자체 JSON이라 안전하지만, 외부 기여 PR을 검토 없이 머지하면 위험.
3. **CI / 테스트 부재** — `.github/workflows` 없음. JSON 스키마 검증 CI만 붙여도 품질 향상.
4. **번역 불균형** — Docker는 영어만 존재, Git도 it/fr은 127개 중 100개만 번역.

---

## 5. Q&A 요약

### Q1. 설치 및 사용법
- **그냥 쓰기**: 라이브 데모 접속 → 주소창의 설치 버튼으로 PWA 설치(오프라인 동작).
- **로컬 실행**:
  ```bash
  git clone https://github.com/bmshin94/dev-commands-cheatsheet.git
  cd dev-commands-cheatsheet
  python -m http.server 8080     # 또는  npx serve .  / VS Code Live Server
  ```
  `index.html` 더블클릭 시 데이터가 안 뜨면 `file://` CORS 제한 → 로컬 서버로 해결.
- **배포**: Settings → Pages → `main / (root)` → 5분이면 `bmshin94.github.io/dev-commands-cheatsheet`.
  배포 후 위 4-1의 옛 URL들을 내 주소로 교체할 것.
- **커스터마이징**: 명령 추가는 JSON 6줄. 새 섹션은 `sections.json` + `{key}.en.json` + 로고 SVG,
  그리고 **`sw.js`의 `ASSETS` 배열 추가 + `CACHE_NAME` 버전 업**을 잊지 말 것.

### Q2. 플러그인 / 스킬 / MCP 중 무엇인가?
**셋 다 아님.** 순수 정적 웹앱(Static Web App). `package.json`도 없음.

| 구분 | 정체 | 실행 주체 | 해당 여부 |
|---|---|---|---|
| 웹앱 | 브라우저에서 도는 페이지 | 사람 | ✅ 이것 |
| 플러그인 | 특정 호스트 앱의 기능 확장 패키지 | 호스트 앱 | ❌ |
| 스킬 | AI에게 절차를 알려주는 md + 스크립트 묶음 | AI 에이전트 | ❌ (CLAUDE.md는 유사 성격) |
| MCP 서버 | 표준 프로토콜로 도구/데이터 제공 | AI 툴 호출 | ❌ |

다만 269개 JSON 데이터가 정제되어 있어 **MCP 서버 / Claude Skill / VSCode 확장으로 전환이 매우 쉬움.**

### Q3. API 토큰이 필요한가?
**전혀 불필요.** 네트워크 호출은 (1) 자기 서버의 JSON fetch, (2) Google Fonts CSS,
(3) 브라우저 내장 Clipboard API, (4) 서비스워커 등록이 전부.
외부 API 0건 · 인증 코드 0줄 · 개인정보 수집 0건.
저장하는 값은 `localStorage`의 언어·섹션 두 개뿐. 운영 비용도 0원(GitHub Pages).

### Q4. 왜 GitHub에서 유명한가? (코드에 남은 근거 기반 분석)
1. **주제가 스테디셀러** — "Git 명령어 까먹음"은 전 연차 공통 고통.
2. **진입장벽 0** — clone 후 더블클릭, `npm install` 없음.
3. **첫인상** — 다크 터미널 + 네온 + 애니메이션, README 상단 preview.png 한 장 승부.
4. **기여 난이도 극저** — 명령 추가 = JSON 6줄. 첫 오픈소스 기여자·Hacktoberfest 유입 구조.
   실제로 외부 기여 PR(`AmarBego/signing`) 머지 이력 존재.
5. **기여 유도 장치** — ISSUE_TEMPLATE, README의 섹션/언어 추가 단계별 가이드.
6. **SEO 풀세팅** — sitemap, robots, canonical, OG, Twitter Card, JSON-LD, 서치콘솔 인증 메타.
7. **다국어 4개** — 비영어권 흡수.
8. **README 마케팅** — 뱃지·통계표·⭐ CTA·"나는 웹개발자가 아니다"류의 겸손 화법.

> 교훈: 스타는 기술 난이도가 아니라 **고통 해결 + 첫인상 + 기여 쉬움**에서 나온다.
> (주: 이 세션은 저장소 접근이 내 fork로 제한되어 실제 스타 수치는 미검증.)

### Q5. 로컬 에이전트 구축에 도움이 되는가?
**도움 됨. 단, 프론트 코드가 아니라 `assets/data/`의 269개 커맨드 데이터셋이 자산.**

| 데이터 특성 | 에이전트 이득 |
|---|---|
| 일관된 스키마 (`command`, `searchDescription`, `descriptionHtml`, `danger`) | 파싱 없이 툴 응답으로 사용 |
| **`danger: true` 플래그 22개** | 파괴적 명령 가드레일을 데이터로 구현 |
| 24개 카테고리 분류 | 의도 라우팅에 활용 |
| 4개 국어 | 다국어 응답 |

활용 4가지:
1. **MCP 서버로 래핑** — `search_command(query, section)` 도구 제공 → 소형 LLM의 명령어 환각 감소.
2. **RAG 지식베이스** — 항목 1개 = 청크 1개로 딱 떨어져 청킹 고민 불필요.
3. **가드레일 룰셋** — danger 22개를 에이전트 실행 차단/확인 목록으로 사용.
4. **Claude Code 스킬화** — `.claude/skills/git-helper/SKILL.md` + `commands.json`.

한계: 명령 *설명*이지 *실행 로직*이 아님(에이전트의 "손"이 아닌 "사전"). 269개는 지식량으로는 소규모.

### Q6~Q7. 요약
- 수익화: 아래 6장 참조.
- React/PHP 이식: 아래 7장 참조.

---

## 6. 수익화 아이디어

> 전제: MIT 라이선스이므로 상업적 이용·수정·재배포 자유. **LICENSE 원문과 원저자 저작권 고지는 반드시 포함.**
> 로고는 Simple Icons(CC0)지만 Git/Docker는 상표이므로 "공식 제휴"로 오인될 표현은 금지.
> 치트시트 *자체*는 무료 대체재(cheat.sh, tldr, devhints)가 많아 수익성이 낮음.
> 돈은 "치트시트"가 아니라 **"치트시트 엔진 + 사설 데이터 + AI 연동"** 에서 나옴.

### 🥇 1. 사내 커맨드 위키 SaaS ("CommandBase")
- **파는 것**: "신입이 배포 명령어를 매번 물어본다"는 문제. 팀 전용 사설 치트시트 + 권한 + Slack 봇 + IDE 확장.
- **논리**: 신입 1명 온보딩에 시니어가 쓰는 시간 월 ~5시간 × 시급 5만원 = 월 25만원 손실 → 월 3~5만원 툴은 즉시 정당화.
- **가격**: Free 0원 / Team $9·인·월 / Business $29·인·월(SSO·감사로그·온프레미스).
- **킬러 기능**: ① `danger` 데이터 기반 **위험명령 승인 워크플로우** ② **검색 분석**(가장 많이 검색된 명령 = 문서화 우선순위) ③ VSCode/JetBrains 확장 + CLI.
- 난이도 ★★★★ / 3~4개월 / 현실 목표 20팀×5명×$9 ≈ 월 $900.

### 🥈 2. AI 에이전트용 커맨드 지식팩 ("DevCommands MCP")
- LLM의 CLI 명령 환각이 실무 최대 골칫거리 → 검증된 269개 데이터가 해답.
- **상품**: 무료 MCP 서버(Git/Docker, 미끼) / **Pro 지식팩 $39 평생**(K8s·AWS CLI·Terraform·psql·ffmpeg 등) /
  **Team 라이선스 $299·년**(사내 팩 제작 도구) / **Safety Guardrail 팩 $79**(파괴적 명령 차단 룰셋).
- MCP 생태계 초기라 선점 이득이 크고, Guardrail 팩은 "에이전트 사고 방지 보험"으로 포지셔닝 가능.
- 난이도 ★★★ / 3~4주 / 현실 목표 월 $500~2,000 (Gumroad·Lemon Squeezy).

### 🥉 3. 한국어 치트시트 허브 + 콘텐츠 (최단 수익화)
1. 269개 한국어 번역 (`git.ko.json`, `docker.ko.json`) — AI 초벌 + 사람 검수.
2. Next.js SSG로 **명령어 1개 = URL 1개** → 롱테일 키워드 수백 개 선점.
3. 수익 스택: 애드센스(월 10만 PV ≈ 20~50만원) + 제휴 마케팅(10~30만원) +
   **PDF 치트시트 5,900원 판매** + **인강("790줄로 만드는 PWA 웹앱") 회당 100~500만원**.
- 한국어 Git 치트시트는 블로그 파편 위주라 검색 1페이지 진입 난이도가 낮음.
- 난이도 ★★ / 2~4주 / 6개월 후 월 30~100만원 목표.

### 그 외
| # | 아이디어 | 모델 | 난이도 |
|---|---|---|---|
| 4 | 치트시트 제네레이터 (JSON 업로드 → 사이트 자동 생성) | 무료 + 커스텀 도메인 $5/월 | ★★★ |
| 5 | 워드프레스 플러그인 (블로그 삽입 위젯) | 라이선스 $29/년 | ★★ |
| 6 | 화이트라벨 납품 (부트캠프·학원 브랜딩판) | 건당 100~300만원 | ★★ |
| 7 | GitHub Sponsors / Buy Me a Coffee / 스폰서 배너 | 월 0~30만원 | ★ |

### 추천 로드맵
```
1주차   한국어 번역 + Next.js 이식 + GitHub Pages 배포 (포트폴리오 + SEO 씨앗)
2~4주   MCP 서버 오픈소스 공개 (무료 미끼) → 커뮤니티 홍보, 스타 수집
2개월   Pro 지식팩 $39 출시 (첫 매출)
3~6개월 사용자 인터뷰로 B2B 수요 검증 후 SaaS 개발
```

### 리스크
- 무료 대체재 다수 → 사설 데이터 또는 AI 연동 없이는 판매 불가.
- AI가 직접 명령어를 알려주는 시대라 일반 치트시트 수요는 감소 추세
  → **AI의 반대편이 아니라 AI 옆자리(MCP)** 에 서는 2번이 가장 미래지향적.

---

## 7. React / PHP 이식 가능성

**가능하며, 난이도는 낮음.** 로직이 `JSON 읽기 → 필터 → 카드 렌더 → 클립보드 복사` 4개뿐.

### React (Vite + React + TS + Tailwind, 또는 Next.js)
```jsx
function CheatSheet() {
  const [section, setSection] = useState("git");
  const [query, setQuery] = useState("");
  const [cat, setCat] = useState("all");
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch(`/data/${section}.en.json`).then(r => r.json()).then(setData);
  }, [section]);

  const visible = useMemo(() => {
    if (!data) return [];
    const q = query.trim().toLowerCase();
    return data.categories
      .filter(c => cat === "all" || c.key === cat)
      .map(c => ({ ...c, commands: c.commands.filter(
        cmd => !q || (cmd.command + cmd.searchDescription).toLowerCase().includes(q)) }))
      .filter(c => c.commands.length > 0);
  }, [data, query, cat]);

  return (/* 검색창 + 탭 + 카드 map */);
}
```
얻는 것: Fuse.js 퍼지 검색(오타 허용), 즐겨찾기/최근 복사, `cmdk` 커맨드 팔레트,
**Next.js SSG로 명령어별 개별 페이지 → SEO 롱테일 유입**(수익화 핵심).
주의: `codeHtml`을 `dangerouslySetInnerHTML`로 쓰면 원본 XSS 이슈를 그대로 상속 →
하이라이팅을 토큰 배열로 데이터화하거나 DOMPurify 적용.

### PHP
```php
<?php
$section = preg_match('/^[a-z]+$/', $_GET['s'] ?? 'git') ? $_GET['s'] : 'git';
$q = trim($_GET['q'] ?? '');
$data = json_decode(file_get_contents("data/{$section}.en.json"), true);

foreach ($data['categories'] as $cat) {
  foreach ($cat['commands'] as $cmd) {
    $hay = mb_strtolower($cmd['command'].' '.$cmd['searchDescription']);
    if ($q !== '' && !str_contains($hay, mb_strtolower($q))) continue;
    // 카드 출력 (htmlspecialchars 필수)
  }
}
```
유리한 점: MySQL + 관리자 CRUD로 **비개발자도 명령어 추가** → B2B 사내 위키 상품화에 직결,
회원/권한/즐겨찾기 구현 용이, 워드프레스 플러그인화, 국내 저가 호스팅에서 바로 구동.

### 선택 가이드
| 목표 | 추천 |
|---|---|
| 포트폴리오 / SEO 수익 | **Next.js** |
| 유료 B2B 사내 위키 | **PHP(Laravel)** 또는 Next + Supabase |
| 빠르게 내 버전 | 원본 그대로 두고 **JSON만 교체** (0시간) |

---

## 8. 결론

- 이 저장소는 **269개 커맨드 데이터셋 + 790줄 프론트엔드**로 이루어진 가볍고 잘 정돈된 학습·실용 자산이다.
- 진짜 가치는 UI가 아니라 **정제된 JSON 데이터와 `danger` 플래그**에 있다.
- 가장 빠른 현금화는 한국어 콘텐츠(3번), 가장 큰 시장은 B2B SaaS(1번),
  가장 미래지향적인 것은 AI 에이전트 지식팩(2번).
- 즉시 할 일 3가지:
  1. 옛 URL(`git-commands-cheatsheet`) 잔재 정리 → SEO 회복
  2. GitHub Pages 배포로 내 데모 확보
  3. `git.ko.json` 한국어 번역 착수
