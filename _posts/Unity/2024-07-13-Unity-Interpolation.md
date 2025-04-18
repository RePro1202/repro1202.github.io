---
title: "[Unity] 보간(Interpolation) 함수"
author: RePro
date: 2024-07-13 10:00:00 +0900
categories: [Programming, Unity]
tags: [unity]
---

# Unity의 보간(Interpolation) 함수 정리

---

## 1. 보간(Interpolation)이란?

- 두 값 사이를 부드럽게 이어주는 수학적 계산
- 애니메이션, 이동, 회전, 스케일 등 다양한 곳에 활용됨
- Unity는 다양한 타입(Vector, Quaternion, float 등)에 대한 보간 API를 제공

---

## 2. 대표적인 보간 함수들

### 1. Mathf.Lerp(a, b, t)

- `a`에서 `b`까지 `t` (0~1) 비율만큼 선형 보간
- `t = 0` → a, `t = 1` → b

```csharp
float value = Mathf.Lerp(0f, 10f, 0.5f); // 결과: 5.0
```

---

### 2. Vector3.Lerp(a, b, t)

- 두 벡터 간 선형 보간
- 벡터의 위치를 부드럽게 연결할 때 사용

```csharp
Vector3 pos = Vector3.Lerp(start, end, Time.deltaTime);
```

---

### 3. Vector3.LerpUnclamped(a, b, t)

- `t`가 0~1 범위를 벗어나도 작동
- 빠르게 뒤로 넘기거나 넘어서서 보간할 때 사용

```csharp
Vector3 extended = Vector3.LerpUnclamped(a, b, 1.5f); // b를 넘어간 위치
```

---

### 4. Vector3.Slerp(a, b, t)

- 구면 선형 보간(Spherical Linear Interpolation)
- 방향(단위 벡터) 보간에 적합, 회전 경로가 더 부드럽고 일정

```csharp
Vector3 dir = Vector3.Slerp(dirA, dirB, t);
```

---

### 5. Quaternion.Lerp(a, b, t)

- 회전(Quaternion) 보간
- Slerp보다 연산이 빠름, 부드러움은 덜할 수 있음

---

### 6. Quaternion.Slerp(a, b, t)

- 회전 보간의 대표 함수
- 부드럽고 자연스러운 회전 경로

```csharp
transform.rotation = Quaternion.Slerp(startRot, endRot, t);
```

---

### 7. Mathf.SmoothStep(a, b, t)

- 보간이 좀 더 부드럽게 시작하고 끝남 (S자 형태)
- `t`는 자동으로 부드러운 곡선에 따라 매핑됨

```csharp
float smooth = Mathf.SmoothStep(0f, 1f, t);
```

---

### 8. Mathf.MoveTowards(current, target, maxDelta)

- 일정 속도로 `current`에서 `target`까지 이동
- `deltaTime`과 함께 쓰면 프레임에 무관한 이동 구현 가능

```csharp
value = Mathf.MoveTowards(value, target, speed * Time.deltaTime);
```

---

### 9. Vector3.MoveTowards(current, target, maxDistanceDelta)

- `Vector3.Lerp`와 유사하지만 일정한 속도로 거리 이동
- 벡터 이동 시 자주 사용됨

```csharp
transform.position = Vector3.MoveTowards(transform.position, target, speed * Time.deltaTime);
```

---

## 3. 요약 비교

| 함수 | 타입 | 특징 |
|------|------|------|
| `Mathf.Lerp` | float | 선형 보간 |
| `Vector3.Lerp` | Vector3 | 위치 보간 |
| `Vector3.Slerp` | Vector3 | 방향 보간 (단위 벡터 간) |
| `Quaternion.Lerp` | Quaternion | 회전 보간 (빠름) |
| `Quaternion.Slerp` | Quaternion | 회전 보간 (부드러움) |
| `Mathf.SmoothStep` | float | 더 부드러운 S자 보간 |
| `MoveTowards` | float/Vector3 | 일정 속도 유지, 목표까지 이동 |

---

## 4. 팁

- `Lerp`는 t값이 0~1 사이일 때만 보간 역할
- `Slerp`는 회전 또는 방향을 부드럽게 처리할 때 선호
- `MoveTowards`는 속도 기반 구현에 필수
