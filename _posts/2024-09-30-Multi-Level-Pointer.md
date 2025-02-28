---
title: C++ 다중 포인터
author: RePro
date: 2024-09-30 10:00:00 +0900
categories: [Programming, C++]
tags: [c++]
---


# C++ 다중 포인터(Multi-Level Pointer) 정리

## 📌 1. 이중 포인터(Double Pointer)란?
이중 포인터는 **포인터를 가리키는 포인터**로, `**` 기호를 사용하여 선언합니다.

### 🔹 **일반 포인터 vs 이중 포인터**
| 구분 | 선언 방식 | 설명 |
|------|----------|------|
| 일반 포인터 | `int* ptr;` | `ptr`은 `int` 값을 가리키는 포인터 |
| 이중 포인터 | `int** ptr;` | `ptr`은 `int*`(포인터)를 가리키는 포인터 |

### **🔹 기본 예제**
```cpp
#include <iostream>

int main() {
    int x = 10;
    int* ptr = &x;   // 1단계 포인터
    int** dptr = &ptr; // 2단계 포인터

    std::cout << "x: " << x << std::endl;
    std::cout << "*ptr: " << *ptr << std::endl;
    std::cout << "**dptr: " << **dptr << std::endl;

    return 0;
}
```
📌 **이중 포인터(`dptr`)는 `ptr`을 가리키며, `**dptr`을 통해 `x`의 값에 접근 가능**.

---

## ✅ 2. 이중 포인터의 주요 활용

### **(1) 포인터를 함수에서 수정 (Call by Reference)**
이중 포인터를 사용하면 **함수에서 포인터 자체를 변경 가능**.

```cpp
void modifyPointer(int** dptr) {
    static int y = 20;
    *dptr = &y;  // 포인터가 가리키는 주소 변경
}
```

### **(2) 동적 2차원 배열 할당**
```cpp
int** matrix = new int*[3];
for (int i = 0; i < 3; i++) {
    matrix[i] = new int[4];
}
```
📌 **이중 포인터(`int** matrix`)를 이용하여 2차원 배열을 동적으로 생성**.

---

## ✅ 3. 다중 포인터(Multi-Level Pointer)란?
다중 포인터는 **이중 포인터 이상(`int***`, `int****` 등)의 포인터를 의미**하며, 포인터의 포인터를 여러 단계 거쳐 가리킬 수 있습니다.

### **🔹 3중 포인터 (`int***`)**
```cpp
int x = 10;
int* ptr = &x;
int** dptr = &ptr;
int*** tptr = &dptr;

std::cout << ***tptr << std::endl;  // 10
```
📌 **`tptr`을 사용하면 `***tptr`을 통해 최종 변수 `x`에 접근 가능**.

---

## ✅ 4. 다중 포인터 활용 사례

### **(1) 3D 배열 동적 할당 (`int***` 사용)**
```cpp
int*** arr = new int**[3];
for (int i = 0; i < 3; i++) {
    arr[i] = new int*[3];
    for (int j = 0; j < 3; j++) {
        arr[i][j] = new int[3];
    }
}
```
📌 **3D 배열을 동적으로 생성하려면 3중 포인터(`int***`)가 필요함**.

### **(2) 트리 구조에서 포인터 수정**
```cpp
struct Node {
    int value;
    Node* left;
    Node* right;
};

void insert(Node** root, int val) {
    if (*root == nullptr) {
        *root = new Node{val, nullptr, nullptr};
    }
}
```
📌 **이중 포인터(`Node** root`)를 사용하면 `root` 포인터를 함수 내에서 직접 수정 가능**.

---

## ✅ 5. 다중 포인터 사용 시 주의할 점
### **🔹 (1) 메모리 해제 (Memory Leak 방지)**
```cpp
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        delete[] arr[i][j];
    }
    delete[] arr[i];
}
delete[] arr;
```
📌 **다중 포인터를 사용할 경우, 메모리 할당된 모든 계층을 `delete[]`로 해제해야 함**.

### **🔹 (2) 다중 역참조 사용 시 가독성 저하**
```cpp
std::cout << ***p;  // ❌ 너무 복잡함
```
📌 **대부분의 경우 이중 포인터(`int**`)까지만 사용하는 것이 일반적**.

---

## ✅ 6. 다중 포인터 vs 2차원 배열
| 구분 | 다중 포인터 (`int**`) | 2차원 배열 (`int arr[M][N]`) |
|------|----------------------|----------------------|
| **메모리 구조** | 동적으로 할당된 행 배열 | 정적인 연속된 메모리 |
| **배열 크기** | 런타임 결정 가능 | 컴파일 타임에 크기 고정 |
| **접근 속도** | 상대적으로 느림 (포인터 연산 필요) | 빠름 (연속된 메모리 접근) |

---

## ✅ 7. 결론
✔ **이중 포인터(`int**`)는 다차원 배열, 포인터 배열, 트리 구조에서 유용**.  
✔ **3D 배열 동적 할당, 복잡한 자료구조 등에서는 3중 포인터(`int***`)도 활용됨**.  
✔ **포인터의 개수가 많아질수록 코드가 복잡해지고 가독성이 떨어짐**.  
✔ **대부분의 경우 이중 포인터(`int**`)까지만 사용하는 것이 일반적**.  

🚀 **다중 포인터는 강력한 기능을 제공하지만, 적절한 사용이 중요합니다!**

