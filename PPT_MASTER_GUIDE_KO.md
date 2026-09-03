# PPT Master 분석 & 활용 가이드 (한국어)

> AI와 함께 이 저장소를 분석하고 나눈 대화를 정리한 문서입니다.
> 프로젝트가 무엇인지 → 어떻게 설치하는지 → 어떻게 쓰는지 → 어떻게 돈을 버는지 → 어떻게 서비스로 확장하는지 순서로 정리했습니다.

## 🔗 관련 링크

| 구분 | 주소 |
|---|---|
| **원본 저장소 (upstream)** | https://github.com/hugohe3/ppt-master |
| **내 저장소 (fork)** | https://github.com/bmshin94/ppt-master |
| 예제 갤러리 (온라인 뷰어) | https://hugohe3.github.io/ppt-master-examples/ |
| 예제 소스 저장소 | https://github.com/hugohe3/ppt-master-examples |
| AtomGit 미러 (중국) | https://atomgit.com/hugohe3/ppt-master |
| 제작자 | Hugo He — https://www.hehugo.com/ |

- 버전: **6.2.0**
- 라이선스: **MIT** (Copyright (c) 2025-2026 Hugo He)

---

## 1. 이게 뭔가요?

**"자료를 던져주면 AI가 편집 가능한 PowerPoint(.pptx)를 만들어주는 워크플로우"** 입니다.

가장 중요한 포인트: **이건 실행 프로그램(앱)이 아닙니다.** 더블클릭할 `.exe`도, `main.py`도 없습니다.
대신 **AI 에이전트(Claude Code 등)가 읽고 따라 하는 초대형 "레시피북 + 도구상자"** 입니다.

### 요리로 비유하면

| 구성 요소 | 역할 |
|---|---|
| 폴더 안의 마크다운 문서들 | 레시피북 📖 |
| 폴더 안의 파이썬 스크립트들 | 주방 도구 🔪 |
| AI 에이전트 | 요리사 👩‍🍳 |
| 내가 준 PDF/DOCX/텍스트 | 재료 🥬 |
| 나오는 `.pptx` 파일 | 완성된 요리 🍽️ |

즉 사용자는 아무것도 실행하지 않습니다. **채팅창에 말만 하면** AI가 이 저장소의 문서를 읽고 스크립트를 순서대로 돌려 PPT를 만듭니다.

### 처리 흐름

```
자료(PDF / DOCX / URL / Markdown / 붙여넣은 텍스트)
   → AI가 내용 분석하고 스토리 구조 설계
   → 페이지별로 SVG를 직접 작성
   → svg_to_pptx.py 가 SVG를 네이티브 DrawingML로 변환
   → exports/이름_타임스탬프.pptx 완성
```

### 왜 특별한가

대부분의 AI PPT 도구는 **이미지로 박제된 슬라이드**를 줍니다. 글자 하나 못 고칩니다.
이 프로젝트는 **PowerPoint에서 도형·글자·색을 하나하나 클릭해서 수정할 수 있는 진짜 네이티브 PPT**를 만듭니다.
차트도 옵션에 따라 "데이터 편집"이 가능한 진짜 PowerPoint 차트 객체로 뽑을 수 있습니다.

> 완성품을 받는 게 아니라, **내가 이어서 다듬을 수 있는 초안**을 받는 도구입니다.

---

## 2. 폴더 구조 분석

| 위치 | 정체 | 규모 |
|---|---|---|
| `skills/ppt-master/SKILL.md` | AI가 가장 먼저 읽는 **진입점**. 요청을 어느 루트로 보낼지 라우팅 | 12KB |
| `skills/ppt-master/workflows/` | 실제 작업 절차서 (3대 루트 + 스테이지들) | 372KB |
| `skills/ppt-master/scripts/` | 실제로 일하는 **파이썬 코드** | 8.9MB / **244개 파일 / 174,487줄** |
| `skills/ppt-master/references/` | 디자인 지식 라이브러리 (비주얼 스타일 18종, 이미지 렌더링 24종, 팔레트 14종) | **45MB** |
| `skills/ppt-master/templates/` | 브랜드 템플릿 21개사, 차트 SVG, 아이콘, 레이아웃, 스키마 | **63MB** |
| `projects/` | **내 작업물이 쌓이는 곳** (결과물 위치) | 초기엔 비어 있음 |
| `docs/` | 공식 문서 (영어 / 중국어) | — |
| `.env.example` | API 키 설정 템플릿 | 12KB |

