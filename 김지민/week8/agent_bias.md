# A Survey on LLM-as-a-Judge

https://awesome-llm-as-a-judge.github.io/


## BackGround
### LLM-as-a-Judge
LLM을 사용하여 사전에 정의된 규칙/기준/선호에 따라 객체, 행동, 판단을 평가하도록 하는 것

채점자/평가자/비평가/검증자/검사자/보상 및 순위 평가 등

### 프롬프트 방식 (In-Context Learning)
- 점수 부여 방식: “답변의 유용성/관련성/정확성/세부성에 따라 1~10점으로 평가하시오.”
- Yes/No 판단
- Pairwise Comparison: 두 응답 중 더 좋은 것 선택
- 객관식 선택
| 방식       | 활용 상황   | 장점        | 단점          |
| -------- | ------- | --------- | ----------- |
| Score    | 절대품질 평가 | 익숙함       | 편향·불안정      |
| Yes/No   | 사실성·정합성 | 간단        | 정보 손실 크다    |
| Pairwise | 비교/랭킹   | 사람 평가와 유사 | 비용 증가       |
| MC       | 선호성 검사  | 다양한 선택    | 프롬프트 설계 어려움 |

### LLM output -> 평가 결과에 반영
1. 특정 토큰 추출
LLM의 출력에서 해당 숫자나 단어를 규칙 기반으로 파싱할 때의 문제점:
LLM이 항상 딱 "Yes"만 출력하는 게 아니라
"Yes.",
"Conclusion: Yes",
"It should be modified."
처럼 계속 변형해서 말함.

그래서:
“마지막 문장은 반드시 The better response is [A/B] 형식으로 적어라” 같은 형식 강제 프롬프트를 쓰거나
few-shot 예시를 써서 출력을 유도.

하지만:
모델이 지시를 완전히 잘 안 따르는 경우
→ 파싱이 깨져서 잘못된 결과를 읽어갈 수 있음.

즉, 후처리 규칙 설계 + 모델의 instruction following 능력 둘 다 중요.


2. Constrained Decoding (제약 디코딩)
“아예 LLM이 쓸 수 있는 토큰을 강제로 제한하자!”
예: JSON 스키마 강제, 특정 포맷 강제

FSM(유한 상태 기계)을 써서 “다음 토큰 후보”를 필터링하고,
그 외 토큰은 확률을 0으로 막아버리는 방식.

장점:
항상 문법적으로 맞는 형식으로만 출력

단점:
모델의 자연스러운 분포를 왜곡 → 품질 저하 가능
구현 복잡, 처리 속도 느려짐

최신 연구들은 이걸 개선하려고:
- DOMINO, XGrammar, SGLang 같은 시스템 제안
- 미리 계산, speculative decoding, grammar 엔진 최적화 등으로 성능 개선

3. Logits 정규화- 확률 점수(0~1)로 쓰려는 경우
방법 예시:
1) 프롬프트: (x ⊕ C) 뒤에 "Yes"를 붙여서
2) auto-regressive로 "Yes" 토큰이 나올 확률 P("Yes"|context) 계산
3) 이걸 Self-consistency score, Self-reflection score 등으로 쓰고
4) 두 점수를 곱해서 최종 점수 ρ_j를 만들기도 함.

또한:
이 추론 단계가 올바른가?
-> LLM이 "Yes"라고 할 확률로 자기 추론을 평가하는 Self-evaluation에도 사용.

4. 문장/단락 선택 (Selecting sentences)
- 평가 결과가 숫자/토큰이 아니라
    - “어떤 reasoning step이 제일 유망한지”
    - “어떤 경로를 따라가야 할지”
- 같은 것을 선택해야 할 때:
    - LLM-as-a-Judge가
        - 여러 행동/서브질문 후보 중
        - 다음에 펼쳐갈 “한 문장/한 스텝”을 선택하면서 추론 트리(reasoning tree) 를 만들어감.

즉, LLM-as-a-Judge가 “어느 문장을 다음 행동으로 채택할지”를 결정해주는 에이전트 역할.


## LLM - Agent 관계
1. Agent 전체를 평가자로 설계 (Agent-as-a-Judge)
2. 기존 Agent 파이프라인의 한 단계에 LLM-as-a-Judge를 삽입

1. Agent 전체를 평가자로 설계 (Agent-as-a-Judge)
장점: 
- 평가 과정 자체가 더 “깊어질” 수 있음
- 단순 프롬프트 한 번 호출이 아니라 필요할 때마다 다시 질문하고, 검색하고, 재평가하는 식의 멀티스텝 평가 가능
단점: 
- 에이전트가 복잡해질수록 내부 결정 과정을 추적·검증하기 어렵다 (explainability 문제)
- 에이전트가 사용하는 도구나 서브-LLM들이 가진 편향이 그대로 쌓여서 확대될 수 있음


2. 기존 Agent 파이프라인의 한 단계에 LLM-as-a-Judge를 삽입
Reflexion, Self-Refine류 접근:
에이전트가 행동/답변을 만든 뒤, **“자기 자신을 평가”**하는 루프를 돌리는 구조

여기서 LLM-as-a-Judge는:
“이 답변이 괜찮았는지?”
“어떤 점을 수정해야 하는지?”
를 평가하고 피드백을 준다.

이 피드백은:
다음 시도에서 사용하는 메모리/힌트가 되고, 행동 정책을 서서히 개선하는 데 쓰임.

장점: 
- Agent 내부에 **“품질 체크 포인트”**를 박아 넣는 효과.
- 잘못된 행동을 초기 단계에서 차단하거나, 수정 유도 가능.

단점:
- 초반 단계에서 잘못된 판단을 하면 → 그 뒤의 계획·행동이 전부 엇나가는 오류 전파(cascade) 문제.
- Judge가 편향되면 Agent의 전체 행동이 그 편향을 따른다.

