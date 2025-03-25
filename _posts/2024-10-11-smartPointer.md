---
title: C++ 스마트 포인터
author: RePro
date: 2024-10-11 10:00:00 +0900
categories: [Programming, C++]
tags: [c++]
---

# C++ 스마트 포인터 정리

## 📌 1. 스마트 포인터(Smart Pointer)란?

스마트 포인터는 **C++에서 메모리를 자동으로 관리**하는 포인터입니다. 일반적인 `new/delete` 기반의 메모리 관리를 대체하며, **메모리 누수 방지 및 안정적인 메모리 해제 기능**을 제공합니다.

C++98에서는 `std::auto_ptr`이 도입되었지만, 설계상의 문제로 인해 **C++11 이후 `std::unique_ptr`, `std::shared_ptr`, `std::weak_ptr`로 대체**되었습니다.

---

## ✅ 2. 주요 스마트 포인터 종류

### **2.1 `std::unique_ptr` (단독 소유, 복사 불가)**
- 단 하나의 스마트 포인터만 특정 객체를 소유할 수 있음.
- **복사 불가능**, 대신 `std::move()`를 사용하여 소유권 이동 가능.

#### 🔹 사용 예제
```cpp
#include <iostream>
#include <memory>

int main() {
    std::unique_ptr<int> ptr = std::make_unique<int>(10);
    std::cout << *ptr << std::endl;  // 출력: 10
    
    std::unique_ptr<int> ptr2 = std::move(ptr);  // 소유권 이동
    std::cout << (ptr ? "ptr is valid" : "ptr is null") << std::endl;
}
```

#### 📌 특징
- **`std::move()`를 사용하여 소유권 이동 가능**.
- **범위를 벗어나면 자동으로 메모리 해제**.
- **배열(`T[]`)도 관리 가능** → `std::unique_ptr<int[]> arr(new int[10]);`

---

### **2.2 `std::shared_ptr` (공유 소유, 참조 카운트)**
- **여러 개의 포인터가 같은 객체를 공유 가능**.
- **참조 카운트(`use_count()`)** 를 사용하여 마지막 `shared_ptr`가 소멸될 때 객체 자동 삭제.

#### 🔹 사용 예제
```cpp
#include <iostream>
#include <memory>

int main() {
    std::shared_ptr<int> ptr1 = std::make_shared<int>(10);
    std::shared_ptr<int> ptr2 = ptr1;  // 참조 카운트 증가
    
    std::cout << *ptr1 << " ref count: " << ptr1.use_count() << std::endl;
}
```

#### 📌 특징
- **여러 포인터가 하나의 객체를 공유 가능**.
- **객체가 더 이상 사용되지 않을 때(`use_count() == 0`) 자동으로 삭제**.
- `std::make_shared<T>()`를 사용하면 **메모리 할당 비용 감소**.

--- 

## `std::shared_ptr` 개요 및 주의점

`std::shared_ptr`는 C++11에서 도입된 스마트 포인터로, 참조 카운트를 통해 메모리 관리를 자동화합니다. 그러나 모든 상황에서 이상적이지 않으며, 단점과 내부 구조를 이해하고 적절히 사용하는 것이 중요합니다.


## 1. 장점

- 참조 카운트를 기반으로 한 **자동 메모리 관리**
- 복수 객체에서 동일 리소스를 공유 가능
- 메모리 누수를 줄이는 데 효과적

---

## 2. 단점 및 주의사항

### 2.1 순환 참조 문제 (Circular Reference)

- 두 객체가 서로를 `shared_ptr`로 참조하면 참조 카운트가 0이 되지 않아 **메모리 누수** 발생

#### 예시
```cpp
struct A;
struct B;

struct A {
    std::shared_ptr<B> b;
};

struct B {
    std::shared_ptr<A> a;
};
```
- 해결: 한쪽은 `std::weak_ptr`로 변경

---

### 2.2 제어 블록 오버헤드

- 객체 외에 **제어 블록(control block)**이 추가로 메모리에 존재
- 제어 블록에는 참조 카운트, deleter 등이 포함됨 → **메모리 사용량 증가**

---

### 2.3 성능 오버헤드

- 참조 카운트 연산은 **thread-safe** → 내부적으로 원자 연산 사용
- 멀티스레드 환경에서는 **shared_ptr의 생성/복사/소멸이 성능 저하** 유발 가능

---

### 2.4 소유권이 불분명

