---
title: "[C++] 가상함수와 vtable"
author: RePro
date: 2024-09-02 10:00:00 +0900
categories: [Programming, C++]
tags: [c++]
---

# C++ 가상 함수(Virtual Function)와 vtable의 동작 원리

C++에서 **가상 함수(Virtual Function)** 는 **상속과 다형성(polymorphism)** 을 구현하는 핵심 개념입니다. 가상 함수를 통해 부모 클래스 포인터(또는 참조)를 통해 자식 클래스의 메서드를 호출할 수 있습니다. 이 기능을 가능하게 하는 구조가 바로 **vtable(virtual table)** 입니다.

---

## 1. 가상 함수(Virtual Function)란?

### 정의
- 가상 함수는 `virtual` 키워드로 선언된 멤버 함수입니다.
- **기본 클래스 포인터를 통해 파생 클래스의 메서드를 호출할 수 있도록 허용**합니다.

```cpp
class Base {
public:
    virtual void speak() {
        std::cout << "Base speaking\n";
    }
};

class Derived : public Base {
public:
    void speak() override {
        std::cout << "Derived speaking\n";
    }
};

int main() {
    Base* p = new Derived();
    p->speak();  // Derived::speak() 호출됨
    delete p;
}
```

### 출력 결과:
```
Derived speaking
```

---

## 2. virtual vs non-virtual 함수

| 항목 | virtual 함수 | 일반 함수 |
|------|---------------|------------|
| 바인딩 시점 | 런타임 | 컴파일 타임 |
| 호출 방식 | vtable 간접 참조 | 직접 호출 |
| 성능 | 간접 호출로 약간 느림 | 빠름 |
| 유연성 | 높음 (다형성 지원) | 낮음 |

---

## 3. vtable (Virtual Table)이란?

### 개요
- 가상 함수 호출을 구현하기 위해 **컴파일러가 자동 생성**하는 **함수 포인터 테이블**
- 클래스마다 하나씩 생성됨
- 가상 함수가 하나라도 있다면 해당 클래스에 vtable이 생성됨

### vptr (Virtual Pointer)
- **객체 내부에 숨겨진 포인터**
- 이 객체가 사용할 vtable을 가리킴

```
객체 구조:
┌─────────────┐
│   vptr      │ → vtable(Class)
│ 멤버 변수들 │
└─────────────┘
```

---

## 4. vtable 동작 원리

### 객체 생성 시
- 컴파일러는 객체에 `vptr`을 삽입
- `vptr`은 해당 클래스의 `vtable` 주소를 저장

### 함수 호출 시
- `vptr`을 통해 `vtable`에 접근
- 해당 함수의 포인터를 찾아 호출 → **동적 바인딩** 수행

```cpp
Base* p = new Derived();
p->speak();  // vptr → vtable(Derived) → &Derived::speak 호출
```

---

## 5. 예시로 보는 vtable 구조

```cpp
class Base {
public:
    virtual void f() { std::cout << "Base::f\n"; }
};

class Derived : public Base {
public:
    void f() override { std::cout << "Derived::f\n"; }
};
```

### vtable 구조:
```
vtable(Base):
+----------------+
| &Base::f       |
+----------------+

vtable(Derived):
+----------------+
| &Derived::f    |
+----------------+
```

### 호출 흐름:
- `Base* p = new Derived();`
- `p->f();` → p → vptr → vtable(Derived) → &Derived::f 호출

---

## 6. 생성자/소멸자에서 virtual 함수 호출이 위험한 이유

### 생성자
- 객체 생성 도중에는 vptr이 아직 base 클래스의 vtable을 가리킴
- 이 시점에 virtual 함수를 호출하면 **base 버전만 호출됨**

### 소멸자
- 파생 클래스가 먼저 소멸되므로 vptr은 base 클래스의 vtable로 돌아감
- 이 시점에 virtual 함수 호출 시 역시 base 버전만 호출됨

### 예제
```cpp
class Base {
public:
    Base() { speak(); } // Base::speak 호출됨
    virtual void speak() { std::cout << "Base\n"; }
    virtual ~Base() { speak(); } // Base::speak 호출됨
};

class Derived : public Base {
public:
    void speak() override { std::cout << "Derived\n"; }
    ~Derived() { std::cout << "Derived destroyed\n"; }
};
```

### 출력 결과
```
Base
Derived destroyed
Base
```

---

## 7. 순수 가상 함수와 추상 클래스

### 순수 가상 함수
```cpp
class Shape {
public:
    virtual void draw() = 0;  // 순수 가상 함수
};
```
- 구현이 없는 virtual 함수
- 해당 클래스를 **추상 클래스**로 만들며, 인스턴스를 생성할 수 없음
- 파생 클래스는 반드시 오버라이딩해야 함

### vtable 구조
- 순수 가상 함수도 vtable에 등록됨 (대개 null 또는 특수 표기)

---

## 8. 기타 특징 및 주의사항

| 항목 | 설명 |
|------|------|
| 클래스마다 vtable 1개 | 객체당 vptr 1개가 자신의 클래스의 vtable을 가리킴 |
| 다중 상속 | 클래스마다 여러 개의 vptr과 vtable이 생길 수 있음 |
| RTTI, dynamic_cast | 내부적으로 vtable 사용 (typeid도 포함) |
| 성능 | 간접 호출이므로 일반 함수보다 약간 느림 |
| 디버깅 | 가상 함수 호출 오류는 런타임 시점에 드러남 |

---

## 9. 요약

| 개념 | 설명 |
|------|------|
| 가상 함수 | virtual 키워드로 선언, 런타임에 결정됨 (동적 바인딩) |
| vtable | 클래스마다 생성되는 함수 포인터 테이블 |
| vptr | 객체 내부 포인터, 해당 클래스의 vtable을 가리킴 |
| 생성자/소멸자 | 이 시점엔 파생 클래스의 vtable을 사용할 수 없음 |
| 추상 클래스 | 순수 가상 함수를 포함한 클래스, 직접 인스턴스 생성 불가 |

가상 함수와 vtable 구조는 객체지향 설계에서 다형성을 구현하는 핵심적인 기반이며, 올바르게 이해하고 사용하면 C++의 유연함과 강력함을 최대한 활용할 수 있습니다.

