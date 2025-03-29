---
title: "[C#] 매개변수 키워드 정리 (`ref`, `out`, `in`, `params`)"
author: RePro
date: 2024-03-02 10:00:00 +0900
categories: [Programming, C#]
tags: [c#]
---

# C# 매개변수 키워드 정리 (`ref`, `out`, `in`, `params`)

C#에서 함수에 인자를 넘기는 방법에는 다양한 키워드가 있습니다.  
이 문서는 `ref`, `out`, `in`, `params`의 **개념, 차이점, 사용 예시, IL 내부 처리, Unity 활용 사례**까지 정리합니다.

---

## 1. 비교 요약표

| 키워드     | 용도                            | 호출 전 초기화 | 함수 내 초기화 | 수정 여부 | 설명 |
|------------|----------------------------------|----------------|----------------|------------|------|
| `ref`      | 값을 전달하고 수정도 허용         | ✅ 필요         | ❌ 선택         | ✅ 가능     | 호출한 쪽 변수 값을 함수에서 수정 가능 |
| `out`      | 함수에서 값을 반환해 넘김         | ❌ 불필요       | ✅ 반드시       | ✅ 가능     | 다중 반환 값처럼 사용 |
| `in`       | 읽기 전용 참조 전달               | ✅ 필요         | ❌ 금지         | ❌ 불가능   | 구조체를 복사 없이 넘기고 변경 못 하게 |
| `params`   | 가변 인자                         | ❌ 불필요       | ❌ 선택         | ✅ 가능     | 배열처럼 여러 개 인자를 넘길 수 있음 |

---

## 2. 예제 코드

### `ref` 예제
```csharp
void Double(ref int x) {
    x *= 2;
}
int a = 5;
Double(ref a);
Console.WriteLine(a); // 10
```

### `out` 예제
```csharp
bool TryDivide(int a, int b, out int result) {
    if (b == 0) {
        result = 0;
        return false;
    }
    result = a / b;
    return true;
}
```

### `in` 예제
```csharp
void PrintLength(in Vector3 pos) {
    Console.WriteLine(pos.magnitude);
    // pos.x = 1; ❌ 컴파일 에러
}
```

### `params` 예제
```csharp
void PrintAll(params int[] numbers) {
    foreach (var n in numbers)
        Console.WriteLine(n);
}
PrintAll(1, 2, 3, 4); // 가변 인자 전달
```

---

## 3. IL (Intermediate Language) 상의 차이점

| 키워드 | IL 표현 | 설명 |
|--------|---------|------|
| `ref`  | `&` 접미사 (`int32&`) | 값의 참조 전달 |
| `out`  | `&` 접미사 (`int32&`) | `ref`와 동일하나, **호출 시 초기화 여부 검사 차이** |
| `in`   | `in` 접두사 + `&` 접미사 | 읽기 전용으로 전달, C# 7.2 이상 |
| `params` | 배열로 처리됨 (`int32[]`) | 컴파일러가 자동으로 배열로 래핑 |

즉, **`ref`와 `out`은 IL에서는 동일한 참조 전달**로 처리되지만, C# 컴파일러 수준에서 **초기화 여부 및 사용 제한을 다르게 검사**합니다.

---

## 4. Unity 및 게임 개발에서의 활용 예

| 키워드 | Unity 사용 예시 | 설명 |
|--------|------------------|------|
| `ref`  | 드물게 직접 수정 함수 | `ModifyStats(ref playerData)` 같은 구조 |
| `out`  | `Physics.Raycast()` | 히트 정보 반환: `out RaycastHit hit` |
| `in`   | 성능 최적화용 구조체 전달 | `in Quaternion rot`, `in Vector3` 등 |
| `params` | 디버그 함수 유틸 | `Log(params object[] messages)` 등 |

### 대표 사례: `Physics.Raycast`

```csharp
Ray ray = new Ray(origin, direction);
RaycastHit hitInfo;
if (Physics.Raycast(ray, out hitInfo)) {
    Debug.Log(hitInfo.point);
}
```

### 대표 사례: `TryGetComponent`

```csharp
if (TryGetComponent(out Rigidbody rb)) {
    rb.AddForce(Vector3.up * 5);
}
```

---

## 5. 요약 정리

| 상황 | 추천 키워드 |
|------|--------------|
| 값을 수정하고 싶을 때 | `ref` |
| 값을 반환하고 싶을 때 | `out` |
| 읽기 전용 성능 최적화 | `in` |
| 유동적인 수의 인자 | `params` |

---

## 참고

- `ref`와 `out`은 값 타입 뿐만 아니라 참조 타입에도 사용 가능
- 구조체는 복사 비용이 크므로 `in`, `ref`, `out`으로 참조 전달이 일반적
- `params`는 **오버로딩 주의** (가장 마지막 매개변수에만 사용 가능)



---

## 6. `params` 오버로딩 시 주의사항

### 왜 주의가 필요한가?

`params`는 가변 인자를 배열처럼 받기 때문에,  
**비슷한 시그니처를 가진 오버로드가 존재하면 컴파일러가 혼동할 수 있습니다.**

### 모호한 오버로딩 예시

```csharp
void Print(string message) {
    Console.WriteLine("Single string: " + message);
}

void Print(params string[] messages) {
    Console.WriteLine("Params: " + string.Join(",", messages));
}

Print("Hello"); // 어떤 메서드가 호출될까? — 모호!
```

- `"Hello"`는 `string` 1개로도 맞고, `params string[]`에도 맞음
- 컴파일러가 모호하다고 판단할 수 있음

---

### C# 컴파일러의 선택 우선순위

1. **정확히 일치하는 시그니처**가 있으면 그걸 선택
2. 그렇지 않으면 `params`로 fallback
3. 모호할 경우 컴파일 에러

---

### 안전한 오버로딩 방법

```csharp
void Log(string label, params string[] messages) { ... }
void Log(params string[] messages) { ... }
```

→ 명확히 구분되는 시그니처를 사용하여 모호성 방지

---
