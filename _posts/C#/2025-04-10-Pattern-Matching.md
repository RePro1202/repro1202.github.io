---
title: "[C#] 패턴 매칭"
author: RePro
date: 2025-04-10 10:00:00 +0900
categories: [Programming, C#]
tags: [c#]
---


## C# 패턴 매칭 정리

### 1. 개요

패턴 매칭(Pattern Matching)은 객체의 타입, 값, 구조 등을 기반으로 분기 로직을 작성할 수 있는 기능이다. 기존 `if`, `switch` 문보다 더 명확하고 간결하게 조건 분기를 구성할 수 있다.  
C# 7.0부터 도입되었으며, 이후 버전에서 점차 확장되었다.

---

### 2. 주요 패턴 종류

#### 2.1 `is` 패턴 (C# 7.0+)

```csharp
if (obj is string s)
{
    Debug.Log(s.Length);
}
```

- 타입 검사와 형변환을 동시에 수행

---

#### 2.2 `switch` 패턴 (C# 7.0+)

```csharp
switch (animal)
{
    case Dog d:
        Debug.Log("멍멍이");
        break;
    case Cat c:
        Debug.Log("야옹이");
        break;
}
```

- 타입 기반 분기 및 암시적 형변환 가능

---

#### 2.3 `when` 조건 패턴

```csharp
switch (player)
{
    case Warrior w when w.Level > 10:
        Debug.Log("고렙 전사");
        break;
    case Warrior w:
        Debug.Log("뉴비 전사");
        break;
}
```

- 타입 + 조건 조합 분기

---

#### 2.4 `switch` 식 (C# 8.0+)

```csharp
string msg = player switch
{
    Warrior w => $"전사 ({w.Level})",
    Mage m => $"마법사 ({m.Mana})",
    _ => "알 수 없음"
};
```

- 간결한 식 기반의 조건 분기

---

#### 2.5 속성 패턴 (C# 8.0+)

```csharp
if (player is Warrior { Level: > 10 })
{
    Debug.Log("10레벨 초과 전사");
}
```

- 속성 값 기반 분기

---

#### 2.6 튜플 패턴 (C# 8.0+)

```csharp
(string action, int amount) = ("heal", 20);

switch ((action, amount))
{
    case ("heal", > 0):
        Debug.Log($"회복 {amount}");
        break;
}
```

- 여러 값의 조합 기반 분기

---

#### 2.7 리스트 패턴 (C# 11.0+)

```csharp
int[] arr = { 1, 2, 3 };
if (arr is [1, 2, 3])
{
    Debug.Log("정확히 [1,2,3]입니다.");
}
```

- 배열/리스트 형태와 값을 비교

---

### 3. 예제

#### 상황: 다양한 스킬 오브젝트에 따라 다른 효과 처리

```csharp
public abstract class Skill {}
public class Fireball : Skill
{
    public float Damage { get; set; }
}
public class Heal : Skill
{
    public float Amount { get; set; }
}

public class SkillHandler : MonoBehaviour
{
    public void UseSkill(Skill skill)
    {
        switch (skill)
        {
            case Fireball f:
                Debug.Log($"불덩이 데미지: {f.Damage}");
                // 이펙트, 사운드 등 처리
                break;

            case Heal h when h.Amount >= 50:
                Debug.Log($"강력한 회복: {h.Amount}");
                // 힐 이펙트 강화
                break;

            case Heal h:
                Debug.Log($"일반 회복: {h.Amount}");
                break;

            default:
                Debug.Log("알 수 없는 스킬");
                break;
        }
    }
}
```

- `switch` + 타입 패턴 + `when` 조건 패턴 조합
- `Skill` 클래스를 상속하는 다양한 타입을 처리
- 유지보수성과 가독성 향상

---

### 4. 버전별 지원 정리

| C# 버전 | 지원 기능                            |
|---------|-------------------------------------|
| 7.0     | `is` 패턴, `switch` 타입/조건 패턴     |
| 8.0     | `switch` 식, 속성 패턴, 튜플 패턴      |
| 9.0     | 논리 패턴 (`and`, `or`), 관계 패턴      |
| 11.0    | 리스트 패턴                           |

> Unity는 기본적으로 사용하는 **.NET 및 C# 버전에 따라 일부 최신 패턴이 제한될 수 있음**  
> Unity 2023 LTS 기준: **C# 10** 지원 (C# 11 일부 지원은 실험적)

