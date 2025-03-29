---
title: "[Unity] 스크립트 종류"
author: RePro
date: 2024-07-15 10:00:00 +0900
categories: [Programming, Unity]
tags: [unity]
---

# Unity 스크립트의 종류

Unity에서 스크립트를 작성할 때 `MonoBehaviour`를 상속할지, 일반 클래스로 만들지, 혹은 `ScriptableObject`를 활용할지는 그 클래스의 **역할과 목적**에 따라 달라집니다. 이 문서는 그 선택 기준과 각각의 구조에 대한 이해를 돕기 위한 가이드입니다.

---

## 1. MonoBehaviour란?

`MonoBehaviour`는 Unity의 **컴포넌트 시스템의 핵심 클래스**로, Unity에서 오브젝트에 부착할 수 있는 스크립트를 만들 때 반드시 상속해야 합니다. 이를 통해 Unity의 생명주기 이벤트 메서드들을 사용할 수 있습니다.

### 주요 특징
- GameObject에 부착 가능
- `Start()`, `Update()`, `OnTriggerEnter()` 등 Unity 생명주기 메서드 사용 가능
- `transform`, `gameObject`, `enabled` 등의 Unity 속성 접근 가능
- Unity 인스펙터에서 필드 노출 및 설정 가능

---

## 2. 일반 클래스란?

C#의 일반 클래스는 `MonoBehaviour`를 상속하지 않으며, Unity의 컴포넌트 시스템과는 무관하게 동작합니다. 로직이나 데이터 구조를 처리하는 **순수한 코드 모듈**로 활용됩니다.

### 언제 사용하는가?

| 조건 | 설명 |
|------|------|
| 순수 로직 또는 데이터 전용 | 알고리즘, 계산, 상태 저장 등 |
| 직접 인스턴스 생성 필요 | `new MyClass()`로 생성 |
| Unity 기능 필요 없음 | `transform`, `Update()` 등 미사용 |
| 테스트, 재사용 용이 | 유닛 테스트 및 모듈화 용이 |

### 예시

```csharp
public class HealthSystem
{
    private int health;
    public void TakeDamage(int dmg) => health -= dmg;
}
```

- Unity 오브젝트에 의존하지 않고 독립적으로 동작함

---

## 3. ScriptableObject란?

`ScriptableObject`는 Unity의 특수한 클래스 타입으로, **에셋 형태로 데이터를 저장하고 공유**할 수 있게 해줍니다. 게임 설정, 스킬 정보, 아이템 데이터 등 재사용 가능한 정적 데이터 저장에 적합합니다.

### 주요 특징
- GameObject에 붙일 수 없음 (MonoBehaviour와 다름)
- 에디터에서 인스펙터로 관리 가능한 에셋으로 저장됨
- **인스턴스를 new로 만들 수 없음**, 대신 `ScriptableObject.CreateInstance<T>()` 또는 `CreateAssetMenu`를 사용
- 런타임과 에디터 모두에서 안전하게 데이터 공유 가능

### 예시

```csharp
[CreateAssetMenu(menuName = "MyGame/SkillData")]
public class SkillData : ScriptableObject
{
    public string skillName;
    public int damage;
}
```

- 프로젝트 폴더에 에셋으로 저장되어 여러 오브젝트가 참조 가능

---

## 4. 비교 요약표

| 항목 | MonoBehaviour | ScriptableObject | 일반 클래스 |
|------|---------------|------------------|--------------|
| GameObject에 부착 | O | ✖ | ✖ |
| 에셋으로 저장 | ✖ | O | ✖ |
| 직접 생성 (new) | ✖ | CreateInstance | O |
| Unity 생명주기 사용 | O | ✖ | ✖ |
| 인스펙터 노출 | O | O | ✖ |
| 재사용성 | 낮음 | 높음 | 높음 |
| 테스트 용이성 | 낮음 | 보통 | 높음 |
| 용도 | 씬 내 동적 행동 | 데이터 보관/공유 | 계산/로직 분리 |

---

## 5. 언제 어떤 것을 선택할까?

| 목적 | 추천 타입 | 이유 |
|------|------------|------|
| 플레이어 이동, 애니메이션 처리 | MonoBehaviour | 프레임별 동작 처리 및 오브젝트 부착 필요 |
| 체력 시스템, 데미지 계산 등 로직 | 일반 클래스 | 단위 테스트 가능, 구조적 관리 |
| 아이템, 스킬, 환경 설정 데이터 | ScriptableObject | 에셋화 가능, 재사용과 공유 편리 |
| UI 버튼 이벤트 처리 | MonoBehaviour | OnClick 이벤트 처리 등 |
| 글로벌 게임 설정 또는 상태 공유 | ScriptableObject | 싱글톤 대체 가능, 직렬화 지원 |

---

## 6. 전체 요약

- **MonoBehaviour**는 Unity에서 오브젝트에 **붙여서 동작할 스크립트**에 적합하며, 생명주기 함수 및 Unity 오브젝트 기능을 사용해야 할 때 선택합니다.
- **일반 클래스**는 Unity에 의존하지 않는 **독립적인 로직, 알고리즘, 데이터 처리**용으로, 생성이 자유롭고 테스트가 쉽습니다.
- **ScriptableObject**는 **재사용 가능한 에셋 형태의 데이터**를 관리할 때 적합하며, 여러 컴포넌트 간 데이터를 공유하거나 에디터 기반 설계를 도울 때 효과적입니다.