## Improvement
### Prompt Design Strategy
1. LLMs’ Task Understanding 개선
(1) Few-shot prompting
가장 많이 쓰이고 효과적인 방법: 프롬프트에 평가 예시를 여러 개 넣어주는 것.

이렇게 하면 LLM은 “무슨 기준으로, 어떤 식으로 점수를 줘야 하는지” 예제를 보고 감을 잡는다.

예: FActScore, SALAD-Bench, GPTScore 등이 이 패턴 사용.

(2) Evaluation Step Decomposition (평가 “단계” 쪼개기)
큰 평가 작업을 여러 작은 단계로 나눠서 프롬프트 안에 적어주는 방법.

예시 느낌:
[1]요약을 처음부터 끝까지 읽어라
[2]중심 주제를 파악해라
[3]문장 간 논리 흐름과 전환(transition)을 확인해라
[4]논리적 순서가 자연스러운지 살펴봐라
  X. 위 기준을 종합해 1~5점으로 점수를 매겨라

(3) Evaluation Criteria Decomposition (평가 “기준” 쪼개기)

“Fluency”을 Grammar, Readability,Engagingness 같은 세부 기준으로 나누고, 각 기준 점수 → 최종 점수를 합성.

(4) 편향 줄이는 프롬프트 설계 (Position Bias 등)
pairwise 비교에서 자주 나타나는 위치 편향(position bias): “앞에 있는 답”을 자꾸 더 높게 평가하는 문제.

이를 줄이려고:
- **응답 순서를 교환(shuffle/swap)**해서 두 번 평가하고,
- 점수를 평균내거나,
- 앞뒤 결과가 충돌하면 “tie”로 처리.

(5) Score → Pairwise 로 task 변환
단일 점수(1~10)로 평가 시: 프롬프트에 민감하고, 난수성 때문에 불안정해질 수 있음.

그래서 몇몇 연구는 점수 평가를 “쌍 비교(pairwise)”로 변환:
ex) PARIS (Liu et al.)
“각 텍스트의 절대 점수” 대신
- 텍스트들 사이의 “누가 더 낫냐”를 여러 번 로컬 비교
- 그 결과로 전체 순서를 전역적으로 정렬
- 사람-LLM 일치도가 더 좋아지고, 결과도 더 안정적.


2. Standardizing LLMs’ Output Format (출력 형식 표준화)
(1) 구조화된 포맷 강제
- JSON 구조로 점수 여러 차원 출력하도록 유도.
- LLM이 코드/JSON 형식 이해를 잘 하는 점을 활용.

(2) 설명(Reason)도 같이 출력시키기
그냥 숫자만 나오면: 이 점수가 프롬프트에 정의한 기준과 진짜로 align 되는지 알기 힘듦.

- CLAIR: 0~100점 + 이유(explanation)를 JSON으로 함께 출력.
- FLEUR: 먼저 점수 부여 → 다시 “왜 이 점수를 줬는지”를 추가 질문해 이유를 생성.

3.Specialized Fine-tuning (특화 파인튜닝)
 메타-evaluation 데이터셋을 만들어 파인튜닝
(1) Evaluation Templates (템플릿 방식)
- 공개된 데이터셋에서 질문/입력을 샘플링
- 미리 정해둔 평가 템플릿에 집어넣음

(2) Deep Transformation (원본을 더 많이 변형)
- 단순한 템플릿 채우기보다 한 단계 더 나아가서:
    - 입력을 변형하거나,
    - “나쁜 답변”을 의도적으로 만들고,
    - 기준/스타일/구조를 바꿔가며 다양하게 데이터 생성.

4. Feedback-Driven Iterative Refinement (피드백 기반 반복 개선)
- INSTRUCTSCORE
    - metric 출력에서 발생한 실패 case들을 모음
    - 각 실패 case에 대해 GPT-4에게 “어디가 문제인지” 피드백 요청
    - 사람 선호와 잘 맞는 설명들을 골라 LLaMA를 반복적으로 파인튜닝

- JADE
    - judge가 낸 평가 결과를 인간이 고쳐 줌
    - 가장 자주 수정된 샘플들을 few-shot 예시 세트에 추가
    → 전체 모델 파라미터를 다시 학습하는 게 아니라, 예시 세트만 갱신해서 점진적으로 개선

- Think-J
오프라인 + 온라인 RL을 둘 다 사용:
- 오프라인: critic 모델을 학습해 judge의 판단을 평가 → 이걸로 SFT/DPO 수행
- 온라인: GRPO(Policy Optimization)로 rule-based reward를 이용해 계속 업데이트
- 사람 선호와 더 가까운 피드백을 사용하면
→ judge 모델이 실시간에 가깝게 개선될 수 있다는 주장.

5. Final Output Optimization Strategy (최종 출력 최적화)
(1)Integrating Multi-Source Evaluation Results
여러 번 평가하거나 여러 모델로 평가한 뒤 통합하는 전략.

(2)Direct Output Optimization
    1) logits + explicit score를 함께 쓰는 방식
    점수(예: 1~10)를 출력할 때:
    - 겉으로 보이는 숫자 말고,
    - 해당 숫자 토큰의 확률분포(logits)를 함께 활용해 스무딩.

    2) Self-verification (자기 검증)
    logits 못 쓰는 상황(대부분의 상용 API)에서도 쓸 수 있는 방법.
    judge가 점수를 낸 후 다시 LLM에게 묻는다: “이 평가에 자신 있어?”
    확신이 부족하거나 일관성 없는 응답은 필터링/폐기.

    이 방식은: 추가 계산은 조금 들지만, 어떤 점수들을 버려야 할지 간단하게 걸러낼 수 있음.

