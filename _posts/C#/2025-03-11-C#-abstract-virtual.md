---
title: "[C#] abstract와 virtual 키워드"
author: RePro
date: 2025-03-11 10:00:00 +0900
categories: [Programming, C#]
tags: [c#]
---

## C#에서 abstract와 virtual 키워드 설명

### 1. abstract 키워드

#### 의미

* **추상 메서드**: 구현 없이 선언만 존재
* 반드시 **자식 클래스에서 override 필요**
* **추상 클래스 안에서만 선언 가능**

#### 사용 목적

* 상속받는 클래스에 **구현을 강제**

#### 예시

```csharp
abstract class Animal
{
    public abstract void Speak();
}

class Dog : Animal
{
    public override void Speak()
    {
        Console.WriteLine("멍멍!");
    }
}
```

`Dog` 클래스는 **반드시 `Speak()` 메서드를 override** 해야 컴파일 가능.

---

### 2. virtual 키워드

#### 의미

* **구현이 있는 메서드**지만 **자식 클래스에서 override 가능**
* 자식 클래스에서 override하지 않으면 **부모 기본 구현 사용**

#### 사용 목적

* **기본 구현 제공 + 필요 시 override 허용**

#### 예시

```csharp
class Animal
{
    public virtual void Speak()
    {
        Console.WriteLine("동물이 소리를 냅니다.");
    }
}

class Dog : Animal
{
    public override void Speak()
    {
        Console.WriteLine("멍멍!");
    }
}
```

`Dog` 클래스는 `Speak()`를 override 했지만, **안 해도 기본 동작 사용 가능**.

---

### 3. 공통점과 차이점 비교

| 구분     | abstract        | virtual             |
| ------ | --------------- | ------------------- |
| 구현 여부  | ❌ 없음            | ✅ 있음                |
| 자식 클래스 | 반드시 override 필요 | 선택적으로 override 가능   |
| 선언 위치  | abstract 클래스 내부 | 일반 클래스 내부           |
| 목적     | 구현 강제           | 기본 구현 + override 허용 |

---

### 4. abstract class vs interface 차이

* **abstract class**: 일부 구현 + 일부 추상 메서드 가능
* **interface**: (C# 8.0 이전) 모든 메서드 구현 없음, (C# 8.0 이후) default 구현 가능

→ `abstract`는 **구현 강제 + 공통 로직 제공** 가능
→ `virtual`은 **기본 로직 제공 + 필요 시 override**

---

### 5. 실전 추천 사용법

| 상황                | 추천 키워드     |
| ----------------- | ---------- |
| 모든 자식 클래스가 반드시 구현 | `abstract` |
| 기본 동작 제공, 필요 시 수정 | `virtual`  |

---

### 6. 예제 비교

```csharp
abstract class Shape
{
    public abstract void Draw();
}

class Circle : Shape
{
    public override void Draw()
    {
        Console.WriteLine("원을 그립니다.");
    }
}

class BaseLogger
{
    public virtual void Log(string message)
    {
        Console.WriteLine("로그: " + message);
    }
}

class FileLogger : BaseLogger
{
    public override void Log(string message)
    {
        Console.WriteLine("파일에 기록: " + message);
    }
}
```

---

### 7. 요약

| 항목          | abstract | virtual       |
| ----------- | -------- | ------------- |
| 구현 존재 여부    | ❌ 없음     | ✅ 기본 구현 존재    |
| override 여부 | 필수       | 선택            |
| 목적          | 구현 강제    | 기본 제공 + 수정 허용 |

→ 규칙을 강제하고 싶으면 `abstract`, 기본 동작 제공 + 필요할 때만 수정 허용하려면 `virtual` 사용 권장.
