---
title: "[C++] 동적 메모리 할당"
author: RePro
date: 2024-08-04 10:00:00 +0900
categories: [Programming, C++]
tags: [c++]
---

# C++ new/delete vs malloc/free 차이 정리

C++에서 동적 메모리를 할당하고 해제할 때는 `new/delete`와 `malloc/free`를 사용할 수 있습니다. 하지만 이 둘은 작동 방식, 안전성, 객체 초기화 등 여러 면에서 중요한 차이가 있습니다.

---

## 1. 주요 차이 요약

| 항목 | `new` / `delete` | `malloc` / `free` |
|------|------------------|-------------------|
| 소속 언어 | C++ | C (C++에서도 사용 가능) |
| 타입 정보 유지 | O (타입 추론됨) | X (void* 반환) |
| 생성자 / 소멸자 호출 | O | X |
| 오버로딩 가능 여부 | O (`operator new`) | X |
| 예외 처리 | 실패 시 `std::bad_alloc` 예외 | 실패 시 `NULL` 반환 |
| 캐스팅 필요 여부 | X | O (명시적 캐스팅 필요) |

---

## 2. `new` / `delete` 예제 (C++ 스타일)

```cpp
#include <iostream>

class MyClass {
public:
    MyClass() { std::cout << "MyClass 생성자 호출\n"; }
    ~MyClass() { std::cout << "MyClass 소멸자 호출\n"; }
};

int main() {
    MyClass* obj = new MyClass();  // 생성자 호출됨
    delete obj;                    // 소멸자 호출됨

    int* arr = new int[5];         // int형 배열 할당
    delete[] arr;                  // 배열 해제 (주의: delete[] 사용)

    return 0;
}
```

### 결과:
```
MyClass 생성자 호출
MyClass 소멸자 호출
```

---

## 3. `malloc` / `free` 예제 (C 스타일)

```cpp
#include <iostream>
#include <cstdlib> // malloc, free
#include <cstring> // memset

class MyClass {
public:
    MyClass() { std::cout << "MyClass 생성자 호출\n"; }
    ~MyClass() { std::cout << "MyClass 소멸자 호출\n"; }
};

int main() {
    // 메모리만 할당됨, 생성자 호출되지 않음
    MyClass* obj = (MyClass*)malloc(sizeof(MyClass));
    memset(obj, 0, sizeof(MyClass)); // 위험: vtable 등 초기화 파괴 가능

    // 직접 생성자 호출도 가능하지만 매우 비추천
    // new(obj) MyClass();  // placement new

    free(obj); // 소멸자 호출되지 않음 → 리소스 누수 가능성

    int* arr = (int*)malloc(sizeof(int) * 5); // 배열 할당
    free(arr);

    return 0;
}
```

### 결과:
- 출력 없음 (생성자/소멸자 호출되지 않음)

---

## 4. 혼용 금지 예제

```cpp
int* p1 = (int*)malloc(sizeof(int));
// delete p1; // 잘못된 해제 → undefined behavior

int* p2 = new int;
// free(p2);  // 잘못된 해제 → undefined behavior
```

> 반드시 `new`는 `delete`로, `malloc`은 `free`로 해제해야 합니다.

---

## 5. 언제 어떤 걸 써야 하나?

| 상황 | 권장 방법 |
|------|-------------|
| 일반적인 객체 생성/삭제 | `new` / `delete` 사용 |
| 생성자/소멸자 포함된 클래스 | `new` / `delete` 필수 |
| C API와 연동할 때 | `malloc` / `free` 가능 |
| 고성능/커스텀 메모리 제어 | 상황에 따라 선택 (Placement new 등) |

---

## 6. 결론

- **C++에서는 `new/delete` 사용이 기본**입니다.
- `malloc/free`는 객체의 생성자/소멸자를 무시하기 때문에 **클래스 타입에는 위험**합니다.
- 둘을 혼용하면 undefined behavior가 발생하므로 **매칭을 정확히 지켜야 합니다.**

C++의 객체 지향 기능을 안전하게 활용하려면 `new/delete`를 사용하고, 필요 시 스마트 포인터(`std::unique_ptr`, `std::shared_ptr`)를 활용하는 것이 바람직합니다.

