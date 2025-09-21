# Adaptive In-conversation Team Building for Language Model Agents

## Captain Agent
Captain Agent는 사용자 프록시(User Proxy)와 소통 
-> 일반적인 작업으로 전략적 계획 수립(계획은 작업 끝날 때까지 반복)

### flow
1. 특정 하위 과업 식별, 필요한 역할 규정, 에이전트 팀 구성
2. 다양한 도구를 활용해 하위 과업 해결 위한 대화 참여
3. 과업 완료 시 reflector llm이 반성 보고서를 captain agent에 제공
4. captain agent는 팀 구성이나 하위 과업 지시 조정, 과정 반복, 최종 결과 제시

## adaptive multi-agent team building
- 과제 확인 시 검색(retrieval), 선택(selection), 생성(generation) 과정을 거치며 rag 방식에 의해 구현

1. agent, 도구 검색
역할 n개 제안하고, 임베딩 유사도에 따라 도구 제안

2. agent 필터링
후보 에이전트 중에 적합한 상위 1개 선택하도록, 기권 메커니즘 포함

3. agent 생성(적절한 agent 찾지 못한 경우)
1) 기본 프로필 생성: 이름, 페르소나, 필요한 기술
2) 시스템 메시지 보강: 일반적인 작업 해결 및 환경 상호작용 지침
3) 도구 검색 및 연결: 생성된 시스템 메시지 기준 적합한 도구 검색 후 새 에이전트 연결

- team memory
captain agent는 구축된 팀을 대화별 임시 메모리에 캐싱 (팀원 정보만, 대화 기록X)

## nested conversation and reflection
1) 메인 대화와는 별도의 새로운 대화 시작 = 중첩 대화
llm 기반 관리자가 대화 기록 바탕으로 에이전트 팀 구성원 중 다음 발언저 적응적 선택
동일한 대화 기록 공유, but 메인 대화 기록에는 접근 안됨
대화 기록에는 이전 에이전트 행동과 그에 대한 피드백 포함

- 대화가 끝나면 reflector가 전체 대화 기록 분석
서로 갈등하면 쉽게 충돌 이어질 수 있으며, 합의 못하고 최대 라운드까지 계속 충돌할 수 있음
- 이땐 captain agent가 모호한 지시 내리거나 하위 과제 충분히 명확하지 않을 때 자주 발생
=> 반성 메커니즘 설계

### 반성 메커니즘이란?
llm 기반 반성자가 대화 기록 요약, 내부 분쟁 확인 후 반성 보고서 작성하여 captain agent에 피드백 제공 -> captain agentㄱ 보고서에 따라 결정

## benefits over static team
- 정적 팀은 팀의 능력 범위 제한할 수 있음 
but llm이 참여하는 긴 컨텍스트를 다루는 건 어려움 & 대화 품질 저하 & 기능 중복 에이전트 참여
vs captain agent는 현재 과업에 맞는 최적화된 에이전트 팀 적응적으로 선택, 구축 가능

### 특이점
- 별도 프롬프트 엔지니어링 없이도 뛰어난 성능 발휘
- 동일한 프롬프트 사용 시 평균 정확도 21.94% 향상
- 작은 모델이라도 뚜렷한 지시 따르기 능력 있으면, 성능 좋음