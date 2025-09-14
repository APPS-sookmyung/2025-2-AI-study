# MA-RAG: Multi-Agent Retrieval-Augmented Generation via Collaborative Chain-of-Thought Reasoning

## 1. 기존 RAG의 한계
- **검색 불일치 (Retrieval Mismatch):** 사용자의 질문 의도와 실제 문서 내용 간의 의미적 차이로 정확한 문서를 찾는 데 실패
- **문맥 비효율성 (Context Inefficiency):** 검색된 모든 정보를 그대로 사용하여, 불필요한 정보(Noise) 발생

---

## 2. MA-RAG의 5개의 에이전트
#### 1. Planner Agent
- 질문을 가장 먼저 분석하여 모호함을 해결하고, 복잡한 질문을 여러 개의 하위 작업으로 분해하여 전체 작업 계획을 수립
- `사고의 사슬(Chain-of-Thought)`의 프롬프팅을 통해 단계별 추론 계획을 생성

#### 2. Step Definer Agent
- 플래너가 세운 추상적인 계획을 받아, 각 단계에 맞는 구체적인 검색어를 생성
- 이전 단계의 결과를 참고하여 다음 검색어를 동적으로 생성-> 문맥을 유지

#### 3. Retrieval Tool
- 외부 데이터베이스에서 관련 문서를 실제로 찾아오는 역할을 수행

#### Extractor Agent
- 검색된 문서 전체를 청크한 거 덧붙이기X
- 현재 작업 단계와 직접적으로 관련된 핵심 문장이나 데이터만 추출

#### 5. QA Agent
- 각 단계에 대한 중간 답변을 생성
- 모든 단계가 끝나면 이를 종합하여 최종 답변을 완성

필요에 따라 에이전트를 호출하는 **온디맨드(On-demand) 방식**