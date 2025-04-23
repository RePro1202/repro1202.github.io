---
title: "고정 시간(Fixed Time)과 FixedUpdate"
author: RePro
date: 2025-03-05 10:00:00 +0900
categories: [Programming, Game]
tags: [math]
---

# 게임에서의 고정 시간(Fixed Time)과 FixedUpdate 정리

게임 엔진에서는 프레임 속도(FPS)와 관계없이 **안정적인 물리 연산이나 시간 기반 처리**를 위해 "고정 시간 간격(Fixed Delta Time)"을 사용하는 구조를 제공합니다.

---

## 1. 고정 시간(Fixed Time)이란?

- **고정된 시간 간격**으로 실행되는 시뮬레이션 시간 단위
- 보통 0.02초 (50Hz)
- 엔진 내부에서 별도로 누적하여 반복 처리
- 주로 물리, 충돌, 네트워크 시뮬레이션, 타이머 등에 사용

---

## 2. 왜 Fixed Time이 필요한가?

| 사용 상황 | 이유 |
|------------|------|
| 물리 엔진 처리 | 일정한 시간 간격이 있어야 시뮬레이션 결과가 안정적임 |
| 네트워크/서버 로직 | 예측 가능성, 재현성 필요 |
| 충돌 판정 | 정확한 위치와 속도 계산을 위해 필요 |
| 애니메이션, 리플레이 시스템 | 일정한 시간 간격 기반의 계산이 요구됨 |

---

## 3. FixedUpdate란?

- Unity에서 제공하는 **고정 시간 간격으로 호출되는 함수**
- `Time.fixedDeltaTime` 간격마다 호출됨
- 입력은 받지 않고, **물리 연산만 처리**해야 함 (입력은 `Update()`에서 처리)

### Unity 예시
```csharp
void FixedUpdate() {
    rb.velocity += Vector3.down * gravity * Time.fixedDeltaTime;
}
```

---

## 4. Fixed Time의 누적 처리 원리

고정 시간은 다음과 같은 방식으로 처리됩니다:

### 누적 처리 루프 구조
```cpp
accumulator += deltaTime;

while (accumulator >= fixedDeltaTime) {
    FixedUpdate();
    accumulator -= fixedDeltaTime;
}
```

- **accumulator**: 누적된 경과 시간
- **fixedDeltaTime**: 고정 간격 시간 (예: 0.02초)
- `FixedUpdate()`는 **필요한 만큼 반복 호출**됨

### 예시 상황
| 상황 | deltaTime | FixedUpdate 호출 수 |
|-------|------------|---------------------|
| 60FPS | 0.01667초 | 0 or 1회 (2프레임에 1번) |
| 30FPS | 0.03333초 | 1~2회 |
| 10FPS | 0.1초 | 최대 5회 |

---

## 5. 실제 시뮬레이션 코드 예시

```cpp
float fixedDeltaTime = 0.02f;
float accumulator = 0.0f;
float currentTime = GetCurrentTime();

while (GameRunning) {
    float newTime = GetCurrentTime();
    float deltaTime = newTime - currentTime;
    currentTime = newTime;

    accumulator += deltaTime;

    while (accumulator >= fixedDeltaTime) {
        FixedUpdate(); // 고정 간격 처리
        accumulator -= fixedDeltaTime;
    }

    Render(); // 프레임은 가변적으로 처리
}
```

---

## 6. 요약 비교: Fixed Time vs Delta Time

| 항목 | Fixed Time | Delta Time |
|------|-------------|-------------|
| 시간 간격 | 고정 (보통 0.02초) | 프레임마다 달라짐 |
| 호출 위치 | FixedUpdate | Update, Tick |
| 사용 목적 | 물리, 충돌, 시뮬레이션 | 애니메이션, 이동, UI 반응 등 |
| 일관성 | 재현성 높음 | 불안정 가능 |
| 누적 처리 | 누적 시간에 따라 반복 호출 | 1프레임 1회 고정 호출 |

---

고정 시간 처리(Fixed Time)는 게임에서 물리적 안정성과 정밀한 계산을 보장하기 위한 핵심 개념입니다. 특히 프레임 수가 불안정하거나 다양한 플랫폼에서 동작할 경우, **일정한 시간 간격으로 처리되는 로직을 분리**하는 것이 안정적인 게임 동작의 기반이 됩니다.

