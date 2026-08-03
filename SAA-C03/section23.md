# Section 23. 머신 러닝

## 목차

1. [Amazon Rekognition](#1-amazon-rekognition)
2. [Amazon Transcribe](#2-amazon-transcribe)
3. [Amazon Polly](#3-amazon-polly)
4. [Amazon Translate](#4-amazon-translate)
5. [Amazon Lex & Connect](#5-amazon-lex--connect)
6. [Amazon Comprehend](#6-amazon-comprehend)
7. [Amazon SageMaker](#7-amazon-sagemaker)
8. [Amazon Forecast](#8-amazon-forecast)
9. [Amazon Kendra](#9-amazon-kendra)
10. [Amazon Personalize](#10-amazon-personalize)
11. [Amazon Textract](#11-amazon-textract)
12. [ML 서비스 요약](#12-ml-서비스-요약)

---

## 1. Amazon Rekognition

### 1.1 개요

- ML을 사용하여 이미지와 영상에서 **객체(objects), 사람(people), 텍스트(text), 장면(scenes)** 감지
- **얼굴 분석(Facial analysis)** 및 **얼굴 검색(Facial search)**: 사용자 인증, 인원 집계
- "친숙한 얼굴" 데이터베이스 생성 가능

### 1.2 주요 사용 사례

- **라벨링(Labeling)**: 이미지/영상 내 객체 태깅
- **콘텐츠 모더레이션(Content Moderation)**: 부적절한 콘텐츠 감지
- **텍스트 감지(Text Detection)**: 이미지 내 텍스트 인식
- **얼굴 감지/분석/검색/인증(Face Detection/Analysis/Search/Verification)**
- **유명인 인식(Celebrity Recognition)**
- **패스 분석(Pathing)**: 스포츠 경기 분석

### 1.3 콘텐츠 모더레이션 (Content Moderation)

- 부적절하거나 공격적이거나 원치 않는 콘텐츠 감지
- 활용 분야: 소셜 미디어, 방송, 광고, 이커머스
- **최소 신뢰 임계값(Minimum Confidence Threshold)** 설정 가능
- 임계값 미달 콘텐츠는 **Amazon Augmented AI (A2I)** 를 통해 사람 검토(Human Review)로 전달

> 💡 **시험 포인트**:
> - Rekognition = 이미지/영상 ML 분석 서비스 (텍스트 추출 아님, Textract와 구분)
> - 얼굴 인식 + 유명인 인식 + 콘텐츠 모더레이션 모두 Rekognition
> - 콘텐츠 모더레이션의 사람 검토 워크플로는 **Amazon A2I** 와 연동
> - 스포츠 경기 선수 동선 분석(Pathing) = Rekognition

---

## 2. Amazon Transcribe

### 2.1 개요

- **음성을 텍스트로 변환(Speech to Text)**: **ASR(Automatic Speech Recognition)** 딥러닝 기반
- 자동으로 **PII(Personally Identifiable Information, 개인 식별 정보) 제거**: Redaction 기능
- **자동 언어 식별(Automatic Language Identification)**: 다중 언어 오디오 지원

### 2.2 주요 사용 사례

- 고객 서비스 통화 자동 전사
- **자막(Closed Captions)** 생성
- 미디어 자산에 메타데이터 추가

> 💡 **시험 포인트**:
> - Transcribe = 음성 → 텍스트 (STT, Speech-to-Text)
> - PII 자동 제거(Redaction) 기능 내장
> - Polly와 반대 방향임을 명심 (Polly = 텍스트 → 음성)
> - 다국어 오디오 처리 시 Automatic Language Identification 활용

---

## 3. Amazon Polly

### 3.1 개요

- 딥러닝을 사용하여 **텍스트를 음성으로 변환(Text to Speech)**
- "말하는(talking)" 애플리케이션 구현 가능

### 3.2 주요 기능

- **Lexicon**: 단어의 발음을 커스터마이즈 (예: 약어, 브랜드 이름, 신조어)
- **SSML(Speech Synthesis Markup Language)**: 발음 세밀 제어
  - 숨소리(Breathing), 속삭임(Whispering), 뉴스 앵커 스타일(Newscaster Speaking Style) 등

> 💡 **시험 포인트**:
> - Polly = 텍스트 → 음성 (TTS, Text-to-Speech)
> - Transcribe와 반대 방향 관계
> - Lexicon = 특정 단어 발음 커스터마이즈
> - SSML = 더 세밀한 음성 표현 제어 (마크업 언어)

---

## 4. Amazon Translate

### 4.1 개요

- **자연스럽고 정확한 언어 번역(Language Translation)** 서비스
- 국제 사용자를 위한 콘텐츠 현지화(Localization)
- 대용량 텍스트의 효율적인 번역 처리

> 💡 **시험 포인트**:
> - Translate = 다국어 번역, 콘텐츠 현지화
> - 대용량 배치 번역에도 사용 가능
> - Comprehend와 혼동 주의 (Comprehend는 언어 분석, Translate는 번역)

---

## 5. Amazon Lex & Connect

### 5.1 Amazon Lex

- **Alexa를 구동하는 기술** 기반
- **ASR**: 음성을 텍스트로 변환
- **NLU(Natural Language Understanding)**: 의도(Intent) 인식
- **챗봇(Chatbot)**, **콜센터 봇(Call Center Bot)** 구축

### 5.2 Amazon Connect

- 전화 수신, 연락 흐름(Contact Flow) 생성, 클라우드 기반 **가상 콜센터(Virtual Contact Center)**
- **선불 비용 없음**, 기존 콜센터 대비 **80% 저렴**
- **CRM 시스템** 및 기타 AWS 서비스와 통합

### 5.3 Lex + Connect 통합 흐름

```
전화 통화 → Amazon Connect → Amazon Lex (의도 식별)
         → AWS Lambda (작업 실행) → CRM 시스템
```

> 💡 **시험 포인트**:
> - Lex = 챗봇 엔진 (Alexa 기반), ASR + NLU 조합
> - Connect = 가상 콜센터, 전화 수신 + 연락 흐름
> - Lex + Connect + Lambda 조합 = 완전한 자동화 콜센터 아키텍처
> - Connect는 기존 솔루션 대비 80% 비용 절감

---

## 6. Amazon Comprehend

### 6.1 개요

- **NLP(Natural Language Processing, 자연어 처리)** 서비스
- 완전 관리형(Fully Managed), 서버리스(Serverless)
- ML을 사용하여 텍스트에서 인사이트 및 관계 추출

### 6.2 주요 기능

| 기능 | 설명 |
|---|---|
| 언어 감지 | 텍스트 언어 자동 식별 |
| 핵심 구문(Key Phrases) | 중요 구문 추출 |
| 엔티티(Entities) | 사람, 장소, 날짜 등 개체 인식 |
| 감성 분석(Sentiment) | 긍정/부정/중립/혼합 분류 |
| 구문 분석(Syntax) | 품사 태깅 |
| 주제 모델링(Topic Modeling) | 문서 주제 그룹화 |

### 6.3 Amazon Comprehend Medical

- 의료 메모(Medical Notes)에서 **PHI(Protected Health Information, 보호 건강 정보)** 감지

> 💡 **시험 포인트**:
> - Comprehend = NLP, 텍스트 분석 (번역 아님)
> - 감성 분석(Positive/Negative/Neutral/Mixed) 4가지 분류
> - Comprehend Medical = PHI 감지 전용 의료 특화 버전
> - 서버리스, 완전 관리형 서비스

---

## 7. Amazon SageMaker

### 7.1 개요

- ML 모델 **구축(Build), 훈련(Train), 배포(Deploy)** 를 위한 완전 관리형 서비스
- ML 각 단계의 **무거운 작업(Heavy Lifting)** 자동화

### 7.2 ML 워크플로우

```
데이터 레이블링 → 모델 구축 → 훈련/튜닝 → 배포 → 모니터링
```

### 7.3 주요 사용 사례

- 커스텀 ML 요구사항이 있는 경우
- 데이터 사이언티스트가 직접 커스텀 모델을 구축/훈련하는 경우

> 💡 **시험 포인트**:
> - SageMaker = 커스텀 ML 모델 전체 라이프사이클 관리
> - 다른 ML 서비스(Rekognition, Comprehend 등)는 특정 기능 특화 → SageMaker는 범용
> - CloudWatch Application Insights가 SageMaker 기반으로 동작
> - "데이터 사이언티스트", "커스텀 모델" 키워드 → SageMaker

---

## 8. Amazon Forecast

### 8.1 개요

- **정확한 예측(Accurate Forecasts)** 을 제공하는 완전 관리형 ML 서비스
- 인간 분석 대비 **50% 더 정확**
- AWS가 ML 인프라 전체 관리

### 8.2 주요 사용 사례

- **제품 수요 계획(Product Demand Planning)**
- **재무 계획(Financial Planning)**
- **리소스 계획(Resource Planning)**
- 예측 시간을 **수개월 → 수 시간**으로 단축

> 💡 **시험 포인트**:
> - Forecast = 시계열 예측, 수요/재무/리소스 계획
> - 인간 분석 대비 50% 더 정확
> - 예측 시간 단축 (months → hours)
> - ML 인프라 관리 불필요

---

## 9. Amazon Kendra

### 9.1 개요

- ML 기반 완전 관리형 **문서 검색 서비스(Document Search Service)**
- 다양한 문서에서 답변 추출: 텍스트, PDF, HTML, PowerPoint, Word, FAQ

### 9.2 주요 기능

- **자연어 검색(Natural Language Search)** 지원
- **증분 학습(Incremental Learning)**: 사용자 인터랙션/피드백으로 선호 결과 자동 학습
- 검색 결과 수동 **파인튜닝(Fine-tune)** 가능

> 💡 **시험 포인트**:
> - Kendra = 엔터프라이즈 문서 검색, 자연어 질의 응답
> - Elasticsearch/OpenSearch와 구분: Kendra는 ML 기반 의미 검색
> - 사용자 피드백 기반 Incremental Learning 자동 적용
> - 다양한 문서 형식(PDF, PPT, Word 등) 지원

---

## 10. Amazon Personalize

### 10.1 개요

- **실시간 개인화 추천(Real-time Personalized Recommendations)** 앱 구축을 위한 완전 관리형 ML 서비스
- **amazon.com** 이 실제로 사용하는 동일한 기술
- ML 경험 없이도 사용 가능

### 10.2 주요 특징

- 통합 시간: **수개월이 아닌 수일(Days, not Months)**
- 사용 사례: 개인화 상품 추천, **재순위화(Re-ranking)**, 맞춤형 다이렉트 마케팅

> 💡 **시험 포인트**:
> - Personalize = 개인화 추천 엔진, amazon.com 동일 기술
> - ML 경험 불필요, 빠른 통합 (days)
> - 재순위화(Re-ranking) + 맞춤 마케팅도 지원
> - Kendra(문서 검색)와 혼동 주의

---

## 11. Amazon Textract

### 11.1 개요

- 스캔된 문서에서 **텍스트 및 데이터 자동 추출(Text and Data Extraction)**
- AI 및 ML 기반
- PDF, 이미지에서 텍스트, 필기체(Handwriting), 양식(Forms), 표(Tables) 추출

### 11.2 주요 사용 사례

| 분야 | 활용 예시 |
|---|---|
| 금융 서비스 | 인보이스, 영수증 처리 |
| 의료 | 의료 기록, 보험 청구 |
| 공공 부문 | 세금 양식, 신분증, 여권 |

> 💡 **시험 포인트**:
> - Textract = 스캔 문서에서 텍스트/데이터 추출 (OCR 고급 버전)
> - Rekognition과 구분: Rekognition은 이미지 분석, Textract는 문서 텍스트 추출
> - 필기체(Handwriting) + 표(Tables) + 양식(Forms) 구조적 추출 가능
> - 금융/의료/공공 분야 문서 처리에 적합

---

## 12. ML 서비스 요약

| 서비스 | 기능 |
|---|---|
| **Rekognition** | 이미지/영상 분석, 얼굴 인식 |
| **Transcribe** | 음성 → 텍스트 (ASR) |
| **Polly** | 텍스트 → 음성 (TTS) |
| **Translate** | 언어 번역 |
| **Lex** | 챗봇 (Alexa 기반, ASR + NLU) |
| **Connect** | 가상 콜센터 |
| **Comprehend** | NLP, 감성 분석 |
| **Comprehend Medical** | PHI 감지 (의료 특화) |
| **SageMaker** | 커스텀 ML 모델 구축/훈련/배포 |
| **Forecast** | 시계열 수요 예측 |
| **Kendra** | ML 기반 문서 검색 |
| **Personalize** | 실시간 개인화 추천 |
| **Textract** | 스캔 문서 텍스트/데이터 추출 |

> 💡 **시험 포인트**:
> - Transcribe(음성→텍스트) ↔ Polly(텍스트→음성) 방향 구분
> - Rekognition(이미지 분석) ↔ Textract(문서 텍스트 추출) 역할 구분
> - Comprehend(NLP 분석) ↔ Translate(번역) 혼동 주의
> - SageMaker = 커스텀 ML 전용, 나머지는 특정 기능 특화 관리형 서비스

---

← [Section 22. 데이터 & 분석](section22.md) | [Section 24. AWS 모니터링 및 감사: CloudWatch, CloudTrail 및 Config](section24.md) →
