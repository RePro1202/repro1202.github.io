---
title: "[C#] 멤버 변수 접근 방식"
author: RePro
date: 2025-05-09 10:00:00 +0900
categories: [Programming, C#]
tags: [c#]
---

# C#에서 멤버 변수 접근 방식 비교

## 1. 주요 방식 개요

| 방식                    | 문법 예시                            | 설명                | 대표 사용 상황           |
| --------------------- | -------------------------------- | ----------------- | ------------------ |
| 필드(field)             | `public int hp;`                 | 아무런 제어 없이 공개됨     | 단순 POCO 모델, DTO    |
| 프로퍼티(property)        | `public int HP { get; set; }`    | 캡슐화 + 자동 접근 제어 가능 | 대부분의 C# 클래스        |
| Get/Set 메서드           | `public int GetHP()` / `SetHP()` | 명시적 캡슐화, Java 스타일 | 특별한 검증/로직 포함 시     |
| 람다식 프로퍼티              | `public int HP => hp;`           | 읽기 전용 단축식         | 내부 계산 기반, 읽기용      |
| Expression-bodied 메서드 | `public int GetHP() => hp;`      | 간단한 getter 메서드 표현 | lightweight getter |
| init-only 프로퍼티        | `public int HP { get; init; }`   | 초기화 후 수정 불가       | Immutable 패턴       |

---

## 2. 방식별 설명

### 2.1 필드 (Field)

```csharp
public class Enemy
{
    public int hp;
}
```

* 외부에서 자유롭게 읽기/쓰기 가능
* 캡슐화 없음 → 예외 상황 제한 불가

---

### 2.2 프로퍼티 (Property)

```csharp
private int hp;
public int HP
{
    get => hp;
    set => hp = Math.Max(0, value); // 음수 방지
}
```

**자동 구현 프로퍼티:**

```csharp
public int HP { get; set; }
```

---

### 2.3 Get/Set 메서드

```csharp
private int hp;
public int GetHP() => hp;
public void SetHP(int value)
{
    hp = Math.Max(0, value);
}
```

* Java 스타일 인터페이스
* API처럼 동작시 명시적 처리 가능

---

### 2.4 람다식 (표현식 기반 프로퍼티)

```csharp
private int hp;
public int HP => hp;
```

또는 계산 기반:

```csharp
public int HealthPercent => hp / maxHp;
```

---

### 2.5 init-only 프로퍼티 (C# 9 이상)

```csharp
public int MaxHP { get; init; }
```

* 생성자나 객체 초기화 시점에만 설정 가능

---

### 2.6 접근자 제어자 조합

```csharp
public int HP { get; private set; } // 외부는 읽기만, 내부는 쓰기 가능
public int HP { get; } = 100;       // 읽기 전용 프로퍼티
```

---

## 3. 용도별 추천

| 상황                       | 추천 방식                             |
| ------------------------ | --------------------------------- |
| 단순 데이터 전달, Unity 인스펙터 노출 | public field 또는 \[SerializeField] |
| 외부 접근 제한 + 내부 캡슐화        | property                          |
| 내부 계산 기반 읽기 전용           | expression-bodied property        |
| 외부에서 초기값만 설정 가능하게        | init-only property                |
| 명확한 함수적 API로 처리하고 싶을 때   | Get/Set 메서드                       |

---

## 4. 결론

* 대부분의 경우에는 프로퍼티 (`get; set;`)를 사용하는 것이 C# 스타일에 가장 적합
* Unity에서는 public field 또는 \[SerializeField] + private 필드 + 프로퍼티 조합이 실무에서 가장 흔함
* Get/Set 메서드는 구체적인 로직이나 의미 있는 네이밍이 필요한 경우에만 사용
* 람다식 표현식은 읽기 전용, 계산 기반 속성에 적합
