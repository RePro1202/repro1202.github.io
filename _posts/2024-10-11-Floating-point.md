---
title: "[C++] 부동소수점 오차 및 비교 방법"
author: RePro
date: 2024-10-11 9:00:00 +0900
categories: [Programming, C++]
tags: [c++]
---

# C++ 부동소수점(Floating-Point) 오차 및 비교 방법

부동소수점은 실수를 컴퓨터 메모리에 저장하고 계산하는 방법입니다. 하지만 이 방식에는 고유한 한계와 오차가 존재합니다. 아래는 주요 개념, 예제, 그리고 실전에서 유의할 점들을 종합한 정리입니다.

---

## 1. 부동소수점이란?
- 부동소수점(Floating-point)은 소수점이 고정되지 않고 떠다니는 방식으로, 값의 크기와 정밀도를 유연하게 표현할 수 있습니다.
- 정수 또는 고정소수점 방식에 비해 더 넓은 범위의 수를 표현할 수 있지만, 정밀도 손실이 발생할 수 있습니다.

---

## 2. IEEE 754 표준
현대 컴퓨터에서 사용하는 부동소수점 표현 방식의 국제 표준입니다.

### 대표 형식
| 형식 | 비트 수 | 정밀도 (소수 자릿수) |
|------|---------|------------------|
| float (단정도) | 32비트 | 약 7자리 |
| double (배정도) | 64비트 | 약 15~17자리 |

---

## 3. 부동소수점 오차 예제

### 3.1 < 0.1 + 0.2 ≠ 0.3 >
```cpp
#include <iostream>
#include <iomanip>

int main() {
    double a = 0.1;
    double b = 0.2;
    double c = a + b;

    std::cout << std::fixed << std::setprecision(20);
    std::cout << "0.1 + 0.2 = " << c << std::endl;
    std::cout << "0.3       = " << 0.3 << std::endl;
    std::cout << std::boolalpha;
    std::cout << "Equal? " << (c == 0.3) << std::endl;
}
```

**출력 예시:**
```
0.1 + 0.2 = 0.30000000000000004441
0.3       = 0.29999999999999998890
Equal? false
```

### 3.2 누적 오차 예제
```cpp
#include <iostream>
int main() {
    double sum = 0.0;
    for (int i = 0; i < 10; ++i) {
        sum += 0.1;
    }
    std::cout << (sum == 1.0) << std::endl; // false
}
```

---

## 4. 오차가 발생하는 이유

### 이진수 표현의 한계
- 일부 십진수(예: 0.1, 0.2)는 이진수로 정확히 표현할 수 없어, 근사값만 저장됨
- 예: `0.1(10진수)` → `0.0001100110011...(2진수 무한 반복)`

### IEEE 754 구조 요약
- float: 1비트 부호 + 8비트 지수 + 23비트 가수
- double: 1비트 부호 + 11비트 지수 + 52비트 가수

---

## 5. C++에서 정밀하게 실수 비교하는 방법

실수끼리 비교할 때는 `==`을 직접 사용하지 말고, 오차 허용 범위(Epsilon)를 고려해야 합니다.

### 절대 오차 기준
```cpp
#include <cmath>
bool nearly_equal(double a, double b, double epsilon = 1e-9) {
    return std::fabs(a - b) < epsilon;
}
```

### 상대 오차까지 고려
```cpp
#include <cmath>
bool almost_equal(double a, double b, double epsilon = 1e-9) {
    return std::fabs(a - b) <= epsilon * std::max(std::fabs(a), std::fabs(b));
}
```

### C++17 이상: `std::nextafter` 활용
```cpp
#include <cmath>
bool safe_equal(double a, double b) {
    return std::nextafter(a, b) == b;
}
```

---

## 6. 디버깅 팁

| 팁 | 설명 |
|-----|------|
| `setprecision(n)` | 정밀 출력으로 오차 확인 |
| 디버거로 16진수 확인 | 내부 표현을 직접 추적 가능 |
| 반복 연산 최소화 | 누적 오차 방지 필요 |
| 문자열 비교 지양 | `to_string()` 결과는 정확하지 않음 |

---

## 8. 요약 정리

| 항목 | 설명 |
|------|------|
| Floating-point | 유연하고 광범위한 실수 표현 방식 |
| IEEE 754 | 부동소수점 표현 표준 (float, double 포함) |
| 비교 방식 | `==` 대신 오차(Epsilon) 기반 비교 권장 |
| 디버깅 | `setprecision`, `isnan`, 반복 합산 시 유의 |
