# Root Inside v0.1

**IR 자료 분석용 프롬프트 생성 시스템**

Gemini API를 활용하여 IR 자료(피치덱, 사업계획서 등)를 체계적으로 분석하기 위한 구조화된 프롬프트를 자동 생성하고 즉시 실행하는 도구.

---

## 시스템 개요

### 목적
복잡한 AI 파이프라인 없이, **프롬프트 엔지니어링만으로** IR 문서를 일관되게 검토하는 시스템 구축.

### 핵심 컨셉
```
사용자 입력 (문서 내용) 
    ↓
프롬프트 템플릿 선택 & 구성
    ↓
Gemini API 호출
    ↓
구조화된 분석 결과
```

### 특징
- ✅ **Zero Setup**: Gemini API 키만 있으면 즉시 실행
- ✅ **프롬프트 중심**: 15종 체계화된 프롬프트 템플릿
- ✅ **무료 Tier**: Gemini 1.5 Flash (무료) 사용
- ✅ **즉시 사용**: CLI/Web UI에서 바로 분석

---

## Tech Stack
```
Core
├─ Python 3.11+ (최소 의존성)
├─ google-generativeai (Gemini API)
└─ pydantic (프롬프트 validation)

Optional (Web UI)
├─ Streamlit (단순 UI)
└─ python-dotenv (환경변수)
```

---

## Quick Start

### 1. 설치
```bash
# Clone
git clone [repo-url]
cd root-inside

# 가상환경
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# 의존성 설치 (최소)
pip install google-generativeai pydantic python-dotenv

# Optional (Web UI)
pip install streamlit
```

### 2. API 키 설정
```bash
# .env 파일 생성
echo "GEMINI_API_KEY=your_api_key_here" > .env
```

**Gemini API 키 발급**: https://makersuite.google.com/app/apikey

### 3. 실행

#### CLI 모드
```bash
# 섹션 라벨링
python main.py label --text "당사는 2024년 매출 50억원을 달성했습니다..."

# 레드플래그 검사
python main.py check --file pitch_deck_text.txt

# 전체 분석
python main.py analyze --file document.txt --output report.json
```

#### Web UI 모드
```bash
streamlit run app.py
```
→ http://localhost:8501 접속

---

## 프롬프트 템플릿 구조

### 템플릿 디렉토리
```
prompts/
├── base/
│   └── system_prompt.txt          # 공통 시스템 프롬프트
├── labeling/
│   ├── section_labeler.txt        # 섹션 분류
│   └── metric_extractor.txt       # 수치 추출
├── checking/
│   ├── consistency_checker.txt    # 정합성 검증
│   ├── red_flag_scanner.txt       # 레드플래그
│   └── kpi_validator.txt          # KPI 검증
├── generation/
│   ├── summary_generator.txt      # 요약 생성
│   ├── question_generator.txt     # 질의 생성
│   └── bull_bear_analyzer.txt     # Bull/Bear 분석
└── templates.yaml                 # 템플릿 메타데이터
```

### 프롬프트 설계 원칙

1. **명확한 역할 정의**
```
역할: [구체적 역할명]
전문성: [요구되는 전문 지식]
목표: [달성해야 할 명확한 목표]
```

2. **구조화된 입력**
```
입력 형식:
- 문서 타입: [pitch_deck|business_plan|financial_statement]
- 분석 범위: [전체|특정 섹션]
- 컨텍스트: [이전 분석 결과 등]
```

3. **제약 조건 명시**
```
제약:
- 추정/가정 금지
- 원문 수치 보존
- 불확실 시 명시
- JSON 포맷 준수
```

4. **출력 포맷 지정**
```json
{
  "result": "...",
  "confidence": 0.0-1.0,
  "evidence": "원문 인용",
  "reasoning": "판단 근거"
}
```

---

## 프롬프트 템플릿 (주요 15종)

