# 🚀 Day 2: Vision AI & Generative AI Fundamentals (Amazon Bedrock)

2일차 과정에서는 AWS의 핵심 AI 서비스인 Amazon Rekognition을 활용한 비전 분석 기법과 Amazon Bedrock을 통한 생성형 AI 기초, 프롬프트 엔지니어링 및 멀티턴 FAQ 챗봇 구현 방법을 학습했습니다.

---

## 📌 학습 목차
1. [Amazon Rekognition: 컴퓨터 비전 및 이미지 라벨링](#1-amazon-rekognition-컴퓨터-비전-및-이미지-라벨링)
2. [Amazon Bedrock: 파운데이션 모델(FM) 및 통합 API](#2-amazon-bedrock-파운데이션-모델fm-및-통합-api)
3. [프롬프트 엔지니어링 핵심 기법 & 업무 자동화](#3-프롬프트-엔지니어링-핵심-기법--업무-자동화)
4. [Amazon Bedrock 기반 학교 FAQ 챗봇 구현](#4-amazon-bedrock-기반-학교-faq-챗봇-구현)
5. [💡 인프라 실무 지식: 멀티모달 & 클라우드 SLA](#5--인프라-실무-지식-멀티모달--클라우드-sla)

---

## 1. Amazon Rekognition: 컴퓨터 비전 및 이미지 라벨링
어제 직접 텍스트 데이터셋을 가공하여 머신러닝 모델을 빌드했다면, 오늘은 완전 관리형 컴퓨터 비전 API인 Amazon Rekognition을 통해 복잡한 커스텀 모델 빌드 없이 이미지 분석을 수행했습니다.
* **DetectFaces API:** 이미지 내 안면을 인식하고 성별, 감정 상태, 눈 개폐 여부 등 구조화된 메타데이터를 추출합니다.
* **DetectText API:** 이미지 속에 포함된 텍스트(영문/숫자 중심)의 위치 정보와 문자열을 바운딩 박스(Bounding Box, 경계 박스 좌표) 형태로 파싱합니다.

---

## 2. Amazon Bedrock: 파운데이션 모델(FM) 및 통합 API

### [2-1] Bedrock의 정의와 필요성
* Amazon Bedrock은 다양한 글로벌 인공지능 기업(Anthropic, AI21, Meta 등)의 고성능 파운데이션 모델을 하나의 단일 **엔드포인트(Endpoint)**로 안전하게 호출하고 활용할 수 있는 완전 관리형 서비스입니다.

### [2-2] Boto3 핵심 API 및 PoC 전략
* `list_foundation_models`: 리전 내에서 사용 가능한 파운데이션 모델 목록과 속성 조사.
* `invoke_model`: 단발성(Single-turn)으로 프롬프트를 전달하고 Payload를 응답받는 기본 API.
* `converse`: 모델별로 상이한 페이로드 구조를 표준화(추상화)하여 멀티턴 대화를 쉽게 구현할 수 있도록 지원하는 **통합 대화 API**.

> 💡 **실무 PoC(Proof of Concept, 개념 검증) 팁:**
> AI 애플리케이션 개발 시 비용·속도·품질의 Trade-off를 고려해야 합니다. 일반적으로는 가장 빠르고 저렴한 대량 처리용 모델(예: Claude Haiku)로 기능 구현 가능 여부(PoC)를 검증한 후, 복잡한 추론이나 높은 정확도가 요구될 때 상위 모델(예: Claude Sonnet)로 전환하거나 프롬프트를 개선하는 것이 FinOps(비용 최적화) 관점에서 유리합니다.

---

## 3. 프롬프트 엔지니어링 핵심 기법 & 업무 자동화

추가적인 모델 파인튜닝(학습) 없이, 프롬프트 최적화만으로 시스템 내부 DB 및 타 API와 연동 가능한 고도화된 업무 자동화를 설계할 수 있습니다.

| 기법 명칭 | 작동 원리 및 특징 | 실무 활용 시나리오 |
| :--- | :--- | :--- |
| **Zero-shot** | 아무런 예시를 주지 않고 지시문만으로 태스크 수행 | 간단하고 명확한 텍스트 분류, 요약 |
| **Few-shot** | 2~5개의 가이드용 예시(Input-Output 쌍)를 포함하여 전달 | 특정 스타일의 안내문 작성, 일관된 포맷팅 |
| **Chain-of-Thought** | "단계별로 차근차근 생각해봐"라는 논리 전개 유도 | 복잡한 계산식 도출, 다단계 자격 심사 로직 |
| **System Prompt** | 대화가 시작되기 전 AI의 역할(Persona)과 제약사항 선언 | 상담원 페르소나 주입, 욕설/비속어 필터링 |
| **Structured Output** | 응답 포맷을 특정 규격(**JSON**)으로 출력하도록 강제 | 고객 문의 $\rightarrow$ `{유형, 긴급도, 담당부서}` 형태의 DB 적재 자동화 |

---

## 4. Amazon Bedrock 기반 학교 FAQ 챗봇 구현

### [4-1] 챗봇 논리 아키텍처 (단순형 RAG 구조)
실무 백엔드 시스템은 `사용자 입력 $\rightarrow$ API Gateway $\rightarrow$ AWS Lambda (검색 로직 + Bedrock 호출) $\rightarrow$ 스트리밍 응답` 골격으로 빌드됩니다. 2일차 실습에서는 이 아키텍처의 핵심 백엔드 클래스(Class) 소스코드를 직접 완성했습니다.