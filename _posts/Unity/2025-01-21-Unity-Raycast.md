---
title: "[Unity] Raycast 함수"
author: RePro
date: 2025-01-21 10:00:00 +0900
categories: [Programming, Unity]
tags: [unity]
---


# Unity의 Raycast 관련 함수 정리

Unity에서 사용 가능한 모든 Raycast 관련 함수들을 카테고리별로 정리한 문서입니다.  
각 함수는 충돌 판정 방식에 따라 동작 특성과 반환값이 다릅니다.

---

## 1. 기본 Raycast (선형 레이저)

| 함수 | 설명 | 반환 | 메모리 할당 |
|------|------|------|--------------|
| `Physics.Raycast(ray)` | 충돌 여부만 반환 | `bool` | X |
| `Physics.Raycast(ray, out RaycastHit hit)` | 첫 번째 충돌 정보 반환 | `bool` + `RaycastHit` | X |
| `Physics.Raycast(origin, direction)` | 시작점과 방향 지정 | `bool` | X |
| `Physics.RaycastAll(...)` | **모든 충돌체 반환** | `RaycastHit[]` | O |
| `Physics.RaycastNonAlloc(ray, resultArray)` | 미리 할당된 배열에 결과 저장 | `int` (충돌 수) | X |

---

## 2. SphereCast (구체 레이저)

| 함수 | 설명 | 반환 | 메모리 할당 |
|------|------|------|--------------|
| `Physics.SphereCast(...)` | 반지름을 가진 구 형태로 레이 캐스팅 | `bool` + `RaycastHit` | X |
| `Physics.SphereCastAll(...)` | 모든 충돌 반환 | `RaycastHit[]` | O |
| `Physics.SphereCastNonAlloc(...)` | 결과를 배열에 저장 | `int` | X |

---

## 3. CapsuleCast (캡슐 형태)

| 함수 | 설명 | 반환 | 메모리 할당 |
|------|------|------|--------------|
| `Physics.CapsuleCast(...)` | 두 점 사이의 캡슐 형태로 충돌 판정 | `bool` + `RaycastHit` | X |
| `Physics.CapsuleCastAll(...)` | 모든 충돌체 반환 | `RaycastHit[]` | O |
| `Physics.CapsuleCastNonAlloc(...)` | 배열에 결과 저장 | `int` | X |

---

## 4. BoxCast (박스 형태)

| 함수 | 설명 | 반환 | 메모리 할당 |
|------|------|------|--------------|
| `Physics.BoxCast(...)` | 반 크기를 기준으로 박스 모양으로 검사 | `bool` + `RaycastHit` | X |
| `Physics.BoxCastAll(...)` | 모든 충돌 반환 | `RaycastHit[]` | O |
| `Physics.BoxCastNonAlloc(...)` | 배열에 결과 저장 | `int` | X |

---

## 5. Overlap 시리즈 (범위 내부 겹침 검사)

| 함수 | 설명 | 반환 | 메모리 할당 |
|------|------|------|--------------|
| `Physics.OverlapSphere(...)` | 구체 내부의 Collider 반환 | `Collider[]` | O |
| `Physics.OverlapBox(...)` | 박스 내부의 Collider 반환 | `Collider[]` | O |
| `Physics.OverlapCapsule(...)` | 캡슐 내부의 Collider 반환 | `Collider[]` | O |
| `Physics.OverlapSphereNonAlloc(...)` | 배열에 결과 저장 | `int` | X |
| `Physics.OverlapBoxNonAlloc(...)` | 〃 | `int` | X |
| `Physics.OverlapCapsuleNonAlloc(...)` | 〃 | `int` | X |

---

## 6. 기타 고급 함수

| 함수 | 설명 |
|------|------|
| `Physics.RaycastCommand.ScheduleBatch(...)` | Burst 컴파일러 + Job 시스템용 레이캐스트 |
| `Physics.ComputePenetration(...)` | 두 콜라이더가 겹쳐 있을 때 침투 깊이 및 방향 계산 |
| `Physics.ClosestPoint(...)` | 콜라이더 표면에서 주어진 점에 가장 가까운 지점 계산 |

---

## 7. 함수 선택 요령

| 필요 | 추천 함수 |
|------|-----------|
| 첫 번째 충돌체만 | `Physics.Raycast` |
| 여러 개 충돌체 | `Physics.RaycastAll` |
| GC 없이 반복 검사 | `Physics.RaycastNonAlloc` |
| 범위 안 겹침 감지 | `Physics.OverlapSphere`, `OverlapBox` 등 |
| 이동 경로 충돌 | `SphereCast`, `CapsuleCast`, `BoxCast` |
| 고성능 대량 검사 | `RaycastCommand` (Job system) |

---

## 요약 비교표

| 형태 | 단일 충돌 | 다중 충돌 | 비할당 |
|------|------------|-------------|----------|
| 선형 | `Raycast` | `RaycastAll` | `RaycastNonAlloc` |
| 구체 | `SphereCast` | `SphereCastAll` | `SphereCastNonAlloc` |
| 캡슐 | `CapsuleCast` | `CapsuleCastAll` | `CapsuleCastNonAlloc` |
| 박스 | `BoxCast` | `BoxCastAll` | `BoxCastNonAlloc` |
| 범위 | `Overlap*` | 없음 (항상 다중) | `Overlap*NonAlloc` |
