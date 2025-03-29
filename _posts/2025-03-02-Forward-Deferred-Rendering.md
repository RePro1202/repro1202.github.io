---
title: Forward vs. Deferred Rendering
author: RePro
date: 2025-03-02 10:00:00 +0900
categories: [Programming, Graphics]
tags: [graphics]
---

# Forward vs. Deferred Rendering

## 1. 개요
Forward Rendering과 Deferred Rendering은 실시간 3D 그래픽스에서 조명과 렌더링을 처리하는 대표적인 방식이다. 이 개념은 Unity뿐만 아니라, Unreal Engine, CryEngine, Frostbite 등의 게임 엔진 및 DirectX, OpenGL, Vulkan 같은 그래픽 API에서도 사용된다.

---

## 2. Forward Rendering (순차 렌더링)
### 개요
- 오브젝트별로 조명을 계산하면서 즉시 렌더링하는 방식
- 객체가 그려질 때 해당 픽셀에 대한 조명을 계산하여 최종 색상을 결정
- 단순한 구조로 구현이 쉬우며, 모바일 및 VR과 같은 환경에서 적합

### 특징
- 모든 픽셀에 대해 조명을 한 번씩 계산
- 멀티패스 렌더링을 지원하여 다양한 조명 효과 적용 가능
- 반투명(Transparent) 오브젝트 처리 용이
- VR 및 모바일 환경에서 적합

### 단점
- 조명이 많아질수록 성능 저하 (각 조명마다 추가적인 드로우 콜 발생)
- 복잡한 조명 효과(SSAO, GI 등) 처리 어려움
- 다수의 그림자를 지원하기 어려움

### Forward Rendering 과정
1. 각 오브젝트에 대해 조명을 계산하며 드로우 콜 발생
2. 픽셀마다 최종 색상을 계산하고 출력
3. 모든 오브젝트가 렌더링되면 화면에 출력

### Forward Rendering 추천 사용처
- 모바일 및 VR 환경 (낮은 연산 비용 요구)
- 조명이 적은 장면 (성능 최적화)
- 반투명 오브젝트가 많은 장면

---

## 3. Deferred Rendering (지연 렌더링)
### 개요
- 모든 객체의 정보를 먼저 렌더링한 후, 조명을 한 번에 계산하는 방식
- GPU를 활용하여 조명 연산을 최적화하며, 다중 조명 환경에서 성능을 극대화
- 고급 그래픽 기능(SSAO, Screen Space Reflections, GI 등) 지원

### 특징
- 조명 계산을 한 번만 수행하여 성능 최적화 가능
- 다중 조명을 효율적으로 처리 가능
- Screen Space 기반 후처리 효과 적용 용이 (SSAO, SSR 등)
- 고급 조명 시스템 지원

### 단점
- 반투명(Transparent) 오브젝트 처리 어려움 (Forward Pass 필요)
- VR, 모바일에서 성능 이슈 발생 가능
- G-Buffer를 사용하여 메모리 사용량 증가

### Deferred Rendering 과정
1. G-Buffer(Geometry Buffer)에 오브젝트 정보를 저장 (색상, 노멀, 깊이 등)
2. G-Buffer를 기반으로 조명 연산 수행
3. 최종 조명 효과를 적용하여 화면에 출력

### Deferred Rendering 추천 사용처
- AAA 게임 및 고품질 그래픽이 필요한 프로젝트
- 다중 조명이 필요한 장면 (도시, 실내 환경 등)
- Screen Space 기반 효과(SSAO, SSR, GI) 활용

---

## 4. Forward Rendering vs. Deferred Rendering 비교
| 비교 항목 | Forward Rendering | Deferred Rendering |
|----------|-----------------|-------------------|
| 렌더링 방식 | 객체별로 조명 계산 | G-Buffer를 사용하여 한 번에 조명 계산 |
| 성능 | 조명이 많아질수록 성능 저하 | 조명 수가 많아도 성능 유지 가능 |
| 메모리 사용량 | 적음 | 많음 (G-Buffer 저장 필요) |
| 조명 처리 방식 | 광원마다 추가적인 패스 실행 | 한번에 조명 계산 가능 (최적화) |
| 반투명 오브젝트 | 쉽게 처리 가능 | 처리 어려움 (Forward Pass 필요) |
| 후처리(Post Processing) | 제한적 | Screen Space 기반 후처리 적용 가능 |
| 적합한 환경 | 모바일, VR, 간단한 씬 | AAA 게임, 복잡한 씬, 다중 조명 환경 |

---

## 5. Forward+ Rendering (하이브리드 방식)
### 개요
- Forward와 Deferred의 장점을 결합한 방식
- Forward 방식의 반투명 오브젝트 처리 장점 + Deferred 방식의 다중 조명 최적화
- Unreal Engine, DirectX, CryEngine 등에서 사용됨

### 특징
- 다중 조명 처리 성능 향상 (GPU 성능 활용)
- 반투명 오브젝트 지원 가능
- Forward의 간결함과 Deferred의 성능을 결합

### 단점
- 구현이 복잡하며, 추가적인 셰이더 작업 필요
- 일부 하드웨어에서 성능 최적화 필요

---

## 6. Forward vs. Deferred 선택 기준
| 선택 기준 | 추천 렌더링 방식 |
|----------|---------------|
| 모바일 & VR 게임 | Forward Rendering |
| PC & 콘솔 게임 (고품질 그래픽) | Deferred Rendering |
| 조명이 적은 게임 (성능 중요) | Forward Rendering |
| 다중 조명이 많은 게임 (도시, 실내 환경 등) | Deferred Rendering |
| 반투명 오브젝트가 많은 씬 | Forward Rendering |
| Screen Space 후처리 효과 (SSAO, SSR 등) | Deferred Rendering |

---

## 7. 결론
- **Forward Rendering** → 조명이 적고, 모바일 & VR에서 성능 최적화가 중요한 경우
- **Deferred Rendering** → 다중 조명이 많고, 고품질 그래픽이 필요한 경우
- **Forward+ Rendering** → Forward와 Deferred의 장점을 결합한 최신 기법

이러한 렌더링 기법은 Unity뿐만 아니라 Unreal Engine, CryEngine, DirectX, OpenGL, Vulkan 등 다양한 그래픽스 엔진과 API에서도 적용되는 보편적인 개념이다. 각 프로젝트의 성능 및 그래픽 요구사항에 따라 적절한 렌더링 방식을 선택하는 것이 중요하다.

