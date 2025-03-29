---
title: "[Unity] 기본 렌더링 파이프라인"
author: RePro
date: 2025-03-01 10:00:00 +0900
categories: [Programming, Unity]
tags: [unity, graphics]
---

# Unity 렌더링 파이프라인(Rendering Pipeline) 종류 및 차이점

## 1. Unity의 렌더링 파이프라인 종류
Unity에서는 렌더링 파이프라인(Rendering Pipeline)을 통해 3D 그래픽을 화면에 렌더링합니다. Unity는 기본적으로 3가지 주요 렌더링 파이프라인을 제공합니다.

| 렌더링 파이프라인 | 특징 | 주 사용처 |
|-----------------|------|---------|
| **Built-in RP** (기본 렌더링 파이프라인) | 전통적인 렌더링 방식, Unity 기본 제공 | 레거시 프로젝트, 빠른 프로토타입 |
| **Universal RP (URP)** | 모바일 & 멀티플랫폼 최적화, 빠른 렌더링 | 모바일, VR/AR, 인디게임 |
| **High Definition RP (HDRP)** | 고품질 그래픽, 하이엔드 그래픽 구현 | AAA 게임, 콘솔, PC |

---

## 2. Built-in RP (기본 렌더링 파이프라인)
### 개요
- Unity의 기본 내장 렌더링 파이프라인 (전통적인 Forward & Deferred 렌더링 방식 사용)
- 설정이 간단하고, 기존 프로젝트와 호환성이 높음
- ShaderLab 기반의 표준 셰이더(Standard Shader) 사용

### 특징
- 기존 Unity 프로젝트와 완벽한 호환성
- 커스텀 셰이더 작성이 쉬움 (기본 ShaderLab 및 Surface Shader 사용)
- Forward & Deferred Rendering 모드 지원
- 조명과 그림자 처리 비용이 상대적으로 높음

### 단점
- 멀티플랫폼 최적화가 부족함 → 모바일/VR에서 성능 이슈
- 커스텀 최적화 어려움 → SRP 기반의 URP/HDRP보다 유연성이 낮음
- 대규모 프로젝트에서는 성능 최적화가 어려움

### 추천 사용처
- 기존 Unity 프로젝트 유지보수
- 빠른 프로토타이핑(Prototype)
- 간단한 2D & 3D 게임 개발

---

## 3. Universal Render Pipeline (URP, 유니버설 렌더 파이프라인)
### 개요
- 멀티플랫폼 지원 & 성능 최적화에 초점을 맞춘 렌더링 파이프라인
- 기존 LWRP(Lightweight RP)의 업그레이드 버전
- 모바일, VR, AR, 닌텐도 스위치 등 저사양 기기에서 높은 성능 제공

### 특징
- 멀티플랫폼 지원 (모바일, VR, AR, 콘솔 등)
- 렌더링 성능 최적화 → GPU 부하 감소, 경량화된 렌더링
- Single-Pass Forward Rendering 방식 사용 → 낮은 성능 비용으로 렌더링 가능
- 커스텀 Renderer Feature 사용 가능 → 후처리(Post Processing) 및 이펙트 적용 유연성

### 단점
- Built-in RP보다 일부 기능(쉐이더, 그림자 등)이 제한됨
- HDRP에 비해 그래픽 품질이 낮음
- 일부 고급 그래픽 효과(SSAO, 고급 GI 등) 미지원

### 추천 사용처
- 모바일, VR/AR, 콘솔, 인디게임
- 성능 최적화가 중요한 프로젝트
- 가벼운 2D & 3D 게임

---

## 4. High Definition Render Pipeline (HDRP, 하이 디피니션 렌더 파이프라인)
### 개요
- 고품질 그래픽을 위한 렌더링 파이프라인
- AAA급 게임, 콘솔, PC, 실시간 CG, 건축 시각화 등에 적합
- Physically Based Rendering (PBR)과 Ray Tracing 지원

### 특징
- 물리 기반 렌더링(PBR) 강화 → 사실적인 조명 및 반사 효과 제공
- 하드웨어 레이 트레이싱 지원 (NVIDIA RTX 등)
- 고급 조명 시스템 지원 (SSGI, SSR, SSAO, Volumetric Lighting 등)
- 데이터 중심 렌더링 구조 → 성능 최적화 가능

### 단점
- 모바일/저사양 기기 지원 불가
- 설정 및 개발 난이도가 높음
- 높은 하드웨어 요구사항 (PC & 콘솔 전용)

### 추천 사용처
- AAA 게임 개발 (PC, 콘솔)
- 실시간 CG & 건축 시각화
- 고품질 그래픽이 필요한 프로젝트

---

## 5. Built-in RP vs URP vs HDRP 비교 정리
| 비교 항목 | Built-in RP | URP (Universal RP) | HDRP (High Definition RP) |
|----------|------------|----------------|----------------|
| 목적 | 기본 렌더링 | 멀티플랫폼 최적화 | 고품질 그래픽 |
| 성능 | 중간 | 높음 (최적화됨) | 낮음 (고사양 필요) |
| 지원 플랫폼 | PC, 모바일, 콘솔 | 모바일, VR, 콘솔 | PC, 콘솔 (고사양) |
| [렌더링 방식] | Forward & Deferred | Forward (최적화됨) | Deferred (고급 그래픽) |
| 조명 시스템 | 기본 조명 | 경량 조명 (성능 최적화) | 물리 기반 조명 (고품질) |
| 후처리(Post Processing) | 기본 제공 | 경량화된 후처리 | 고급 후처리 지원 |
| 커스텀 셰이더 | ShaderLab 사용 | Shader Graph 사용 | Shader Graph 사용 |
| 레이 트레이싱 | 미지원 | 미지원 | 지원 (RTX) |
| 모바일 지원 | 가능 | 최적화됨 | 불가능 |

---

## 6. 어떤 렌더링 파이프라인을 선택해야 할까?
| 프로젝트 유형 | 추천 렌더링 파이프라인 |
|------------|----------------|
| 빠른 프로토타입 | Built-in RP |
| 모바일 게임 개발 | URP |
| VR/AR 개발 | URP |
| 인디게임 & 경량 3D 게임 | URP |
| AAA급 PC/콘솔 게임 | HDRP |
| 고품질 건축 시각화 & 실시간 CG | HDRP |

---

## 7. 결론
- **Built-in RP** → 기존 프로젝트 유지보수, 빠른 프로토타이핑
- **URP** → 모바일, VR/AR, 인디게임, 멀티플랫폼 최적화
- **HDRP** → AAA급 게임, 고품질 그래픽, 실시간 CG

Unity 프로젝트의 성능 및 그래픽 품질에 맞춰 적절한 렌더링 파이프라인을 선택하는 것이 중요하다.

