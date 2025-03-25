---
title: lvalue, rvalue 그리고 for(auto) 반복문
author: RePro
date: 2025-02-24 10:00:00 +0900
categories: [Programming, C++]
tags: [c++]
---

# C++ lvalue, rvalue 그리고 for(auto) 반복문

## 1. lvalue와 rvalue 개념

### 1.1 lvalue (left value)
**"메모리에 이름이 있고, 주소를 가질 수 있는 값"**

#### lvalue 특징
- **변수처럼 메모리에 저장된 값**
- **주소를 가질 수 있음** (`&` 연산자로 참조 가능)
- **대입문의 왼쪽(lvalue = rvalue)에서 사용 가능**

#### lvalue 예제
```cpp
int x = 10;   // x는 lvalue (메모리에 저장됨)
int y = x;    // x는 좌변에서 사용 가능 (lvalue)
int* p = &x;  // x는 주소를 가질 수 있음
```

---

### 1.2 rvalue (right value)
**"임시로 생성되는 값, 주소를 가질 수 없는 값"**

#### rvalue 특징
- **임시 값 (메모리에 저장되지 않음)**
- **주소를 가질 수 없음** (`&` 연산자로 주소를 얻을 수 없음)
- **대입문의 오른쪽에서만 사용 가능 (`lvalue = rvalue`)**

#### rvalue 예제
```cpp
int x = 10; 
int y = x + 5;  // (x + 5)는 rvalue (연산 결과로 생성된 임시 값)
int* p = &(x + 5); // ❌ 오류! rvalue는 주소를 가질 수 없음
```

---

### 1.3 lvalue와 rvalue 비교

| 구분 | lvalue | rvalue |
|------|-----------------|------------------|
| **메모리 위치** | **메모리에 저장됨** | **임시 값 (메모리에 없음)** |
| **주소 (& 연산자 가능 여부)** | ✅ 가능 (`&x`) | ❌ 불가능 (`&(x + 5)`) |
| **대입 연산에서의 위치** | 좌변에서 사용 가능 | 우변에서 사용 가능 |
| **예제** | `int x = 10;` | `int y = x + 5;` |

---

## 2. lvalue와 rvalue 참조 (`&`, `&&`)

#### (1) lvalue 참조 (`&`)
```cpp
int a = 10;
int& ref = a;  // ✅ lvalue 참조 가능
ref = 20;      // a의 값이 20으로 변경됨
```

#### (2) rvalue 참조 (`&&`)
```cpp
int&& r = 10; // ✅ rvalue 참조 가능
r = 20;       // r에 값 변경 가능
```

---

## 3. for(auto ) 반복문에서의 복사와 참조

### 3.1 기본 동작: 값 복사 (Copy)
```cpp
for (auto num : vec) {  // 복사됨
    num += 10;  // vec의 원본 데이터는 변경되지 않음
}
```
- `num`은 `vec`의 요소를 **복사한 값**이므로, **원본 데이터(vec)는 변경되지 않음**.

---

### 3.2 참조(`&`)를 사용하여 원본 수정
```cpp
for (auto& num : vec) {  // 참조 사용
    num += 10;  // 원본 vec가 변경됨
}
```
- `auto& num`을 사용하면 **vec의 원본 데이터를 직접 수정할 수 있음**.

---

### 3.3 `auto&&` (범용 참조) 사용
```cpp
for (auto&& num : vec) {
    num += 10;  // 원본 변경
}
```
- `auto&&`는 **lvalue와 rvalue 모두 받을 수 있는 범용 참조**.
- 보통 **템플릿 코드**에서 많이 사용됨.

---

## 4. auto&&가 lvalue와 rvalue를 모두 받을 수 있는 이유

```cpp
template <typename T>
void func(T&& arg) {   // arg는 lvalue와 rvalue 모두 받을 수 있음
    // ...
}
```

### 컴파일러가 `T&&`를 해석하는 방식

1. **lvalue를 전달하는 경우**
   ```cpp
   int x = 10;
   func(x);  // lvalue 전달
   ```
   - `x`는 lvalue이므로 `T`는 `int&`로 추론됨.
   - 따라서 `T&&`는 `int& &&` → **int& (lvalue 참조)** 가 됨.
   - 즉, **lvalue가 전달되면 T&&는 lvalue 참조(`&`)로 변환됨.**

2. **rvalue를 전달하는 경우**
   ```cpp
   func(20);  // rvalue 전달
   ```
   - `20`은 rvalue이므로 `T`는 `int`로 추론됨.
   - 따라서 `T&&`는 `int&&` 그대로 유지됨.
   - 즉, **rvalue가 전달되면 T&&는 rvalue 참조(`&&`)로 유지됨.**

### 정리된 변환 규칙

| 입력 값 | `T`가 추론되는 타입 | `T&&`가 변환되는 타입 |
|---------|----------------|----------------|
| `int x = 10; func(x);` | `int&` | `int&` |
| `func(20);` | `int` | `int&&` |

- 즉, **`T&&`는 입력된 값(lvalue/rvalue)에 따라 다르게 해석됨.**

---

## 5. 결론

- **lvalue** → **이름이 있고, 메모리에 저장되는 값 (변수 등)**  
- **rvalue** → **연산 결과로 생성되는 임시 값 (주소 없음)**  
- **lvalue 참조(`&`)** → **lvalue를 참조 (주소 가짐)**  
- **rvalue 참조(`&&`)** → **rvalue를 참조 (이동 연산자에서 유용)**  
- **for(auto num : container)** → **복사 발생, 원본 변경되지 않음**  
- **for(auto& num : container)** → **참조 사용, 원본 변경 가능**  
- **for(auto&& num : container)** → **lvalue, rvalue 모두 받을 수 있음 (범용 참조)**  
- **`auto&&`는 lvalue 입력 시 lvalue 참조, rvalue 입력 시 rvalue 참조로 동작함**

