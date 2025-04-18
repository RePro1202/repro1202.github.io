---
title: "[C#] protected 키워"
author: RePro
date: 2025-03-29 10:00:00 +0900
categories: [Programming, C#]
tags: [c#]
---

# protected 키워드 정리

`protected`는 **C++**, **C#**, **Java** 등 **객체지향 프로그래밍(OOP)** 언어에서  
**접근 제어자**로 사용됩니다.  
주로 **상속 관계**에서 **멤버 변수나 메서드의 접근 범위를 제어**하는 데 사용됩니다.  

---

## protected의 특징  
1. **클래스 내부와 상속받은 자식 클래스에서 접근 가능**  
2. **다른 외부 클래스나 객체에서는 접근 불가능**  
3. **상속 관계를 염두에 둔 접근 제어**  
4. **함수와 클래스**에 사용할 수 있음  

---

## protected 멤버 변수 사용 예시

### C++에서의 예시
```cpp
#include <iostream>

class Base {
protected:
    int protectedValue;

public:
    Base() : protectedValue(100) {}
};

class Derived : public Base {
public:
    void ShowValue() {
        std::cout << "Protected Value: " << protectedValue << std::endl;
    }
};

int main() {
    Derived d;
    d.ShowValue();
    return 0;
}
```
출력:
```
Protected Value: 100
```

### C#에서의 예시
```csharp
using System;

class Base
{
    protected int protectedValue = 42;
}

class Derived : Base
{
    public void DisplayValue()
    {
        Console.WriteLine("Protected Value: " + protectedValue);
    }
}

class Program
{
    static void Main()
    {
        Derived d = new Derived();
        d.DisplayValue();
    }
}
```
출력:
```
Protected Value: 42
```

---

## protected 함수 사용 예시

### C++에서의 예시
```cpp
#include <iostream>

class Animal {
protected:
    void Speak() {
        std::cout << "Animal sound" << std::endl;
    }
};

class Dog : public Animal {
public:
    void Bark() {
        Speak();
    }
};

int main() {
    Dog dog;
    dog.Bark();
    return 0;
}
```
출력:
```
Animal sound
```

### C#에서의 예시
```csharp
using System;

class Animal
{
    protected void Speak()
    {
        Console.WriteLine("Animal sound");
    }
}

class Dog : Animal
{
    public void Bark()
    {
        Speak();
    }
}

class Program
{
    static void Main()
    {
        Dog dog = new Dog();
        dog.Bark();
    }
}
```
출력:
```
Animal sound
```

---

## protected 생성자 사용 예시

### C++에서의 예시
```cpp
#include <iostream>

class Base {
protected:
    Base() {
        std::cout << "Base constructor" << std::endl;
    }
};

class Derived : public Base {
public:
    Derived() {
        std::cout << "Derived constructor" << std::endl;
    }
};

int main() {
    Derived d;
    return 0;
}
```
출력:
```
Base constructor
Derived constructor
```

---

## protected 클래스 사용 예시

### C++에서의 예시
```cpp
#include <iostream>

class Outer {
protected:
    class Inner {
    public:
        void Greet() {
            std::cout << "Hello from Inner" << std::endl;
        }
    };
};

class Derived : public Outer {
public:
    void CallInner() {
        Inner inner;
        inner.Greet();
    }
};

int main() {
    Derived d;
    d.CallInner();
    return 0;
}
```
출력:
```
Hello from Inner
```

---

## protected와 다른 접근 제어자 비교

| 접근 제어자        | 동일 클래스 내부 | 상속받은 클래스 내부 | 외부 클래스 |
|------------------|----------------|---------------------|-------------|
| **public**        | O              | O                   | O           |
| **protected**     | O              | O                   | X           |
| **private**       | O              | X                   | X           |
| **internal** (C#) | O              | X                   | O (같은 어셈블리) |
| **private protected** (C#) | O | O (같은 어셈블리) | X |

---

## 정리
1. `protected`는 **클래스 내부**와 **상속받은 자식 클래스**에서만 접근할 수 있는 접근 제어자입니다.  
2. **함수**, **생성자**, **중첩 클래스** 등에 사용하여 **상속 구조에서 접근 범위를 제한**할 수 있습니다.  
3. **외부 접근이 불가능**하므로 **상속 구조를 명확히 관리**할 때 유용합니다.  
4. **public**과 **private**의 중간 단계로, **상속과 캡슐화**를 동시에 고려할 때 적절한 선택입니다.
