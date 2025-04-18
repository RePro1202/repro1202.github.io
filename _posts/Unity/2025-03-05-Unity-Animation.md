---
title: "[Unity] 애니메이션"
author: RePro
date: 2025-03-05 10:00:00 +0900
categories: [Programming, Unity]
tags: [unity]
---

## 기본 애니메이션 적용과정

### 캐릭터 객체

캐릭터 객체 - Animator 컴포넌트 추가.

Animator 컴포넌트 - Controller와 Avatar 연결.

애니메이션 자체에 좌표이동이 있다면 필요에 따라 Apply Root Motion 옵션 조정.

---

### Animator

Window - Animator로 열 수 있다. 노드로 스테이트 머신 구성.

애니메이션 에셋(.fbx 등) 드래그 드롭. 드롭된 노드를 좌클릭해서 Entry에서 나오는 디폴트 스테이트 변경가능.

노드 우클릭 후 Make Transition 으로 다른 노드와 연결.

* AnimatorTransitionBase
  
  화살표 클릭해서 인스펙터 확인. Has Exit Time 활성화시 종료 시점에 자동으로 다음 애니메이션 재생. 해제하면 코드에서 트리거 해야 전환됨. [Fixd Duration, Transition Duration, Transition Offset, Interruption Souce 설명 추가.]

---

### Animation

Window - Animation 오픈. Animator를 가진 캐릭터 선택, 들어있는 Animation들을 타임 라인 형태로 볼 수 있으며, 수정가능.

---

## 캐릭터 손에 장비 들게 하기

단순한 방법으로는 신 하이어라키에서 캐릭터 선택, 본 계층구조에서 원하는 부위 선택해서 자식으로 넣고 위치조정.
이때 월드에 한번 배치하고 자식으로 넣지 않고, 하이어라키에서 바로 넣을때 스케일 값 차이로 너무 작거나 크게 나올 수 있음.

그런데 하나에서 위치를 맞춰도 다른 애니메이션에서는 위치가 부자연스러울 수 있음. 이때는 애니메이션을 수정해야 하는데 Animation창에서 보면 Read-Only로 되어 수정이 불가능 할 수 있음.
이럴때는 해당 애니메이션 클립의 복사본을 만들면 수정 할 수 있음. 복사본과 원본 이름이 같지 않게 수정해주고 애니메이터에서 카피본으로 교체후 수정.