### 1. 섹션 라벨러 (`prompts/labeling/section_labeler.txt`)
```
# 역할
당신은 IR 문서 구조 분석 전문가입니다.

# 작업
주어진 텍스트를 다음 13개 섹션 중 하나로 분류하세요.

## 섹션 정의
1. **Problem**: 해결하려는 시장 문제, 고객 페인포인트
2. **Solution**: 제품/서비스 솔루션 설명
3. **Market**: TAM/SAM/SOM, 시장 규모, 성장성
4. **Product**: 제품 기능, 로드맵, 기술 스택
5. **Business_Model**: 수익 모델, 가격 정책, 유닛 이코노믹스
6. **Traction**: 매출, 고객 수, 성장 지표
7. **Financials**: 재무제표, 비율, 예측
8. **Competition**: 경쟁사, 차별화 포인트
9. **GTM**: Go-to-Market 전략, 채널, 파트너십
10. **Risks**: 사업 리스크, 규제, 의존성
11. **Use_of_Proceeds**: 자금 사용 계획
12. **Team**: 팀 구성, 경력, 자문단
13. **Other**: 위 카테고리에 해당하지 않음

# 입력
"""
{user_text}
"""

# 출력 형식
반드시 다음 JSON 형식으로 답변하세요:
{
  "label": "섹션명",
  "confidence": 0.0-1.0,
  "rationale": "분류 근거 (30자 이내 원문 인용)",
  "key_phrases": ["핵심 키워드1", "핵심 키워드2"]
}

# 규칙
- 원문의 숫자, 단위, 고유명사를 정확히 보존하세요
- 불확실할 경우 confidence < 0.7로 표시하세요
- 추정이나 해석을 추가하지 마세요
- 여러 섹션에 해당할 경우 가장 지배적인 하나만 선택하세요
```

### 2. 수치 추출기 (`prompts/labeling/metric_extractor.txt`)
```
# 역할
당신은 재무/사업 지표 추출 전문가입니다.

# 작업
텍스트나 표에서 정량 지표를 추출하여 구조화하세요.

# 추출 대상 지표
## 재무 지표
- revenue (매출)
- gross_margin (매출총이익률)
- operating_margin (영업이익률)
- ebitda
- net_income (순이익)
- cash_balance (현금)

## 사업 지표
- customers (고객 수)
- arpu (고객당 평균 매출)
- cac (고객획득비용)
- ltv (고객생애가치)
- churn_rate (이탈률)
- retention_rate (잔존율)
- mau/dau (월간/일간 활성 사용자)

## 운영 지표
- headcount (인원)
- capex (자본적지출)
- opex (운영비)

# 입력
"""
{user_text}
"""

# 출력 형식
{
  "metrics": [
    {
      "name": "revenue",
      "value": 5000000000,
      "unit": "KRW",
      "period": "2024",
      "period_type": "annual",
      "source": "원문 인용 (20자 이내)",
      "confidence": 0.95
    }
  ]
}

# 규칙
1. **단위 변환 금지**: 원문의 단위를 그대로 사용하세요
   - "50억원" → value: 5000000000, unit: "KRW"
   - "5 million USD" → value: 5000000, unit: "USD"

2. **기간 명시**: 해당 수치의 기간을 정확히 표시하세요
   - period_type: annual|quarterly|monthly|cumulative

3. **불확실성 처리**: 
   - 명확한 수치가 아니면 confidence < 0.8
   - 추정치는 is_estimated: true 추가

4. **계산 금지**: 원문에 있는 수치만 추출하세요
   - 성장률은 계산하지 말고 원문에 있으면 추출
```

