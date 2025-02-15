---
title: 가상 메모리
author: RePro
date: 2025-01-13 10:00:00 +0900
categories: [Blogging, Tutorial]
tags: [favicon]
---

# 스택과 힙의 메모리 주소 관계 및 실제 배치

## 1. 가상 메모리(Virtual Memory)란?

가상 메모리는 운영체제가 제공하는 메모리 관리 기법으로, 각 프로세스마다 독립적인 메모리 공간을 제공하여 실제 물리 메모리와 무관하게 연속된 주소 공간을 사용하는 것처럼 보이게 한다. 이를 통해 다음과 같은 이점이 있다.

- **프로세스 간 메모리 보호:** 한 프로세스가 다른 프로세스의 메모리에 접근하지 못하도록 보호한다.
- **주소 공간 독립성:** 각 프로세스는 자신의 가상 메모리 주소 공간을 가지며, 다른 프로세스와 충돌 없이 독립적으로 실행된다.
- **메모리 확장:** 실제 물리 메모리가 부족하더라도 디스크의 일부를 사용하여 부족한 메모리를 보완(스왑)할 수 있다.
- **효율적인 메모리 사용:** 필요한 부분만 물리 메모리에 로드하고 나머지는 디스크에 둠으로써 효율적인 메모리 사용이 가능하다.

운영체제는 가상 메모리를 페이지(page) 단위로 나누고, 페이지 테이블(Page Table)을 통해 가상 주소를 물리 주소로 변환하여 실제 메모리에 접근하도록 한다.

## 2. "스택은 높은 주소, 힙은 낮은 주소"라는 말의 의미

일반적으로 스택은 높은 주소부터, 힙은 낮은 주소부터 부여된다고 알려져 있다. 이는 가상 메모리 주소 공간(Logical/Virtual Address Space) 기준의 설명이다.

### 가상 메모리 기준으로 흔히 그리는 구조

```
높은 주소
+-------------------+
|      스택(Stack)  |  ↓ 밑으로 성장
|-------------------|
|      ...          |
|-------------------|
|      힙(Heap)     |  ↑ 위로 성장
+-------------------+
낮은 주소
```

- **스택(Stack)**: 함수 호출이나 지역 변수 생성 시, 높은 주소에서 낮은 주소 방향으로 확장(감소)한다.
- **힙(Heap)**: `new`, `malloc()` 등으로 동적 메모리를 할당하면, 낮은 주소에서 높은 주소 방향으로 확장(증가)한다.

이는 가상 주소 공간에서의 배치 모습일 뿐이며, 물리 메모리 주소와 직접적인 관계는 없다.

## 3. 가상 주소와 물리 주소는 다르다

실제 메모리(RAM)에는 스택이 위, 힙이 아래로 정렬되어 들어가는 것이 아니다. 운영체제가 페이지 테이블(Page Table)을 통해 가상 주소를 물리 주소에 매핑하기 때문에, 가상 주소 공간에서 스택이 힙보다 높다고 해서, 물리 주소에서도 스택이 힙보다 높은 주소에 있다는 보장은 없다.

### 예시

| 가상 주소 (논리 주소) | 물리 주소 (실제 RAM) |
|------------------|------------------|
| 스택의 `0x7FFFxxxx` | `0x20000000` |
| 힙의 `0x60000000` | `0x30000000` |

위와 같이 가상 주소는 스택이 힙보다 높지만, 물리 주소에서는 힙이 더 높은 경우도 충분히 있을 수 있다.

## 4. 스택과 힙의 물리 주소 관계는 OS와 상황에 따라 다르다

같은 프로그램을 여러 번 실행해도 물리 메모리 배치는 달라질 수 있다. 페이지 테이블에 의해 가상 주소와 물리 주소가 동적으로 매핑되므로, 스택이 항상 물리적으로 더 높은 주소에 있다고는 단정할 수 없다.

## 5. 실제 주소 비교해보기 (리눅스에서)

리눅스에서는 `/proc/self/pagemap`을 활용해 가상 주소와 물리 주소를 확인할 수 있다.

```cpp
#include <iostream>
#include <fstream>

#define PAGE_SIZE 4096

uintptr_t get_physical_address(uintptr_t virtual_address) {
    std::ifstream pagemap("/proc/self/pagemap", std::ios::binary);
    if (!pagemap) {
        std::cerr << "Failed to open /proc/self/pagemap" << std::endl;
        return 0;
    }

    uintptr_t page_index = virtual_address / PAGE_SIZE;
    uintptr_t offset = page_index * sizeof(uint64_t);

    pagemap.seekg(offset, std::ios::beg);
    uint64_t entry;
    pagemap.read(reinterpret_cast<char*>(&entry), sizeof(entry));

    if (!(entry & (1ULL << 63))) {
        std::cerr << "Page not present in memory" << std::endl;
        return 0;
    }

    uintptr_t physical_address = (entry & ((1ULL << 55) - 1)) * PAGE_SIZE + (virtual_address % PAGE_SIZE);
    return physical_address;
}

int main() {
    int stackVar = 42;
    int* heapVar = new int(100);

    uintptr_t stack_virtual = reinterpret_cast<uintptr_t>(&stackVar);
    uintptr_t heap_virtual = reinterpret_cast<uintptr_t>(heapVar);

    uintptr_t stack_physical = get_physical_address(stack_virtual);
    uintptr_t heap_physical = get_physical_address(heap_virtual);

    std::cout << "Stack Virtual Address: " << std::hex << stack_virtual << std::endl;
    std::cout << "Stack Physical Address: " << std::hex << stack_physical << std::endl;

    std::cout << "Heap Virtual Address: " << std::hex << heap_virtual << std::endl;
    std::cout << "Heap Physical Address: " << std::hex << heap_physical << std::endl;

    delete heapVar;

    return 0;
}
```

이 코드를 실행하면 가상 주소와 물리 주소가 우리가 예상한 배치와 다를 수 있다는 것을 직접 확인할 수 있다.

## 6. 결론

| 기준 | 스택이 힙보다 항상 높은가? |
|------|------------------------|
| **가상 주소** | 보통 높음 (높은 주소부터 내려감) |
| **물리 주소** | 항상 높다고 할 수 없음 (OS가 임의 배치) |

- 가상 주소 공간에서는 스택이 힙보다 항상 높다.
- 하지만 물리 주소에서는 스택이 힙보다 항상 높다고 보장할 수 없다.
- 페이지 테이블이 가상 주소와 물리 주소를 동적으로 매핑하기 때문이다.

---

인위적 연속성(artificial continuity)이란 가상기억장치의 개념에서 가상공간의 연속적 주소가 실제 물리적인 공간상에서 연속일 필요가 없다는 것을 뜻하는 말이다. 사상 표를 통해서 가상기억장치의 주소로부터 실기억장치의 주소를 알 수 있다.