### 눈여겨볼 스크립트

| 파일 | 역할 |
|---|---|
| `svg_to_pptx.py` | **핵심 변환 엔진** (이 하위 패키지만 53,823줄) |
| `image_gen.py` | AI 이미지 생성 (15종 이상 백엔드 지원) |
| `image_search.py` | 무료 웹 이미지 검색 + 라이선스 자동 처리 |
| `pptx_animations.py` / `pptx_transitions.py` | 애니메이션 / 화면전환 |
| `notes_to_audio.py` | 발표자 노트 → 음성 나레이션 |
| `powerpoint_video.py` | 덱 → 영상(mp4) |
| `svg_quality_checker.py` | 품질 검사 게이트 |
| `project_manager.py` | 프로젝트 초기화/관리 |
| `attribution_guard.py` | 저작권 파일 무결성 검사 (변조 시 스킬 정지) |

### 중요한 인사이트 💡

코드는 전체의 일부일 뿐이고, **진짜 자산은 `references/`(45MB)와 `templates/`(63MB)** 입니다.
그리고 이 둘은 전부 **마크다운과 SVG** — 즉 **프로그래밍 언어와 무관**합니다.
"AI에게 주는 지식"이 이 프로젝트의 본체입니다.

---

## 3. 3가지 사용 루트

| 루트 | 언제 쓰나 | 무엇이 보존되나 |
|---|---|---|
| **Generate PPTX** | 자료/주제로 새 덱 생성 (메인 기능) | 원본의 사실관계. 스토리 구조와 페이지 수는 바뀔 수 있음 |
| **Create Template** | 기존 PPT에서 브랜드/스타일/레이아웃/덱 템플릿 추출해 재사용 | 디자인 시스템을 자산화 |
| **Edit Native PPTX** | 기존 .pptx 디자인 유지한 채 내용만 교체 | **안 건드린 페이지는 바이트 단위로 그대로 보존** |

추가 프로필: **Beautify**(내용 유지 + 디자인만 재작업), **Quick**(확인 절차 생략 원샷), **Image to PPTX**(이미지/스크린샷을 편집 가능한 슬라이드로 복원).

---

## 4. 설치 가이드

### 4-1. 준비물 딱 2개

**① Python 3.10 이상** — 이것만 진짜로 "설치"합니다.

```bash
python3 --version    # macOS / Linux
python --version     # Windows
```

