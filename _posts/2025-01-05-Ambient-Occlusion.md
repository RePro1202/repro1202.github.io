---
title: Ambient Occlusion (AO)
author: RePro
date: 2025-01-05 10:00:00 +0900
categories: [Programming, Graphics]
tags: [graphics]
---

# Ambient Occlusion (AO) 개요

## 1. Ambient Occlusion이란?
Ambient Occlusion(AO)은 빛이 도달하기 어려운 공간을 어둡게 표현하여 더욱 현실적인 조명 효과를 구현하는 기술이다. 즉, 빛이 직접 닿지 않는 모서리, 틈새, 접합부, 물체 간 좁은 공간을 어둡게 만들어 입체감을 강조하는 기술이다.

이 기술은 전역 조명(Global Illumination, GI) 기법의 일부로, 광원의 직접적인 영향 없이 환경에 따른 그림자를 표현하는 데 사용된다.

---

## 2. 왜 Ambient Occlusion이 필요한가?
- 일반적인 조명 모델은 직접 광원에서 오는 빛만 계산하므로, 물체 사이의 그림자 효과가 부족할 수 있음.
- AO를 적용하면 주변 환경에 의해 발생하는 미세한 그림자를 추가하여 더욱 현실적인 조명 효과를 구현할 수 있음.
- 광원과 무관하게 물체 자체의 구조적 특징을 반영하여 그림자를 표현할 수 있음.

예제: AO가 없는 경우
- 물체가 공중에 떠 있는 듯한 부자연스러운 조명 표현
- 벽과 바닥, 또는 오브젝트 간의 접촉 부분이 밝게 표현되어 비현실적으로 보일 수 있음

예제: AO가 적용된 경우
- 모서리, 틈새, 접합부 등에 미세한 그림자가 생겨 입체감이 증가
- 더욱 자연스러운 공간감과 사실적인 조명 표현 가능

---

## 3. Ambient Occlusion의 원리
AO는 표면 주변의 기하학적 구조를 분석하여, 주변 물체에 의해 빛이 차단될 확률을 계산한다.

1. 각 표면의 특정 반경 내에 있는 주변 지형을 분석
2. 해당 반경 내에서 빛이 차단되는 비율을 계산
3. 빛이 차단되는 정도에 따라 그림자의 강도를 결정
4. 조명과 무관하게 어두운 영역을 추가하여 더욱 입체적인 느낌을 표현

이 과정을 통해 AO는 실시간 렌더링에서 추가적인 현실감을 제공할 수 있다.

---

## 4. Ambient Occlusion의 주요 기법
AO는 다양한 방식으로 구현될 수 있으며, 사용 목적과 성능 요구 사항에 따라 여러 기법이 존재한다.

| 기법 | 설명 | 장점 | 단점 |
|------|------|------|------|
| **Screen Space Ambient Occlusion (SSAO)** | 화면 기반으로 AO를 계산하는 실시간 기술 | 빠른 연산, 실시간 적용 가능 | 정확도가 낮음, 먼 거리 표현 어려움 |
| **Scalable Ambient Obscurance (SAO)** | SSAO의 개선된 버전, 더 넓은 영역에서 AO를 계산 | SSAO보다 정밀한 표현 가능 | SSAO보다 연산 비용 증가 |
| **Horizon-Based Ambient Occlusion (HBAO)** | 시각적인 품질을 향상한 SSAO 방식 | 고품질 AO 표현 | 높은 연산 비용 |
| **Voxel-Based Ambient Occlusion (VXAO)** | 볼륨 데이터를 활용하여 AO를 계산 | 정확하고 자연스러운 AO 표현 | 매우 높은 연산 비용 |
| **Ray-Traced Ambient Occlusion (RTAO)** | 레이 트레이싱을 활용한 고품질 AO | 물리적으로 정확한 AO | 고사양 GPU 필요 |

---

## 5. Unity와 Unreal Engine에서 Ambient Occlusion

### (1) **Unity에서 Ambient Occlusion 적용 방법**
#### URP (Universal Render Pipeline)
- Post Processing Volume을 추가하여 SSAO 활성화 가능