- 여러 곳에서 객체를 소유할 수 있어, 객체의 **생명 주기 예측이 어려움**
- 리소스 생명주기를 명확하게 제어해야 한다면 `unique_ptr`이 적합함

---

### 2.5 잘못된 사용: shared_ptr(p.get())

```cpp
std::shared_ptr<Foo> a(new Foo);
std::shared_ptr<Foo> b(a.get()); // 위험: 서로 다른 제어 블록!
```
- `a`와 `b`는 서로 다른 제어 블록을 가짐 → **double delete 위험**


---

### **2.3 `std::weak_ptr` (순환 참조 방지, 참조 카운트 증가 없음)**
- `shared_ptr`의 **참조 카운트를 증가시키지 않고 객체를 참조**할 수 있음.
- **객체가 존재하는지(`expired()`) 확인 후 사용 가능**.

#### 🔹 사용 예제
```cpp
#include <iostream>
#include <memory>

int main() {
    std::weak_ptr<int> wptr;
    {
        std::shared_ptr<int> sptr = std::make_shared<int>(100);
        wptr = sptr;  // 참조 카운트 증가 안함
    } // sptr이 범위를 벗어나면 객체 소멸

    if (auto sp = wptr.lock()) {
        std::cout << *sp << std::endl;
    } else {
        std::cout << "객체가 소멸됨" << std::endl;
    }
}
```

#### 📌 특징
- `shared_ptr`과 함께 사용하여 **순환 참조(Circular Reference) 방지**.
- `lock()`을 사용하여 **유효한 `shared_ptr`를 얻을 수 있음**.

---

## ✅ 3. `std::auto_ptr` (C++17에서 제거됨)

`std::auto_ptr`은 **C++98에서 도입된 스마트 포인터**지만, **복사 시 소유권이 강제로 이동되는 문제**로 인해 **C++11에서 사용이 비추천되고, C++17에서 완전히 제거됨**.

#### 🔹 `std::auto_ptr` 사용 예제 (비추천)
```cpp
#include <iostream>
#include <memory>

int main() {
    std::auto_ptr<int> ptr1(new int(10));
    std::auto_ptr<int> ptr2 = ptr1;  // ptr1의 소유권이 ptr2로 이동
    
    std::cout << (ptr1.get() == nullptr ? "ptr1 is null" : "ptr1 is valid") << std::endl;
}
```
📌 `std::auto_ptr` 대신 **`std::unique_ptr` 또는 `std::shared_ptr`를 사용할 것**.

---

## ✅ 4. 부가적인 스마트 포인터

### **4.1 `boost::scoped_ptr` (단순한 `unique_ptr`)**
- Boost 라이브러리에서 제공하는 스마트 포인터.
- `unique_ptr`과 유사하지만, **소유권 이동(`std::move()`)이 불가능**.

### **4.2 `boost::intrusive_ptr` (사용자 정의 참조 카운트)**
- `shared_ptr`과 유사하지만, **참조 카운트를 사용자가 직접 관리**.

### **4.3 `boost::shared_array` (배열 관리 스마트 포인터)**
- `shared_ptr`의 배열 버전.
- C++11 이후에는 `std::unique_ptr<T[]>`로 대체 가능.

---

## ✅ 5. 스마트 포인터 비교

| 스마트 포인터 | 소유권 | 참조 카운트 | 주요 특징 |
|--------------|--------|------------|------------|
| **`std::unique_ptr`** | **단독 소유** | ❌ 없음 | 복사 불가, 이동(`std::move()`) 가능 |
| **`std::shared_ptr`** | **공유 소유** | ✅ 있음 | 참조 카운트 증가/감소 |
| **`std::weak_ptr`** | **공유 소유 (X)** | ❌ 없음 | `shared_ptr`의 참조 카운트 증가 없이 참조 |
| **`std::auto_ptr`** | **단독 소유 (비추천)** | ❌ 없음 | C++17에서 제거됨 |

---

## ✅ 6. 결론

✔ **스마트 포인터는 `new/delete` 없이 자동 메모리 관리를 지원**.  
✔ **C++11 이후 `std::unique_ptr`, `std::shared_ptr`, `std::weak_ptr`을 사용해야 함**.  
✔ **`std::auto_ptr`은 C++17에서 제거되었으며, `std::unique_ptr`로 대체**.  
✔ **`std::shared_ptr`을 사용할 때는 순환 참조를 방지하기 위해 `std::weak_ptr`을 활용**.  
