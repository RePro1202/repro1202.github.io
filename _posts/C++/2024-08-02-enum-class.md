---
title: "[C++] enum class"
author: RePro
date: 2024-08-02 10:00:00 +0900
categories: [Programming, C++]
tags: [c++]
---


# C++의 enum vs enum class 정리

---

## 1. enum과 enum class의 정의

### enum (전통적인 열거형, C 스타일)

```cpp
enum Color {
    RED,
    GREEN,
    BLUE
};
```

- 열거자(`RED`, `GREEN`, `BLUE`)가 **전역 스코프**에 노출됨
- 열거자는 기본적으로 `int`로 취급되며 암묵적 형변환 가능
- 이름 충돌 가능성 있음
- 타입 안정성 낮음

---

### enum class (스코프 열거형, C++11부터 도입)

```cpp
enum class Color {
    RED,
    GREEN,
    BLUE
};
```

- 열거자(`RED`, `GREEN`, `BLUE`)가 **스코프 안에 있음**
- `Color::RED`처럼 **명시적으로 접근해야 함**
- 암묵적 형변환이 불가능하여 타입 안전성이 높음
- 이름 충돌 방지

---

## 2. enum과 enum class의 차이점

| 항목 | enum | enum class |
|------|------|-------------|
| 접근 방식 | 전역 이름으로 접근 (예: `RED`) | 스코프를 통한 접근 (예: `Color::RED`) |
| 형 변환 | int로 암묵적 변환 가능 | 명시적 형변환만 가능 |
| 타입 안전성 | 낮음 | 높음 |
| 이름 충돌 | 가능 | 없음 |
| 사용 가능 버전 | C부터 가능 | C++11 이상에서 사용 가능 |

---

## 3. 스코프처럼 사용한다는 의미

### 전통 enum

```cpp
enum Status {
    OK,
    ERROR
};

int a = OK;  // OK
```

- 열거자 `OK`, `ERROR`가 전역에 존재함

### enum class

```cpp
enum class Status {
    OK,
    ERROR
};

int a = static_cast<int>(Status::OK);  // OK
int b = OK;          // 컴파일 에러
```

- `OK`, `ERROR`는 `Status`라는 이름 공간 안에 있는 **스코프 멤버**

---

## 4. 명시적 형변환 예시

```cpp
enum class Mode { AUTO = 0, MANUAL = 1 };

int value = static_cast<int>(Mode::AUTO); // 명시적 변환만 가능
```

---

## 5. 이름 충돌 방지 예시

```cpp
enum class Animal { DOG, CAT };
enum class Color { RED, GREEN };

Animal a = Animal::DOG;
// a = Color::RED;  // 컴파일 오류: 타입 불일치
// a = 0;           // 컴파일 오류: 암묵적 int 변환 불가
```

---

## 6. 요약

| 항목 | enum | enum class |
|------|------|-------------|
| 열거자 이름 접근 | 전역에서 직접 접근 | 열거형 이름을 통해 접근 |
| 스코프 구분 | 없음 | 있음 |
| 이름 충돌 가능성 | 있음 | 없음 |
| 타입 안전성 | 낮음 | 높음 |
| int 변환 | 암묵적 | 명시적만 가능 |

---

**결론**: `enum class`는 스코프 제한과 타입 안전성을 제공하여 코드의 명확성과 안전성을 높이므로 일반적으로 `enum class` 사용이 권장.