### 3. 정합성 검사기 (`prompts/checking/consistency_checker.txt`)
```
# 역할
당신은 IR 문서 정합성 검증 전문가입니다.

# 작업
문서 내 주장과 수치, 가정 간의 불일치를 탐지하세요.

# 검증 항목
1. **텍스트 vs 표 불일치**
   - 본문에서 언급한 수치와 표의 수치가 다른 경우

2. **슬라이드 간 가정 충돌**
   - 앞 페이지와 뒷 페이지의 전제가 모순되는 경우

3. **재무 공식 오류**
   - GM = Revenue - COGS
   - Operating Margin = EBIT / Revenue
   - 위 공식이 성립하지 않는 경우

4. **시계열 논리 오류**
   - YoY 성장률과 실제 수치가 맞지 않음
   - QoQ와 YoY가 상충

5. **비율 합계 오류**
   - 비중 합계가 100%가 아님
   - TAM > SAM > SOM 순서 위반

# 입력
"""
{user_text}
"""

# 출력 형식
{
  "inconsistencies": [
    {
      "type": "text_table_mismatch",
      "severity": "high|medium|low",
      "description": "본문에서 '매출 50억'이라 했으나 표에는 48억으로 기재",
      "evidence": {
        "text_claim": "원문 인용",
        "table_value": "표에서 추출한 값",
        "location": "페이지 5, 슬라이드 3"
      },
      "impact": "투자자가 실제 매출을 과대평가할 위험"
    }
  ],
  "summary": {
    "total_issues": 3,
    "high_severity": 1,
    "medium_severity": 2
  }
}

# 규칙
- 의심스러운 경우 보수적으로 판단 (flag 처리)
- 명백한 오타는 제외 (예: 50억 vs 50.1억)
- 근거가 불충분하면 "판단 유보" 명시
```

### 4. 레드플래그 스캐너 (`prompts/checking/red_flag_scanner.txt`)
```
# 역할
당신은 IR 문서 리스크 탐지 전문가입니다.

# 작업
다음 15가지 레드플래그를 검사하고 발견된 항목을 보고하세요.

# 레드플래그 체크리스트

## 시장/경쟁 (3개)
1. **unrealistic_tam**: TAM이 비현실적으로 큼
   - 체크: TAM > 국가 GDP × 해당 산업 비중
   - 또는 출처 없이 "XX조원 시장"만 언급

2. **weak_competitor_analysis**: 경쟁 분석 부실
   - 경쟁사 언급 없음
   - 또는 "경쟁자 없음" 주장

3. **market_assumption_vague**: 시장 가정 모호
   - 점유율 목표 근거 없음
   - 성장 시나리오 불명확

## 수치/재무 (5개)
4. **kpi_definition_missing**: 핵심 KPI 정의 부재
   - CAC/LTV/ARPU 산식 없음
   - 코호트 기준 불명확

5. **unit_inconsistency**: 단위 혼재
   - 억원, 백만원, % 혼용
   - 통화 단위 불일치 (원/달러)

6. **margin_cogs_contradiction**: 마진-원가 모순
   - GM 상승하는데 COGS% 동일 또는 상승
   - 규모의 경제 주장하나 단위 원가 개선 없음

7. **financial_formula_error**: 재무 공식 오류
   - 성장률 계산 오류
   - 비율 합계 100% 초과

8. **yoy_qoq_conflict**: 기간별 성장률 충돌
   - YoY 50% 상승인데 각 분기 성장률 합산 시 불일치

## 차트/시각화 (2개)
9. **chart_axis_distortion**: 그래프 왜곡
   - Y축 0 시작 안 함
   - 축 단위 불명확
   - 비선형 축 사용

10. **vanity_metrics_only**: 허영 지표만 강조
    - MAU/다운로드만 언급, 매출/잔존율 없음
    - 누적 지표만 있고 기간별 추이 없음

## 운영/전략 (3개)
11. **use_of_proceeds_vague**: 자금 사용처 모호
    - "운영 자금" 같은 포괄적 표현만
    - 마일스톤과 연결 안 됨

12. **customer_concentration_high**: 고객 집중도 높음
    - Top 3 고객이 매출의 50% 이상
    - 단일 고객 의존도 언급 없음

13. **opex_capex_unclear**: 비용 구조 불명확
    - 고정비/변동비 구분 없음
    - CAPEX 계획 누락

## 리스크/거버넌스 (2개)
14. **regulatory_risk_absent**: 규제 리스크 미언급
    - 인허가 필요한 사업인데 언급 없음
    - 법적 제약 사항 누락

15. **audit_opinion_missing**: 감사의견 누락
    - 재무제표 있으나 감사의견 없음
    - 특기사항 설명 없음

# 입력
"""
{user_text}
"""

# 출력 형식
{
  "red_flags": [
    {
      "flag_id": "unrealistic_tam",
      "severity": "high",
      "title": "TAM 과장 의심",
      "description": "국내 전체 시장이 50조원이라 주장하나, 관련 산업 통계(12조원)의 4배",
      "evidence": {
        "quote": "국내 TAM 50조원 추정",
        "location": "페이지 5"
      },
      "why_it_matters": "시장 잠재력 과대평가로 투자 판단 왜곡 가능",
      "followup_question": "TAM 산정 근거 및 출처(리포트명, 연도) 요청",
      "recommendation": "공개된 시장조사 리포트로 교차검증 필요"
    }
  ],
  "summary": {
    "total_flags": 5,
    "by_severity": {
      "high": 2,
      "medium": 2,
      "low": 1
    },
    "by_category": {
      "market": 1,
      "financial": 2,
      "risk": 2
    }
  }
}

# 판단 기준
- **High**: 투자 판단에 직접 영향, 즉시 확인 필요
- **Medium**: 신뢰도에 영향, 실사 과정에서 확인
- **Low**: 개선 권장 사항, 중요도 낮음

# 규칙
- 보수적 판단 (의심스러우면 flag)
- 근거 반드시 명시 (원문 인용)
- 판단 불가 시 null 반환, 추측 금지
```

