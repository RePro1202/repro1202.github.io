---
title: "[C++] 멀티스레드, 데드락과 경쟁 상태"
author: RePro
date: 2024-09-05 11:00:00 +0900
categories: [Programming, C++]
tags: [c++]
---

# C++ 멀티스레드에서의 데드락(Deadlock)과 경쟁 상태(Race Condition)

멀티스레드 프로그래밍에서는 성능 향상을 위해 여러 스레드를 사용하지만, 공유 자원 접근 시 **데드락**이나 **경쟁 상태**와 같은 문제에 주의해야 합니다. 이 문서에서는 이 두 가지 주요 문제를 구체적인 예제와 함께 설명합니다.

---

## 1. 데드락 (Deadlock)

### 정의
두 개 이상의 스레드가 서로 자원을 점유한 상태에서, 서로의 자원을 기다리며 **무한히 대기 상태에 빠지는 현상**입니다.

### 예시 코드
```cpp
std::mutex m1, m2;

void thread1() {
    std::lock_guard<std::mutex> lock1(m1);
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    std::lock_guard<std::mutex> lock2(m2);  // m2 대기
}

void thread2() {
    std::lock_guard<std::mutex> lock2(m2);
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    std::lock_guard<std::mutex> lock1(m1);  // m1 대기
}
```

### 발생 조건 (Coffman 조건)
| 조건 | 설명 |
|------|------|
| 상호 배제 | 자원을 하나의 스레드만 사용할 수 있음 |
| 점유와 대기 | 자원을 점유한 상태에서 다른 자원을 기다림 |
| 비선점 | 자원을 강제로 뺏을 수 없음 |
| 순환 대기 | 원형으로 자원을 기다리는 상태가 있음 |

### 예방 방법
- 락 순서 고정 (`std::lock(m1, m2)` 사용)
- `std::scoped_lock`, `std::lock_guard`로 동시 락 획득
- `try_lock`으로 타임아웃 시도
- 리소스 최소화 및 락 범위 최소화

### 안전한 락 예시
```cpp
std::scoped_lock lock(m1, m2);  // C++17 이상 안전 락
```

---

## 2. 경쟁 상태 (Race Condition)

### 정의
두 개 이상의 스레드가 **동시에 공유 자원에 접근**할 때, 실행 순서에 따라 결과가 **비정상적으로 달라지는 상황**입니다.

### 예시 코드
```cpp
int counter = 0;

void increment() {
    for (int i = 0; i < 100000; ++i)
        ++counter;  // 위험: 공유 자원 수정
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);

    t1.join();
    t2.join();

    std::cout << "counter: " << counter << "\n";  // 결과 불확실
}
```

### 발생 이유
- `counter++`는 원자적 연산이 아님 (읽기 → 증가 → 쓰기)
- 두 스레드가 동시에 같은 값을 읽고 덮어씌움 → 손실 발생

### 해결 방법
| 방법 | 설명 |
|------|------|
| `std::mutex` | 락으로 공유 자원 보호 |
| `std::atomic<int>` | 원자적 연산 사용 |
| `std::lock_guard` | 안전한 락 범위 보장 |
| ThreadSanitizer | 런타임 경쟁 상태 탐지 툴 |

### 예시 (atomic으로 수정)
```cpp
std::atomic<int> counter = 0;

void increment() {
    for (int i = 0; i < 100000; ++i)
        ++counter;
}
```

---

## 3. 데드락 vs 경쟁 상태 비교

| 항목 | 데드락 | 경쟁 상태 |
|------|--------|------------|
| 정의 | 서로 자원을 기다리며 무한 대기 | 동시에 자원 접근 → 결과 예측 불가 |
| 증상 | 프로그램이 멈춤 | 계산 오류, 데이터 손상 |
| 탐지 | 상대적으로 명확 | 디버깅 매우 어려움 |
| 해결 | 락 순서 통일, scoped_lock | mutex, atomic 사용 |

---

## 결론
멀티스레드 프로그래밍에서는 반드시 다음을 고려해야 합니다:
- **락 사용 시 순서와 범위**를 철저히 관리해 데드락 방지
- **공유 변수는 atomic 또는 mutex로 보호**하여 경쟁 상태 방지

안전한 병렬 처리를 위해서는 `std::mutex`, `std::atomic`, `scoped_lock`, `thread sanitizer` 등 C++의 동기화 도구를 적극 활용해야 합니다.

