> **ARCHIVED** -- This is the original Korean LANGUAGE_POLICY.md. Superseded by [../LANGUAGE_POLICY.md](../LANGUAGE_POLICY.md).

# Language Policy -- Owner ↔ Agents (Claude Code x GPT-5.3-Codex)

작성일: 2026‑02‑14  
작성자: System Owner

---

## 1. 기본 원칙

- System Owner는 한국어와 영어로 자유롭게 지시(instruction)를 내릴 수 있다.
- Claude Code와 GPT‑5.3‑Codex 에이전트는 **내부 대화, 설계 문서, 코드 코멘트, 로그, 템플릿 등 “에이전트 간/대상 시스템 내부” 텍스트를 기본적으로 영어로 사용한다.**[web:32][web:37]
- Owner가 한국어로 지시하더라도, 에이전트는 다음 전략을 따른다.
  - 지시를 이해한 뒤, **내부 설계/코드/스펙은 영어로 생성**한다.
  - Owner에게 반환하는 요약/보고는 “영어 원문 + (필요 시) 한국어 요약” 구조를 허용한다.[web:36]

---

## 2. 왜 영어를 기본 언어로 사용하는가

- 대부분의 최신 LLM(Claude 계열, GPT‑5.x 계열)은 **영어 데이터 비중이 높고 토크나이저도 영어 중심**으로 설계되어 있어, 동일 내용이라도 영어 표현이 토큰 효율·정확도에서 유리한 경향이 있다.[web:32][web:38]
- 연구·벤치마크에 따르면, Claude와 GPT 계열 모두 한국어를 잘 처리하지만, **정교한 코드·기술 문서·툴 호출 등에서는 영어가 더 안정적이고 일관된 결과**를 준다.[web:33][web:36][web:41]
- 코드/스펙/프로토콜은 다른 도구(검색, 정적 분석, CI 등)와 연동될 가능성이 크므로, **일관된 영어 기반 형식**이 유지되는 것이 장기 유지보수에 유리하다.

---

## 3. Owner 지시와 에이전트 응답

- Owner는 다음 두 가지 방식을 자유롭게 혼용한다.
  - 한국어: “Phase 2 레드팀에서 메모리/동시성 위주로 더 세게 물어봐줘.”
  - 영어: “In Phase 2, focus the red-team review on memory and concurrency issues.”
- 에이전트 동작 규칙:
  - Owner 입력이 한국어일 경우:
    1. 내부적으로 영어 요약을 생성한다.
    2. 설계/코드/스펙/로그는 영어로 작성한다.
    3. Owner 요청 시 한국어 요약·설명을 추가할 수 있다.
  - Owner 입력이 영어일 경우:
    - 그대로 영어 컨텍스트를 사용하여 작업한다.

---

## 4. 아티팩트별 언어 정책

- `.aisync/BLUEPRINT.json`
  - 모든 키와 값(특히 `core_principles`, `constraints`, `io_contracts`, `security_policies`, `non_functional_requirements`)은 영어로 작성한다.[file:31]
  - Owner가 한국어 설명을 추가해야 하는 경우, 별도 필드(예: `"notes_ko"`) 또는 별도 문서에 기록한다.

- `.aisync/AUDIT_LOG.md`
  - 레드팀 리뷰 본문(Description, Evidence/PoC Idea, Impact, Suggested Mitigation, Recommendations)은 영어로 작성한다.[file:31][web:13]
  - 필요 시 맨 아래에 `## Owner Notes (Korean Summary)` 섹션을 두고, Claude가 한국어로 핵심 내용을 요약할 수 있다.

- `specs/01_interface_def.ts` 및 기타 스펙 파일
  - 타입 이름, 필드 이름, 주석, 설명은 모두 영어로 작성한다.[file:31]
  - 한국어 예시는 주석으로만 제한적으로 추가한다.

- `src/**` 코드 및 테스트
  - 코드, 함수/변수 이름, 주석, 로그 메시지, 에러 문자열, CI/배포 스크립트 내 텍스트는 영어를 기본으로 작성한다.[file:31]
  - 사용자-facing 텍스트(에러 메시지, UI 문자열)는 국제화 레이어(i18n)에서 별도 관리하며, 한국어/영어 리소스는 번역 파일을 통해 제공한다.[web:33][web:35]