- **Windows**: [python.org](https://www.python.org/downloads/) 에서 다운로드 → ⚠️ 설치 시 **"Add python.exe to PATH" 체크 필수** (가장 흔한 실수)
- **macOS**: `brew install python`
- **Ubuntu/Debian**: `sudo apt install python3 python3-pip`

**② AI 에이전트** — 파일을 읽고 쓰고 명령을 실행할 수 있는 AI 도구

| 종류 | 예시 |
|---|---|
| CLI | **Claude Code** (제작자 추천), Codex CLI, Gemini CLI |
| IDE 확장 | VS Code / JetBrains용 Claude Code, GitHub Copilot, Cline |
| IDE 자체 | Cursor, Windsurf, Zed, Trae |

> 코딩 지식은 필요 없습니다. 여기서 이 도구들은 **"파일을 다룰 줄 아는 채팅창"** 역할만 합니다.

### 4-2. 프로젝트 받기

**방법 A — Git clone (추천)** : 나중에 업데이트가 명령어 한 줄로 끝납니다.

```bash
git clone https://github.com/hugohe3/ppt-master.git
cd ppt-master
```

내 fork를 쓰려면:

```bash
git clone https://github.com/bmshin94/ppt-master.git
cd ppt-master
```

**방법 B — ZIP 다운로드** : GitHub → 초록 `Code` 버튼 → `Download ZIP`
> 저장소가 1GB를 넘어 무겁습니다. 가볍게 쓰려면 Releases 탭의 `ppt-master-skill-*.zip` (약 56MB, 예제만 빠지고 기능은 동일)

**방법 C — 스킬 마켓플레이스**

```bash
npx skills add hugohe3/ppt-master
```

Claude Code 안에서:

```
/plugin marketplace add hugohe3/ppt-master
/plugin install ppt-master@ppt-master
```

> 방법 C는 스킬 파일만 받습니다. 파이썬 의존성은 **설치된 스킬 경로에서 따로** 설치해야 합니다.

### 4-3. 의존성 설치 (가장 중요)

```bash
pip install -r requirements.txt
```

> `pip`이 인식되지 않으면 → `python -m pip install -r requirements.txt`

주요 의존성: `python-pptx`(PPT 조립), `PyMuPDF`(PDF 읽기), `Pillow`·`numpy`(이미지), `skia-pathops`(도형 불리언 연산), `uharfbuzz`(글자 폭 측정), `edge-tts`(음성), `flask`(실시간 프리뷰), `mammoth`(DOCX), `beautifulsoup4`·`requests`(웹) 등 19개.

### 4-4. 설치 확인

```bash
python3 -c "import pptx; import fitz; print('All core dependencies OK')"
```

`All core dependencies OK` 가 출력되면 성공입니다.

### 4-5. API 키 설정 (선택이지만 강력 추천)

키가 없어도 PPT는 나오지만 **사진 품질 차이가 큽니다.**

```bash
cp .env.example .env    # 그 다음 .env 편집
```

**A) 무료 웹 이미지 검색 (비용 0원, 가성비 최고)**

```bash
PEXELS_API_KEY=your-key      # pexels.com 무료 가입
PIXABAY_API_KEY=your-key     # pixabay.com 무료 가입
```

> 키가 없으면 Openverse / Wikimedia만 사용합니다. 동작은 하지만 일반 사용자 업로드가 많아 품질이 고르지 않습니다.

**B) AI 이미지 생성 (유료, 품질 최상)**

```bash
IMAGE_BACKEND=openai
OPENAI_API_KEY=sk-xxx
OPENAI_MODEL=gpt-image-2
```

```bash
IMAGE_BACKEND=gemini
GEMINI_API_KEY=your-key
GEMINI_MODEL=gemini-3.1-flash-image
```

> 그 외 MiniMax, Qwen, Zhipu, Volcengine, Stability, Flux(BFL), Ideogram, SiliconFlow, fal, Replicate, OpenRouter 등 지원.
> 사용 가능한 백엔드 확인: `python3 skills/ppt-master/scripts/image_gen.py --list-backends`

**C) 음성 나레이션(TTS)** — `edge-tts`는 무료로 바로 동작. ElevenLabs / CosyVoice / MiniMax / Qwen 등 유료 옵션도 지원.

**`.env` 탐색 순서**: 현재 프로세스 환경변수 → 작업 디렉터리 → 스킬 디렉터리 → clone 저장소 루트 → `~/.ppt-master/.env`

---

## 5. 사용법

### 5-1. 시작하기

1. **AI 도구에서 `ppt-master` 폴더를 엽니다.** (IDE는 File → Open Folder, CLI는 `cd ppt-master` 후 실행)
2. **자료를 `projects/` 안에 넣습니다.** (PDF, DOCX, 이미지 등)
3. **채팅창에 말합니다.** 끝입니다.

### 5-2. 명령 예시

**① 기본 (AI가 한 번 확인받고 진행)**

```
projects/report.pdf 로 PPT 만들어줘
```

