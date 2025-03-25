---
title: "[수학] 순열과 조합"
author: RePro
date: 2025-01-08 10:00:00 +0900
categories: [Programming, Math]
tags: [math, algorithm]
---

# 순열(Permutation)과 조합(Combination) 정리 (시간복잡도 포함)

게임 개발, 알고리즘 문제, 수학 전반에서 자주 쓰이는 개념인 **순열과 조합**에 대해  
개념, 수학 공식, 구현 방법, STL 활용, 중복 조합까지 완전 정리한 문서입니다.

---

## 1. 순열과 조합의 기본 개념 비교

| 구분 | 의미 | 예시 | 수학 기호 |
|------|------|------|-----------|
| 순열 | 순서를 고려한 선택 | 3명 중 2명을 뽑아 줄 세우는 경우 | P(n, r) 또는 nPr |
| 조합 | 순서를 고려하지 않는 선택 | 3명 중 2명을 뽑는 경우 | C(n, r) 또는 nCr |

---

## 2. 수학 공식

### 순열
$$
P(n, r) = \frac{n!}{(n - r)!}
$$

### 조합
$$
C(n, r) = \frac{n!}{r!(n - r)!}
$$

---

## 3. 팩토리얼 및 기본 구현

```cpp
long long factorial(int n) {
    long long res = 1;
    for (int i = 2; i <= n; ++i)
        res *= i;
    return res;
}
// 시간복잡도: O(n)
```

### 순열 함수

```cpp
long long permutation(int n, int r) {
    return factorial(n) / factorial(n - r);
}
// 시간복잡도: O(n)
```

### 조합 함수

```cpp
long long combination(int n, int r) {
    return factorial(n) / (factorial(r) * factorial(n - r));
}
// 시간복잡도: O(n)
```

---

## 4. 조합의 DP(파스칼 삼각형) 구현

```cpp
long long comb[101][101];

void buildCombinationTable(int maxN) {
    for (int n = 0; n <= maxN; ++n) {
        comb[n][0] = comb[n][n] = 1;
        for (int r = 1; r < n; ++r) {
            comb[n][r] = comb[n - 1][r - 1] + comb[n - 1][r];
        }
    }
}
// 시간복잡도: O(n^2)
```

---

## 5. STL로 순열 생성 – `next_permutation`

```cpp
std::vector<int> v = {1, 2, 3};
std::sort(v.begin(), v.end());
do {
    for (int x : v) std::cout << x << ' ';
    std::cout << '\n';
} while (std::next_permutation(v.begin(), v.end()));
// 시간복잡도: O(n! × n)
```

---

## 6. STL로 조합 생성 – 마스크 + `prev_permutation`

```cpp
std::vector<int> data = {10, 20, 30, 40, 50};
std::vector<int> mask(5, 0);
std::fill(mask.begin(), mask.begin() + 3, 1);

do {
    for (int i = 0; i < 5; ++i)
        if (mask[i]) std::cout << data[i] << ' ';
    std::cout << '\n';
} while (std::prev_permutation(mask.begin(), mask.end()));
// 시간복잡도: O(nCr × n)
```

### 왜 `prev_permutation`을 사용할까?

- 마스크 `{1,1,1,0,0}`은 내림차순 상태이므로
- `prev_permutation()`을 써야 사전순으로 모든 조합을 생성 가능

---

## 7. 조합의 백트래킹(DFS) 구현

```cpp
std::vector<int> comb;
void dfs(int start, int n, int r) {
    if (comb.size() == r) {
        for (int x : comb) std::cout << x << ' ';
        std::cout << '\n';
        return;
    }
    for (int i = start; i <= n; ++i) {
        comb.push_back(i);
        dfs(i + 1, n, r);
        comb.pop_back();
    }
}
// 시간복잡도: O(nCr)
```

---

## 8. 각 방식 비교 요약

| 목적 | 추천 방식 | 시간복잡도 | 특징 |
|------|-------------|-------------|--------|
| 조합 수 계산 | 공식 / DP | O(n) ~ O(n²) | 빠름 |
| 모든 조합 출력 | 마스크 + `prev_permutation` | O(nCr × n) | STL 활용 |
| 조건부 탐색 | 백트래킹(DFS) | O(nCr) | 유연성 뛰어남 |

---

## 9. `next_permutation` vs `prev_permutation` 정리

| 항목 | next_permutation | prev_permutation |
|------|------------------|------------------|
| 동작 방향 | 사전순 증가 | 사전순 감소 |
| 시작 정렬 | 오름차순 필요 | 내림차순 필요 |
| 전체 순열 생성 | 정렬 필수 | 역정렬 필수 |

---

## 10. 중복 조합

### 공식

$$
\binom{n + r - 1}{r}
$$

### 마스크 구성 (별 `*`, 구분자 `|`)

- 예: `**||` → 첫 번째 항목 2개 선택
- 총 칸 수: `n + r - 1`

### 예제 (A, B, C 중 2개 중복 선택)

