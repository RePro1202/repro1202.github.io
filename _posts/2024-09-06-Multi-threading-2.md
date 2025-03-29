---
title: "[C++] 멀티스레드, 세마포어와 뮤텍스"
author: RePro
date: 2024-09-06 11:00:00 +0900
categories: [Programming, C++]
tags: [c++]
---

# C++에서의 세마포어(Semaphore)와 뮤텍스(Mutex)

멀티스레드 프로그래밍에서 **동기화(synchronization)** 는 매우 중요한 이슈이며, 이를 해결하기 위한 대표적인 도구가 **세마포어**와 **뮤텍스**입니다. 이 문서에서는 두 개념의 차이, 용도, 예제는 물론, Producer-Consumer 문제, OS별 구현 차이, 다양한 C++ 뮤텍스 종류까지 함께 정리합니다.

---

## 1. 뮤텍스 (Mutex: Mutual Exclusion)

### 정의
- 한 번에 하나의 스레드만 공유 자원에 접근하도록 허용하는 락
- **상호 배제**를 보장하여 크리티컬 섹션 보호

### 상세
- 내부적으로 **플래그(locked/unlocked)** 상태를 가짐
- `lock()` 함수 호출 시:
  - 뮤텍스가 잠겨 있지 않으면 잠그고 반환
  - 잠겨 있으면 **해제될 때까지 블로킹됨**
- `unlock()` 호출 시:
  - 다음 대기 중인 스레드에 락을 넘김 또는 해제

### 구현
- 대부분의 OS에서는 **커널 수준 객체**로 관리
- 일부는 사용자 수준 뮤텍스와 커널 뮤텍스 혼합 구조
- deadlock 방지를 위해 `try_lock()`, `timed_lock()`도 제공됨

### 사용 예제
```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx;

void printSafe(const std::string& msg) {
    std::lock_guard<std::mutex> lock(mtx);
    std::cout << msg << std::endl;
}

int main() {
    std::thread t1(printSafe, "Thread 1");
    std::thread t2(printSafe, "Thread 2");
    t1.join();
    t2.join();
    return 0;
}
```

---

## 2. 세마포어 (Semaphore)

### 정의
- 정수 값을 사용해 **리소스 개수**를 관리하는 동기화 도구
- 값이 0이면 대기, 양수면 감소하고 통과

### 상세
- **공유 자원에 접근 가능한 개수(N)** 를 표현하는 정수
- `acquire()` 또는 `wait()`를 호출하면:
  - 값이 0보다 크면 1 감소하고 통과
  - 값이 0이면 대기 상태로 전환
- `release()` 또는 `signal()`을 호출하면:
  - 값이 1 증가하고 대기 중인 스레드 하나를 깨움

### 구현
- POSIX/Linux: `sem_t` 구조체 기반으로 관리
- Windows: `HANDLE` 객체로 관리되며, 커널 리소스를 사용
- C++20: `std::counting_semaphore`, `std::binary_semaphore` 표준 제공 (원자적 카운트 + 내부 대기 큐)

### C++20 예제 (`std::counting_semaphore`)
```cpp
#include <iostream>
#include <thread>
#include <semaphore>

std::counting_semaphore<3> sem(3);  // 최대 3개 동시 허용

void task(int id) {
    sem.acquire();
    std::cout << "Task " << id << " running\n";
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::cout << "Task " << id << " finished\n";
    sem.release();
}

int main() {
    std::thread t1(task, 1);
    std::thread t2(task, 2);
    std::thread t3(task, 3);
    std::thread t4(task, 4);
    t1.join(); t2.join(); t3.join(); t4.join();
    return 0;
}
```

---

## 3. Producer-Consumer 문제 (세마포어 활용)

```cpp
#include <iostream>
#include <thread>
#include <queue>
#include <mutex>
#include <condition_variable>

std::queue<int> buffer;
const unsigned int MAX_SIZE = 5;
std::mutex mtx;
std::condition_variable cv_producer, cv_consumer;

void producer() {
    for (int i = 1; i <= 10; ++i) {
        std::unique_lock<std::mutex> lock(mtx);
        cv_producer.wait(lock, [] { return buffer.size() < MAX_SIZE; });

        buffer.push(i);
        std::cout << "Produced: " << i << std::endl;

        cv_consumer.notify_one();
    }
}

void consumer() {
    for (int i = 1; i <= 10; ++i) {
        std::unique_lock<std::mutex> lock(mtx);
        cv_consumer.wait(lock, [] { return !buffer.empty(); });

        int val = buffer.front();
        buffer.pop();
        std::cout << "Consumed: " << val << std::endl;

        cv_producer.notify_one();
    }
}

int main() {
    std::thread p(producer);
    std::thread c(consumer);
    p.join();
    c.join();
    return 0;
}
```

---

## 4. OS별 세마포어 구현 차이

| 항목 | Windows | Linux (POSIX) |
|------|---------|----------------|
| API 이름 | CreateSemaphore / WaitForSingleObject | `sem_t`, `sem_init`, `sem_wait` |
| 초기값 설정 | `CreateSemaphore(NULL, initial, max, NULL)` | `sem_init(&sem, 0, initial)` |
| C++20 표준 | `std::counting_semaphore` 지원 | GCC 11 이상에서 지원 |
| 주의점 | 커널 객체 (핸들) | POSIX 스레드 라이브러리 필요 |

> C++20 이상에서는 플랫폼 독립적으로 `std::counting_semaphore`, `std::binary_semaphore` 사용 권장

---

## 5. C++의 다양한 뮤텍스 종류

### `std::recursive_mutex`
- **자기 자신을 여러 번 lock 할 수 있음** (재귀적으로 호출되는 함수에 유용)
```cpp
#include <iostream>
#include <mutex>

std::recursive_mutex rec_mtx;

void recursive(int count) {
    if (count == 0) return;
    rec_mtx.lock();
    std::cout << "recursing: " << count << std::endl;
    recursive(count - 1);
    rec_mtx.unlock();
}
```

### `std::shared_mutex` (C++17)
- 여러 **reader(읽기)** 는 동시에 접근 가능
- **writer(쓰기)** 는 단독 접근

```cpp
#include <iostream>
#include <shared_mutex>
#include <thread>

std::shared_mutex sharedMtx;
int sharedData = 0;

void reader() {
    std::shared_lock lock(sharedMtx);
    std::cout << "Read: " << sharedData << std::endl;
}

void writer() {
    std::unique_lock lock(sharedMtx);
    sharedData++;
    std::cout << "Written: " << sharedData << std::endl;
}
```

---

## 6. 요약 비교표

| 항목 | 뮤텍스 | 세마포어 |
|------|--------|----------|
| 자원 제어 | 하나만 접근 | 여러 개 제한 가능 |
| 내부 값 | 0 또는 1 | 0 이상 정수형 |
| 소유권 개념 | 있음 (스레드 기반) | 없음 |
| 대표 구현 | `std::mutex`, `lock_guard` | `std::counting_semaphore` |
| 대표 용도 | 크리티컬 섹션 보호 | Producer-Consumer 등 제한 자원 제어 |

---

## 7. 참고: 락 프리 (Lock-Free)와 비교

| 항목 | 락 기반 | 락 프리 (`std::atomic`) |
|------|---------|------------------|
| 동기화 방식 | 락으로 보호 | 원자적 연산으로 충돌 방지 |
| 데드락 위험 | 있음 | 없음 |
| 성능 | 낮음 (락 비용) | 높음 (경량 연산) |
| 복잡도 | 비교적 쉬움 | 구현 난이도 높음 |
