---
title: "[Unity] 생성자"
author: RePro
date: 2024-07-16 10:00:00 +0900
categories: [Programming, Unity]
tags: [unity]
---

# Unity에서 생성자 사용

---

## 1. 생성자란?

- 생성자(Constructor)는 클래스의 인스턴스가 생성될 때 호출되는 특별한 메서드입니다.
- 일반적으로 필드 초기화, 기본 설정 작업을 수행합니다.

하지만 Unity에서는 클래스의 종류에 따라 **생성자 사용 방식이 다릅니다.**

---

## 2. MonoBehaviour에서의 생성자 사용

### 사용 가능 여부

| 항목 | 가능 여부 |
|------|-----------|
| 생성자 선언 | 가능 |
| new 키워드로 생성 | ❌ (AddComponent 사용해야 함) |
| 생성자 내 Unity 엔진 객체 접근 | ❌ (gameObject, transform 등 사용 불가) |
| 내부 필드 초기화 | 가능 (안전) |

### 예시

```csharp
public class MyComponent : MonoBehaviour
{
    private string label;

    public MyComponent()
    {
        label = "[MyComponent]";
        // gameObject.name = "Fail"; // ❌ gameObject는 null 상태
    }

    void Awake()
    {
        Debug.Log(label + " initialized");
    }
}
```

### 주의사항
- 생성자 내에서 Unity 엔진 관련 속성 사용은 불가
- 반드시 `AddComponent<T>()`로 인스턴스화해야 함

---

## 3. ScriptableObject에서의 생성자 사용

### 사용 가능 여부

| 항목 | 가능 여부 |
|------|-----------|
| 생성자 선언 | 가능 (권장 안 함) |
| new 키워드로 생성 | ❌ (CreateInstance 사용해야 함) |
| 필드 초기화 | 제한적으로 가능 |
| Unity 속성 접근 | 대부분 불가 (에셋 생성 시점엔 엔진 컨텍스트 없음) |

### 안전한 생성 방법

```csharp
[CreateAssetMenu]
public class SkillData : ScriptableObject
{
    public string skillName;

    public static SkillData Create(string name)
    {
        var data = CreateInstance<SkillData>();
        data.skillName = name;
        return data;
    }
}
```

- 생성자보다는 정적 팩토리 메서드를 통해 초기화하는 패턴이 일반적

---

## 4. 일반 클래스에서의 생성자 사용

| 항목 | 가능 여부 |
|------|-----------|
| 생성자 선언 | ✅ 자유롭게 사용 가능 |
| new 키워드로 생성 | ✅ |
| Unity 엔진 요소 접근 | ❌ (MonoBehaviour 아님) |
| 필드 초기화 | ✅ |

### 예시

```csharp
public class HealthSystem
{
    private int health;

    public HealthSystem(int maxHealth)
    {
        health = maxHealth;
    }

    public void TakeDamage(int dmg) => health -= dmg;
}
```

- 일반적인 C# 생성자 활용 가능
- 가장 유연하고 예측 가능

---

## 5. 비교 요약

| 항목 | MonoBehaviour | ScriptableObject | 일반 클래스 |
|------|---------------|------------------|--------------|
| 생성자 선언 | 가능 (주의 필요) | 가능 (비추천) | 자유 |
| new 생성 가능 | ✖ | ✖ (CreateInstance 사용) | O |
| Unity 객체 접근 | ✖ (생성자 시점엔 불가) | ✖ | ✖ |
| 생성자에서 초기화 권장 방식 | 필드 초기화 정도만 | 팩토리 메서드 활용 | 생성자 직접 활용 |
| 초기화 대안 | Awake(), Start() | OnEnable() 등 | 생성자 |

---

## 6. 언제 어떤 방식?

| 목적 | 추천 |
|------|--------|
| Unity 컴포넌트 기능 포함 | MonoBehaviour + Awake |
| 데이터 에셋 또는 공유 데이터 | ScriptableObject + CreateInstance |
| 순수 로직, 계산용 | 일반 클래스 + 생성자 |

---

## 7. 결론

- MonoBehaviour는 생성자를 쓸 수 있지만, **Unity 오브젝트에 접근하면 안 됩니다.**
- ScriptableObject도 **생성자보단 팩토리 패턴이 안전**합니다.
- 일반 클래스는 **가장 자유롭게 생성자 활용**이 가능하며, 테스트와 구조화에 적합합니다.
