---
title: "[C#] 제네릭 인터페이스(Generic Interface)"
author: RePro
date: 2025-05-07 11:00:00 +0900
categories: [Programming, C#]
tags: [c#]
---

# C#의 제네릭 인터페이스(Generic Interface)

## 1. 개념 정의

**제네릭 인터페이스**란, 인터페이스 정의 시 **타입 매개변수(Generic Type Parameter)** 를 지정하여 다양한 타입에 대해 일관된 계약(contract)을 제공할 수 있도록 만든 **유형 일반화 인터페이스**이다.

```csharp
public interface IRepository<T>
{
    void Add(T item);
    T GetById(int id);
}
```

---

## 2. 필요성과 장점

### 비제네릭 인터페이스의 한계

```csharp
public interface IRepository
{
    void Add(object item);
    object GetById(int id);
}
```

* 타입 안전성 없음
* 형변환 필요
* 런타임 오류 가능성

### 제네릭 인터페이스의 장점

* 컴파일 타임 타입 안정성
* 형변환 없이 구체 타입 그대로 사용
* 다양한 엔티티 타입에 재사용 가능

---

## 3. 사용 예제

### 인터페이스 정의

```csharp
public interface IRepository<T>
{
    void Add(T item);
    T GetById(int id);
}
```

### 구현 클래스

```csharp
public class UserRepository : IRepository<User>
{
    private List<User> users = new();

    public void Add(User item) => users.Add(item);
    public User GetById(int id) => users.FirstOrDefault(u => u.Id == id);
}
```

---

## 4. 다형성과 타입 지정

```csharp
IRepository<User> repo = new UserRepository();
repo.Add(new User { Id = 1, Name = "홍길동" });
```

---

## 5. DI와의 결합 예시

```csharp
services.AddScoped<IRepository<User>, UserRepository>();
services.AddScoped<IRepository<Product>, ProductRepository>();
```

---

## 6. 다중 제네릭 매개변수 인터페이스

```csharp
public interface IMapper<TSource, TDestination>
{
    TDestination Map(TSource source);
}
```

---

## 7. 서비스 구조와 결합 예시

```csharp
public class Service<T>
{
    private readonly IRepository<T> repository;

    public Service(IRepository<T> repo)
    {
        repository = repo;
    }

    public T Find(int id) => repository.GetById(id);
}
```

---

## 8. 제약 조건과 상속 예시

```csharp
public interface IEntity { int Id { get; } }

public interface IRepository<T> where T : IEntity
{
    T GetById(int id);
}
```

---

## 9. 요약 정리

| 항목      | 설명                          |
| ------- | --------------------------- |
| 재사용성    | 다양한 타입에 대해 구현 가능            |
| 타입 안정성  | 컴파일 타임에서 오류 사전 방지           |
| DI 호환성  | IoC 컨테이너와 결합하여 유연한 구조 구성 가능 |
| 유지보수 용이 | 인터페이스 변경 없이 구현체 확장 가능       |

---

## 결론

**제네릭 인터페이스**는 C#에서 **유연성과 타입 안정성**을 모두 제공하는 강력한 기능이다. 서비스 추상화, 리포지토리 패턴, 매퍼 패턴 등 여러 디자인 패턴에서 중심 역할을 하며, 의존성 주입 구조와도 궁합이 좋다.
