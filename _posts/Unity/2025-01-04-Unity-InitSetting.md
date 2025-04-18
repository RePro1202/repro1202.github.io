---
title: "[Unity] 프로젝트 초기 설정"
author: RePro
date: 2025-01-04 10:00:00 +0900
categories: [Programming, Unity]
tags: [unity]
---

## 1. Post Processing

Project Settings - Graphics
Default Render Pipeline - Asset - Data
* Screen Space Ambient Occlusion


Global Volume
* ToneMapping
* Vignette
* Color Adjustments
* Bloom

Directional Light
* Emission

Camera - Rendering
* Post Processing
* Anti-aliasing


## 2. 애니메이션 Setup

1. 인간 캐릭터 애니메이션일 경우, 임포트 후 (세팅 에셋) -> Rig -> Animation Type -> Humanoid


2. (세팅 에셋) -> Animation 하단. 
   1. 클립이름 적정한지 확인. 
   2. Loop Time 확인. 
   3. 미리보기 캐릭터가 정면을 보는지 확인. 아니라면 Root Transform Rotation -> Based Upon 조정 or offset 조정


## 3. 스크립트 실행 순서 조정

Edit - Project Settings - Script Excution Order

여기서 특정 스크립트의 실행 순서를 조정할 수 있음.
스크립트를 드래그앤 드롭해서 리스트에 추가하고 순서 조정 가능.


## 4. 3d 환경에서 텍스쳐를 배치하는 방법.

2d 이미지 파일을 배치하기 위해서 머테리얼을 만들고 해당 머테리얼의

머테리얼 - Surface Input - Base Map 옆에 작은 네모칸에 해당 이미지 파일 드랍.

Png 투명도 적용법: 위에서 만든 머테리얼을 적용한(매쉬 랜더러) 객체 하단 머테리얼 설정에서 / Surface Options - Surface Tpye - Transparent 적용.


## 5. [SerializeField] 주의.
프리펩 내부의 객체는 참조가 유지 되지만 씬에 배치된 프리펩 외부 객체는 참조가 깨질 수 있다. 때문에 [SerializeField] 객체 내부에서 사용하자.


## 6. 프레임 고정

```csharp
Application.targetFrameRate = 30;
```


## UI

### Grid Layout Group
자식 객체들을 자동으로 그리드 레이아웃 스타일로 만들어 준다.