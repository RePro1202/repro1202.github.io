---
title: "[CS] 메모리 풀링"
author: RePro
date: 2025-02-13 10:00:00 +0900
categories: [CS]
tags: [cs, c++]
---

# **메모리 풀 (Memory Pool) 개념 및 이론적 설명**

## **1. 메모리 풀(Memory Pool)이란?**

**메모리 풀**은 미리 일정 크기의 메모리를 할당해 두고, 필요할 때 이를 나누어 제공하고 다시 반환받아 재사용하는 메모리 관리 기법입니다.\
즉, **동적 메모리 할당과 해제의 비용을 줄이기 위해 메모리를 미리 할당하여 효율적으로 관리하는 기법**입니다.

### **왜 필요한가?**

- **할당/해제 성능 향상** → `malloc/free`는 느리고, 메모리 파편화(Fragmentation)를 유발할 수 있음.
- **일관된 메모리 관리** → 특정 크기의 객체를 자주 생성/소멸하는 경우 효율적임.
- **실시간 시스템에서 중요** → 실시간 게임, 네트워크 서버, 임베디드 시스템 등에서는 예측 가능한 성능이 필요.

---

## **2. 메모리 풀의 동작 방식**

### **1) 초기화 단계**

- 프로그램 시작 시, 또는 특정 시점에 **큰 덩어리의 메모리(Chunk)** 를 할당.
- 이를 **슬롯(Slot) 또는 블록(Block)** 으로 나누어 관리.

### **2) 할당(Allocation)**

- 메모리가 필요할 때, 미리 할당된 **사용 가능한 블록을 반환**.
- 새로운 `malloc` 호출 없이 즉시 메모리를 사용할 수 있음.

### **3) 해제(Deallocation)**

- 메모리를 반납할 때, **실제 해제하지 않고 메모리 풀에 반환**.
- 반환된 블록은 다시 사용할 수 있도록 목록(Freelist)에 저장됨.

### **기본적인 메모리 풀 코드 예시시**

```cpp
class MemoryPool {
private:
    std::vector<void*> freeList; // 사용할 수 있는 블록 목록
    size_t blockSize;

public:
    MemoryPool(size_t size, size_t count) : blockSize(size) {
        for (size_t i = 0; i < count; ++i) {
            freeList.push_back(malloc(size));  // 미리 메모리 할당
        }
    }

    void* allocate() {
        if (freeList.empty()) return nullptr;  // 풀에 메모리가 없으면 nullptr 반환
        void* ptr = freeList.back();
        freeList.pop_back();
        return ptr;
    }

    void deallocate(void* ptr) {
        freeList.push_back(ptr); // 반납된 메모리를 다시 풀에 추가
    }

    ~MemoryPool() {
        for (void* ptr : freeList) {
            free(ptr);
        }
    }
};
```

 **할당 요청 시 미리 준비된 블록을 반환하고, 해제 시 다시 리스트에 추가하는 방식.**

---

## **3. 메모리 풀의 장점**

**1) 빠른 메모리 할당 및 해제**

- `malloc/free` 대신 사전 할당된 블록을 활용하므로 속도가 빠름.
- **O(1) 시간 복잡도로 동작 가능**, 즉 빠르게 메모리를 재사용.

**2) 메모리 파편화(Fragmentation) 방지**

- 일반적인 동적 할당(`malloc/new`)은 내부 및 외부 파편화 발생 가능.
- 메모리 풀은 **고정된 크기의 블록**을 재사용하므로 파편화 문제를 줄임.

**3) 실시간 성능 유지**

- 게임, 네트워크 서버, 임베디드 시스템 등 **실시간 응답성이 중요한 환경에서 예측 가능한 성능** 제공.

**4) 메모리 관리가 용이**

- 특정 객체의 수명이 종료될 때 전체 메모리 풀을 한 번에 정리할 수 있음.

---

## **4. 메모리 풀의 단점 및 주의점**

**1) 초기 메모리 사용량 증가**

- 미리 메모리를 할당하므로 초기에 **메모리 사용량이 많아질 수 있음**.

**2) 크기 변화에 대한 유연성이 낮음**

- **고정된 크기의 객체**에 최적화되어 있어 크기가 가변적인 객체에는 적절하지 않을 수 있음.

**3) 메모리 낭비 가능성**

- 필요 이상으로 많은 블록을 할당하면 프로그램의 메모리 사용량이 증가할 위험이 있음.

