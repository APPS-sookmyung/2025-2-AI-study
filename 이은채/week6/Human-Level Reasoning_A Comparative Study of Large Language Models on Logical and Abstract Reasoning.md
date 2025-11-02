# Human-Level Reasoning: A Comparative Study of Large Language Models on Logical and Abstract Reasoning (Benjamin G. Moreira, 2025)

## 연구 개요

이 논문은 GPT-4o, Claude 3 Opus, Gemini 1.5 Pro, Grok 2, Llama 3, Mistral Large 6개의 대표 LLM을 대상으로, 논리적 사고(logical reasoning) 및 추상적 사고(abstract reasoning) 능력을 인간과 비교분석한 연구이다.

- 논리적 사고(logical reasoning) : 주어진 전제·규칙으로부터 결론을 이끌어내는 능력, 예: “모든 A는 B이다. C는 A이다. 그럼 C는 B인가?” 같은 형태
- 추상적 사고(abstract reasoning) : 구체적 지식이나 사전 학습된 패턴을 넘어서서 새로운 문제 상황을 유연하게 해석하고 유추하는 능력(예: 유사한 구조/패턴을 서로 다른 맥락에 적용)
- 논문에서는 이 둘을 ‘유체지능(fluid intelligence)’의 핵심 요소로 보고 있으며, 단순한 텍스트 생성능력(superficial language ability)과 구분됨

### 핵심 질문

1. 최신 LLM들은 인간 수준의 논리적 추론 능력을 보유했는가?
2. 모델마다 어떤 유형의 논리문제에서 강점/약점을 보이는가?
3. 인간 사고 방식과 LLM의 사고 방식은 어떤 점에서 다르게 보이는가?

---

## 서론

- LLM은 언어 생성 능력에서 뛰어나지만, 진정한 논리적 사고력은 불확실함
- 인간의 사고는 '지식 기반'이지만 LLM은 '패턴 기반 확률적 추론'에 의존함
- 연구는 연역, 귀납, 가설, 유추 추론의 4가지 영역으로 세분화하여 테스트함

---

## 실험 설계

### 데이터셋

- 심리학적 사고력 평가 문제(Wason Selection Task, Raven's Progressive Matrices 등) + 신규 설계된 언어 기반 퍼즐 사용

### 참가자

- 인간 참가자 120명(대학생 이상) + 6개 LLM (GPT-4o, Claude 3 Opus, Gemini 1.5 Pro, Grok 2, Llama 3, Mistral Large)

### 평가 지표

- 정답률(Accuracy)
- 추론 일관성(Coherence)
- 설명 품질(Explanation Quality)
- 오류 유형(Error Typology: factual-사실오류 / logical-논리모순 / self-contradictory-자기모순)

---

## 결과

### 전체 성능 순위

GPT-4o > Claude 3 Opus = Gemini 1.5 Pro > Grok 2 > Llama 3 > Mistral Large

### 영역별 결과

- 연역적 추론 : 전제들이 모두 참이라고 가정할 때, 그 전제로부터 반드시 따라오는 결론을 도출하는 유형
	GPT-4o / Grok 2 (조건문 오류 - 비가 오면 길이 젖는다, 길이 젖었을 때 비가 왔는가?)
- 귀납적 추론 : 몇가지 관찰로부터 일반 규칙을 가설로 세우는 유형
	Claude 3 / Gemini (패턴 과잉일반화 - 불확실성이나 다른 환경 변수들을 생각하지 못함)
- 가설 추론 : 표면이 달라도 구조가 같은 문제를 매핑해 해결하는 유형
	GPT-4o, Claude 3 / Llama, Mistral (가설을 탐색 하기보다, 가장 자주 본 패턴 하나를 출력)
- 유추 추론 : 표면이 달라도 구조가 같은 문제를 매핑해 해결하는 유형
	Gemini 1.5 / GPT-4o (관계/구조 매핑보다 어휘/문맥 반복에 집착)

### 인간 비교

- 상위 3개 모델은 평균 인간 대학생 수준의 정확도에 근접.
- 모델들은 논리적 모순을 스스로 인지하지 못하는 경우가 많았음 (인간은 보통 “이말이 옳은가?”를 스스로 검토하지만, LLM은 검토 없이 답을 냄)
- 그러나 오류 패턴은 인간과 다름
    - 인간: 지식 부족으로 인한 오답 (= 문제 이해 부족, 지식 부족)
    - LLM: 논리 구조 오류로 인한 오답 (= 논리적 절차/전제 설정 자체에 오류)

---

## 실험 해석

- LLM은 정답률은 인간에 근접할 정도로 높지만, 사고 경로가 인간과 다름
	1. 메타인지적 자기 검증(Self-check) 과정이 부재함
	2. 장기 일관성이나 기억 유지 능력이 약함
	3. 확률적 패턴 기반 예측에 강하지만 형식논리에는 약함

### 제안

- 메타인지 모듈 추가 : 자기검증 루프를 프롬프트로 강제, 생성자-검증자 분리, 반례 생성 프롬프트
- 다단계 추론(Reasoning Chain) 강화 : 계획→분해→도구검증→통합의 다단계 추론 파이프라인
- 기억 유지 보강 : 명시적 메모리, 스냅샷

---

## 결론

- 최신 상위 LLM들은 지식기반 문제나 단순 추론문제에서는 인간 평균 수준에 근접하지만 사고방식은 인간과 다름
- 인간 수준의 사고와 비교하면 자기검토, 사고 방식의 일관성, 논리구조 처리능력 측면이 부족함
- 진정한 Human-Level Reasoning을 위해서는
    - 메타인지
    - 기억 유지
    - 다단계 추론 구조가 필요함

