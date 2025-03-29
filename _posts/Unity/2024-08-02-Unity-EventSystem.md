---
title: "[Unity] 이벤트 시스템"
author: RePro
date: 2024-08-02 10:00:00 +0900
categories: [Programming, Unity]
tags: [unity, c#]
---

# Unity의 이벤트 시스템
---

## 1. 각 용어 개요

| 용어 | 설명 |
|------|------|
| **Action** | 반환값이 없는 메서드를 참조하는 델리게이트 타입 |
| **EventHandler** | 이벤트 발생 시 호출할 수 있는 .NET 표준 델리게이트 |
| **Delegate** | 사용자가 직접 정의하는 메서드 참조 타입 |
| **UnityEvent** | Unity 전용의 직렬화 가능한 이벤트 시스템으로, 인스펙터에서 이벤트를 연결할 수 있음 |

---

## 2. Action (System.Action)

### 정의
- `System.Action`은 반환값이 없는 메서드를 가리킬 수 있는 **제네릭 델리게이트 타입**입니다.
- `Action`은 **매개변수 없이 반환값도 없는 메서드**를 참조할 수 있고,
- `Action<T>`, `Action<T1, T2, ..., T16>` 형태로 최대 16개의 매개변수를 지원합니다.

### 사용 예
```csharp
Action sayHello = () => Console.WriteLine("Hello");
sayHello();  // 출력: Hello

Action<int> printNumber = (n) => Console.WriteLine(n);
printNumber(100);  // 출력: 100
```

### 특징
- 매우 간결하고 빠름
- null 체크 후 호출 (`?.Invoke()`)
- Unity와 궁합이 좋고 자주 사용됨

---

## 3. EventHandler

### 정의
- .NET에서 표준으로 제공하는 이벤트 델리게이트
- 시그니처는 `void(object sender, EventArgs e)`로 고정됨
- `sender`는 이벤트를 발생시킨 객체, `e`는 이벤트 데이터를 포함

### 예시
```csharp
public event EventHandler OnDeath;

void Die() {
    OnDeath?.Invoke(this, EventArgs.Empty);
}
```

### EventHandler<T> 사용 예
```csharp
public class DamageEventArgs : EventArgs {
    public int DamageAmount { get; }
    public DamageEventArgs(int dmg) => DamageAmount = dmg;
}

public event EventHandler<DamageEventArgs> OnDamaged;

void TakeDamage(int dmg) {
    OnDamaged?.Invoke(this, new DamageEventArgs(dmg));
}
```

### 특징
- 이벤트 구조를 정형화하여 일관된 설계 가능
- 재사용성과 확장성이 뛰어남
- WPF, WinForms, 외부 .NET 라이브러리에서도 표준으로 사용

---

## 4. Delegate

### 정의
- 사용자가 직접 정의하는 메서드 참조 타입
- C#에서 `delegate` 키워드로 정의

```csharp
public delegate void MyDelegate(string message);
public MyDelegate onMessage;

onMessage += (msg) => Console.WriteLine(msg);
```

### 특징
- 가장 기본적인 델리게이트 방식
- `Action`, `Func`, `EventHandler` 모두 결국 delegate 기반
- 자유롭게 시그니처를 정의할 수 있음

---

## 5. UnityEvent

### 정의
- `UnityEngine.Events.UnityEvent`
- Unity 에디터에서 **인스펙터를 통해 이벤트를 연결**할 수 있는 직렬화 가능 이벤트 시스템

### 사용 예
```csharp
[SerializeField] private UnityEvent onComplete;

void Complete() {
    onComplete.Invoke();
}
```

### 특징
- 비프로그래머도 Unity 에디터에서 이벤트 연결 가능
- 애니메이션 이벤트, UI 버튼 등과 자연스럽게 연동
- 성능은 덜하지만 유지보수성과 유연성이 뛰어남

---

## 6. 비교표

| 항목 | Action | EventHandler | Delegate | UnityEvent |
|------|--------|--------------|----------|------------|
| 반환값 | void | void | 정의에 따름 | void |
| 매개변수 | 자유 | (object, EventArgs) | 자유 | 제한적 |
| 설계 목적 | 간단한 콜백 | .NET 이벤트 처리 | 사용자 정의 콜백 | Unity 에디터 연결 |
| 사용 대상 | 코드 전용 | .NET 앱, 라이브러리 | 확장 가능 구조 | 인스펙터 이벤트 |
| 직렬화 | 불가 | 불가 | 불가 | 가능 |
| 인스펙터 연결 | 불가 | 불가 | 불가 | 가능 |
| Unity 친화도 | 높음 | 낮음 | 중간 | 매우 높음 |
| 성능 | 매우 높음 | 높음 | 높음 | 상대적으로 낮음 |

---

## 7. 어떤 것을 언제 사용해야 하나?

| 상황 | 추천 방식 |
|------|------------|
| 간단한 콜백 처리 | `Action` |
| .NET 스타일 이벤트 구현 | `EventHandler`, `EventHandler<T>` |
| 사용자 정의 콜백 시그니처 | `delegate` |
| 디자이너가 이벤트를 연결해야 함 | `UnityEvent` |

---

## 8. 요약

- `Action`: 간단하고 가벼운 코드 콜백 처리에 최적. Unity 내부에서도 자주 사용
- `EventHandler`: 정형화된 .NET 이벤트 시스템. 확장성과 일관성이 필요할 때 적합
- `delegate`: 특별한 시그니처가 필요한 경우 직접 정의 가능
- `UnityEvent`: Unity 인스펙터를 통한 연결이 필요할 때 필수적