**4) 멀티스레드 환경에서 동기화 문제**

- 여러 스레드에서 동시에 메모리 풀을 사용하면 **경쟁 조건(Race Condition)** 발생 가능.

---

## **5. 메모리 풀의 활용 사례**

### **1) 게임 개발**

- **게임 객체 관리 (예: 탄환, NPC, 파티클 등)**
  - 탄환이 사라질 때마다 `new` / `delete` 하면 성능 문제가 발생.
  - **오브젝트 풀(Object Pool) 패턴**과 결합하여 활용.

### **2) 네트워크 서버**

- 패킷(Buffer) 관리
  - 네트워크 서버에서 클라이언트와 데이터를 주고받을 때, 매번 `malloc/free` 하면 성능 저하.
  - **고정 크기의 버퍼를 미리 생성**하여 빠르게 할당/해제.

### **3) 실시간 시스템 (임베디드, 금융 시스템)**

- 실시간 환경에서는 **메모리 할당의 예측 가능성**이 중요.
- 메모리 풀을 활용하면 **할당/해제의 지연 없이 일정한 성능 유지** 가능.

---

## **6. 메모리 풀 vs 오브젝트 풀**

메모리 풀과 오브젝트 풀은 비슷하지만 약간의 차이가 있습니다.

| 구분        | 메모리 풀 (Memory Pool) | 오브젝트 풀 (Object Pool) |
| --------- | ------------------- | -------------------- |
| **주요 목적** | 메모리 할당/해제 비용 감소     | 객체 생성/소멸 비용 감소       |
| **관리 단위** | 메모리 블록 (raw memory) | 객체 인스턴스 (C++ 클래스 등)  |
| **사용 방식** | `malloc/free` 대체    | `new/delete` 대체      |
| **예제**    | 네트워크 버퍼, 그래픽 리소스    | 총알, 적 캐릭터, UI 요소     |

 **즉, 오브젝트 풀은 메모리 풀을 기반으로 "객체"까지 포함해서 관리하는 방식**입니다.

---

## **7. 결론**

- **메모리 풀은 미리 할당된 메모리를 재사용하여 성능을 최적화하는 기법**.
- **게임, 네트워크 서버, 실시간 시스템 등에서 사용됨**.
- **빠른 할당/해제와 메모리 파편화 방지의 장점이 있지만, 초기 메모리 사용량 증가 등의 단점도 존재**.
- **멀티스레드 환경에서는 동기화 고려가 필요**.
- **오브젝트 풀과 혼용되어 게임 개발에서 자주 활용됨**.

---

# C++ 메모리 풀링 실험

## 1. 간단한 객체 풀 (Static Object Pool)
### 코드
```cpp
#include <iostream>
#include <vector>

class TestObject {
public:
    int a, b, c;
    float x, y, z;

    void initialize(int val) { a = b = c = val; x = y = z = val * 1.0f; }
};

class ObjectPool {
private:
    std::vector<TestObject> pool;
    std::vector<bool> available;
public:
    ObjectPool(size_t size) : pool(size), available(size, true) {}

    TestObject* allocate() {
        for (size_t i = 0; i < pool.size(); ++i) {
            if (available[i]) {
                available[i] = false;
                return &pool[i];
            }
        }
        return nullptr; // 풀이 가득 찼음
    }

    void deallocate(TestObject* obj) {
        size_t index = obj - &pool[0]; // 객체 위치 찾기
        if (index < pool.size()) {
            available[index] = true;
        }
    }
};

int main() {
    ObjectPool pool(10);

    TestObject* obj1 = pool.allocate();
    obj1->initialize(10);
    std::cout << "Allocated obj1: " << obj1->a << std::endl;

    pool.deallocate(obj1);
    std::cout << "Deallocated obj1\n";

    return 0;
}
```

---

