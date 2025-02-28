---
title: C++ 참조
author: RePro
date: 2024-10-05 10:00:00 +0900
categories: [Programming, C++]
tags: [c++]
---

# C++ 참조(Reference) 정리

## 📌 1. C++ 참조(Reference)란?

참조(Reference)는 **변수의 별칭(Alias)** 으로, 원본 변수와 같은 메모리 주소를 공유합니다.

```cpp
int x = 10;
int& ref = x;  // x를 참조하는 ref 선언
ref = 20;  // x의 값도 20으로 변경됨
std::cout << x;  // 출력: 20
```
📌 **`ref`와 `x`는 같은 메모리 공간을 공유하므로 `ref`를 변경하면 `x`도 변경됩니다.**

---

## ✅ 2. 참조의 기본 특징
1. **한 번 참조하면 변경 불가** → `int& ref = x;` 이후 `ref`를 다른 변수에 참조할 수 없음.
2. **초기화가 반드시 필요** → `int& ref;` 는 **오류 발생!** (참조는 `nullptr`이 될 수 없음).
3. **메모리 주소 공유** → 참조는 원본 변수와 같은 주소를 가짐.

```cpp
int x = 10;
int& ref = x;
std::cout << &x << " == " << &ref << std::endl; // 같은 주소
```

---

## ✅ 3. 참조의 종류

### 🔹 (1) 일반 참조 (Lvalue Reference)
```cpp
int x = 10;
int& ref = x;
ref = 30;  // x도 30으로 변경됨
```

### 🔹 (2) 상수 참조 (Const Reference)
```cpp
const int x = 10;
const int& ref = x;  // x를 읽기 전용으로 참조
```

### 🔹 (3) Rvalue 참조 (Move Reference, `&&`)
```cpp
int&& rref = 10;  // 임시 값(10)을 참조
std::cout << rref;  // 10
```
📌 **Rvalue 참조(`&&`)를 사용하면 불필요한 복사를 방지하여 성능 최적화 가능**.
[이곳 참고](/2024-10-16-Ravlue-move.md)

---

## ✅ 4. 참조 vs 포인터 비교

| 구분 | 참조 (`&`) | 포인터 (`*`) |
|------|-----------|-------------|
| **NULL 가능 여부** | ❌ 불가능 | ✅ 가능 (`nullptr`) |
| **초기화 여부** | ✅ 반드시 초기화 필요 | ❌ 필요 없음 |
| **변경 가능 여부** | ❌ 한 번 설정하면 변경 불가 | ✅ 가리키는 주소 변경 가능 |
| **역참조 필요 여부** | ❌ 불필요 (직접 사용) | ✅ 필요 (`*ptr`) |

```cpp
int x = 10;
int& ref = x;  // 참조
int* ptr = &x; // 포인터

*ptr = 20;  // x = 20
ref = 30;   // x = 30
```

---

## ✅ 5. 참조 vs 포인터: 언제 사용해야 할까?

### **🔹 참조를 사용하는 것이 좋은 경우**
✅ **(1) 함수 인자로 원본 데이터를 수정할 때**
```cpp
void modify(int& num) { num += 10; }
int main() {
    int x = 10;
    modify(x);
    std::cout << x;  // 20
}
```

✅ **(2) 함수에서 불필요한 복사를 방지할 때**
```cpp
void print(const std::string& text) { std::cout << text << std::endl; }
std::string str = "Long Text";
print(str);  // 복사 없이 원본 사용
```

✅ **(3) 특정 변수를 별칭(alias)로 사용할 때**
```cpp
int x = 100;
int& alias = x;  // x의 별칭
alias = 200;
std::cout << x;  // 200
```

✅ **(4) Rvalue를 활용하여 성능 최적화할 때 (이동 시멘틱)**
```cpp
void process(std::string&& str) { std::cout << "Processing: " << str << std::endl; }
process("Temporary String");
```

---

### **🔹 포인터를 사용하는 것이 좋은 경우**
✅ **(1) 참조를 변경해야 할 때**
```cpp
int a = 10, b = 20;
int* ptr = &a;
ptr = &b; // ✅ 가능
```

✅ **(2) `nullptr`을 허용해야 할 때**
```cpp
int* ptr = nullptr;  // ✅ 가능
```

✅ **(3) 동적 메모리 할당을 할 때 (`new`/`delete`)**
```cpp
int* ptr = new int(10);
delete ptr;
```

✅ **(4) 배열을 가리킬 때**
```cpp
int arr[3] = {10, 20, 30};
int* ptr = arr;
ptr++;
```

✅ **(5) 함수에서 변수의 주소를 저장해야 할 때**
```cpp
void modifyPointer(int** ptr) {
    static int y = 20;
    *ptr = &y;
}
```

---

## ✅ 6. 참조 vs 포인터 비교 정리

| 사용 목적 | 참조 (`&`) | 포인터 (`*`) |
|----------|------------|--------------|
| **값을 변경해야 하는 경우** | ✅ 사용 가능 | ✅ 사용 가능 |
| **참조하는 대상 변경 가능 여부** | ❌ 불가능 | ✅ 가능 |
| **NULL(`nullptr`) 가능 여부** | ❌ 불가능 | ✅ 가능 |
| **불필요한 복사 방지 (`const &`)** | ✅ 사용 가능 | ❌ 불필요한 복사 발생 가능 |
| **동적 메모리 할당 (`new/delete`)** | ❌ 불가능 | ✅ 가능 |
| **배열을 다룰 때 (`ptr++`)** | ❌ 불가능 | ✅ 가능 |

---

## ✅ 7. 결론
✔ **참조(`&`): 안전한 별칭, 함수 인자로 사용할 때, 복사 방지(`const &`)**  
✔ **포인터(`*`): 가리키는 대상 변경 가능, `nullptr` 허용, 동적 메모리 할당, 배열 처리**  
✔ **Rvalue 참조(`&&`): 이동 연산자(move semantics)를 활용하여 성능 최적화**  

🚀 **포인터와 참조를 적절히 활용하여 C++ 코드를 더욱 효율적으로 작성하세요!**