### 5. Executive Summary 생성기 (`prompts/generation/summary_generator.txt`)
```
# 역할
당신은 투자심사역 관점의 IR 요약 전문가입니다.

# 작업
문서를 분석하여 투자자용 1페이지 Executive Summary를 작성하세요.

# 포함 요소
1. **Company Snapshot** (2-3문장)
   - 비즈니스 모델
   - 타겟 고객
   - 핵심 가치 제안

2. **Bull Points** (3개)
   - 투자 매력 포인트
   - 정량 근거 포함

3. **Bear Points** (3개)
   - 투자 우려 사항
   - 리스크 요인

4. **Key Metrics** (5-7개)
   - 매출, 성장률, 마진
   - CAC, LTV, Churn 등

5. **Top Risks** (3-5개)
   - 시장, 재무, 운영, 규제 리스크

6. **Next Actions** (3개)
   - 실사 시 확인 필요 항목

# 입력
"""
{analysis_results}
"""

# 출력 형식
# Executive Summary: [회사명]

## Company Snapshot
[2-3 문장으로 회사 설명]

## Investment Highlights

### Bull Case
1. **[제목]**: [설명 + 정량 근거]
2. **[제목]**: [설명 + 정량 근거]
3. **[제목]**: [설명 + 정량 근거]

### Bear Case
1. **[제목]**: [우려 사항 + 영향]
2. **[제목]**: [우려 사항 + 영향]
3. **[제목]**: [우려 사항 + 영향]

## Key Metrics
| Metric | Value | Note |
|--------|-------|------|
| 매출 (2024) | XX억원 | YoY +XX% |
| GM | XX% | - |
| CAC/LTV | 1:X | - |

## Top Risks
1. **[리스크명]**: [설명]
2. **[리스크명]**: [설명]
3. **[리스크명]**: [설명]

## Recommended Actions
- [ ] [확인 필요 항목 1]
- [ ] [확인 필요 항목 2]
- [ ] [확인 필요 항목 3]

# 톤
- 중립적, 사실 기반
- 간결하고 스캔 가능하게 (불릿 포인트 활용)
- 전문 용어는 최소화하되 정확하게
```