```cpp
std::vector<std::string> base = {"A", "B", "C"};
std::vector<int> mask(4, 0);
std::fill(mask.begin(), mask.begin() + 2, 1);

do {
    int stars = 0;
    std::vector<std::string> result;
    for (int i = 0; i < 4; ++i) {
        if (mask[i]) stars++;
        else {
            result.insert(result.end(), stars, base[result.size()]);
            stars = 0;
        }
    }
    result.insert(result.end(), stars, base[result.size()]);
    for (auto& s : result) std::cout << s;
    std::cout << '\n';
} while (std::prev_permutation(mask.begin(), mask.end()));
// 시간복잡도: O(C(n+r-1, r) × n)
```

---

## 마무리 요약

| 항목 | 설명 |
|------|------|
| 순열 생성 | next_permutation 사용 (정렬 필요) |
| 조합 생성 | 마스크 + prev_permutation 사용 |
| 중복 조합 | `n + r - 1`칸에서 마스크 구성 |
| 조건부 조합 | DFS(백트래킹) 방식 적합 |
| 성능 비교 | 조합 출력은 O(nCr × n), 계산만 할 경우 O(n) 가능 |



void buildCombinationTable(int maxN) {
    for (int n = 0; n <= maxN; ++n) {
        comb[n][0] = comb[n][n] = 1;
        for (int r = 1; r < n; ++r) {
            comb[n][r] = comb[n - 1][r - 1] + comb[n - 1][r];
        }
    }
}
// 시간복잡도: O(n^2)
```

---

## 7. C++ STL로 순열 만들기 (`std::next_permutation`)

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3};
    do {
        for (int x : v)
            std::cout << x << ' ';
        std::cout << '\n';
    } while (std::next_permutation(v.begin(), v.end()));
}
// 시간복잡도: O(n! × n)
```

---

## 8. C++ STL로 조합 만들기 (`std::prev_permutation` + 마스크)

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    int n = 5, r = 3;
    std::vector<int> data = {10, 20, 30, 40, 50};
    std::vector<int> mask(n, 0);
    std::fill(mask.begin(), mask.begin() + r, 1);

    do {
        for (int i = 0; i < n; ++i) {
            if (mask[i])
                std::cout << data[i] << ' ';
        }
        std::cout << '\n';
    } while (std::prev_permutation(mask.begin(), mask.end()));
}
// 시간복잡도: O(nCr × n)
```

---

## 9. 요약 정리

| 항목 | 조합 (Combination) | 순열 (Permutation) |
|------|--------------------|---------------------|
| 의미 | 순서 없이 선택 | 순서 있게 선택 |
| 공식 | nCr = n! / r!(n-r)! | nPr = n! / (n-r)! |
| 사용 예 | 복권 번호 뽑기 | 줄 세우기, 경로 정하기 |
| STL 지원 | X (마스크 + perm 활용) | `next_permutation()` |
| 시간복잡도 | O(n) 또는 O(n^2) | O(n!) |

- `next_permutation`은 순열을 사전순으로 생성합니다.
- 조합을 사전순으로 출력하려면 입력과 마스크를 정렬한 뒤 `prev_permutation()` 사용이 효과적입니다.
- 필요 시 백트래킹 또는 DFS 방식으로도 생성 가능.

---

## 10. 조합 생성 방식 비교 (백트래킹 vs STL)


| 방식 | 특징 | 시간복잡도 | 장점 | 단점 |
|------|------|------------|------|------|
| STL `prev_permutation` (마스크) | 마스크 조작으로 조합 생성 | O(nCr × n) | 구현 간단, STL 지원 | 출력만 가능, 탐색 제어 어려움 |
| 백트래킹 (DFS) | 탐색 트리를 따라 직접 선택 | O(nCr) | 조건부 탐색 가능, 유연함 | 구현 복잡, 재귀 깊이 큼 |
| 수학적 계산 (nCr 함수) | 조합 수만 구함 | O(n) 또는 O(n²) | 빠름, 값만 구할 때 적합 | 직접 조합은 만들 수 없음 |

---

###  백트래킹 DFS 예제 (C++)

```cpp
#include <iostream>
#include <vector>
using namespace std;

vector<int> comb;
void dfs(int start, int n, int r) {
    if (comb.size() == r) {
        for (int x : comb) cout << x << ' ';
        cout << '\n';
        return;
    }
    for (int i = start; i <= n; ++i) {
        comb.push_back(i);
        dfs(i + 1, n, r);
        comb.pop_back();
    }
}

int main() {
    int n = 5, r = 3;
    dfs(1, n, r);
}
```

---

###  목적별 추천 방식

| 목적 | 추천 방식 |
|------|-----------|
| 모든 조합을 빠르게 출력 | STL 마스크 + `prev_permutation` |
| 조건부 조합 탐색 | 백트래킹 (DFS) |
| 조합 개수만 구함 | 수학 공식(nCr) 또는 DP |