AI가 디자인 스펙을 확인합니다:

```
AI: 디자인 스펙을 확인할게요
    [템플릿] B) 자유 디자인
    [형식]   PPT 16:9
    [분량]   8-10장
```

**② 빠른 생성 (확인 생략, 원샷)**

```
projects/report.pdf 로 8장짜리 PPT 빠르게 만들어줘, 확인 안 해도 돼
```

> 명시한 것은 지키고, 명시하지 않은 것은 AI가 알아서 결정합니다.
> 단 **되돌리기(resume) 불가** — 컨텍스트가 날아가면 처음부터 다시 해야 합니다.

**③ 자료 없이 주제만**

```
"2026 AI 트렌드" 주제로 10장짜리 발표자료 만들어줘
```

**④ 기존 PPT 재활용 (Edit Native PPTX)**

```
회사양식.pptx 디자인 그대로 두고 내용만 이걸로 바꿔줘
```

**⑤ 디자인만 재작업 (Beautify)**

```
이 pptx 내용은 그대로 두고 디자인만 다시 해줘
```

**⑥ 템플릿 만들기**

```
projects/brand/our_deck.pptx 로 재사용 가능한 Deck 템플릿 만들어줘
```

### 5-3. 결과물 위치

```
projects/프로젝트명_20260903/
├── sources/      원본 자료 및 변환된 마크다운
├── analysis/     추출된 사실 정보
├── notes/        페이지별 내용 정리
├── svg_output/   작업 중인 슬라이드 SVG
├── svg_final/    자체 완결형 미리보기 SVG
├── templates/    프로젝트 레벨 템플릿
└── exports/
    └── 프로젝트명_20260903_1430.pptx   ← 최종 결과물
```

> `projects/` 하위는 `.gitignore`로 제외되어 있습니다.

### 5-4. 부가 기능

**실시간 프리뷰 & 직접 편집** — 생성 중 브라우저가 열립니다 (`http://localhost:5050`, 점유 시 다음 빈 포트)
- 슬라이드가 만들어지는 걸 실시간으로 확인
- **AI 없이 직접 수정**: 요소 클릭 → 글자/색/폰트/크기 변경, 드래그 이동, 방향키 미세조정(Shift = 10px), `Ctrl+Z` 되돌리기
- **또는 주석 남기기**: 요소 클릭 → 원하는 수정사항 입력 → `Add annotation` → `Apply changes` → 채팅창에서 "내 주석 반영해줘"

**애니메이션 & 화면전환** — 기본은 꺼져 있고, 요청하면 등장/강조/모션경로/퇴장 애니메이션과 화면전환을 넣습니다.

**나레이션 & 영상** — 발표자 노트 → TTS 음성 → mp4 영상까지 생성 가능.

**다양한 캔버스** — 16:9 PPT 외에 3:4 카드뉴스, 1:1 정사각, 9:16 스토리, A4 인쇄용 등 지원.

### 5-5. 업데이트

```bash
# git clone으로 설치한 경우
python3 skills/ppt-master/scripts/update_repo.py
```

> 최신 버전을 받고, `requirements.txt`가 바뀌면 의존성도 자동 동기화합니다.
> ZIP 설치는 새 ZIP을 받아 `.env`와 `projects/`만 복사해오면 됩니다.

### 5-6. 문제 해결

| 증상 | 해결 |
|---|---|
| AI가 절차를 잊고 딴짓함 | `skills/ppt-master/SKILL.md 읽어줘` 라고 지시 |
| 도형과 선만 나옴 | 대부분 모델 성능 문제 → 더 좋은 모델로 교체 |
| 글자 넘침 / 요소 어긋남 | 해당 페이지를 지목해 "이 부분 고쳐줘" |
| `pip` 인식 안 됨 | `python -m pip` 사용 |
| Windows에서 python 못 찾음 | PATH 재확인 → `docs/windows-installation.md` |
| 그 외 | **`docs/faq.md`** (실사용자 제보 기반으로 계속 업데이트) |

