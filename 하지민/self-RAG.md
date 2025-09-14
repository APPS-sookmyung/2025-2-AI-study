## SELF-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection (2023)

### 목적 (Objective)

- LLM의 hallucination(사실 오류) 문제를 줄이고,
- 불필요한 retrieval(고정 개수 문서 검색)의 한계를 극복하기 위해
- SELF-RAG(Self-Reflective RAG) 프레임워크 제안.
- 핵심: on-demand retrieval + self-critique 를 통해 factuality & controllability 개선.

---

### 주요 구성요소 (Main Components)

사용 모델

- Base LM: LLaMA2 (7B, 13B)
- Retriever: Contriever (dense retriever)
- Critic Model: GPT-4 기반 distillation → reflection token 생성

핵심 기술

- Reflection Tokens (4종류) 1. Retrieve: 검색 필요 여부 2. ISREL: 문서 관련성 평가 3. ISSUP: 답변이 근거로 뒷받침되는지 평가 4. ISUSE: 응답의 전반적 유용성 평가
- Self-Reflection Loop
- Retrieve → Generate → Critique 과정에서 자기 평가
- Customizable Decoding
- inference 시 fact vs creativity 균형 조정 가능

---

SELF-RAG 동작 과정 (Method) 1. 프롬프트 입력
사용자가 질문/명령을 입력 2. Retrieve 판단
모델이 reflection token으로 검색 필요 여부 결정 3. 문서 검색 + 생성
관련 문서들을 참조하여 candidate 답변 생성 4. Critique (자기평가)

- ISREL: 문서 관련성
- ISSUP: 답변-근거 일치도
- ISUSE: 응답 품질 5. 최종 선택
  beam search + critique 점수를 이용해 최적 답변 선택

---

실험 및 결과 (Experiments & Results)

- Tasks: PubHealth (fact verification), ARC (reasoning), PopQA, TriviaQA (open-domain QA), Biography generation, ASQA (long-form QA)
- 비교 Baseline: ChatGPT, Llama2-chat, Alpaca, Toolformer, SAIL 등
- 결과:
- SELF-RAG (7B, 13B) → 모든 태스크에서 baseline보다 성능 우위
- 특히 ASQA에서 citation precision 크게 개선 (ChatGPT보다 정확한 출처 기반)
- Human evaluation에서도 reflection token 판단이 사람 평가와 높은 일치율

---

Ablation & 분석

- No Critic / No Retriever: 성능 급락 → critic + adaptive retrieval 둘 다 핵심
- Retrieve Top-1 고정: conventional RAG보다 성능 낮음 → adaptive retrieval의 중요성 확인
- Inference Customization: ISSUP weight ↑ → 사실성 ↑, fluency ↓ → trade-off 제어 가능

---

결론 (Conclusion)

- SELF-RAG는 LLM이 스스로 검색 필요성을 판단하고 자기평가하는 프레임워크
- 기존 RAG의 고정적 retrieval 방식 한계를 극복
- 다양한 태스크에서 사실성, citation accuracy, controllability 개선
- 실무적으로 fact-critical한 작업(예: 의료/법률 QA, 논문 요약 등)에 특히 유용

---

참고 (Reference)
Akari Asai, Zeqiu Wu, Yizhong Wang, Avirup Sil, Hannaneh Hajishirzi.
Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection. arXiv:2310.11511, 2023.
