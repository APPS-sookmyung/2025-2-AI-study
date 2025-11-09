# Towards Effective GenAI Multi-Agent Collaboration: Design and Evaluation for Enterprise Applications

## 개요

### 협업 에이전트
- 중앙집중형: 하나의 중앙 에이전트가 다른 에이전트에게 작업 할당하고 조정
- 분산형: 각 에이전트가 자율적으로 판단하고 협력

### 계층형 에이전트 구조
- supervisor agent: 작업 계획 세우기, 작업 분해 및 할당, 하위 agent 간의 소통 조율
- 장점: 각 에이전트 자신의 역할 맞는 맥락만 유지, 개발과 테스트 훨씬 용이

### agent 간 지식 교환
- 기본적으로 동기 방식으로 통신하며, 응답 받을 때까지 송신자의 실행 멈춤
- 향후 비동기 통신 도입해, 응답 기다리는 동안 다른 작업 수행

### multi-agent 평가
- 이 논문이 assertion-based benchmarking이라는 새로운 자동 평가 방식을 제안
    - payload referencing
    - 동적 라우팅

### 다중 에이전트 프레임워크 및 플랫폼
- CrewAI
    - 각 에이전트 역할 부여
- AutoGen
    - 다중 에이전트 간 복잡한 대화와 협업을 가능하게 하는 프레임워크
    - 에이전트들이 반복적 상호작용을 통해 정보를 교환하고 결과를 개선
- Langgraph
    - 에이전트의 상호작용을 DAG 형태로 표현하는 혁신적인 구조 제시
    - 작업 간 의존 관계 시각화하고 관리
    - 각 agent 행동이 이전 단계의 결과에 기반하도록 정보 흐름과 의사 결정 최적화

### 종단형 에이전트 평가 프레임워크
- AgentEva
    - 3개의 평가용 에이전트(CriticAgent, QuantifierAgent, VerifierAgent)가 협력하여 대화를 평가
        - CriticAgent: 작업 성공 기준(criteria) 목록을 생성
        - QuantifierAgent: 대화가 기준을 충족했는지 수치로 평가
        - VerifierAgent: 평가 결과의 정확성과 완전성을 검증

- AgentQuest
    - “진행률(progress rate)”이라는 새로운 지표를 제안
    - 에이전트가 특정 마일스톤(milestone) —  목표를 달성하기 위한 환경 내 상태(state)에 도달했는지를 기준으로 성공 여부를 측정
    - 게임 환경에 주로 사용

- ToolSandbox
    - 각 세션마다 마일스톤(발생해야 하는 이벤트)과 마인필드(발생하면 안 되는 이벤트)를 사전에 정의

## 모델링
### 에이전트 간 통신
1. 통합된 인터페이스
- 시스템 내에서 사용자는 또 하나의 “에이전트”로 간주
- 사용자–Supervisor 간, Supervisor–Specialist 간 모든 상호작용이 동일한 인터페이스를 사용

2. 병렬 통신
- Supervisor Agent는 여러 Specialist Agent와 동시에 통신

3. 기존 함수 호출 기능 활용
- 통신 자체를 “하나의 도구(tool)”로 모델링함으로써, Supervisor Agent는 기존의 함수 호출 구조를 그대로 활용해 메시지를 보낼 수 있음.

### payload 참조
페이로드 참조는 대용량 콘텐츠(예: 코드 블록)를 효율적으로 주고받기 위한 메커니즘
-> 목적: Supervisor Agent의 지연시간을 줄이는 것
ex) Supervisor Agent(A)가 Specialist Agent(B)에게 다른 Agent(C)의 결과를 기반으로 한 작업을 요청해야 한다면,
A는 굳이 C의 결과 전체를 다시 생성할 필요가 없음.
- 작동방식
    - Specialist Agent가 코드 등 구조화된 콘텐츠를 생성하면 시스템이 자동 감지한다.
    - 해당 콘텐츠는 “Payload”로 식별되어 고유 ID와 태그가 부여된다.
    - Supervisor Agent는 새 메시지를 보낼 때 전체 내용을 재생성하지 않고, 이전 Payload ID를 참조한다.
    - 시스템은 전송 직전에 태그를 실제 Payload 내용으로 자동 치환한다.

### 동적 라우팅
Supervisor Agent는 보통 여러 Specialist Agent와 여러 번의 통신을 거쳐 복잡한 작업을 수행한다.
하지만 간단한 요청까지 항상 이 과정을 거치면 불필요한 지연과 오버헤드가 발생한다.
-> 동적 에이전트 라우팅으로 Supervisor를 거치지 않고, 요청을 직접 관련 Specialist Agent에게 전달하는 방식
ex) 사용자의 요청이 단순히 “문법 교정해줘”라면, Supervisor를 거칠 필요 없이 바로 Language Agent로 라우팅된다.
- “이 요청은 단순 라우팅으로 처리 가능하다”고 판단되면 Supervisor를 건너뜀.
- 확신이 없으면 기존의 전체 조정 프로세스(Orchestration)를 실행.
- Supervisor는 언제나 이 라우팅 과정을 인지하고 있으며, 직접 처리하지 않아도 해당 대화 기록은 시스템 메모리에 저장된다.

## multiagent 평가
### Assertion-Based Evaluation Framework
테스트 케이스처럼 “이 대화가 성공하려면 반드시 참이어야 하는 조건(assertion)”을 정의하고 자동으로 검증하는 방식

- Benchmark Data (데이터 구성요소)
: 시나리오 + 입력 문제 + 단언문 목록 (user/system 기준으로 나눔)
    - User-side assertions: 사용자가 관찰할 수 있는 행위 (예: 응답의 정확성)
    - System-side assertions: 내부 동작(예: 툴 호출, 매개변수 정확성, 규칙 준수 등)

- Simulation
: User Simulator(시나리오 설명과 입력 문제) + Action Simulator로 실제 대화 환경을 재현

- Assertion Judge
: 시뮬레이션 결과를 LLM이 자동으로 판정(True/False + 근거)

### metrics
- 모든 단언이 참일 때 성공
- 통신 지연, 응답 시간, 토큰 수 등으로 시스템 속도 평가
- 올바른 에이전트로 메시지를 보내는 정확도·속도 평가