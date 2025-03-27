---
title: "[Unity] Transform 회전 관련 속성"
author: RePro
date: 2024-07-19 10:00:00 +0900
categories: [Programming, Unity]
tags: [unity]
---

# Unity의 Transform 회전 관련 속성 정리

---

## 1. transform.rotation

### 설명
- 현재 Transform의 **회전 정보를 쿼터니언(Quaternion)** 형태로 나타냅니다.
- 내부적으로 사용하는 회전 표현 방식이며, 계산과 보간에 유리합니다.

### 특징
- 타입: `Quaternion`
- 짐벌락(Gimbal Lock) 문제 없음
- API 예시: `Quaternion.Lerp()`, `LookRotation()`, `SetLookRotation()`

### 예제
```csharp
transform.rotation = Quaternion.Euler(0, 90, 0);
```

---

## 2. transform.eulerAngles

### 설명
- 현재 Transform의 회전을 **오일러 각도(Euler Angles)** 로 나타냅니다.
- XYZ 회전을 도(degree) 단위로 제공합니다.

### 특징
- 타입: `Vector3`
- 외부에서는 사람이 이해하기 쉬운 값으로 표현
- 내부에서는 쿼터니언으로 변환되어 저장됨
- 직관적이지만 역변환 시 예상과 다를 수 있음

### 예제
```csharp
transform.eulerAngles = new Vector3(0, 180, 0);
```

---

## 3. transform.forward

### 설명
- 현재 Transform이 바라보는 **전방 방향 벡터**
- **Local Z+ 방향을 기준**으로 회전에 따라 계산됨

### 특징
- 타입: `Vector3`
- 이동 방향, 레이캐스트, 거리 계산 등에 활용
- 회전 값(rotation)에 따라 자동으로 변경됨

### 예제
```csharp
Vector3 dir = transform.forward * 10f;
transform.position += dir * Time.deltaTime;
```

---

## 4. 세 속성 비교 요약

| 속성 | 설명 | 데이터 타입 | 사용 예 |
|------|------|--------------|----------|
| `transform.rotation` | 쿼터니언 회전값 | `Quaternion` | 회전 보간, 회전 설정 |
| `transform.eulerAngles` | XYZ 오일러 각도 | `Vector3` | 디버깅, 직관적 설정 |
| `transform.forward` | 전방 방향 벡터 | `Vector3` | 이동 방향 계산 |

---

## 5. 관계 요약

```plaintext
eulerAngles ⇄ Quaternion(rotation)
      ↓
   회전 방향
      ↓
forward / right / up 벡터 계산됨
```

---

## 6. 팁

- `rotation`은 직접 회전을 보간하거나 정확한 회전 연산이 필요할 때 사용
- `eulerAngles`는 직관적으로 회전값을 설정하고 싶을 때 유용 (단, 정확성에 주의)
- `forward`, `right`, `up`은 **Transform이 기준축(Local axis)** 을 기준으로 방향 계산에 매우 자주 사용됨
