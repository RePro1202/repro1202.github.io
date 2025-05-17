---
title: "[Unity] 스테이트 머신 구현"
author: RePro
date: 2025-05-06 10:00:00 +0900
categories: [Programming, Unity]
tags: [unity]
---

## Unity 상태머신(State Machine) 구현 예시

### 1. 상태머신 개념

* **상태(State)**: 객체가 가질 수 있는 상태 (예: Idle, Move, Attack)
* **상태 전이(Transition)**: 상태가 다른 상태로 전환되는 규칙
* **상태별 로직**: 각 상태마다 독립적으로 동작 수행

---

### 2. 상태머신 기본 구조 (클래스 기반)

#### 2.1 상태 인터페이스 정의

```csharp
public interface IState
{
    void Enter();
    void Execute();
    void Exit();
}
```

모든 상태는 `Enter()`, `Execute()`, `Exit()` 메서드를 포함.

---

#### 2.2 상태 구현 클래스

```csharp
public class IdleState : IState
{
    public void Enter() { Debug.Log("Idle: Enter"); }
    public void Execute() { Debug.Log("Idle: Execute"); }
    public void Exit() { Debug.Log("Idle: Exit"); }
}

public class MoveState : IState
{
    public void Enter() { Debug.Log("Move: Enter"); }
    public void Execute() { Debug.Log("Move: Execute"); }
    public void Exit() { Debug.Log("Move: Exit"); }
}
```

각 상태마다 동작 정의.

---

#### 2.3 상태머신 관리 클래스

```csharp
public class StateMachine
{
    private IState currentState;

    public void ChangeState(IState newState)
    {
        if (currentState != null)
            currentState.Exit();

        currentState = newState;

        if (currentState != null)
            currentState.Enter();
    }

    public void Update()
    {
        if (currentState != null)
            currentState.Execute();
    }
}
```

`ChangeState()`로 상태 변경, `Update()`로 현재 상태 실행.

---

#### 2.4 Unity 컴포넌트 연결

```csharp
using UnityEngine;

public class Enemy : MonoBehaviour
{
    private StateMachine stateMachine;

    private void Start()
    {
        stateMachine = new StateMachine();
        stateMachine.ChangeState(new IdleState());
    }

    private void Update()
    {
        stateMachine.Update();

        if (Input.GetKeyDown(KeyCode.Space))
        {
            stateMachine.ChangeState(new MoveState());
        }
    }
}
```

---

### 3. 실행 결과

1. 게임 시작 시 `"Idle: Enter"` 출력
2. 매 프레임 `"Idle: Execute"` 출력
3. 스페이스바 누르면:

   * `"Idle: Exit"`
   * `"Move: Enter"`
   * 이후 매 프레임 `"Move: Execute"` 출력

---

### 4. 구조 요약

```
Enemy (MonoBehaviour)
 └ StateMachine
     ├ IState (interface)
     ├ IdleState
     └ MoveState
```

---

### 5. 이 방식의 장점

* 상태 추가/수정 시 클래스만 추가하면 됨
* 각 상태 코드가 독립적 (SRP 원칙 만족)
* 상태 전이 로직이 명확하고 깔끔

---

### 6. 대안 방법

더 복잡하거나 시각적 상태머신 필요 시:

* Unity Animator + Animation State Machine 사용
* Unity Visual Scripting
* PlayMaker 같은 3rd Party FSM Asset 사용

<br>

---


## Unity 상태머신 구현 방식 비교 (클래스 기반 vs enum 기반)

### 1. 기본 개념

| 방식      | 설명                               |
| ------- | -------------------------------- |
| 클래스 기반  | 각 상태를 별도의 **클래스**로 정의            |
| enum 기반 | 상태를 **enum 값**으로 구분, 상태마다 분기문 작성 |

---

### 2. 코드 구조 예시

#### 클래스 기반 예시

```csharp
interface IState { void Enter(); void Execute(); void Exit(); }
class IdleState : IState { ... }
class MoveState : IState { ... }

StateMachine.ChangeState(new IdleState());
StateMachine.Update(); // 현재 상태의 Execute() 실행
```

- 상태마다 독립 클래스 + 메서드 호출

#### enum 기반 예시

```csharp
enum State { Idle, Move }
State currentState;

void Update()
{
    switch (currentState)
    {
        case State.Idle:
            // Idle 로직
            break;
        case State.Move:
            // Move 로직
            break;
    }
}
```

- `switch` 문 안에 상태별 로직 작성

---

### 3. 주요 차이점

| 항목             | 클래스 기반            | enum 기반              |
| -------------- | ----------------- | -------------------- |
| 상태 확장성         | ✔ 클래스 추가만 필요      | ❌ switch-case 수정 필요  |
| 상태 코드 분리       | ✔ 각 상태 클래스 별도 파일  | ❌ 모든 상태 로직 한 파일에 작성  |
| 유지보수성          | ✔ 각 상태 독립 → 영향 적음 | ❌ 상태 늘어날수록 switch 복잡 |
| 초보자 이해         | ❌ 추상화 개념 필요       | ✔ 직관적이고 간단함          |
| 실행 속도          | 거의 동일 (최적화 영향 미미) | 거의 동일                |
| 다형성 (override) | ✔ OOP 다형성 지원      | ❌ 불가능                |

---

### 4. enum 기반이 적합한 경우

- 상태 개수가 **적고 간단**할 때
- **간단한 상태 전이**만 필요할 때
- **프로토타입, 테스트 코드 작성**

예: `Idle`, `Move`, `Attack` 정도의 간단한 상태

---

### 5. 클래스 기반이 적합한 경우

- 상태마다 **복잡한 로직** 포함
- **상태별 Enter, Execute, Exit 메서드 필요**
- **상태 추가/삭제/수정**이 자주 일어날 때
- **상태를 다른 객체에서도 재사용**해야 할 때

예: 적 AI, 플레이어 상태, 보스 패턴 관리

---

### 6. 예시 비교

#### 클래스 기반 상태 전환

```csharp
stateMachine.ChangeState(new MoveState());
```

→ `MoveState.Enter()`, `MoveState.Execute()`, `MoveState.Exit()` 자동 관리

#### enum 기반 상태 전환

```csharp
currentState = State.Move;
switch (currentState) { case State.Move: ... }
```

→ 상태별 진입/종료 로직 직접 호출 필요

---

### 7. 추천 상황 요약

| 상황            | 추천 방식   |
| ------------- | ------- |
| 상태 개수 적고 간단   | enum 기반 |
| 상태 많고 복잡      | 클래스 기반  |
| 유지보수성, 확장성 중시 | 클래스 기반  |
| 빠른 구현, 프로토타입  | enum 기반 |

---

**결론:**

* **클래스 기반 → 확장성과 유지보수성**
* **enum 기반 → 간단함과 빠른 작성**
