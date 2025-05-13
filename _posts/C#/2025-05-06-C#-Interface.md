---
title: "[C#] 인터페이스 문법 with Unity"
author: RePro
date: 2025-05-06 10:00:00 +0900
categories: [Programming, C#]
tags: [c#, unity]
---

## C# (Unity 기준) 인터페이스 문법 상세 설명

### 1. 인터페이스란?

* 메서드 시그니처(선언)만 정의
* 구현 내용은 클래스에서 담당
* 다중 상속 가능
* 객체 간 공통 동작 보장

> "이런 기능은 있어야 한다"는 규칙/약속만 제공

---

### 2. 기본 문법

```csharp
public interface IExample
{
    void DoSomething();
}
```

구현 클래스:

```csharp
public class ExampleClass : IExample
{
    public void DoSomething()
    {
        Debug.Log("DoSomething 실행됨");
    }
}
```

---

### 3. 인터페이스 특징

| 특징        | 설명                      |
| --------- | ----------------------- |
| 다중 상속 가능  | 클래스는 여러 인터페이스 구현 가능     |
| 접근 제한자 불가 | 메서드는 기본 public (명시 불필요) |
| 필드 정의 불가  | 변수 선언 불가능 (속성은 가능)      |
| 생성자 정의 불가 | 생성자 작성 불가               |

---

### 4. 다중 인터페이스 구현 예시

```csharp
public interface IDamageable
{
    void TakeDamage(int amount);
}

public interface IHealable
{
    void Heal(int amount);
}

public class Player : IDamageable, IHealable
{
    public void TakeDamage(int amount)
    {
        Debug.Log($"{amount} 데미지 받음");
    }

    public void Heal(int amount)
    {
        Debug.Log($"{amount} 회복함");
    }
}
```

---

### 5. Unity에서의 사용 예시 (다형성)

```csharp
public interface IDamageable
{
    void TakeDamage(int damage);
}

public class Enemy : MonoBehaviour, IDamageable
{
    public void TakeDamage(int damage)
    {
        Debug.Log($"Enemy: {damage} 피해 받음");
    }
}

public class DestructibleObject : MonoBehaviour, IDamageable
{
    public void TakeDamage(int damage)
    {
        Debug.Log($"파괴 오브젝트: {damage} 피해 받음");
    }
}

void Attack(GameObject target)
{
    IDamageable damageable = target.GetComponent<IDamageable>();
    if (damageable != null)
    {
        damageable.TakeDamage(10);
    }
}
```

→ `Enemy`와 `DestructibleObject` 모두 같은 방식으로 처리 가능 (다형성)

---

### 6. 속성 포함 가능

```csharp
public interface IHasHealth
{
    int Health { get; set; }
}

public class Enemy : MonoBehaviour, IHasHealth
{
    public int Health { get; set; } = 100;
}
```

→ 속성도 반드시 구현해야 함

---

### 7. 인터페이스 vs 추상 클래스

| 항목       | 인터페이스 | 추상 클래스 |
| -------- | ----- | ------ |
| 다중 상속    | O     | X      |
| 필드/변수    | X     | O      |
| 생성자      | X     | O      |
| 기본 구현 제공 | X     | O      |

---

### 8. 명시적 인터페이스 구현

```csharp
interface IWalk { void Move(); }
interface IRun { void Move(); }

public class Character : IWalk, IRun
{
    void IWalk.Move() { Debug.Log("걷기"); }
    void IRun.Move() { Debug.Log("달리기"); }
}
```

→ 같은 시그니처 충돌 시 구분해 구현 가능

---

### 9. Unity 기본 인터페이스 예시

| 인터페이스                | 역할        |
| -------------------- | --------- |
| IPointerClickHandler | UI 클릭 처리  |
| IBeginDragHandler    | 드래그 시작 처리 |
| IEndDragHandler      | 드래그 끝 처리  |
| IDragHandler         | 드래그 중 처리  |

Unity의 이벤트 시스템에서 인터페이스 기반 이벤트 처리에 사용됨.

---

### 10. 요약

* 인터페이스는 **규약(약속) 제공, 구현은 클래스 담당**
* **다중 상속 가능**, 메서드/속성 선언 가능
* **다형성, 컴포넌트 상호작용에 유용**
* Unity에서는 **이벤트 처리, 메시지 전송, 공통 동작 보장**에 자주 사용