## 2. 스마트 포인터 기반 객체 풀
### 코드
```cpp
#include <iostream>
#include <vector>
#include <memory>

class TestObject {
public:
    int a, b, c;
    float x, y, z;
    TestObject() { std::cout << "TestObject 생성\n"; }
    ~TestObject() { std::cout << "TestObject 소멸\n"; }
};

class ObjectPool {
private:
    std::vector<std::unique_ptr<TestObject>> pool;
public:
    ObjectPool(size_t size) {
        for (size_t i = 0; i < size; ++i) {
            pool.push_back(std::make_unique<TestObject>());
        }
    }

    std::unique_ptr<TestObject> allocate() {
        if (!pool.empty()) {
            std::unique_ptr<TestObject> obj = std::move(pool.back());
            pool.pop_back();
            return obj;
        }
        return nullptr; // 풀이 비었음
    }

    void deallocate(std::unique_ptr<TestObject> obj) {
        pool.push_back(std::move(obj));
    }
};

int main() {
    ObjectPool pool(5);

    auto obj1 = pool.allocate();
    auto obj2 = pool.allocate();

    std::cout << "객체 사용 중...\n";

    pool.deallocate(std::move(obj1));
    pool.deallocate(std::move(obj2));

    return 0;
}
```

---

## 3. 커스텀 메모리 풀 & 성능 비교
### 코드
```cpp
#include <iostream>
#include <vector>
#include <chrono>

class MemoryPool {
private:
    struct Node {
        Node* next;
    };

    std::vector<char> memoryBlock;
    Node* freeList;
    size_t chunkSize;
    size_t poolSize;

public:
    MemoryPool(size_t chunkSize, size_t poolSize) 
        : chunkSize(chunkSize), poolSize(poolSize), freeList(nullptr) {
        memoryBlock.resize(chunkSize * poolSize);
        
        for (size_t i = 0; i < poolSize; ++i) {
            void* ptr = &memoryBlock[i * chunkSize];
            reinterpret_cast<Node*>(ptr)->next = freeList;
            freeList = reinterpret_cast<Node*>(ptr);
        }
    }

    void* allocate() {
        if (!freeList) {
            std::cerr << "메모리 풀 고갈!" << std::endl;
            return nullptr;
        }
        void* allocatedMemory = freeList;
        freeList = freeList->next;
        return allocatedMemory;
    }

    void deallocate(void* ptr) {
        if (!ptr) return;
        reinterpret_cast<Node*>(ptr)->next = freeList;
        freeList = reinterpret_cast<Node*>(ptr);
    }
};

struct TestObject {
    int a, b, c;
    float x, y, z;
};

void benchmarkNewDelete(size_t iterations) {
    auto start = std::chrono::high_resolution_clock::now();
    for (size_t i = 0; i < iterations; ++i) {
        TestObject* obj = new TestObject();
        delete obj;
    }
    auto end = std::chrono::high_resolution_clock::now();
    std::cout << "new/delete 실행 시간: "
              << std::chrono::duration<double, std::milli>(end - start).count()
              << " ms\n";
}

void benchmarkMemoryPool(size_t iterations, MemoryPool& pool) {
    std::vector<TestObject*> allocatedObjects;
    allocatedObjects.reserve(iterations);

    auto start = std::chrono::high_resolution_clock::now();
    for (size_t i = 0; i < iterations; ++i) {
        allocatedObjects.push_back(static_cast<TestObject*>(pool.allocate()));
    }
    for (auto obj : allocatedObjects) {
        pool.deallocate(obj);
    }
    auto end = std::chrono::high_resolution_clock::now();
    std::cout << "메모리 풀 실행 시간: "
              << std::chrono::duration<double, std::milli>(end - start).count()
              << " ms\n";
}

int main() {
    constexpr size_t ITERATIONS = 100000;
    constexpr size_t CHUNK_SIZE = sizeof(TestObject);
    constexpr size_t POOL_SIZE = ITERATIONS;

    MemoryPool pool(CHUNK_SIZE, POOL_SIZE);

    std::cout << "성능 비교 시작...\n";
    benchmarkNewDelete(ITERATIONS);
    benchmarkMemoryPool(ITERATIONS, pool);

    return 0;
}
```

---

## 4. 정리

| 방식 | 특징 | 장점 | 단점 |
|------|------|------|------|
| **1. 간단한 객체 풀** | 미리 생성된 객체를 벡터에서 관리 | 매우 빠름, 동적 할당 없음 | 풀 크기 고정, 유연성 낮음 |
| **2. 스마트 포인터 풀** | `std::unique_ptr` 활용 | 자동 메모리 관리, 안전성 ↑ | `std::move` 필요, 약간의 성능 손실 |
| **3. 커스텀 메모리 풀** | 직접 메모리 블록 관리 | 유연성 높음, 빠른 할당/해제 | 구현 복잡 |
