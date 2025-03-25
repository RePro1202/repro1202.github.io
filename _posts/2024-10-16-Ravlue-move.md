---
title: "[C++] Rvalue 참조(&&)와 std::move()"
author: RePro
date: 2024-10-16 10:00:00 +0900
categories: [Programming, C++]
tags: [c++]
---


# C++ Rvalue 참조(`&&`)와 `std::move()` 사용 원리 및 주의점

## 1. Rvalue 참조(`&&`)를 사용한 성능 최적화 원리

### 1.1 개념: 복사 vs 이동
C++에서 객체를 전달할 때 **값을 복사하면 원본 데이터를 유지하면서 새로운 객체를 생성**하지만, **복사는 비용이 크기 때문에** C++11부터 **이동(Move) 연산을 도입하여 불필요한 복사를 방지**할 수 있습니다.

🔹 **복사(Copy) 방식**
```cpp
std::string str1 = "Hello";
std::string str2 = str1; // str1을 복사하여 str2에 저장 (새로운 메모리 할당 발생)
```
- `str2`는 `str1`의 데이터를 **복사**하여 새롭게 할당된 메모리에 저장됨.
- **데이터가 클 경우 불필요한 성능 비용이 발생**.

🔹 **이동(Move) 방식 (`&&` 사용)**
```cpp
std::string str1 = "Hello";
std::string str2 = std::move(str1); // str1의 자원을 str2로 이동 (복사 없음)
```
- `str2`는 `str1`의 **내부 데이터를 가리키는 포인터만 이동**.
- `str1`은 **비어 있는 상태가 되고 추가적인 메모리 할당이 발생하지 않음**.

- **즉, Rvalue 참조(`&&`)를 사용하면 복사를 하지 않고, 기존 객체의 자원을 그대로 "이동"할 수 있음 → 불필요한 복사를 방지하여 성능 최적화 가능**.

---

## 2. `std::move()` 사용 시 주의할 점

### 🔹 2.1 `std::move()` 사용 후 원본 객체는 사용하지 말 것
```cpp
#include <iostream>
#include <string>

int main() {
    std::string str1 = "Hello";
    std::string str2 = std::move(str1);

    std::cout << "str1: " << str1 << std::endl;  // ⚠️ str1은 비어 있을 가능성 있음!
    std::cout << "str2: " << str2 << std::endl;  // Hello
}
```
- **이동 후 원본 객체(`str1`)를 다시 사용하면 미정의 동작(UB)이 발생할 수 있으므로 주의해야 함**.

 **안전한 방법**
```cpp
std::string str = "Hello";
std::string newStr = std::move(str);

if (str.empty()) {
    std::cout << "str is empty, safe to proceed" << std::endl;
}
```

---

###  2.2 이동 후 객체를 사용하면 미정의 동작 발생 가능
```cpp
std::vector<int> v1 = {1, 2, 3};
std::vector<int> v2 = std::move(v1);

std::cout << "v1 size: " << v1.size() << std::endl;  // ⚠️ 비어 있을 가능성 있음
std::cout << "v2 size: " << v2.size() << std::endl;  // 3
```
- **이동 후 `v1`은 비어 있을 수 있으므로, 다시 사용하지 않는 것이 좋음**.

---

### 2.3 이동 생성자/이동 연산자가 없으면 `std::move()`가 "복사"를 수행할 수도 있음
```cpp
class MyClass {
public:
    std::string data;

    MyClass(const std::string& str) : data(str) {}  // 복사 발생
};

int main() {
    MyClass obj1("Hello");
    MyClass obj2 = std::move(obj1);  // ⚠️ 복사가 발생할 수도 있음!
}
```
**올바른 이동 생성자 추가**
```cpp
MyClass(MyClass&& other) noexcept {
    data = std::move(other.data);
}
```
- **이동 생성자를 명시적으로 구현해야 `std::move()`가 복사 대신 "진짜 이동"을 수행함**.

---

## 3. `std::move()`를 사용하면 비효율적인 경우: 함수 인자로 `const std::string&`을 사용할 수 있을 때

### 🔹 3.1 `const std::string&` vs `std::string&&` vs `std::string` 비교
| 함수 인자 유형 | 복사 발생 여부 | 이동 가능 여부 | 주로 사용하는 상황 |
|--------------|------------|------------|----------------|
| `const std::string&` | ❌ 없음 | ❌ 불가능 | 읽기 전용, 복사 없이 빠르게 전달 |
| `std::string` | ✅ 있음 | ❌ 불가능 | 복사가 필요할 때 (값을 저장하는 경우) |
| `std::string&&` | ❌ 없음 | ✅ 가능 | Rvalue를 이동할 때 |

- **즉, 함수에서 `std::string`을 단순히 읽기만 한다면 `const std::string&`을 사용하는 것이 가장 효율적**.

 **올바른 코드 (복사 없이 전달)**