설정 방법:
1. Post Processing Volume을 추가
2. Ambient Occlusion 효과 추가 및 활성화
3. Intensity, Radius 등의 값을 조절하여 강도 조정

#### HDRP (High Definition Render Pipeline)
- HDRP에서는 더욱 정밀한 AO 적용 가능
- Ray-Traced Ambient Occlusion(RTAO) 지원

설정 방법:
1. Global Volume을 추가
2. Ambient Occlusion Override 추가
3. Sample Count, Intensity 조절하여 품질 조정

[유니티 문서](https://docs.unity3d.com/kr/Packages/com.unity.render-pipelines.universal@14.0/manual/post-processing-ssao.html)

---

### (2) **Unreal Engine에서 Ambient Occlusion 적용 방법**
#### SSAO (Screen Space Ambient Occlusion)
- Unreal Engine의 기본 AO 기법으로 실시간 SSAO 적용 가능

설정 방법:
1. **Post Process Volume** 추가
2. **Ambient Occlusion Intensity** 조절
3. **Radius, Quality** 조정하여 AO 효과 세밀하게 조절

#### Distance Field Ambient Occlusion (DFAO)
- 거리 필드 데이터를 활용하여 SSAO보다 더 정교한 AO를 계산하는 방식
- Static Mesh에 최적화된 AO 방식으로, 동적 오브젝트에는 제한적

설정 방법:
1. **Project Settings → Rendering → Generate Mesh Distance Fields 활성화**
2. **Post Process Volume에서 Ambient Occlusion Distance 설정**
3. **AO Quality 및 Radius 조정**

#### Ray-Traced Ambient Occlusion (RTAO)
- 하드웨어 기반 Ray Tracing을 활용한 AO 기법
- RTX GPU에서 하드웨어 가속을 통해 실행 가능
- 현실적인 AO 효과를 제공하지만 높은 연산 비용이 필요함

설정 방법:
1. **Project Settings → Ray Tracing 활성화**
2. **Post Process Volume에서 Ambient Occlusion Method를 Ray Traced로 변경**
3. **Sample Count 및 Intensity 조정**

---

## 6. Ambient Occlusion과 Global Illumination(GI) 차이점
AO와 GI는 모두 빛의 확산과 그림자를 표현하는 기술이지만, 근본적인 차이가 있다.

| 비교 항목 | Ambient Occlusion (AO) | Global Illumination (GI) |
|----------|-----------------|-----------------|
| **역할** | 오브젝트 간 그림자 표현 | 빛의 확산과 반사 표현 |
| **광원 영향** | 직접적인 광원 없이 그림자 표현 | 광원의 영향을 받음 |
| **계산 방식** | 환경 구조 기반 그림자 생성 | 광원과 표면 간의 상호작용을 계산 |
| **적용 방식** | Post Processing 또는 쉐이더 기반 | 광원, 재질, GI 시스템 활용 |

---

## 7. Ambient Occlusion의 한계
- 빛의 방향을 고려하지 않음: AO는 주변 환경만을 고려하므로 직접 광원과의 관계를 반영하지 않음
- 정확도가 제한적: SSAO와 같은 기법은 먼 거리의 AO를 정확하게 표현하지 못할 수 있음
- 성능 부담: 고품질 AO 기법(HBAO, VXAO, RTAO)은 연산 비용이 높아 성능 저하 가능성이 있음

---

## 8. 결론
- Ambient Occlusion(AO)은 환경에 의한 그림자 효과를 추가하여 사실적인 입체감을 구현하는 기술
- 다양한 AO 기법(SSAO, HBAO, VXAO, RTAO)이 있으며, 성능과 품질에 따라 선택 가능
- Unity와 Unreal Engine에서 각각 SSAO, DFAO, RTAO 등을 활용하여 AO 효과를 적용 가능
- GI(Global Illumination)와 함께 사용하면 더욱 사실적인 조명 효과 구현 가능

AO는 실시간 그래픽스 및 게임에서 사실적인 조명 효과를 표현하는 핵심 기술 중 하나이며, 성능과 품질 요구 사항에 맞춰 적절한 기법을 선택하여 활용하는 것이 중요하다.

