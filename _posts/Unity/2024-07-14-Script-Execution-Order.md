---
title: "[Unity] 스크립트 실행 순서 조정 Script Execution Order"
author: RePro
date: 2024-07-14 10:00:00 +0900
categories: [Programming, Unity]
tags: [unity]
---

# Unity Script Execution Order 

---

## 1. Script Execution Order란?

Unity에서 여러 MonoBehaviour 스크립트가 있을 때,  
**어떤 스크립트가 먼저 실행되고 어떤 스크립트가 나중에 실행될지를 제어**하는 개념입니다.

특히 `Awake()`, `Start()`, `Update()` 등의 호출 순서를 정해  
**게임 로직 간 의존성 문제**를 예방할 수 있습니다.

---

## 2. Unity의 기본 실행 순서

### 초기화 단계

| 순서 | 메서드 | 설명 |
|------|--------|------|
| 1 | `Awake()` | 오브젝트가 생성될 때 호출 |
| 2 | `OnEnable()` | 오브젝트가 활성화될 때 호출 |
| 3 | `Start()` | 모든 Awake가 끝난 후 한 번 호출됨 |

### 프레임 루프 단계

| 메서드 | 설명 |
|--------|------|
| `Update()` | 매 프레임마다 호출 |
| `LateUpdate()` | 모든 `Update()` 후 호출 |
| `FixedUpdate()` | 고정 시간 간격으로 호출 (물리용) |

> `Start()`는 모든 `Awake()`가 호출된 후 실행됩니다.

---

## 3. 문제 예시

```csharp
void Start() {
    GameManager.Instance.DoSomething();  // GameManager가 아직 초기화되지 않았다면 NullReference 예외 발생
}
```

- `GameManager`의 Start가 더 늦게 실행될 경우 문제가 발생할 수 있음

---

## 4. Script Execution Order 설정 방법

1. Unity 메뉴 → **Edit > Project Settings > Script Execution Order**
2. 스크립트를 등록하고 실행 순서를 숫자로 지정

| 숫자 | 우선 순위 |
|------|------------|
| 작은 값 | 먼저 실행 |
| 큰 값 | 나중에 실행 |

예: `-100`이면 다른 `0`보다 먼저 실행됨

---

## 5. 대표적인 활용 사례

| 사용 사례 | 설명 |
|-----------|------|
| 싱글톤 매니저 초기화 | 다른 컴포넌트보다 먼저 Awake 실행 필요 |
| 카메라 LateUpdate 제어 | 플레이어 이동 후 따라붙기 |
| 물리 처리 정확성 | FixedUpdate 순서 조정 필요 시 |

---

## 6. 팁

- 가능한 한 **모든 스크립트에 설정하지 말고**, 핵심 의존성만 설정하는 것이 좋습니다.
- 복잡한 의존 관계는 **명시적 Init() 호출**로 처리하는 방법도 좋습니다.
- **Script Execution Order 설정은 디버깅이 어려우므로 최소화가 유지보수에 유리합니다.**

---

## 7. 요약

| 항목 | 설명 |
|------|------|
| 기능 | MonoBehaviour 메서드 실행 순서를 제어 |
| 기본 순서 | Awake → OnEnable → Start → Update → LateUpdate |
| 설정 위치 | Edit > Project Settings > Script Execution Order |
| 우선순위 숫자 | 작을수록 먼저 실행됨 |
| 주의점 | 핵심 스크립트만 설정, 과도한 사용 지양 |
