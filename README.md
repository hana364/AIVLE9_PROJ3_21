# AIVLE9_PROJ3_21

---

# 💄 Cosmetic Brand Review Analysis PoC

> **LangGraph Multi-Agent 기반: 속성 감성 분석 시스템**

본 프로젝트는 코스메틱 브랜드의 프리미엄 브랜드 가치를 정량화하기 위해 단순 감성 분석을 넘어 제품의 세부 속성별로 신뢰도 높은 인사이트를 추출하는 AI 에이전트 시스템입니다.

---

## 🏗️ 1. Multi-Agent WorkFlow

LangGraph를 활용하여 각 에이전트의 독립적인 책임과 협업 구조를 설계했습니다.

### 🤖 Agent Roles & Responsibilities

* **Supervisor Agent**: 전체 상태(`State`)를 관리하며, 분석 결과의 완성도에 따라 다음 단계(`Analyzer`, `Critic`, 혹은 `Finish`)를 결정하는 관제탑 역할을 수행합니다.
* **Analyzer Agent (Analysis Node)**:
* **Task**: 리뷰 원문에서 뷰엘라의 핵심 속성(보습, 향, 성분 등)을 식별합니다.
* **Tech**: `Pydantic`을 활용하여 구조화된 JSON 데이터(`AspectSentiment`)를 강제 출력합니다.


* **Critic Agent (Review Node)**:
* **Task**: 분석된 결과가 리뷰 원문과 일치하는지, 할루시네이션은 없는지 검증합니다.
* **Feedback**: 수정이 필요한 경우 상세한 `reason_code`를 포함하여 재분석을 요청합니다.



---

## 🛠️ 2. Core Implementation (Based on Code)

### ✅ State Management

에이전트 간의 통신은 LangGraph의 `State`를 통해 이루어지며 분석 이력과 검증 결과가 누적 관리됩니다.

```python
class AgentState(TypedDict):
    # 리뷰 원문 및 분석 타겟 정보
    reviews: List[str]
    # 에이전트별 분석 결과 (누적)
    analysis_data: Annotated[list, operator.add]
    # Critic의 최종 승인 여부 및 피드백
    critique: str
    # Supervisor의 제어 경로
    next_node: str

```

### ✅ Reason-Code 기반 재시도 로직

단순한 루프가 아니라, **비판 에이전트의 논리적 근거**에 따라 분석 노드로 재진입하는 조건부 엣지를 구성했습니다.

```python
def router(state: AgentState):
    if "REJECT" in state["critique"]:
        return "analyzer"  # 재분석 수행
    return "end"           # 최종 결과 확정

```

---

## 📈 3. Monitoring & Quality Assurance (LangSmith)

분석 성능의 블랙박스를 해소하기 위해 **LangSmith**를 통한 모니터링을 지원합니다.

* **Execution Trace**: 각 에이전트 노드(`Analyzer` -> `Critic`) 간의 입출력 및 토큰 사용량 실시간 추적
* **Root Cause Analysis**: `reason_code` 기반의 실패 케이스를 수집하여 프롬프트 엔지니어링 및 성능 개선 데이터셋으로 활용
* **Latency Check**: 대규모 리뷰 배치 처리 시의 병목 구간을 시각화하여 최적화 수행

---

## 📊 4. Business Insights & Dashboard

분석된 데이터는 최종적으로 비즈니스 의사결정을 돕는 대시보드 구조로 변환됩니다.

| 분석 항목 | 데이터 내용 | 마케팅 활용 포인트 |
| --- | --- | --- |
| **속성별 감성** | 보습력(0.9), 향(0.4) 등 | 제품의 강점 강조 및 약점 보완 계획 수립 |
| **핵심 키워드** | "천연 유래", "무자극", "고급스러운" | 광고 카피 및 상세 페이지 키워드 최적화 |
| **원문 매칭** | 특정 점수에 대한 실제 고객 보이스 | CS 대응 가이드 및 고객 페르소나 정교화 |

---

## 💡 5. Technical Insights (Conclusion)

* **Self-Correction**: Critic 노드를 통해 LLM의 자기 수정 능력을 극대화하여 분석 정확도를 PoC 수준 이상으로 확보
* **Batch Scalability**: DB 배치 처리 로직과 결합하여 대규모 리뷰 데이터를 안정적으로 처리 가능한 운영 환경 구축
* **Brand Value Alignment**: 모든 분석 로직은 해당 화장품 브랜드의 '프리미엄 자연주의' 정체성을 반영하여 편향 없는 인사이트를 도출

---