### 5-7. 꼭 알아야 할 현실

- **모델 성능이 곧 결과 품질입니다.** 제작자 권장 조합: 큰 컨텍스트(약 100만 토큰) 모델 (Claude 또는 Kimi K3) + AI 이미지 생성(`gpt-image-2` 또는 `gemini-3.1-flash-image`).
- **한 방에 완벽한 덱은 나오지 않습니다.** 90%를 만들어주고 마지막 다듬기는 사람 몫입니다. 애초에 "편집 가능하게" 뽑아주는 이유가 그것입니다.
- 제작자도 README에 명시: *"This is a tool, not a wishing well"* (요술램프가 아닙니다).

---

## 6. 수익화 아이디어

### 6-0. 법적 조건 먼저

**MIT 라이선스이므로 상업적 사용이 가능합니다.** 팔아도 되고 수정해도 됩니다. 단 두 가지를 지켜야 합니다.

1. **LICENSE 파일과 저작권 고지 유지** (Copyright (c) 2025-2026 Hugo He)
2. `scripts/attribution_guard.py` 라는 **무결성 검사기**가 있습니다. LICENSE / SPONSORS 파일을 지우거나 변조하면 **스킬이 실행 자체를 거부**합니다.

> 즉 "내가 만든 것처럼 포장해 재배포"는 막혀 있고, **도구로 사용해 수익을 내는 것은 완전히 자유**입니다.
> 생성된 PPT 결과물은 당연히 내 것입니다.

### 6-1. PPT 제작 대행 (진입장벽 최저, 즉시 가능)

크몽 · 숨고 · 탈잉 · Fiverr

| 항목 | 숫자 |
|---|---|
| 시장가 | 장당 1~3만원 (10장 = 15~30만원) |
| 원가 | 덱당 3천~1만원 (AI 토큰 + 이미지) |
| 소요시간 | 초안 30분 + 다듬기 1~2시간 |
| **마진** | **80~90%** |

> **차별점**: 경쟁자 대부분은 "이미지로 박제된 PPT"를 납품합니다.
> **"고객이 직접 수정할 수 있는 진짜 PPT"** 라는 점을 상세페이지 전면에 내세우세요. 후기가 갈리는 지점이 정확히 여기입니다.

### 6-2. 버티컬 특화 (단가 상승 포인트)

"아무거나 다 해드립니다"보다 **한 분야만 파는 게 단가가 3배** 입니다.

| 분야 | 근거 |
|---|---|
| 정부지원사업 사업계획서 | 예비창업패키지·초기창업패키지 시즌 수요 폭발. 건당 30~100만원 |
| IR 덱 / 투자제안서 | 스타트업 필수, 단가 최상위 |
| 강의자료 / 학원 교재 | 반복 발주 → **구독 계약** 전환 가능 |
| 병원·법률·부동산 브로슈어 | 디자이너를 못 쓰는 소상공인 시장, 경쟁 적음 |
| 카드뉴스 / 인스타 콘텐츠 | 3:4, 9:16, 1:1 지원 → SNS 대행으로 확장 |

> 특화하면 브랜드 템플릿을 한 번 만들어 재활용하므로 갈수록 빨라집니다.

### 6-3. B2B 사내 자동화 구축 (단가 최고)

```
기업 방문 → 그 회사 PPT 양식으로 브랜드 템플릿 제작
        → 사내 세팅 + 직원 교육
        → 유지보수 월 계약
```

**세일즈 포인트가 강력합니다.**
- **자료가 회사 밖으로 나가지 않음** (AI 모델 통신 외 전부 로컬 처리) → 금융·공공·의료가 SaaS형 PPT 도구를 못 쓰는 이유가 바로 이것
- **PPT 구독료 0원** (오픈소스)
- 회사 고유 디자인 그대로 유지

> 구축비 300~1000만원 + 월 유지비 수준. 한 곳만 확보해도 규모가 큽니다.

