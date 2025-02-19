---
title: try-catch, throw 예외 처리
author: RePro
date: 2024-08-09 10:00:00 +0900
categories: [Blogging, Tutorial]
tags: [favicon]
---

# C++ 예외 처리 (try-catch, throw) 정리

## 1. 예외(Exception)란?

- 프로그램 실행 중에 **비정상적인 상황**이 발생하는 것.
- 예: 0으로 나누기, 메모리 부족, 파일 열기 실패 등.

이러한 **예외 상황이 발생하면 프로그램이 즉시 중단되는 것이 아니라**, 개발자가 지정한 처리를 수행하도록 흐름을 변경할 수 있다.

---

## 2. try-catch문의 기본 구조

```cpp
try {
    // 예외 발생 가능성이 있는 코드
}
catch (예외_타입 변수명) {
    // 예외 발생 시 실행되는 코드
}
```

### 동작 과정
1. `try` 블록 안의 코드 실행
2. **예외 발생 시 `catch` 블록으로 이동**
3. **예외가 없으면 `catch`는 건너뛰고 다음 코드 실행**

---

## 3. throw 키워드

`throw`는 **예외를 발생시키는 키워드**이다.

```cpp
throw 표현식;
```

- `표현식`은 **예외 객체**(숫자, 문자열, 클래스 인스턴스 등)일 수 있다.
- `throw`가 실행되면 즉시 **현재 실행 중인 함수가 종료**되고, **가장 가까운 `catch` 블록으로 제어가 넘어간다**.

### throw 예시

```cpp
int divide(int a, int b) {
    if (b == 0) {
        throw "Division by zero";  // 문자열 예외 발생
    }
    return a / b;
}
```

---

## 4. 예제 코드

```cpp
#include <iostream>

int main() {
    try {
        int a = 10;
        int b = 0;
        if (b == 0) {
            throw "Division by zero error";
        }
        int result = a / b;
    }
    catch (const char* errorMessage) {
        std::cout << "Exception caught: " << errorMessage << std::endl;
    }

    std::cout << "Program continues..." << std::endl;

    return 0;
}
```

### 실행 결과

```
Exception caught: Division by zero error
Program continues...
```

---

## 5. 여러 개의 catch문

서로 다른 종류의 예외를 처리하려면 **여러 개의 `catch`문**을 사용할 수 있다.

```cpp
try {
    throw 404;
}
catch (int errorCode) {
    std::cout << "Error Code: " << errorCode << std::endl;
}
catch (const char* message) {
    std::cout << "Error Message: " << message << std::endl;
}
```

---

## 6. 모든 예외 처리하기

모든 예외를 처리하려면 **`...`(와일드카드)**를 사용한다.

```cpp
try {
    throw "Unknown error";
}
catch (...) {
    std::cout << "Unknown Exception Caught!" << std::endl;
}
```

---

## 7. 예외 객체 사용

객체를 던지고 받을 수도 있다.

```cpp
#include <iostream>
#include <stdexcept>

int main() {
    try {
        throw std::runtime_error("Runtime Error Occurred");
    }
    catch (const std::runtime_error& e) {
        std::cout << "Caught: " << e.what() << std::endl;
    }
}
```

---

## 8. throw 단독 사용 (재던지기)

`throw;`만 단독으로 사용하면, **현재 잡고 있는 예외를 다시 던진다(재전달)**는 의미다.

```cpp
try {
    throw std::runtime_error("에러 발생");
}
catch (const std::runtime_error& e) {
    std::cout << "Caught: " << e.what() << std::endl;
    throw;  // 현재 잡은 예외 다시 던지기
}
```

---

## 9. 주의사항

- **예외를 잡지 않으면 프로그램은 강제 종료된다.**
- 모든 코드에 예외 처리를 붙이면 오히려 복잡해질 수 있으므로, **예외가 발생할 가능성이 큰 중요한 부분에만 적용하는 것이 좋다.**

---

## 10. 요약

| 키워드  | 설명 |
|---------|-------------------------------------------|
| `try`   | 예외 발생 가능성이 있는 코드 블록 |
| `catch` | 예외가 발생했을 때 실행되는 블록 |
| `throw` | **예외 발생(던지기)** |
| `throw;`| **현재 잡고 있는 예외 다시 던지기(재전달)** |
| `...`   | 모든 예외 처리 |

---

## 11. 결론

예외 처리는 프로그램의 안정성을 높이는 데 중요한 역할을 한다.
`try-catch`와 `throw`를 적절히 활용하여 비정상적인 상황에서도 프로그램이 안전하게 동작하도록 설계하는 것이 바람직하다.

