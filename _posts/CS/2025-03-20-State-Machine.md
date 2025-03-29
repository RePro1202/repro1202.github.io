---
title: "[CS] 상태 머신(State Machine)"
author: RePro
date: 2025-03-20 10:00:00 +0900
categories: [CS]
tags: [cs]
---

# 상태 머신(State Machine)과 FSM 정리

---

## 1. 상태 머신(State Machine)란?

- 시스템이 가질 수 있는 여러 상태(state) 중 하나에 있고,
- 입력(이벤트)에 따라 상태를 전이(transition)하며 동작하는 추상 모델

### 구성 요소

| 구성 요소 | 설명 |
|-----------|------|
| 상태 (State) | 시스템이 놓일 수 있는 하나의 조건 또는 상황 |
| 입력 (Input/Event) | 상태 전이를 유발하는 자극 |
| 전이 (Transition) | 현재 상태에서 다른 상태로의 전이 |
| 행동 (Action) | 상태 진입/이탈/유지 중 발생하는 동작 |

---

## 2. FSM(Finite State Machine)과 차이점

| 항목 | 상태 머신 (State Machine) | FSM (Finite State Machine) |
|------|---------------------------|-----------------------------|
| 의미 | 포괄적인 개념 (추상 모델) | 유한 개수의 상태를 가지는 특수한 상태 머신 |
| 상태 수 | 제한 없음 (무한 상태 머신도 포함) | 유한한 상태 집합만 허용 |
| 사용 맥락 | 일반 이론/프로그래밍 모델 | 컴파일러 이론, 게임 로직 등 |
| 관계 | 상위 개념 | 하위 개념 (특정한 경우) |

→ FSM은 상태 머신의 한 종류이며, 일반적으로 우리가 "상태 머신"이라고 부를 때 FSM을 지칭하는 경우가 많음.

---

## 3. 게임 개발에서 상태 머신 사용 예시

### 공통 사용처

- AI 상태 전이 (Idle → Patrol → Chase → Attack)
- 플레이어 행동 처리 (Idle, Run, Jump, Die)
- UI 상태 전환 (메뉴, 인벤토리, 옵션 등)
- 애니메이션 전이 (Animator 시스템)
- 퀘스트/시나리오 흐름 제어

---

## 4. Unity에서 상태 머신

### 사용 방식

- Animator의 Mecanim 시스템 → 시각적 FSM
- 직접 FSM 구현 (상태 인터페이스 + 상태 전이 매니저)
- Coroutines 기반 상태 로직 흐름 제어 가능

### 특징

- 시각화된 애니메이션 FSM으로 진입장벽이 낮음
- AnimatorController 기반의 Transition Rule 설정이 용이
- 스크립트로도 FSM 쉽게 구현 가능 (enum + switch, 클래스 기반)

---

## 5. Unreal Engine에서 상태 머신

### 사용 방식

- AnimGraph + State Machine 노드로 애니메이션 FSM 구현
- C++ 또는 Blueprint로 FSM 직접 구현 가능

### 특징

- State Machine 노드를 통해 애니메이션 블렌딩, 전이 규칙 시각화
- UStateMachineComponent 또는 직접 C++ FSM 구현이 유연함
- 복잡한 상태 전이에서도 Blueprint로 직관적 구현 가능

---

## 6. Unity vs Unreal FSM 비교

| 항목 | Unity | Unreal Engine |
|------|-------|---------------|
| 기본 FSM 제공 | AnimatorController + Mecanim | AnimGraph + State Machine |
| 시각화 도구 | 강력함 (노드 기반) | 강력함 (Blueprint 연동) |
| 코드 기반 FSM | MonoBehaviour + 클래스 FSM | C++/Blueprint 모두 유연 |
| 난이도 | 비교적 쉬움 | 복잡하지만 고성능 구현 가능 |
| 커스터마이징 | enum + switch로 간단하게 구현 | 상태 객체/클래스 기반으로 구조화 가능 |

---

## 7. 실전 예시 (간략화된 FSM 구조)

```cpp
// 상태 인터페이스
class IState {
public:
    virtual void Enter() = 0;
    virtual void Update() = 0;
    virtual void Exit() = 0;
};

// 상태 매니저 예시
class StateMachine {
    IState* current;
public:
    void ChangeState(IState* next) {
        current->Exit();
        current = next;
        current->Enter();
    }

    void Update() {
        current->Update();
    }
};
```

---

## 8. 요약

| 항목 | 설명 |
|------|------|
| 상태 머신 | 상태/입력/전이/행동으로 구성된 추상 모델 |
| FSM | 유한 상태를 갖는 상태 머신 |
| 게임 활용 | AI, 캐릭터 행동, 애니메이션, UI 흐름 등 |
| Unity | Animator 기반 시각 FSM + 코드 FSM 구현 |
| Unreal | AnimGraph 기반 FSM + Blueprint 연동 |

---

게임에서는 주로 애니메이션에 사용되지만 FSM, 오토마타는 깊이있는 내용이라 추가로 정리 예정.