### 6-4. 템플릿 / 디자인 에셋 판매

`Create Template` 기능으로 브랜드·레이아웃 템플릿을 제작해 판매 (미리캔버스, 크몽 템플릿샵, Etsy).
**한 번 만들고 무한 판매 = 패시브 인컴.**

### 6-5. 콘텐츠 / 교육 (초기 현금흐름)

설치 진입장벽이 높다는 게 오히려 기회입니다.
- 유튜브 세팅 튜토리얼
- 전자책 / 노션 가이드 판매 (2~5만원)
- 온라인 강의 플랫폼
- 블로그 애드센스 + 제휴 링크

> 도구로 만든 결과물보다 **"도구 쓰는 법"** 이 먼저 돈이 되는 경우가 많습니다.

### 6-6. SaaS 웹서비스 (난이도 최상, 신중히)

- Gamma, Tome 등 강자가 이미 존재
- API 비용 + 서버비로 적자 나기 쉬움
- "AI 에이전트가 대화하며 만드는" 구조라 무인 서버로 옮기기 까다로움
- 원작자가 계속 업데이트 → 따라가는 유지보수 부담

### 6-7. 시작 전 체크리스트

| 항목 | 내용 |
|---|---|
| 원샷 완벽 아님 | 90% 생성 + 10% 수작업 → **무인 자동화보다 "대행 서비스"가 오히려 정답** |
| 모델 값이 곧 품질 | 원가 아끼려 싼 모델 쓰면 후기가 망가짐 |
| 이미지 저작권 | CC BY 이미지는 크레딧이 슬라이드에 자동 삽입됨. 상업 납품이면 Pexels/Pixabay 키를 넣거나 AI 생성 이미지 사용 |
| 저작권 고지 | 툴 자체를 재배포할 때만 해당. 결과물 PPT는 내 것 |

### 6-8. 추천 로드맵

```
1개월차: 대행 시작 (크몽) + 포트폴리오 10개 확보
   ↓
2~3개월차: 반응 좋은 분야 하나로 특화
   ↓
4개월차~: 해당 분야 템플릿 자산화 → 단가 인상 + 소요시간 단축
   ↓
6개월차~: 단골 기업 확보 → B2B 구축/구독으로 전환
```

리스크가 거의 없고, 실패해도 PPT 제작 역량이 남습니다.

---

## 7. PHP로 만들 수 있나?

### 결론

- ✅ **PHP로 "껍데기(웹 서비스)" 만들기 = 가능하고 권장**
- ❌ **PHP로 "전부 다시 구현" = 비추천**

### 왜 전부 다시 짜면 안 되는가

**규모 문제**

| 항목 | 규모 |
|---|---|
| 파이썬 파일 | 244개 |
| 전체 코드 | **174,487줄** |
| SVG→PPTX 변환 엔진만 | **53,823줄** |

**대체 라이브러리 부재 문제** — PHP 생태계에 동등한 대안이 없습니다.

| 파이썬 | 역할 | PHP 대안 |
|---|---|---|
| `skia-pathops` | 도형 Boolean 연산 (합집합/차집합) | **없음** |
| `uharfbuzz` | 글자 폭 정밀 측정 (텍스트 넘침 방지) | **없음** |
| `python-pptx` | PPT 내부 OOXML 조립 | PHPPresentation 존재하나 커스텀 도형 지원이 약함 |
| `PyMuPDF` | PDF 파싱 | Smalot/pdfparser (성능·정확도 열세) |

> 특히 `uharfbuzz`(글자 측정)가 없으면 **텍스트가 슬라이드 밖으로 넘칩니다.** PPT 자동 생성의 핵심 품질 요소입니다.

### 권장 아키텍처 — 역할 분담

