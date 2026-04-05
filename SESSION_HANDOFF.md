# 세션 핸드오프 가이드

## 현재 상태
- **저장소**: `faeqsu10/test-10`
- **브랜치**: `claude/check-repo-connection-c1nyt`
- **세션 ID**: `session_01Ld94vNFPjx2ijMdUjUZLKN`

---

## 완료된 작업

### 1. 이미지 텍스트 편집 하네스 팀 구축
harness-project 패턴(Plan→Generate→Evaluate 재귀 루프)을 참고하여 이미지 편집 전용 하네스 팀 구성 완료.

**구조:**
```
test-10/
├── CLAUDE.md                    ← 오케스트레이터 (재귀 루프 로직)
├── agents/
│   ├── planner.md               ← 이미지 분석 + 편집 계획
│   ├── generator.md             ← 이미지 편집 실행
│   ├── evaluator.md             ← 정량 평가 (절대 관대 금지)
│   └── evaluation_criteria.md   ← 4차원 채점 기준
├── projects/
│   └── poster-date-edit/        ← 테스트 프로젝트
│       ├── SPEC.md              ← Planner 출력
│       ├── SELF_CHECK.md        ← Generator 자체 점검
│       ├── QA_REPORT.md         ← Evaluator 평가 (합격 7.9/10)
│       ├── original_backup.png  ← 원본 포스터
│       └── output/
│           └── modified_image.png ← 수정된 포스터
├── .claude/plugins/harness/     ← harness 플러그인
├── .mcp.json                    ← mcp-image 서버 설정
├── .gitignore                   ← .env 보호
└── .env                         ← Gemini API 키 (세션 종료 시 소멸)
```

### 2. 하네스 파이프라인 테스트 완료 (Pillow 기반)
- **R1**: 6.6/10 조건부 합격 (폰트 너무 작음)
- **R2**: 7.9/10 합격 (폰트 크기/굵기 개선, 경계 블렌딩)
- 텍스트 정확도 10/10, 주변 보존 9/10, 시각 일관성 7/10

### 3. MCP 리서치 완료
4개 MCP 비교 조사 완료:
| MCP | 적합도 | 핵심 기능 |
|-----|--------|----------|
| mcp-image (Gemini) | ★★★★★ | 자연어로 이미지 직접 편집 |
| Figma MCP | ★★★★☆ | 텍스트 레이어 정밀 편집 |
| Paper MCP | ★★★☆☆ | 디자인 읽기/쓰기 |
| Stitch (Google) | ★★★☆☆ | UI 디자인 재생성 |

---

## 다음 단계 (WSL에서 이어서 할 작업)

### Step 1: 저장소 클론 및 환경 세팅
```bash
git clone https://github.com/faeqsu10/test-10.git
cd test-10
git checkout claude/check-repo-connection-c1nyt
```

### Step 2: .env 파일 재생성
```bash
echo "GEMINI_API_KEY=여기에_API키_입력" > .env
```

### Step 3: mcp-image MCP 서버 설치
```bash
# Claude Code CLI에서
claude mcp add mcp-image -- npx -y mcp-image

# 또는 .mcp.json이 이미 있으므로 Claude Code 실행만 하면 자동 로드
claude
```

### Step 4: mcp-image로 포스터 편집 테스트
Claude Code에서:
```
mcp-image의 edit_image 도구로 projects/poster-date-edit/original_backup.png의
날짜를 수정해줘. 15→22, 13→20, 14→21
```

### Step 5: Generator 에이전트 업그레이드
generator.md를 수정하여 Pillow 대신 mcp-image(edit_image)를 사용하도록 변경.
이후 하네스 파이프라인을 다시 돌려서 품질 비교.

### Step 6: 다른 MCP도 테스트
- Figma MCP: `claude mcp add --transport http figma https://mcp.figma.com/mcp`
- Paper MCP: Paper Desktop 설치 후 `claude mcp add paper --transport http http://127.0.0.1:29979/mcp`
- Stitch MCP: `npx @_davideast/stitch-mcp init`

---

## Gemini API 키
- **키**: AIzaSyCvcwfy0IW1ufZN3HnLT3Zt6R2lDe7xnow
- **발급처**: Google AI Studio (https://aistudio.google.com/apikey)
- **용도**: mcp-image의 이미지 생성/편집

---

## 핵심 아이디어 (대화에서 논의된 것)

1. **Generator를 MCP 기반으로 업그레이드**: Pillow 픽셀 조작 → mcp-image/Figma/Stitch 등 외부 도구 활용
2. **이미지 직접 수정이 어려우면 유사 이미지 재생성 후 수정**: 원본 스타일을 참고해 새로 만들고, 처음부터 올바른 값으로 텍스트 배치
3. **Plan-Generate-Evaluate 재귀 루프**: 정량 평가 기준으로 자동 반복, 7.0 이상 합격
4. **만드는 AI와 평가하는 AI 분리**: Generator와 Evaluator를 별도 서브에이전트로 실행하여 객관성 확보
