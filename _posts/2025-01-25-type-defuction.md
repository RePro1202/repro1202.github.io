---
title: C++ & C# 타입 추론(Type Deduction)
author: RePro
date: 2025-01-25 10:00:00 +0900
categories: [Programming]
tags: [c++, c#]
---


# C++ & C# 타입 추론(Type Deduction) 정리

## 1. 타입 추론(Type Deduction)이란?

타입 추론이란 **컴파일러가 변수나 표현식의 타입을 자동으로 결정하는 과정**을 의미한다. C++과 C#에서는 각각 `auto`, `decltype`, `var`, `dynamic` 등을 사용하여 타입을 추론할 수 있다.

---

## 2. C++의 타입 추론

### 2.1 `auto`를 이용한 타입 추론
C++11부터 `auto` 키워드를 사용하면 **컴파일러가 변수의 타입을 자동으로 결정**할 수 있다.

#### 🔹 기본 예제
```cpp
int x = 10;
auto y = x;  // y의 타입은 int로 추론됨
```

#### 🔹 `auto`를 이용한 함수 반환값
```cpp
auto add(int a, int b) {
    return a + b;  // 반환값의 타입을 자동 추론 (int)
}
```

### 2.2 `decltype`을 이용한 타입 추론
**`decltype`**은 **주어진 표현식(변수, 연산 결과 등)의 타입을 추론**할 때 사용된다.

#### 🔹 기본 예제
```cpp
int x = 10;
decltype(x) y = 20;  // y의 타입은 int로 추론됨
```

#### 🔹 연산 결과의 타입 추론
```cpp
int a = 5;
double b = 2.5;
decltype(a + b) result;  // result의 타입은 double
```

### 2.3 템플릿에서의 타입 추론
C++의 템플릿을 사용할 때 **컴파일러는 전달된 인자에 따라 템플릿의 타입을 자동으로 추론**한다.

#### 🔹 함수 템플릿에서 타입 추론
```cpp
template <typename T>
void print(T value) {
    std::cout << value << std::endl;
}

int main() {
    print(10);    // T는 int로 추론됨
    print(3.14);  // T는 double로 추론됨
    print("Hello"); // T는 const char*로 추론됨
}
```

### 2.4 `T&&`에서의 타입 추론 (완벽한 전달)
```cpp
template <typename T>
void func(T&& arg) {
    std::cout << typeid(arg).name() << std::endl;
}

int main() {
    int x = 10;
    func(x);   // T = int&, T&& = int&
    func(20);  // T = int,  T&& = int&&
}
```

| 입력 값 | `T`가 추론된 타입 | `T&&`로 변환된 타입 |
|---------|----------------|----------------|
| `func(x);` | `int&` | `int&` |
| `func(20);` | `int` | `int&&` |

---

## 3. C#의 타입 추론

### 3.1 `var`를 이용한 타입 추론
C#에서는 `var` 키워드를 사용하면 **변수의 타입을 자동으로 결정**할 수 있다.

#### 🔹 기본 예제
```csharp
var x = 10;       // x의 타입은 int로 추론됨
var y = "hello";  // y의 타입은 string으로 추론됨
var z = 3.14;     // z의 타입은 double로 추론됨
```

### 3.2 `dynamic`을 이용한 런타임 타입 결정
`dynamic` 키워드는 **런타임에 타입이 결정되는 변수**를 만들 때 사용된다.

```csharp
dynamic value = 10;
Console.WriteLine(value.GetType()); // 출력: System.Int32

value = "hello";  
Console.WriteLine(value.GetType()); // 출력: System.String
```

### 3.3 `typeof`를 이용한 타입 정보 가져오기
C#에서는 `typeof`를 사용하여 특정 타입 정보를 가져올 수 있다.

```csharp
Console.WriteLine(typeof(int));       // 출력: System.Int32
Console.WriteLine(typeof(string));    // 출력: System.String
```

### 3.4 `default`를 이용한 기본값 타입 추론
C#에서는 `default` 키워드를 사용하면 특정 타입의 기본값을 반환할 수 있다.

```csharp
int a = default;      // a는 0
bool b = default;     // b는 false
string s = default;   // s는 null
```

---

## 4. `var` vs `dynamic` vs `object` 비교

| 키워드 | 타입 결정 시점 | 타입 변경 가능 여부 | 성능 |
|--------|-------------|----------------|------|
| `var` | **컴파일 타임** | ❌ 불가능 | ✅ 빠름 |
| `dynamic` | **런타임** | ✅ 가능 | ❌ 느림 |
| `object` | **컴파일 타임** | ✅ 가능 (형변환 필요) | 중간 |

```csharp
var v = "hello";       // 컴파일 타임에 string으로 결정됨
dynamic d = "hello";   // 런타임에 string으로 결정됨
object o = "hello";    // object 타입이지만, 형변환 필요
```

---

## 5. C++의 `auto` vs C#의 `var`

| 기능 | C++ `auto` | C# `var` |
|------|-----------|-----------|
| **타입 추론 시점** | **컴파일 타임** | **컴파일 타임** |
| **형변환 가능 여부** | ❌ 불가능 | ❌ 불가능 |
| **사용 제한** | **템플릿과 함께 사용 가능** | **초기화 필수** |
| **함수 반환 타입** | ✅ 가능 | ❌ 불가능 |

```cpp
auto x = 10;  // C++에서 auto 사용
```
```csharp
var x = 10;   // C#에서 var 사용
```
- **둘 다 컴파일 타임에 타입이 결정됨**.
- C++의 `auto`는 **템플릿과 함께 사용 가능**, C#의 `var`는 **함수 반환 타입으로 사용할 수 없음**.

---

## 6. 결론

✔ **C++의 `auto` vs C#의 `var`** → **컴파일 타임 타입 추론, 반드시 초기화 필요**  
✔ **`decltype` (C++)** → **변수나 표현식의 타입을 추론**  
✔ **`dynamic` (C#)** → **런타임 타입 결정, 하지만 성능 저하 가능성 있음**  
✔ **`typeof` (C#)** → **타입 정보를 가져올 때 사용**  
✔ **`default` (C#)** → **타입의 기본값을 반환**  

🚀 **C++의 `auto`와 C#의 `var`는 비슷하지만, `dynamic`은 C++에 없는 기능!**