### 6. 질의 리스트 생성기 (`prompts/generation/question_generator.txt`)
```
# 역할
당신은 실사(Due Diligence) 전문가입니다.

# 작업
분석 결과를 바탕으로 데이터룸 확인 및 경영진 질의 리스트를 생성하세요.

# 질문 카테고리 (각 3-5개)
1. **시장/경쟁**
   - TAM/SAM 산정 근거
   - 경쟁 우위 검증
   - 시장 점유율 달성 가능성

2. **제품/기술**
   - 기술 차별성
   - IP/특허 현황
   - 로드맵 실행 가능성

3. **재무**
   - 매출 인식 정책
   - 비용 구조 상세
   - 재무 가정 민감도

4. **운영**
   - 코호트 분석 상세
   - CAC/LTV 계산 근거
   - 고객 집중도

5. **리스크/법무**
   - 규제 대응 계획
   - 계약 조건
   - 소송/분쟁 여부

# 입력
"""
{analysis_results}
"""

# 출력 형식
# DD Question List

## 1. Market & Competition
1. **TAM Verification**
   - Q: TAM 50조원 산정 시 사용한 리포트 원문 제공 요청
   - 확인 자료: 시장조사 보고서, 업계 통계

2. **Market Share**
   - Q: 3년 내 점유율 10% 목표의 달성 경로는?
   - 확인 자료: GTM 전략 상세, 파트너십 계약서

[...계속...]

## 2. Financial
1. **Revenue Recognition**
   - Q: 매출 인식 시점은? (인도/검수/결제)
   - 확인 자료: 회계정책, 주요 계약서

2. **Customer Concentration**
   - Q: Top 3 고객의 개별 매출 비중은?
   - 확인 자료: 고객별 매출 내역 (최근 2년)

[...총 20-30개 질문...]

# 질문 작성 원칙
- 구체적이고 답변 가능한 형태로
- Yes/No가 아닌 정량/정성 답변 유도
- 확인 필요 자료 명시
```

---

## 코드 구조

### 프로젝트 디렉토리
```
root-inside/
├── prompts/              # 프롬프트 템플릿
│   ├── base/
│   ├── labeling/
│   ├── checking/
│   └── generation/
├── src/
│   ├── prompt_builder.py    # 프롬프트 조립
│   ├── gemini_client.py     # API 호출
│   ├── response_parser.py   # 응답 파싱
│   └── analyzer.py          # 통합 분석기
├── main.py              # CLI 진입점
├── app.py               # Streamlit UI
├── .env                 # API 키
└── requirements.txt
```

### 핵심 모듈: `src/prompt_builder.py`
```python
from pathlib import Path
from typing import Dict, Any
import yaml

class PromptBuilder:
    """프롬프트 템플릿을 로드하고 사용자 입력과 결합"""
    
    def __init__(self, prompts_dir: str = "prompts"):
        self.prompts_dir = Path(prompts_dir)
        self.templates = self._load_templates()
    
    def _load_templates(self) -> Dict[str, str]:
        """모든 프롬프트 템플릿 로드"""
        templates = {}
        for prompt_file in self.prompts_dir.rglob("*.txt"):
            key = prompt_file.stem
            templates[key] = prompt_file.read_text(encoding="utf-8")
        return templates
    
    def build(
        self, 
        template_name: str, 
        user_input: str,
        context: Dict[str, Any] = None
    ) -> str:
        """
        프롬프트 템플릿에 사용자 입력을 주입
        
        Args:
            template_name: 템플릿 이름 (예: "section_labeler")
            user_input: 분석할 텍스트
            context: 추가 컨텍스트 (이전 분석 결과 등)
        
        Returns:
            완성된 프롬프트 문자열
        """
        if template_name not in self.templates:
            raise ValueError(f"템플릿 '{template_name}' 없음")
        
        template = self.templates[template_name]
        
        # 기본 치환
        prompt = template.replace("{user_text}", user_input)
        
        # 컨텍스트 추가
        if context:
            for key, value in context.items():
                prompt = prompt.replace(f"{{{key}}}", str(value))
        
        return prompt
    
    def get_system_prompt(self) -> str:
        """공통 시스템 프롬프트 반환"""
        return self.templates.get("system_prompt", "")
```

