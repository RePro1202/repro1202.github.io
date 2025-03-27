---
title: "[Unity] GetComponent vs TryGetComponent 비교"
author: RePro
date: 2024-07-20 10:00:00 +0900
categories: [Programming, Unity]
tags: [unity]
---

# GetComponent vs TryGetComponent 비교 정리

---

## 1. 개요

Unity에서 컴포넌트를 가져올 때 주로 사용되는 API는 다음 두 가지입니다:

- `GetComponent<T>()`
- `TryGetComponent<T>(out T component)`

둘 다 동일한 목적(컴포넌트 접근)을 갖지만, **성능**, **안정성**, **코드 스타일** 면에서 차이가 존재합니다.

---

## 2. 비교 요약

| 항목 | GetComponent<T>() | TryGetComponent<T>(out T) |
|------|--------------------|----------------------------|
| 반환값 | 컴포넌트 or `null` | 성공 여부 (`bool`) |
| null 체크 | 필요 | 필요 없음 |
| 성능 | 상대적으로 느림 | 더 빠름 (GC 줄임) |
| 사용 위치 | 일반 상황 | 반복문, 성능 민감한 곳 |
| 예시 | `GetComponent<Rigidbody>()` | `TryGetComponent(out Rigidbody rb)` |

---

## 3. GetComponent<T>()

### 설명
- `T` 타입의 컴포넌트를 가져옵니다.
- 존재하지 않으면 `null` 반환 → 반드시 null 체크 필요

### 예제
```csharp
Rigidbody rb = GetComponent<Rigidbody>();
if (rb != null) {
    rb.AddForce(Vector3.up);
}
```

---

## 4. TryGetComponent<T>(out T)

### 설명
- `GetComponent`와 같은 기능이지만 더 **성능 최적화됨**
- 결과를 `out`으로 전달하고, 성공 여부를 `bool`로 반환
- Unity 2020.1 이후 도입됨

### 예제
```csharp
if (TryGetComponent(out Rigidbody rb)) {
    rb.AddForce(Vector3.up);
}
```

---

## 5. 성능 비교

- `TryGetComponent`는 내부적으로 더 적은 GC를 발생시키고,
- null 반환이 아닌 `bool` 체크로 **조건 분기 효율**이 높음
- 반복 루프, `Update()`, `OnTriggerStay()`에서 유리함

---

## 6. 선택 가이드

| 상황 | 추천 API |
|------|-----------|
| 초기화 시, 단발성 호출 | `GetComponent<T>()` |
| 성능이 중요한 루프 내 호출 | `TryGetComponent<T>()` |
| 코드 가독성과 안전성 중시 | `TryGetComponent<T>()` |

---

## 7. 결론

- 기능적으로는 유사하지만, `TryGetComponent`는 **성능**과 **안전성**에서 우위
- Unity 2020 이상을 사용 중이라면, 되도록 `TryGetComponent` 사용을 고려
