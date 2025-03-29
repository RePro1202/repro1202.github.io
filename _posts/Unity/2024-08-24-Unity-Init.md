---
title: "[Unity] 의존성 초기화 순서 문제"
author: RePro
date: 2024-08-24 10:00:00 +0900
categories: [Programming, Unity]
tags: [unity]
---

# Unity에서 의존성 초기화 순서 문제

게임 개발 중 여러 컴포넌트 간에 초기화 순서가 중요할 경우, 단순히 `Start()`에 초기화를 맡기면 **불안정한 실행 순서**로 인해 예기치 않은 버그가 발생할 수 있습니다.

이 문서는 초기화 순서 문제를 해결하는 실전 전략과 예제를 정리한 자료입니다.

---

## 1. 문제 개요

Unity는 `Start()` 메서드의 호출 순서를 보장하지 않기 때문에 다음과 같은 상황에서 문제가 발생할 수 있습니다:

- A는 B를 참조하여 `Start()`에서 초기화
- B도 `Start()`에서 내부 상태를 설정
- Unity가 A → B 순으로 `Start()`를 호출하면 A는 초기화되지 않은 B를 참조하게 됨

---

## 2. 해결 전략

| 전략 | 설명 |
|------|------|
| **Init() 메서드 분리** | 수동으로 초기화 순서를 지정 |
| **Script Execution Order 설정** | Unity 설정으로 호출 순서 고정 |
| **Awake()와 Start() 분리** | Awake는 Start보다 먼저 호출됨 |
| **중앙 관리자 패턴** | 초기화 순서를 직접 통제 |
| **ScriptableObject 활용** | 미리 정의된 정적 데이터 사용 |

---

## 3. 가장 안전한 방식: Init() 함수 분리

### 예제

```csharp
public class B : MonoBehaviour
{
    public string config;

    public void Init()
    {
        config = "B 초기화 완료";
        Debug.Log("B 초기화됨");
    }
}

public class A : MonoBehaviour
{
    public B b;

    public void Init()
    {
        Debug.Log("A: B의 설정 = " + b.config);
    }
}

public class InitController : MonoBehaviour
{
    public A a;
    public B b;

    void Start()
    {
        b.Init();   // B 먼저 초기화
        a.Init();   // 이후 A에서 B를 안전하게 참조
    }
}
```

- 명시적 순서로 초기화되므로 안정적
- 게임 시작 시점이나 로딩 후 자동 수행 가능

---

## 4. 보조 방법: Script Execution Order

### 설정 위치
- `Edit > Project Settings > Script Execution Order`

### 사용법
- B 스크립트를 A보다 위에 배치 → B의 `Awake()` / `Start()`가 먼저 실행됨

**주의:** 코드 외부 설정이므로 의존 관계를 파악하기 어려움

---

## 5. Awake() vs Start() 분리 활용

| 메서드 | 호출 시점 | 용도 |
|--------|------------|------|
| `Awake()` | 오브젝트 활성화 시 즉시 호출 | 자체 초기화, 외부 의존성 없음 |
| `Start()` | 프레임 시작 직전에 호출 | Unity 오브젝트 참조 필요 시 사용 |

### 예시
```csharp
public class B : MonoBehaviour
{
    public string config;

    void Awake()
    {
        config = "B에서 설정됨";
    }
}

public class A : MonoBehaviour
{
    public B b;

    void Start()
    {
        Debug.Log("B의 config: " + b.config);
    }
}
```

---

## 6. 중앙 관리자로 통제하는 방식

### 구조

```csharp
public class GameInitializer : MonoBehaviour
{
    public PlayerManager playerManager;
    public EnemyManager enemyManager;

    void Start()
    {
        playerManager.Init();
        enemyManager.Init();
    }
}
```

- 각 모듈의 `Init()`을 순서대로 호출해 초기화
- 대규모 프로젝트에서 유용

---

## 7. ScriptableObject로 의존성 분리

- ScriptableObject는 런타임 이전에 데이터를 정의할 수 있음
- 데이터를 미리 세팅해두고 참조만 하도록 하면 초기화 타이밍 문제 회피 가능

---

## 8. 요약

| 해결 방법 | 장점 | 사용 추천 상황 |
|------------|------|----------------|
| Init() 메서드 | 명시적, 안전함 | 대부분의 상황 |
| Script Execution Order | 간편함 | 소규모 테스트용 |
| Awake/Start 분리 | 간단한 순서 제어 | 내부 전용 초기화 |
| 중앙 초기화 | 전체 흐름 통제 가능 | 대규모 초기화 |
| ScriptableObject | 런타임 이전 정의 | 정적 데이터 참조 |

---

**초기화 순서를 명시적으로 제어하는 것이 버그를 예방하는 가장 좋은 방법입니다.**