### 핵심 모듈: `src/gemini_client.py`
```python
import google.generativeai as genai
from typing import Optional, Dict, Any
import json
import os
from dotenv import load_dotenv

load_dotenv()

class GeminiClient:
    """Gemini API 래퍼"""
    
    def __init__(
        self, 
        api_key: Optional[str] = None,
        model: str = "gemini-1.5-flash"  # 무료 tier
    ):
        self.api_key = api_key or os.getenv("GEMINI_API_KEY")
        if not self.api_key:
            raise ValueError("GEMINI_API_KEY 필요")
        
        genai.configure(api_key=self.api_key)
        self.model = genai.GenerativeModel(model)
    
    def generate(
        self, 
        prompt: str,
        temperature: float = 0.1,  # 일관성 위해 낮게
        response_format: str = "json"
    ) -> Dict[str, Any]:
        """
        프롬프트를 Gemini에 전달하고 응답 파싱
        
        Args:
            prompt: 완성된 프롬프트
            temperature: 0.0~1.0 (낮을수록 일관적)
            response_format: "json" 또는 "text"
        
        Returns:
            파싱된 응답 딕셔너리
        """
        generation_config = {
            "temperature": temperature,
            "max_output_tokens": 2048,
        }
        
        # JSON 응답 강제
        if response_format == "json":
            prompt += "\n\n반드시 유효한 JSON 형식으로만 답변하세요."
        
        response = self.model.generate_content(
            prompt,
            generation_config=generation_config
        )
        
        text = response.text
        
        # JSON 파싱 시도
        if response_format == "json":
            try:
                # 마크다운 코드 블록 제거
                if "```json" in text:
                    text = text.split("```json")[1].split("```")[0]
                elif "```" in text:
                    text = text.split("```")[1].split("```")[0]
                
                return json.loads(text.strip())
            except json.JSONDecodeError as e:
                return {
                    "error": "JSON 파싱 실패",
                    "raw_response": text,
                    "exception": str(e)
                }
        
        return {"response": text}
    
    def analyze_with_retry(
        self, 
        prompt: str, 
        max_retries: int = 3
    ) -> Dict[str, Any]:
        """API 호출 재시도 로직"""
        for attempt in range(max_retries):
            try:
                return self.generate(prompt)
            except Exception as e:
                if attempt == max_retries - 1:
                    return {
                        "error": "API 호출 실패",
                        "exception": str(e)
                    }
                # 지수 백오프
                import time
                time.sleep(2 ** attempt)
```

### 핵심 모듈: `src/analyzer.py`
```python
from typing import Dict, Any, List
from .prompt_builder import PromptBuilder
from .gemini_client import GeminiClient
from .response_parser import ResponseParser

class IRAnalyzer:
    """통합 IR 분석기"""
    
    def __init__(self):
        self.prompt_builder = PromptBuilder()
        self.gemini = GeminiClient()
        self.parser = ResponseParser()
    
    def analyze_full(self, document_text: str) -> Dict[str, Any]:
        """
        전체 파이프라인 실행
        
        1. 섹션 라벨링
        2. 수치 추출
        3. 정합성 검사
        4. 레드플래그 스캔
        5. 요약 생성
        6. 질의 리스트 생성
        """
        results = {}
        
        # 1. 섹션 라벨링
        print("📋 섹션 분류 중...")
        section_prompt = self.prompt_builder.build(
            "section_labeler",
            document_text
        )
        results["sections"] = self.gemini.generate(section_prompt)
        
        # 2. 수치 추출
        print("🔢 수치 추출 중...")
        metric_prompt = self.prompt_builder.build(
            "metric_extractor",
            document_text
        )
        results["metrics"] = self.gemini.generate(metric_prompt)
        
        # 3. 정합성 검사
        print("✅ 정합성 검증 중...")
        consistency_prompt = self.prompt_builder.build(
            "consistency_checker",
            document_text,
            context={"metrics": results["metrics"]}
        )
        results["consistency"] = self.gemini.generate(consistency_prompt)
        
        # 4. 레드플래그
        print("🚩 레드플래그 스캔 중...")
        flag_prompt = self.prompt_builder.build(
            "red_flag_scanner",
            document_text
        )
        results["red_flags"] = self.gemini.generate(flag_prompt)
        
        # 5. 요약 생성
        print("📝 요약 생성 중...")
        summary_prompt = self.prompt_builder.build(
            "summary_generator",
            document_text,
            context={"analysis_results": results}
        )
        results["summary"] = self.gemini.generate(
            summary_prompt, 
            response_format="text"
        )
        
        # 6. 질의 리스트
        print("❓ 질의 리스트 생성 중...")
        question_prompt = self.prompt_builder.build(
            "question_generator",
            document_text,
            context={"analysis_results": results}
        )
        results["questions"] = self.gemini.generate(
            question_prompt,
            response_format="text"
        )
        
        return results
