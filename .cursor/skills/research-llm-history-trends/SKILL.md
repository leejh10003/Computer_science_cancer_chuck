---
name: research-llm-history-trends
description: Use when the user asks to 조사, 연구, 답변, 확장, 갱신, 검토, or 최신화 LLM 모델
  발달사·계보·기술 전환점·현재 동향 in this OpenKnowledge project, or to maintain its LLM
  research question set. Do not use for generic wiki scaffolding; hand that work
  to apply-llm-wiki.
---
# LLM 역사·동향 조사 규칙

## 목적

이 프로젝트에서 LLM 모델의 발달사와 최신 동향을 조사하거나 질문 세트를 갱신할 때 일관된 구조와 근거 사슬을 유지한다.

## 작업 절차

1. 먼저 `open-knowledge` 스킬을 읽고 모든 프로젝트 Markdown 읽기·쓰기를 OpenKnowledge MCP로 처리한다.
2. `exec("cat questions/llm-model-history-and-current-trends.md")`로 기준 질문을 읽는다.
3. 쓰기 전에 대상 폴더를 읽는다.
   - 질문을 바꾸면 `exec("ls -A questions")`
   - 출처를 추가하면 `exec("ls -A sources")`
   - 분석을 추가하면 `exec("ls -A research")`
4. 새 문서는 가능한 경우 폴더 템플릿으로 만든다.
   - 질문 세트: `questions/question-set`
   - 출처 카드: `sources/source-note`
   - 분석 노트: `research/research-note`
5. 사용자가 “최신”, “현재”, “요즘”, 특정 기준일 이후의 변화를 묻거나 동향 갱신을 요청하면, 기억에 의존하지 말고 웹에서 현재 정보를 확인한다.
6. 웹 또는 외부 파일을 근거로 쓰기 전에 `open-knowledge` 스킬의 ingest 절차를 따른다. 출처 문서에는 `source_url`, `source_type`, `published_at`, `captured_at` 메타데이터를 가능한 범위에서 기록한다.
7. 출처 우선순위는 다음과 같이 둔다.
   - 논문, 기술 보고서, 모델 카드, 공식 저장소와 표준 문서
   - 독립 벤치마크와 재현 연구
   - 신뢰할 수 있는 2차 해설
   공급자의 주장과 독립적으로 확인된 결과를 분리한다.
8. 분석 문서의 모든 사실 주장에는 `sources/` 아래 로컬 출처 문서를 문장 가까이에 링크한다. 외부 URL만 직접 인용하지 않는다. 근거가 없으면 `(TODO: needs source)`로 표시하거나 주장을 보류한다.
9. 역사적 사건은 발표일, 보고서 공개일, 제품 출시일, 웨이트 공개일을 구분한다. 최신 동향에는 조사 기준일과 확인일을 기록한다.
10. 합의된 사실, 논쟁 중인 해석, 전망을 서로 다른 문장이나 섹션으로 분리한다.
11. 질문 문서는 다음 경우에만 갱신한다.
    - 기존 질문으로 포착되지 않는 지속적인 기술 축이 등장했다.
    - 표현이 중복되거나 답변 가능성이 낮아 구조 개선이 필요하다.
    - 기준일 또는 반복 점검 항목을 갱신해야 한다.
12. 작업이 끝나면 변경 범위에 `audit`을 실행하고 깨진 내부 링크와 Markdown 오류를 해결한다. 루트에 `log.md`가 있으면 그 문서의 기록 규칙을 따른다.

## 산출물 기준

- 질문은 예/아니오보다 비교, 인과, 증거, 한계, 반례를 끌어내는 형태를 우선한다.
- 모델별 나열보다 전환점과 기술 축을 중심으로 구성한다.
- 최신성 주장은 기준일 없이 쓰지 않는다.
- 각 분석은 어떤 기준 질문에 답하는지 명시한다.
- 불확실성과 반대 근거를 생략하지 않는다.

## 경계

일반적인 지식베이스 초기화, 폴더 스캐폴딩, Obsidian 호환 위키 설계가 목적이면 이 스킬을 쓰지 말고 `apply-llm-wiki`에 맡긴다. 이 스킬은 이 프로젝트 안에서 LLM 역사·동향의 질문 설계, 근거 조사, 갱신에만 사용한다.
