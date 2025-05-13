---
title: "[Unity] Random"
author: RePro
date: 2025-03-09 10:00:00 +0900
categories: [Programming, Unity]
tags: [unity]
---


## Unity에서 랜덤값을 얻는 방법

### 1. UnityEngine.Random 클래스

Unity에서 가장 일반적으로 사용하는 랜덤 클래스.

#### 주요 함수

| 함수                                   | 설명                            |
| ------------------------------------ | ----------------------------- |
| `Random.Range(int min, int max)`     | `min` 이상 `max` 미만의 **정수** 반환  |
| `Random.Range(float min, float max)` | `min` 이상 `max` 이하의 **실수** 반환  |
| `Random.value`                       | 0.0f 이상 1.0f 이하의 **float** 반환 |
| `Random.insideUnitSphere`            | 반지름 1인 구 안의 랜덤 벡터 반환          |
| `Random.insideUnitCircle`            | 반지름 1인 원 안의 랜덤 벡터 반환          |
| `Random.onUnitSphere`                | 반지름 1인 구 표면상의 랜덤 벡터 반환        |

#### 예시

```csharp
int randomInt = Random.Range(1, 11);
float randomFloat = Random.value;
float randomAngle = Random.Range(0f, 360f);
Vector2 randomPos = Random.insideUnitCircle * 5f;
```

---

### 2. System.Random 클래스

C# 표준 라이브러리 랜덤 클래스.

#### 예시

```csharp
System.Random rng = new System.Random();
int number = rng.Next(10);
double randomDouble = rng.NextDouble();
```

**주의:** Unity Editor에서는 `System.Random`이 프레임마다 같은 시드를 가질 수 있음 → 게임플레이 로직엔 `UnityEngine.Random` 권장.

---

### 3. Random.InitState(int seed)

`UnityEngine.Random`의 시드(seed)를 설정해 랜덤값 시퀀스를 고정.

```csharp
Random.InitState(1234);
int val1 = Random.Range(0, 10);
int val2 = Random.Range(0, 10);
```

→ 항상 같은 값 반환.

---

### 4. Unity Random과 System.Random 차이

| 구분        | UnityEngine.Random     | System.Random       |
| --------- | ---------------------- | ------------------- |
| 주 용도      | 게임 내 랜덤값 (위치, 색상 등)    | 비게임 로직, 알고리즘        |
| 시드 설정     | `Random.InitState()`   | `new Random(seed)`  |
| float 지원  | O                      | NextDouble 후 캐스팅 필요 |
| Vector 지원 | O (insideUnitSphere 등) | X                   |

---

### 5. 추천 사용 상황

| 상황                 | 권장 클래스                              |
| ------------------ | ----------------------------------- |
| 게임 오브젝트 위치, 물리 랜덤화 | UnityEngine.Random                  |
| UI 애니메이션, 색상 랜덤    | UnityEngine.Random                  |
| 암호학 제외 일반 로직 랜덤    | System.Random                       |
| 반복 가능한 결과 필요 (시드)  | UnityEngine.Random 또는 System.Random |

---

### 6. Random.Range(int, int) 범위 주의

* **정수:** `min` 이상, `max` 미만 → `Random.Range(0, 10)` → 0\~9 반환
* **실수:** `min` 이상, `max` 이하 → `Random.Range(0f, 1f)` → 0.0\~1.0 포함

---

### 7. 요약

* Unity 게임 로직: **UnityEngine.Random 권장**
* 벡터, float, int 모두 지원
* 시드 고정 필요 → `Random.InitState()`
* 비게임 로직(데이터 처리) → `System.Random`