```
┌─────────────────────────────────┐
│   PHP (Laravel)                 │
│   회원가입 · 결제 · 파일 업로드     │
│   대시보드 · 다운로드 · 관리자      │
└──────────┬──────────────────────┘
           │ Queue로 작업 전달
           ▼
┌─────────────────────────────────┐
│   Python 엔진 (기존 그대로)        │
│   PPT 생성 → 파일 출력            │
└─────────────────────────────────┘
```

PHP는 잘하는 것(웹/결제/관리)을, Python은 잘하는 것(문서 처리)을 맡습니다.
억지 조합이 아니라 **실무 표준 패턴**입니다.

### 구현 스케치 (Laravel)

```php
// app/Jobs/GeneratePptJob.php
class GeneratePptJob implements ShouldQueue
{
    public function handle()
    {
        $project = "proj_{$this->orderId}";

        // 1) 프로젝트 초기화
        Process::run("python3 scripts/project_manager.py init {$project}");

        // 2) AI 에이전트에 작업 위임 (Claude Code CLI 등)
        Process::timeout(1800)->run([
            'claude', '-p',
            "projects/{$project}/sources/ 자료로 PPT 빠르게 만들어줘"
        ]);

        // 3) 결과물 회수
        $pptx = glob(base_path("projects/{$project}/exports/*.pptx"))[0];
        Storage::disk('s3')->put("orders/{$this->orderId}.pptx", file_get_contents($pptx));

        // 4) 고객 알림
        $this->order->user->notify(new PptReadyNotification());
    }
}
```

### 좋은 소식

이 프로젝트의 알맹이인 `references/`(45MB)와 `templates/`(63MB)는 **전부 마크다운과 SVG**입니다.
즉 **프로그래밍 언어와 무관한 자산**이며, PHP로 감싸든 Java로 감싸든 그대로 활용됩니다.
코드는 5%, 나머지 95%는 "AI에게 주는 지식"입니다.

### 진짜 어려운 건 기술이 아니라 운영

| 문제 | 현실 |
|---|---|
| 속도 | 덱 하나에 10~30분. 웹에서 대기 UX 설계 필요 |
| 원가 | 요청 1건당 3천~1만원. **월정액 무제한 = 즉사** |
| 품질 편차 | 사람 검수가 필요 → 완전 무인 서비스와 상성이 나쁨 |
| 키 관리 | 유저별 키를 받을지, 내가 부담할지 (부담 시 크레딧제 필수) |
| 서버 | 공유호스팅 불가. VPS 이상 + 큐 워커 필요 |

### 단계별 접근 권장

**1단계 — PHP 관리자 도구부터 (1~2주, 리스크 0)**
```
주문 접수 → 파일 업로드 → 큐 등록 → 수동 실행 → 납품
```
대행 서비스의 **내부 업무 자동화**. 고객에게 노출되지 않아 부담이 없고 작업 시간이 크게 줄어듭니다.

**2단계 — 반자동 웹서비스 (1~2개월) ← 최적 지점**
```
고객 주문/결제 → 자동 생성 → 10분 검수 → 자동 발송
```
품질을 지키면서 손이 덜 갑니다.

**3단계 — 완전 자동 SaaS**
데이터가 쌓이고 원가 구조가 확정된 뒤에만.

### 과금 모델

```
✗ 월정액 무제한   → 헤비유저 한 명에 적자
✓ 크레딧 충전제   → 1크레딧 = 1덱
✓ BYOK 옵션      → 고객이 자기 API 키를 넣으면 할인 (원가 0원)
```

---

## 8. 한 줄 요약

> **PPT Master는 "AI에게 주는 PPT 제작 지식 + 도구 세트"입니다.**
> 설치는 Python 하나면 끝이고, 사용은 채팅으로 말하는 게 전부이며,
> 결과물은 **PowerPoint에서 계속 편집 가능한 진짜 PPT**입니다.
> 수익화는 "대행 → 버티컬 특화 → 템플릿 자산화 → B2B" 순서가 가장 안전하고,
> 서비스화한다면 **PHP는 껍데기, Python은 엔진**으로 역할을 나누세요.