- `STATE.yaml`
  - 모든 키와 값(`phase`, `owner_model`, `status`, `reason` 등)은 영어를 사용한다.[file:31]
  - Owner가 한국어 코멘트를 남길 경우 YAML 주석(`# ...`)으로만 작성한다.

- `docs/**, README, CHANGELOG`
  - 기본 문서는 영어로 작성한다.
  - 한국어 문서가 필요할 경우 `docs/ko/**` 디렉토리에 별도로 유지한다.
  - 동일 내용에 대한 다국어 문서를 유지할 때는, 영어 버전을 “소스 오브 트루스”로 간주한다.

---

## 5. .ai_sync 및 CLI 메시지

- `.ai_sync` Manager와 관련 CLI(`ai-sync status`, `ai-sync next`, `ai-sync check`)의 기본 출력 언어는 영어다.[file:31]
- 향후 확장:
  - 옵션 예: `ai-sync status --locale=ko` 형태로 일부 한국어 메시지를 제공할 수 있으나,
  - 내부 상태(STATE.yaml)와 로그는 여전히 영어를 기준으로 유지한다.

---

## 6. 예시 워크플로우

1. Owner (한국어 지시)  
   - “이번 기능에서 Internal Mesh에서만 돌아가는 보안 AI 비서 설계를 Phase 1에서 정리해줘.”

2. Claude Code  
   - 한국어 지시를 영어로 요약.  
   - `.aisync/BLUEPRINT.json`, `specs/01_interface_def.ts`를 영어로 작성.  
   - 필요 시 Owner에게 한국어 요약(예: `docs/ko/feature_x_summary.md`) 제공.

3. GPT‑5.3‑Codex  
   - 영어 BLUEPRINT/스펙을 기반으로 `.aisync/AUDIT_LOG.md`를 영어로 작성하고 PoC/테스트를 구현.

4. Owner  
   - 필요 시 AUDIT_LOG 하단의 `Owner Notes (Korean Summary)` 섹션 또는 별도 한국어 문서를 통해 결과를 확인.


## Reference

### 언어(영어 vs 한국어)·토크나이저 관련

- **"How does a Language‑Specific Tokenizer affect LLMs?"**
  - 내용: 언어별 토크나이저 설계가 토큰 효율·성능에 미치는 영향 분석. 영어 중심 토크나이저가 비영어 언어에서 비효율을 만드는 사례.
  - 위치: arXiv에 공개된 논문.

- **"From LLM to NMT: Advancing Low‑Resource Machine Translation with Claude"**
  - 내용: 저자원 언어에서 LLM 기반 번역 성능 분석. 영어 vs 저자원 언어 간 성능 격차, LLM과 기존 NMT 비교.
  - 위치: arXiv, 기계번역/저자원 언어 관련 논문.

- **"Claude AI 영어 vs 한국어 성능 차이 분석" – 콩나무콩 블로그**
  - 내용: Claude를 영어/한국어로 사용했을 때의 성능 차이를 사례 중심으로 분석. 영어가 더 자연스럽고 안정적인 결과를 주는 경향 보고.
  - 위치: jbeanstalk.com의 블로그 글.

- **"Multilingual capabilities of GPT: A study of structural ambiguity"**
  - 내용: GPT 계열 모델이 영어·한국어·일본어 등 다양한 언어에서 구조적 모호성을 어떻게 처리하는지 평가. 고자원 언어(영어) 우위 경향.
  - 위치: PLOS ONE 또는 PMC(NCBI)에 공개된 논문.

- **"Navigating Korean LLM Research #1: Models"**
  - 내용: 한국어 LLM 연구 리뷰. 한국어 전용/멀티링구얼 모델의 특징, 토크나이저, 성능 비교.
  - 위치: Hugging Face 블로그 시리즈(한국어 LLM 정리).

- **"Performance evaluation of large language models on multilingual tasks" (유사 제목의 Nature/Scientific Reports 논문)**
  - 내용: 다국어 태스크에서 여러 LLM의 성능 비교. 고자원 vs 저자원 언어 성능 차이.
  - 위치: Nature 계열 저널(Scientific Reports 등)의 논문.

