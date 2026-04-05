# 이미지 텍스트 편집 하네스 오케스트레이터

이 프로젝트는 3-Agent 하네스 구조로 동작합니다.
사용자의 이미지 편집 요청을 받아, Planner → Generator → Evaluator 파이프라인을 자동 실행합니다.

---

## 프로젝트 구조

```
test-10/
├── agents/                  ← 공통 에이전트 지시서
│   ├── planner.md
│   ├── generator.md
│   ├── evaluator.md
│   └── evaluation_criteria.md
├── CLAUDE.md                ← 이 파일 (오케스트레이터)
├── projects/                ← 프로젝트별 독립 폴더
│   ├── poster-date-edit/
│   │   ├── SPEC.md
│   │   ├── SELF_CHECK.md
│   │   ├── QA_REPORT.md
│   │   ├── original_backup.png
│   │   └── output/modified_image.png
│   └── ...
```

---

## 실행 방법

사용자가 이미지 편집을 요청하면, **프로젝트 이름**을 먼저 정합니다.
프로젝트 이름이 없으면 사용자에게 물어봅니다.

모든 파일 경로는 `projects/{프로젝트명}/` 아래에 생성됩니다.

---

## 실행 흐름

```
[사용자 편집 요청 + 원본 이미지 + 프로젝트 이름]
       ↓
  ① Planner 서브에이전트
     → projects/{프로젝트명}/SPEC.md 생성
       ↓
  ② Generator 서브에이전트
     → projects/{프로젝트명}/output/modified_image.png 생성
     → projects/{프로젝트명}/SELF_CHECK.md 작성
       ↓
  ③ Evaluator 서브에이전트
     → projects/{프로젝트명}/QA_REPORT.md 작성
       ↓
  ④ 판정 확인
     → 합격 (7.0+): 완료 보고
     → 조건부/불합격: ②로 돌아가 피드백 반영 (최대 3회 반복)
```

---

## 서브에이전트 호출 방법

각 단계에서 Agent 도구를 사용하여 서브에이전트를 호출합니다.
중요: 각 서브에이전트는 독립된 컨텍스트에서 실행됩니다.
Generator와 Evaluator를 반드시 다른 서브에이전트로 호출하세요 (분리가 핵심).

---

## 단계별 실행 지시

`{PROJECT}` = `projects/{프로젝트명}` 경로

### 단계 0: 프로젝트 폴더 생성

새 프로젝트면 `projects/{프로젝트명}/output/` 디렉토리를 생성합니다.
원본 이미지를 `{PROJECT}/original_backup.png`로 복사합니다.

### 단계 1: Planner 호출

```
agents/planner.md 파일을 읽고, 그 지시를 따라라.
agents/evaluation_criteria.md 파일도 읽고 참고하라.

원본 이미지: {원본 이미지 경로}
사용자 요청: [사용자가 준 편집 요청]

원본 이미지를 Read 도구로 시각적으로 확인하고, 
Python (Pillow)으로 텍스트 영역을 분석하라.

결과를 {PROJECT}/SPEC.md 파일로 저장하라.
```

### 단계 2: Generator 호출

최초 실행 시:
```
agents/generator.md 파일을 읽고, 그 지시를 따라라.
agents/evaluation_criteria.md 파일도 읽고 참고하라.
{PROJECT}/SPEC.md 파일을 읽고, 편집 계획에 따라 이미지를 수정하라.

원본 이미지: {PROJECT}/original_backup.png

결과를 {PROJECT}/output/modified_image.png 파일로 저장하라.
완료 후 {PROJECT}/SELF_CHECK.md를 작성하라.
```

피드백 반영 시 (2회차 이상):
```
agents/generator.md 파일을 읽고, 그 지시를 따라라.
agents/evaluation_criteria.md 파일도 읽고 참고하라.
{PROJECT}/SPEC.md 파일을 읽어라.
{PROJECT}/QA_REPORT.md 파일을 읽어라. 이것이 QA 피드백이다.

원본 이미지: {PROJECT}/original_backup.png
(주의: 원본에서 다시 시작할 것. 이전 수정본 위에 덧칠하지 말 것)

QA 피드백의 "구체적 개선 지시"를 모두 반영하여 이미지를 수정하라.
"방향 판단"이 "완전히 다른 접근 시도"이면 편집 전략 자체를 바꿔라.

결과를 {PROJECT}/output/modified_image.png 파일로 저장하라.
완료 후 {PROJECT}/SELF_CHECK.md를 업데이트하라.
```

### 단계 3: Evaluator 호출

```
agents/evaluator.md 파일을 읽고, 그 지시를 따라라.
agents/evaluation_criteria.md 파일을 읽어라. 이것이 채점 기준이다.
{PROJECT}/SPEC.md 파일을 읽어라. 이것이 편집 계획서다.
{PROJECT}/original_backup.png를 Read 도구로 확인하라. 이것이 원본이다.
{PROJECT}/output/modified_image.png를 Read 도구로 확인하라. 이것이 검수 대상이다.

검수 절차:
1. 원본과 수정본을 시각적으로 비교하라
2. SPEC.md의 수정 항목이 정확히 반영되었는지 확인하라
3. 수정 영역 외 손상 여부를 Python으로 검증하라
4. evaluation_criteria.md에 따라 4개 항목을 채점하라
5. 최종 판정(합격/조건부/불합격)을 내려라
6. 불합격 또는 조건부 시, 구체적 개선 지시를 작성하라

결과를 {PROJECT}/QA_REPORT.md 파일로 저장하라.
```

### 단계 4: 판정 확인

{PROJECT}/QA_REPORT.md를 읽고 판정을 확인합니다.

- "합격" → 사용자에게 완료 보고
- "조건부 합격" 또는 "불합격" → 단계 2로 돌아가 피드백 반영
- 최대 반복 횟수: 3회. 3회 후에도 불합격이면 현재 상태로 전달하고 이슈를 보고

---

## 완료 보고 형식

```
## 하네스 실행 완료

**프로젝트**: {프로젝트명}
**결과물**: projects/{프로젝트명}/output/modified_image.png
**수정 항목 수**: X개
**QA 반복 횟수**: X회
**최종 점수**: 텍스트정확도 X/10, 시각일관성 X/10, 주변보존 X/10, 배경자연스러움 X/10 (가중 X.X/10)

**실행 흐름**:
1. Planner: [분석 결과 한 줄]
2. Generator R1: [첫 편집 결과 한 줄]
3. Evaluator R1: [판정 결과 + 핵심 피드백 한 줄]
4. Generator R2: [수정 내용 한 줄] (있는 경우)
5. Evaluator R2: [판정 결과] (있는 경우)
...
```

---

## 주의사항

- 서브에이전트 호출 시, 반드시 필요한 파일을 읽도록 지시하세요
- Generator와 Evaluator는 반드시 다른 서브에이전트로 호출하세요 (분리가 핵심)
- 각 단계 완료 후, 생성된 파일이 존재하는지 확인하세요
- Generator는 항상 원본(original_backup.png)에서 시작해야 합니다 (수정본 위에 덧칠 금지)
- 새 프로젝트 시작 시 프로젝트 이름을 먼저 확인하세요
