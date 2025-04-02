---
title: "[C++] null pointer"
author: RePro
date: 2024-10-01 10:00:00 +0900
categories: [Programming, C++]
tags: [c++, unreal]
---

# C++에서 null pointer exception vs undefined behavior

C++에서는 포인터를 잘못 사용했을 때 두 가지 주요 문제가 발생할 수 있습니다: **null pointer exception**과 **undefined behavior**. 언리얼 엔진은 C++ 기반이지만, 자체적으로 안정성을 높이기 위한 여러 장치를 제공합니다. 이 문서에서는 두 개념의 차이와 언리얼에서의 대처 방식까지 함께 정리합니다.

---

## 1. null pointer exception (널 포인터 예외)

### 정의
널 포인터 예외는 Java, C#, Python 등의 메모리 안전 언어에서 **널 포인터를 참조할 경우 자동으로 발생하는 런타임 예외**입니다.

```java
String s = null;
s.length();  // NullPointerException 발생
```

### C++에서는 해당 개념이 없음
- C++은 널 포인터 참조 시 예외를 발생시키지 않음
- 대신 **undefined behavior**로 이어짐

---

## 2. undefined behavior (정의되지 않은 동작)

### 정의
C++에서 스펙상 정의되지 않은 동작을 의미합니다. 컴파일러나 OS가 이를 제어하지 않으며, 결과는 예측 불가입니다.

### 예시 (C++)
```cpp
int* p = nullptr;
*p = 10;  // undefined behavior
```

### 대표 사례
| 상황 | 설명 |
|------|------|
| 널 포인터 역참조 | `*nullptr` |
| 해제된 메모리 접근 | `delete p; *p = 5;` |
| 배열 범위 초과 | `a[10] = 1;` (배열이 5개일 때) |
| 초기화 안된 변수 사용 | `int x; std::cout << x;` |

---

## 3. 차이점 요약

| 구분 | null pointer exception | undefined behavior |
|------|------------------------|--------------------|
| 발생 시점 | 런타임 예외 | 예측 불가한 시점 (컴파일러도 모름) |
| 처리 | 운영체제가 예외 발생 | 아무 동작도 보장 안 됨 |
| 언어 | Java, C# 등 | C, C++ |
| 디버깅 난이도 | 쉬움 | 어려움 |

---

## 4. 언리얼 엔진에서의 처리 방식

언리얼은 C++ 기반이지만, 널 포인터 접근에 대한 **안전 장치와 디버깅 도구**를 제공합니다.

### `check()` 매크로
- 조건이 false면 **즉시 크래시** 발생 (개발 중 확인 용도)
```cpp
check(MyActor != nullptr);
MyActor->DoSomething();
```

### `ensure()` 매크로
- 조건이 false면 **로그 출력** 후 **실행 계속** (비파괴적)
```cpp
if (ensure(MyActor != nullptr)) {
    MyActor->DoSomething();
}
```

### 스마트 포인터 사용
| 타입 | 설명 |
|------|------|
| `TWeakObjectPtr<T>` | 가비지 컬렉션된 객체를 자동으로 무효화함 |
| `TSharedPtr<T>` / `TSharedRef<T>` | 참조 카운트 기반 메모리 관리 |
| `TUniquePtr<T>` | 단독 소유 포인터 |

```cpp
TWeakObjectPtr<AActor> SafeActor = SomeActor;
if (SafeActor.IsValid()) {
    SafeActor->Destroy();
}
```

### 블루프린트는 Null 보호 내장
- 블루프린트에서는 널 접근 시 자동으로 경고 또는 무시됨 → 크래시 방지

---

## 5. 최종 요약

| 항목 | C++ | 언리얼 엔진 |
|------|-----|--------------|
| 널 포인터 예외 | 없음 | 없음 (하지만 check/ensure로 대체 가능) |
| 널 포인터 역참조 | undefined behavior | check/ensure, 스마트 포인터로 방어 가능 |
| 자동 검사 | 없음 | 블루프린트는 내장 체크 제공 |
| 안정성 도구 | 수동으로 유효성 검사 필요 | `IsValid()`, `ensure()`, `TWeakObjectPtr` 등 지원 |

C++의 강력함은 포인터를 직접 다룰 수 있는 유연성에 있지만, 그만큼 위험성도 큽니다. 언리얼 엔진은 그 위에 안전망을 일부 제공하며, 개발자가 이를 잘 활용하면 안정적인 코드 작성을 도울 수 있습니다.

