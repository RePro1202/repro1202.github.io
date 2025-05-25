---
title: "Delta Time"
author: RePro
date: 2025-03-05 10:00:00 +0900
categories: [Programming, Game]
tags: [math, unity, unreal]
---

# 게임에서의 Delta Time 정리

게임 개발에서 **Delta Time(델타 타임)** 은 프레임 간 시간 차를 의미하며, 프레임 수에 관계없이 일관된 움직임과 연산을 가능하게 하는 핵심 개념입니다.

---

## 1. Delta Time이란?

- **정의**: 두 프레임 사이의 경과 시간 (단위: 초)
- **수식**:  
  $$
  	{deltaTime} = {currentTime} - {previousTime}
  $$
- 보통 `float` 타입으로, 초 단위로 표현됩니다.

---

## 2. 왜 필요한가?

프레임 수가 높거나 낮을 때, 움직임이 일관되지 않으면 게임이 버벅이거나 너무 빠르게 작동할 수 있습니다.

### 예제: 속도 100 픽셀/초

| FPS | Delta Time (초) | 이동량/프레임 | 1초 동안 이동량 |
|-----|------------------|----------------|------------------|
| 60  | 1 / 60 ≈ 0.01667 | 100 × 0.01667 ≈ 1.67 | 1.67 × 60 = 100 |
| 30  | 1 / 30 ≈ 0.03333 | 100 × 0.03333 ≈ 3.33 | 3.33 × 30 = 100 |

### 수식:
$$
\text{이동량} = \text{속도 (pixels/sec)} \times \text{deltaTime (sec)}
$$

---

## 3. 어떤 상황에서 사용되나?

| 사용 예 | 설명 |
|---------|------|
| 캐릭터 이동 | 일정한 속도로 움직이도록 함 |
| 카메라 이동 | 부드럽고 일관된 이동 보장 |
| 물리 연산 | 중력, 속도, 마찰 등 정확도 확보 |
| 애니메이션 진행률 | 초 단위 비율 계산 가능 |
| 타이머, 쿨다운 | 실제 시간 기반 지연 처리 |

---

## 4. Unity에서의 Delta Time

### 코드 예시
```csharp
void Update() {
    float speed = 5f;
    transform.position += Vector3.forward * speed * Time.deltaTime;
}
```
- `Time.deltaTime`: 이전 프레임 이후 경과 시간
- 초당 5유닛 이동

### 기타 활용 예
```csharp
transform.Rotate(Vector3.up * 90f * Time.deltaTime);
```
```csharp
float timer += Time.deltaTime;
if (timer > 2f) { DoSomething(); timer = 0f; }
```

- `FixedUpdate`에서 `Delta Time`이 호출되면 `fixedDeltaTime`이 반환된다다
- `Update`에서 `FixedUpdate` 호출은 `FixedUpdate`을 반환한다

---

## 5. Unreal Engine에서의 Delta Time

### C++ 코드 예시
```cpp
void AMyActor::Tick(float DeltaTime) {
    Super::Tick(DeltaTime);
    FVector NewLocation = GetActorLocation();
    float Speed = 300.0f;
    NewLocation += GetActorForwardVector() * Speed * DeltaTime;
    SetActorLocation(NewLocation);
}
```

- `Tick(float DeltaTime)`은 프레임마다 호출되며, `DeltaTime`은 자동으로 전달됨

### 블루프린트
- `Event Tick` → `Delta Seconds` 핀을 통해 사용

---

## 6. 엔진 내부에서 Delta Time은 어떻게 계산되는가?

### 공통 방식
```cpp
float deltaTime = currentTime - previousTime;
previousTime = currentTime;
```

### Unity 내부
- C++ 레벨에서 매 프레임 시간 측정 후 C# 엔진에 `Time.deltaTime`으로 전달됨

### Unreal 내부
- `UGameEngine::Tick()` 등에서 `FPlatformTime::Seconds()`로 시간 측정
- `FApp::GetDeltaTime()`을 통해 전역 델타 제공

---

## 7. 고정 시간(Fixed Time)과의 차이

- **Delta Time**: 매 프레임 달라짐
- **Fixed Delta Time**: 일정한 시간 간격 (예: 물리 처리용)
- Unity의 `FixedUpdate()`, Unreal의 `Physics Tick()` 등에서 사용

---

## 8. 요약

| 항목 | 설명 |
|------|------|
| 정의 | 두 프레임 간의 시간 차 (초 단위) |
| 목적 | FPS와 무관한 일관된 움직임 보장 |
| 핵심 수식 | `이동량 = 속도 × deltaTime` |
| Unity 사용 | `Time.deltaTime` |
| Unreal 사용 | `Tick(float DeltaTime)` 전달 |
| 프레임 보정 | 속도는 시간 기준이므로 프레임 수 영향 없음 |