```

### CLI: `main.py`
```python
import argparse
import json
from pathlib import Path
from src.analyzer import IRAnalyzer

def main():
    parser = argparse.ArgumentParser(
        description="Root Inside - IR 문서 분석"
    )
    parser.add_argument(
        "command",
        choices=["label", "extract", "check", "scan", "analyze"],
        help="실행할 분석 명령"
    )
    parser.add_argument(
        "--text",
        help="직접 입력할 텍스트"
    )
    parser.add_argument(
        "--file",
        help="분석할 파일 경로"
    )
    parser.add_argument(
        "--output",
        help="결과 저장 경로",
        default="output.json"
    )
    
    args = parser.parse_args()
    
    # 입력 로드
    if args.text:
        document_text = args.text
    elif args.file:
        document_text = Path(args.file).read_text(encoding="utf-8")
    else:
        print("--text 또는 --file 필요")
        return
    
    # 분석기 초기화
    analyzer = IRAnalyzer()
    
    # 명령 실행
    if args.command == "analyze":
        results = analyzer.analyze_full(document_text)
    else:
        # 개별 분석 (label, extract, check, scan)
        method_map = {
            "label": "analyze_sections",
            "extract": "extract_metrics",
            "check": "check_consistency",
            "scan": "scan_red_flags"
        }
        method = getattr(analyzer, method_map[args.command])
        results = method(document_text)
    
    # 결과 저장
    with open(args.output, "w", encoding="utf-8") as f:
        json.dump(results, f, ensure_ascii=False, indent=2)
    
    print(f"✅ 완료! 결과: {args.output}")

if __name__ == "__main__":
    main()
```

### Streamlit UI: `app.py`
```python
import streamlit as st
from src.analyzer import IRAnalyzer
import json

st.set_page_config(
    page_title="Root Inside - IR Analyzer",
    page_icon="🔍",
    layout="wide"
)

st.title("🔍 Root Inside")
st.caption("IR 자료 분석 시스템 (Gemini 기반)")

# 사이드바
with st.sidebar:
    st.header("분석 옵션")
    
    analysis_type = st.selectbox(
        "분석 유형",
        ["전체 분석", "섹션 라벨링", "수치 추출", "레드플래그 스캔"]
    )
    
    st.divider()
    
    st.markdown("### API 설정")
    api_key_input = st.text_input(
        "Gemini API Key",
        type="password",
        help="https://makersuite.google.com/app/apikey"
    )

# 메인
tab1, tab2 = st.tabs(["📄 텍스트 입력", "📁 파일 업로드"])

with tab1:
    document_text = st.text_area(
        "분석할 문서 내용을 입력하세요",
        height=300,
        placeholder="예: 당사는 2024년 매출 50억원을 달성했으며..."
    )

with tab2:
    uploaded_file = st.file_uploader(
        "텍스트 파일 업로드",
        type=["txt", "md"]
    )
    if uploaded_file:
        document_text = uploaded_file.read().decode("utf-8")
        st.text_area("파일 내용", document_text, height=200)