```cpp
void process(const std::string& str) {  // 복사 없이 참조
    std::cout << str << std::endl;
}

int main() {
    std::string msg = "Hello";
    process(msg);  // ✅ 불필요한 복사 없음
}
```

 **이동 연산을 활용한 효율적인 코드**
```cpp
class Logger {
private:
    std::string message;
public:
    void setMessage(std::string&& msg) {
        message = std::move(msg);  // ✅ 이동 연산 수행 (복사 없음)
    }
};
```

- **이동 연산(`std::move()`)을 사용하면 새로운 복사를 수행하지 않고 기존 데이터를 이동할 수 있음**.

---

 **정리**
| 함수 목적 | 올바른 인자 유형 | `std::move()` 사용 여부 |
|---------|-------------|----------------|
| 단순히 읽기 | `const std::string&` | ❌ 사용하지 않음 |
| 내부 저장 필요 | `std::string&&` | ✅ 사용 가능 |
| 복사본 저장 | `std::string` | ❌ 복사 발생 |

**함수에서 읽기만 할 때는 `const std::string&`을 사용하여 불필요한 `std::move()`를 피하는 것이 가장 효율적입니다**

---

# Google C++ 스타일 가이드: Rvalue 참조 정리

## Rvalue 참조란?

- Rvalue 참조는 임시 객체(temporary objects)에만 바인딩 가능한 참조 타입이다.
- 구문은 기존 참조와 유사하지만 `&&`를 사용한다.

```cpp
void f(std::string&& s);
```

- 위는 Rvalue 참조를 매개변수로 받는 함수의 예다.

---

## Rvalue 참조의 주요 사용 사례

### 1. 이동 생성자 및 이동 할당 연산자 정의

- 복사 대신 이동을 통해 성능 최적화

```cpp
class MyVector {
public:
    std::vector<int> data;

    MyVector(MyVector&& other) noexcept : data(std::move(other.data)) {}

    MyVector& operator=(MyVector&& other) noexcept {
        if (this != &other) {
            data = std::move(other.data);
        }
        return *this;
    }
};
```

---

### 2. 복사 불가능하지만 이동 가능한 타입 구현

```cpp
std::unique_ptr<int> ptr1 = std::make_unique<int>(42);
std::unique_ptr<int> ptr2 = std::move(ptr1);
```

---

### 3. 완벽한 전달 (Perfect Forwarding)

```cpp
template <typename T, typename Arg>
std::unique_ptr<T> make_unique(Arg&& arg) {
    return std::unique_ptr<T>(new T(std::forward<Arg>(arg)));
}
```

---

## 권장 지침 요약 및 예시

### 1. 이동 생성자 및 할당자 정의

```cpp
class MyData {
public:
    std::vector<int> data;

    MyData(MyData&& other) noexcept : data(std::move(other.data)) {}

    MyData& operator=(MyData&& other) noexcept {
        if (this != &other) {
            data = std::move(other.data);
        }
        return *this;
    }
};
```

---

### 2. `&&` 한정 멤버 함수

```cpp
class Builder {
public:
    std::string name;

    Builder&& SetName(std::string n) && {
        name = std::move(n);
        return std::move(*this);
    }

    void Print() const {
        std::cout << "Name: " << name << std::endl;
    }
};
```

---

### 3. 전달 참조와 std::forward

```cpp
template <typename T>
void Wrapper(T&& arg) {
    Process(std::forward<T>(arg));
}

void Process(const std::string& s) {
    std::cout << "복사: " << s << std::endl;
}

void Process(std::string&& s) {
    std::cout << "이동: " << s << std::endl;
}
```

---

### 4. 오버로드 쌍 정의

```cpp
class Handler {
public:
    void SetValue(const std::string& value) {
        std::cout << "복사" << std::endl;
        this->value = value;
    }

    void SetValue(std::string&& value) {
        std::cout << "이동" << std::endl;
        this->value = std::move(value);
    }

private:
    std::string value;
};
```

---

## 참고

- `const T&`, `T&&` 대신 값을 직접 받는 방법도 있음 (C++17 이상)
- 전달 참조는 `template <typename T> void func(T&& arg)` 형태일 때만 유효

---

## 요약

| 항목 | 설명 |
|------|------|
| 이동 최적화 | 복사 없이 자원 이전 가능 |
| && 한정자 | Rvalue 객체에서만 호출 허용 |
| 전달 참조 | Rvalue/Lvalue 구분 없이 전달 가능 |
| std::move / std::forward | 이동과 전달을 위한 핵심 함수 |


---
참고 : 
[구글 스타일 가이드 번역글](https://anothel.tistory.com/223?utm_source=chatgpt.com#toc1)

원문 : [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html#Rvalue_references)
