---
title: "[C++] 생성자와 소멸자의 동작"
author: RePro
date: 2024-09-03 10:00:00 +0900
categories: [Programming, C++]
tags: [c++]
---

# C++ 생성자와 소멸자의 동작 원리와 주의할 점

C++에서 객체가 생성되거나 파괴될 때 실행되는 **생성자(Constructor)** 와 **소멸자(Destructor)** 는 객체 생명주기의 핵심 요소입니다. 이 두 함수는 객체의 초기화와 정리에 사용되며, 특히 상속과 다형성이 관련된 상황에서는 특별한 주의가 필요합니다.

---

## 1. 생성자(Constructor)의 동작

생성자는 객체가 생성될 때 자동으로 호출되어 **멤버 변수 초기화** 와 **리소스 할당** 등을 수행합니다.

### 생성자 호출 순서
1. **Base 클래스 생성자 호출**
2. **멤버 초기화 리스트 실행**
3. **Derived 클래스 생성자 본문 실행**

객체가 생성될 때는 항상 **기본 클래스(Base)** 부터 생성이 시작되고, 이후 파생 클래스(Derived)의 생성자가 호출됩니다.

### 생성자 내부에서 주의할 점
- 생성자 안에서 virtual 함수를 호출하면 **예상과 다른 결과**를 초래할 수 있음 (자세한 설명은 아래 참고)
- 멤버 초기화 순서가 선언 순서와 다르면 **컴파일러 경고 또는 비정상 동작** 가능

---

## 2. 소멸자(Destructor)의 동작

소멸자는 객체가 파괴될 때 자동으로 호출되어 **메모리 해제, 파일 닫기, 네트워크 정리** 등 리소스 반환 작업을 수행합니다.

### 소멸자 호출 순서
1. **Derived 클래스 소멸자 호출**
2. **멤버 소멸자 호출**
3. **Base 클래스 소멸자 호출**

소멸자는 생성자의 역순으로 호출됩니다. 파생 클래스부터 파괴되고 마지막에 기본 클래스가 파괴됩니다.

### 소멸자 주의사항
- 상속 구조에서 **Base 소멸자를 반드시 virtual로 선언**해야 **다형성을 지원하는 객체가 완전히 파괴됨**
- 그렇지 않으면 파생 클래스 소멸자가 호출되지 않고 메모리 누수 발생 가능
- 소멸자 내부에서 virtual 함수 호출 시 주의가 필요함 (아래 설명 참고)

---

## 3. 생성자/소멸자에서 virtual 함수 호출이 위험한 이유

C++에서는 생성자 또는 소멸자 내부에서 virtual 함수를 호출하는 것을 **권장하지 않습니다**. 그 이유는 객체의 생성 및 파괴 시점에 **vtable(virtual table)** 이 완전히 설정되지 않기 때문입니다.

### 생성자에서 virtual 함수 호출 시 문제
```cpp
class Base {
public:
    Base() {
        print();  // virtual 함수
    }
    virtual void print() {
        std::cout << "Base::print()\n";
    }
};

class Derived : public Base {
public:
    void print() override {
        std::cout << "Derived::print()\n";
    }
};

int main() {
    Derived d;
    return 0;
}
```

#### 출력 결과
```
Base::print()
```

→ `Derived::print()`는 호출되지 않고 Base 버전이 호출됨.

### 왜 이런 일이 발생하는가?
- 객체 생성 시점에는 아직 Derived 클래스가 완전히 생성되지 않았기 때문에
- 컴파일러는 이 시점에 **vptr(virtual pointer)** 을 Base의 vtable로 설정함
- 따라서 virtual 함수 호출은 항상 Base의 버전으로 제한됨

### 소멸자에서도 유사한 문제 발생
```cpp
class Base {
public:
    virtual ~Base() {
        cleanup();  // virtual 함수
    }
    virtual void cleanup() {
        std::cout << "Base cleanup\n";
    }
};

class Derived : public Base {
public:
    ~Derived() {
        std::cout << "Derived destroyed\n";
    }
    void cleanup() override {
        std::cout << "Derived cleanup\n";
    }
};
```

#### 출력 결과
```
Derived destroyed
Base cleanup
```

→ `Derived::cleanup()`은 호출되지 않음. 이미 Derived 부분은 파괴된 상태이기 때문.

### 핵심 요약
| 시점 | vptr이 가리키는 vtable | 호출되는 virtual 함수 |
|------|-------------------------|-------------------------|
| 생성자 내부 | Base 클래스 | Base 버전 |
| 소멸자 내부 | Base 클래스 | Base 버전 |

---

## 4. 안전하게 virtual 함수 활용하는 방법

### 방법 1: 후처리 초기화 함수 분리
```cpp
class Base {
public:
    void init() { virtualInit(); }
protected:
    virtual void virtualInit() {}
};

class Derived : public Base {
protected:
    void virtualInit() override {
        // 안전한 호출 위치
    }
};

int main() {
    Derived d;
    d.init();  // 생성 후에 안전하게 호출
}
```

### 방법 2: 팩토리 함수 패턴 사용
- 생성자는 최소한으로 구성하고, 별도 함수에서 virtual 함수 호출 포함 초기화 수행

### 방법 3: 프레임워크 제공 후처리 훅 사용 (예: Unreal Engine)
- Unreal에서는 생성자에서 로직을 제한하고, `BeginPlay()`나 `PostInitializeComponents()` 등에서 virtual 호출을 수행함

---

## 5. 관련 개념 및 패턴

| 개념 / 패턴 | 설명 |
|--------------|-------|
| Factory Method | 객체 생성과 초기화를 분리 |
| Template Method | 베이스 클래스에서 virtual로 확장 포인트 제공 |
| CRTP | 템플릿 기반 다형성, virtual 없이 컴파일 타임 결정 |

---
## 6. 결론 요약

| 항목 | 설명 |
|------|------|
| 생성자 호출 순서 | Base → 멤버 초기화 → Derived |
| 소멸자 호출 순서 | Derived → 멤버 소멸자 → Base |
| 생성자에서 virtual 호출 | 위험. Base 버전만 호출됨 |
| 소멸자에서 virtual 호출 | 위험. 이미 Derived 파괴됨 |
| vtable/vptr의 역할 | virtual 함수 연결 및 호출 관리 |
| 안전한 대안 | init(), 팩토리 함수, BeginPlay() 등 후처리 함수 사용 |

C++에서 생성자와 소멸자는 단순히 초기화와 정리 이상으로 객체 구조의 핵심을 반영하는 요소입니다. 특히 상속과 다형성을 사용할 경우, 이들 함수가 동작하는 시점과 방식에 대한 정확한 이해는 안정적이고 오류 없는 코드를 작성하는 데 매우 중요합니다.