# 분석 실행
if st.button("🚀 분석 시작", type="primary"):
    if not document_text:
        st.error("문서 내용을 입력하세요")
    else:
        with st.spinner("분석 중..."):
            try:
                analyzer = IRAnalyzer()
                
                if analysis_type == "전체 분석":
                    results = analyzer.analyze_full(document_text)
                    
                    # 결과 표시
                    col1, col2 = st.columns(2)
                    
                    with col1:
                        st.subheader("📊 추출된 지표")
                        st.json(results.get("metrics", {}))
                        
                        st.subheader("✅ 정합성 검증")
                        st.json(results.get("consistency", {}))
                    
                    with col2:
                        st.subheader("🚩 레드플래그")
                        flags = results.get("red_flags", {}).get("red_flags", [])
                        for flag in flags:
                            severity = flag.get("severity", "low")
                            color = {
                                "high": "🔴",
                                "medium": "🟡",
                                "low": "🟢"
                            }[severity]
                            
                            with st.expander(f"{color} {flag.get('title', 'N/A')}"):
                                st.write(flag.get('description', ''))
                                st.code(flag.get('evidence', {}).get('quote', ''))
                    
                    st.divider()
                    
                    st.subheader("📝 Executive Summary")
                    st.markdown(results.get("summary", {}).get("response", ""))
                    
                    st.subheader("❓ 질의 리스트")
                    st.markdown(results.get("questions", {}).get("response", ""))
                
                # JSON 다운로드
                st.download_button(
                    "📥 결과 다운로드 (JSON)",
                    data=json.dumps(results, ensure_ascii=False, indent=2),
                    file_name="analysis_result.json",
                    mime="application/json"
                )
                
            except Exception as e:
                st.error(f"오류 발생: {str(e)}")
```

---

## Usage Examples

### 1. CLI - 전체 분석
```bash
python main.py analyze \
  --file samples/pitch_deck.txt \
  --output report.json
```

### 2. CLI - 레드플래그만
```bash
python main.py scan --file samples/financial_statement.txt
```

### 3. Python Script
```python
from src.analyzer import IRAnalyzer

analyzer = IRAnalyzer()

document = """
당사는 국내 전자상거래 시장(TAM 50조원)을 타겟으로 하며,
2024년 매출 50억원, 영업이익률 15%를 달성했습니다.
"""

results = analyzer.analyze_full(document)
print(results["red_flags"])
```

---

## Gemini API 요금 (무료 Tier)

### Gemini 1.5 Flash (권장)
- **무료 한도**: 15 RPM (분당 15회)
- **Rate Limit**: 1,500 RPD (일당 1,500회)
- **입력**: 128K 토큰 (한글 문서 ~50페이지)
- **출력**: 8K 토큰

### 비용 절감 팁
1. **프롬프트 최적화**: 불필요한 반복 제거
2. **배치 처리**: 여러 섹션을 하나의 호출로
3. **캐싱**: 동일 문서 재분석 시 결과 재사용
4. **Temperature 낮게**: 일관성 ↑, 토큰 소비 ↓

---

## Troubleshooting

### 1. "API key not valid" 에러
```bash
# .env 파일 확인
cat .env
# GEMINI_API_KEY=...

# 또는 환경변수 직접 설정
export GEMINI_API_KEY="your_key_here"
```

### 2. JSON 파싱 실패
- Gemini가 마크다운으로 응답하는 경우 있음
- `response_parser.py`에서 자동 정리하지만, 실패 시:
```python
# 프롬프트 끝에 강조
prompt += "\n\n중요: 반드시 {로 시작하는 유효한 JSON만 출력하세요."
```

### 3. Rate Limit 초과
```python
# src/gemini_client.py에 재시도 로직 추가됨
# 실패 시 2, 4, 8초 대기 후 재시도
```

---

## Roadmap

### v0.2 (2주)
- [ ] 프롬프트 템플릿 15종 완성
- [ ] 웹 UI 개선 (차트, 하이라이팅)
- [ ] PDF 직접 업로드 지원 (OCR)

### v0.3 (4주)
- [ ] 프롬프트 버전 관리
- [ ] A/B 테스트 (프롬프트 성능 비교)
- [ ] 사용자 피드백 수집 UI

### v1.0
- [ ] 벤치마크 데이터셋 구축
- [ ] 프롬프트 자동 최적화 (DSPy)
- [ ] 멀티모달 (이미지 차트 분석)

---

## License

**Internal Use Only** - Confidential

---

**API Key 발급**: https://makersuite.google.com/app/apikey  
**Gemini API Docs**: https://ai.google.dev/docs
