---
title: Tonemapping (톤 매핑)
author: RePro
date: 2025-03-03 10:00:00 +0900
categories: [Programming, Graphics]
tags: [graphics, unity]
---

# Tonemapping (톤 매핑)

## 1. 개요
Tone Mapping(톤 매핑)은 HDR(High Dynamic Range) 이미지를 LDR(Low Dynamic Range)로 변환하는 과정이다. 즉, 밝기 범위가 넓은 HDR 이미지를 인간이 인식 가능한 범위(LDR)로 조정하는 기술이다.

Unity의 Post Processing Stack에서는 톤 매핑 기능을 제공하며, HDRP에서는 더욱 정교한 톤 매핑이 가능하다.

---

## 2. 왜 Tonemapping이 필요한가?
디지털 화면은 HDR(High Dynamic Range, 높은 밝기 범위) 데이터를 직접 표시할 수 없다. 따라서 HDR 데이터를 SDR(8-bit LDR) 디스플레이에서도 표현할 수 있도록 변환하는 과정이 필요하다.

예제: 톤 매핑이 없을 경우
- 너무 밝은 부분이 하얗게 날아감 (Clipping)
- 너무 어두운 부분이 완전히 검게 보임 (Crushed Blacks)
- HDR 정보가 LDR로 변환될 때 디테일이 손실됨

예제: 톤 매핑 적용 후
- 밝기 범위를 조정하여 자연스러운 색감 유지
- 밝고 어두운 영역을 부드럽게 표현
- 물리 기반 렌더링(PBR)과 조화를 이루어 사실적인 결과 제공

---

## 3. Unity에서 Tonemapping 적용 방법
### (1) URP (Universal Render Pipeline)
- URP에서는 Post Processing 효과 중 Color Grading에서 Tonemapping을 활성화할 수 있다.

설정 방법 (URP)
1. Post Process Volume을 추가
2. Color Grading을 활성화
3. Tonemapping 항목에서 원하는 톤 매핑 알고리즘 선택

### (2) HDRP (High Definition Render Pipeline)
- HDRP에서는 Volume Component에서 Tonemapping을 추가하여 적용 가능
- 더 정밀한 톤 매핑 제어 가능

설정 방법 (HDRP)
1. Volume 오브젝트 추가 (Global Volume 사용 가능)
2. Tonemapping 효과를 Volume Component에 추가
3. Mode에서 원하는 톤 매핑 모드 선택

---

## 4. Unity에서 지원하는 Tonemapping 모드
Unity에서는 여러 톤 매핑 알고리즘을 제공하며, 각각의 방식이 이미지를 변환하는 방식이 다르다.

| 톤 매핑 모드 | 설명 | 특징 |
|------------|-----------------|----------------------|
| None | 톤 매핑 없음 (원본 HDR 유지) | 밝은 부분이 쉽게 날아감 |
| Neutral | 기본적인 톤 매핑 적용 | 색상 왜곡 없음, 대비 보정 |
| ACES (Academy Color Encoding System) | 영화 산업에서 사용되는 표준 톤 매핑 | 사실적이고 자연스러운 조명 표현 |
| Filmic | 필름 느낌의 색감 제공 | 부드러운 명암 대비 적용 |
| Reinhard | 간단한 톤 매핑 기법 | 빠르고 가벼운 계산 방식 |
| Custom | 사용자 지정 톤 매핑 | 셰이더를 직접 구현 가능 |

---

## 5. 주요 톤 매핑 모드 비교
| 모드 | 밝은 영역 표현 | 색상 보정 | 특징 |
|------|-------------|---------|-------|
| None | 과다 노출 (Clipping) | 없음 | HDR 그대로 출력 |
| Neutral | 대비 보정 | 색상 왜곡 없음 | 기본적인 톤 매핑 |
| ACES | 자연스럽게 조정 | 사실적인 색감 | 영화와 같은 느낌 |
| Filmic | 부드럽게 조정 | 따뜻한 색감 | 영화 같은 필름 룩 |
| Reinhard | 밝기 자동 조절 | 색상 왜곡 있음 | 가볍고 빠른 방식 |

---

## 6. Tonemapping과 Gamma vs Linear Rendering
- Unity에서는 Gamma Space와 Linear Space를 지원한다.
- Linear Rendering(선형 공간)을 사용할 경우, 톤 매핑이 더욱 자연스럽게 적용된다.

Gamma vs Linear 선택 기준
- 모바일 / 저사양 프로젝트 → Gamma Space
- 고품질 그래픽 (HDRP / PBR 활용) → Linear Space + Tonemapping 적용

---

## 7. Tonemapping이 적용된 예제
### (1) 톤 매핑 없음 (None)
```csharp
postProcessVolume.profile.GetSetting<ColorGrading>().tonemapper.value = Tonemapper.None;
```
HDR이 클리핑되며, 너무 밝거나 너무 어두운 부분이 손실됨

### (2) ACES 톤 매핑 적용
```csharp
postProcessVolume.profile.GetSetting<ColorGrading>().tonemapper.value = Tonemapper.ACES;
```
가장 현실적인 색감을 유지하는 톤 매핑 방식

### (3) Reinhard 톤 매핑 적용
```csharp
postProcessVolume.profile.GetSetting<ColorGrading>().tonemapper.value = Tonemapper.Reinhard;
```
간단한 톤 매핑 방식, 성능이 가볍고 빠름

---

## 8. 결론
- Tonemapping은 HDR 데이터를 LDR 화면에 자연스럽게 변환하는 과정
- 밝기 범위를 조절하여 색상이 날아가거나 뭉개지는 현상을 방지
- Unity에서는 URP와 HDRP에서 톤 매핑을 지원하며, 다양한 톤 매핑 모드를 제공
- ACES 톤 매핑이 가장 사실적인 결과를 제공하며, 게임 및 영화에서 표준으로 사용됨
- 모바일 환경에서는 Reinhard와 같은 가벼운 방식이 적합

결론적으로, Tonemapping을 활용하면 더 사실적이고 안정적인 그래픽을 구현할 수 있다

