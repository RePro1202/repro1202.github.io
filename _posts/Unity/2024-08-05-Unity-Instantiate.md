---
title: "[Unity] Instantiate"
author: RePro
date: 2024-08-05 10:00:00 +0900
categories: [Programming, Unity]
tags: [unity, c#]
---

# Unity에서의 GameObject.Instantiate() 함수 정리

Unity에서 `GameObject.Instantiate()` 함수는 **게임 오브젝트(GameObject)나 컴포넌트(Component)를 복제(인스턴스화)** 하는 데 사용됩니다.

---

## 1. 기본 사용법

```csharp
GameObject clone = Instantiate(original);
```

### 오버로드 예시

```csharp
// 기본 사용법
GameObject clone = Instantiate(original);

// 부모 설정
GameObject clone = Instantiate(original, parent);

// 위치와 회전 설정
GameObject clone = Instantiate(original, position, rotation);

// 부모와 함께 위치 설정
GameObject clone = Instantiate(original, parent, false); // 월드 좌표 유지
GameObject clone = Instantiate(original, parent, true);  // 부모의 로컬 좌표로 이동
```

---

## 2. 동작 원리

1. **메모리 할당**:  
   - Unity는 `Instantiate` 호출 시 복제본을 힙 메모리에 할당합니다.  
   - 복제된 오브젝트는 **원본의 모든 구성 요소를 포함**합니다.  

2. **복제 과정**:
   - GameObject와 모든 **컴포넌트(Component)** 를 복제합니다.  
   - **Transform 정보(위치, 회전, 크기)** 도 복제됩니다.  
   - 원본과 동일한 **자식 객체 계층 구조**까지 포함합니다.  

3. **컴포넌트 초기화**:
   - `Awake()` → `OnEnable()` 순으로 호출됩니다.  
   - **Start()** 는 다음 프레임에 호출됩니다.  
   - **복제본이 활성화 상태로 생성**되므로, 복제 후 즉시 사용 가능합니다.  

---

## 3. 유사 함수와 비교

| 함수                            | 역할                           | 특징                                       |
|--------------------------------|--------------------------------|--------------------------------------------|
| **Instantiate()**               | 오브젝트 복제                   | 가장 일반적인 복제 방식                     |
| **InstantiateAsync()**          | 비동기 복제                     | 대용량 오브젝트 복제 시 사용                 |
| **Object.Instantiate()**        | 컴포넌트 복제                   | 모든 Unity 객체 복제 가능                   |
| **InstantiatePrefab()**         | 프리팹 복제 전용                | 성능 최적화                                 |
| **ScriptableObject.CreateInstance()** | 데이터 객체 생성 | 메모리 부담이 적고 데이터 전용               |

---

## 4. InstantiateAsync()의 특징

- Unity 2021 이후부터 **비동기 인스턴스화**를 지원합니다.  
- `Addressables`를 사용하여 **비동기 로딩**을 할 수 있습니다.  

```csharp
var handle = Addressables.InstantiateAsync("BulletPrefab");
handle.Completed += (op) =>
{
    GameObject bullet = op.Result;
    bullet.GetComponent<Rigidbody>().AddForce(Vector3.forward * 10f);
};
```

- **대용량 오브젝트를 비동기 복제**할 때 사용  
- **로딩 중 프레임 끊김을 방지**  

---

## 5. Instantiate() 사용 시 주의사항

1. **메모리 누수**:
   - Instantiate로 생성한 오브젝트는 **직접 삭제**해야 합니다.
   - `Destroy(clone);`으로 제거하지 않으면 메모리 누수 발생

2. **프리팹 관리**:
   - 프리팹을 직접 수정하지 말고, 복제본에 수정 사항을 적용해야 합니다.  

3. **성능 문제**:
   - 실시간으로 많은 객체를 복제하면 **프레임 드랍**이 발생할 수 있습니다.  
   - **Object Pooling** 기법을 활용하여 성능 개선 가능  

---

## 6. Object Pooling 예제

```csharp
public class BulletPool : MonoBehaviour
{
    public GameObject bulletPrefab;
    private Queue<GameObject> pool = new Queue<GameObject>();

    void Start()
    {
        for (int i = 0; i < 10; i++)
        {
            GameObject bullet = Instantiate(bulletPrefab);
            bullet.SetActive(false);
            pool.Enqueue(bullet);
        }
    }

    public GameObject GetBullet()
    {
        if (pool.Count > 0)
        {
            GameObject bullet = pool.Dequeue();
            bullet.SetActive(true);
            return bullet;
        }
        return Instantiate(bulletPrefab);
    }

    public void ReturnBullet(GameObject bullet)
    {
        bullet.SetActive(false);
        pool.Enqueue(bullet);
    }
}
```

---

## 7. 정리

| 함수명                           | 용도                          | 특징                                      |
|--------------------------------|--------------------------------|-------------------------------------------|
| `Instantiate()`                 | 오브젝트를 복제                | 가장 일반적, 모든 컴포넌트 복제              |
| `InstantiateAsync()`            | 비동기 복제                    | 대용량 복제에 유리, 끊김 방지                |
| `Object.Instantiate()`          | 모든 Unity 객체 복제            | GameObject 외에도 사용 가능                 |
| `InstantiatePrefab()`           | 프리팹 복제 전용               | 성능 향상, 특정 상황에서만 사용              |
| `ScriptableObject.CreateInstance()` | 데이터 객체 생성 | 메모리 부담 적음, 데이터 전용                 |

- `GameObject.Instantiate()`는 가장 많이 사용되는 복제 메서드지만, 성능과 메모리 관리에 주의해야 합니다.  
- 비동기 방식이나 오브젝트 풀링을 적절히 사용하여 성능 문제를 해결하는 것이 중요합니다.  
