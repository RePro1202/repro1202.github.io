---
title: "[Unity] Gamekit3D 샘플 프로젝트 분석 1"
author: RePro
date: 2025-05-11 10:00:00 +0900
categories: [Programming, Unity]
tags: [unity]
---

# MeleeWeapon.cs

플레이어와 몬스터가 사용하는 공격에 대한 컴포넌트,
무기 or 몬스터의 공격 위치에 AttackRoot에 배치

## 주요 구조 및 역할
**클래스: Gamekit3D.MeleeWeapon**
- 무기를 휘두를 때의 충돌 감지, 피격 처리, 이펙트 재생, 사운드 출력 등을 담당
- SphereCastNonAlloc을 활용하여 부채꼴 모양이 아니라 구 형태로 검출
- 에디터 디버그용 Gizmos 표시까지 포함



## 멤버 정리
| 멤버                              | 설명                             |
| ------------------------------- | ------------------------------ |
| `damage`                        | 공격 데미지                         |
| `AttackPoint[] attackPoints`    | 공격 판정 지점들 (구 모양, 위치와 반지름 포함)   |
| `targetLayers`                  | 데미지를 줄 수 있는 레이어 마스크            |
| `hitParticlePrefab`             | 피격 이펙트 파티클                     |
| `hitAudio`, `attackAudio`       | 피격/공격 시 재생할 사운드                |
| `m_Owner`                       | 무기를 가진 캐릭터 (자기 자신은 무시)         |
| `m_InAttack`, `m_IsThrowingHit` | 공격 중인지, 투척 공격인지 여부             |
| `m_PreviousPos`                 | 각 공격 포인트의 이전 위치 저장 (이동 벡터 계산용) |


## 주요 동작 흐름
### 1. BeginAttack()
- `attackAudio` 재생
- 현재 공격 중임을 표시 (`m_InAttack = true`)
- `attackPoints` 위치를 기록해 이전 위치 초기화

### 2. FixedUpdate()
- 공격 중일 때만 작동
- 각 `AttackPoint`에 대해:
- 현재 위치에서 이전 위치로의 이동 벡터 계산
  - 이동한 방향으로 `SphereCastNonAlloc` 실행
  - 감지된 Collider에 대해 `CheckDamage()` 호출
  - 이전 위치 업데이트

```csharp
Ray r = new Ray(worldPos, attackVector.normalized);
Physics.SphereCastNonAlloc(r, pts.radius, ..., attackVector.magnitude);
```
- 움직이지 않았을 경우 미세한 forward 벡터로 대체 → 정지 시에도 판정 가능하게 함

### 3. CheckDamage()
- 충돌체에 `Damageable` 컴포넌트가 있는지 검사
- `m_Owner`와 동일한 객체라면 무시
- `targetLayers` 검사로 유효 대상인지 필터링
- 데미지를 적용하고 파티클과 사운드 출력

### 4. EndAttack()
- 공격 종료 처리 (`m_InAttack = false`)
- 에디터 디버깅용 위치 리스트 클리어


## 디버그 지원 기능
**OnDrawGizmosSelected()**
- 각 `AttackPoint`의 현재 위치를 구형으로 시각화
- `previousPositions` 리스트를 선으로 연결해서 공격 경로 시각화


<br>

---

# ChomperSMBPursuit.cs

## 개요

`ChomperSMBPursuit`는 Unity의 Gamekit3D 프레임워크에서 사용하는 `SceneLinkedSMB<ChomperBehavior>` 기반의 상태머신 동작 클래스입니다.  
이 클래스는 **Chomper AI 캐릭터가 추격(Pursuit) 상태일 때**의 로직을 담당합니다.

---

## 핵심 역할

- 타겟 감지 및 재탐색
- 타겟과의 거리 계산 후 공격 여부 판단
- `NavMeshAgent`를 통한 추격 지속 또는 중단
- 다중 AI 추격을 위한 슬롯 기반 포지셔닝

---

## 메서드 흐름: `OnSLStateNoTransitionUpdate()`

```csharp
public override void OnSLStateNoTransitionUpdate(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
```

> 애니메이션 상태가 유지되는 동안 매 프레임 호출

### 1. 타겟 탐색
```csharp
m_MonoBehaviour.FindTarget();
```

### 2. 경로 상태 확인 (유효하지 않으면 추격 중단)
```csharp
if (navmeshAgent.pathStatus == PathPartial || PathInvalid)
    StopPursuit();
```

### 3. 타겟 유효성 검사
```csharp
if (target == null || target.respawning)
    StopPursuit();
```

### 4. 거리 계산 및 공격 조건 판단
```csharp
if (toTarget.sqrMagnitude < attackDistance²)
    TriggerAttack();
```

### 5. 추격 유지 조건 (슬롯 위치 기반)
```csharp
if (assignedSlot != -1)
{
    Vector3 targetPoint = 타겟 위치 + 슬롯 방향 * 0.9 * 공격 거리;
    SetTarget(targetPoint);
}
```

### 6. 그 외 조건에서는 추격 종료
```csharp
else
{
    StopPursuit();
}
```

---

## 주요 메서드/변수 설명

| 이름 | 설명 |
|------|------|
| `FindTarget()` | 타겟 재탐색 수행 |
| `RequestTargetPosition()` | 현재 타겟 위치 요청 |
| `TriggerAttack()` | 공격 전환 수행 |
| `SetTarget(Vector3)` | NavMeshAgent의 목적지 설정 |
| `StopPursuit()` | 추격 상태 종료 처리 |
| `assignedSlot` | 다중 AI의 슬롯 ID (포메이션 위치) |
| `followerData.distributor` | 슬롯 분포를 제공하는 로직 |

---

## 장점 요약

-  **명확한 FSM 상태 분리**
-  **NavMesh 경로 유효성 체크 포함**
-  **공격 거리 판정은 `sqrMagnitude`로 최적화**
-  **다중 AI 분산 추격(formation) 지원**
-  **타겟 사라짐/리스폰 상태 대응**

---

이 코드는 Gamekit3D 기반 FSM에서 `Chomper` AI의 **추격 상태(Pursuit State)** 를 정의합니다.

- 타겟이 없거나 거리가 너무 멀면 추격 종료
- 공격 범위 안이면 공격 전환
- 그 외에는 formation을 유지하며 타겟 추적

이 상태는 추후 Idle → Pursuit → Attack → Back 등의 **전이 FSM 설계의 일부로 활용**됩니다.


<br>

---

# TargetDistributor.cs

### 핵심 목적
여러 적 AI가 타겟(플레이어 등)을 공격할 때,
서로 겹치지 않도록 타겟 주변에 원형 분산된 슬롯 위치를 배정하는 시스템입니다.
이 시스템은 Formation 추격, 무리지어 공격 등에 사용됩니다.

### 클래스 구조 요약
```csharp
public class TargetDistributor : MonoBehaviour
```
- 슬롯 분포 및 할당 관리
- **타겟 주변 방향(Arc)** 을 계산하여 각 AI에게 중복되지 않게 방향 슬롯을 분배
- AI들은 `TargetFollower`를 통해 이 시스템에 등록됨

---

### 내부 클래스: TargetFollower
```csharp
public class TargetFollower
```
- 각 추격 AI가 이 클래스를 통해 distributor와 연결됨

주요 변수:
| 변수              | 설명                         |
| --------------- | -------------------------- |
| `requireSlot`   | 슬롯이 필요한지 표시                |
| `assignedSlot`  | 현재 할당된 슬롯 인덱스 (-1이면 없음)    |
| `requiredPoint` | 추격 목표 위치                   |
| `distributor`   | 연결된 `TargetDistributor` 참조 |

---

### 슬롯 구조 및 분배 방식

#### arcsCount
- 타겟을 중심으로 배치될 슬롯 수 (ex. 8이면 360도 / 8 = 45도 간격)
- 각 슬롯은 `m_WorldDirection[i]`에 대응하는 방향을 가짐

#### OnEnable()
- 슬롯 방향 초기화 (360도 회전하면서 방향 분포 생성)
- `Vector3.forward`부터 시작해서 좌회전 방향으로 순환

---

### 작동 흐름
#### RegisterNewFollower()
- 새 AI를 이 디스트리뷰터에 등록하고 `TargetFollower` 객체 반환

#### LateUpdate()
- 매 프레임, 모든 follower 중 `requireSlot == true`인 경우 슬롯 할당
- 이전 슬롯이 있다면 해제 후 재할당 (`m_FreeArcs[]`)

---

### 핵심 메서드: GetFreeArcIndex(TargetFollower)
이 메서드는 실제로 **"어디 방향 슬롯을 할당할지"** 를 계산합니다.

1. `requiredPoint` 기준으로 원하는 방향 벡터 계산
2. 현재 위치에서 그 방향으로 Raycast 실행
3. 장애물이 없고 슬롯이 비어 있으면 그 방향 사용
4. 아니면 좌/우로 번갈아가며 가까운 빈 슬롯을 탐색
5. 전부 막혀 있으면 `-1` 반환

```csharp
Vector3 wanted = follower.requiredPoint - transform.position;
...
int wantedIndex = Mathf.RoundToInt(angle / arcDegree);
...
while (offset <= halfCount)
{
    // 왼쪽/오른쪽 슬롯 번갈아 체크
}
```

---

### 기타 기능

| 함수                     | 설명                     |
| ---------------------- | ---------------------- |
| `GetDirection(index)`  | 슬롯 인덱스를 3D 방향 벡터로 변환   |
| `FreeIndex(index)`     | 슬롯 수동 해제               |
| `UnregisterFollower()` | follower 삭제 및 슬롯 반환 처리 |

### 동작 시각화 예

```scss
        [1]     [0]
   [2]             [7]
       [3]   X   [6]
            [4] [5]
          (target)

→ AI가 타겟을 중심으로 각 방향 슬롯을 균등하게 배치
→ 겹침 방지, 공격 진입각 분산
```

---

`TargetDistributor`는 Gamekit3D에서
다중 적 AI가 타겟을 겹치지 않게 둘러싸는 구조를 만들기 위한 핵심 컴포넌트입니다.

- AI는 `requiredPoint`를 전달하고
- 시스템은 장애물, 방향 등을 고려하여 가장 적절한 슬롯 방향을 할당
- 이 정보로 AI는 `SetTarget()`에 목적지를 전달하여 자연스럽게 포위/추격

<br>

---

# SceneLinkedSMB\<T\>

## 개요

`SceneLinkedSMB<TMonoBehaviour>`는 Unity에서 애니메이션 상태 머신(`StateMachineBehaviour`)을 확장하여 특정 `MonoBehaviour` 인스턴스와 연결된 상태 기반 FSM 처리를 가능하게 만드는 구조이다.

---

## 제네릭 클래스 구조

```csharp
public class SceneLinkedSMB<TMonoBehaviour> : SealedSMB
    where TMonoBehaviour : MonoBehaviour
```

* `TMonoBehaviour`는 연결될 `MonoBehaviour` 타입 (예: `PlayerController`, `EnemyAI` 등)
* `SealedSMB`는 기본 `StateMachineBehaviour`의 세 가지 메서드를 봉인하여 상태 로직을 이 클래스에서만 처리하도록 강제

---

## 주요 필드 및 초기화 함수

```csharp
protected TMonoBehaviour m_MonoBehaviour;
bool m_FirstFrameHappened;
bool m_LastFrameHappened;
```

### 초기화

```csharp
public static void Initialise (Animator animator, TMonoBehaviour monoBehaviour)
```

* Animator에 연결된 `SceneLinkedSMB<T>`를 모두 찾아 `InternalInitialise()` 호출로 컴포넌트를 캐싱

---

## 상태 처리 흐름

### 상태 진입

```csharp
OnStateEnter()
```

* `m_FirstFrameHappened = false`
* `OnSLStateEnter` 호출 (두 버전: Playable 포함/비포함)

### 상태 업데이트

```csharp
OnStateUpdate()
```

* Animator가 Transition 중일 때:

  * 진입 중 상태: `OnSLTransitionToStateUpdate`
  * 이탈 중 상태: `OnSLTransitionFromStateUpdate`
* 완전 진입 직후: `OnSLStatePostEnter`
* Transition이 없을 때: `OnSLStateNoTransitionUpdate`
* 이탈 시작 시점: `OnSLStatePreExit`

### 상태 종료

```csharp
OnStateExit()
```

* `m_LastFrameHappened = false`
* `OnSLStateExit` 호출 (두 버전)

---

## 상태별 가상 함수 목록

| 메서드 이름                          | 설명                      |
| ------------------------------- | ----------------------- |
| `OnSLStateEnter`                | 상태로 진입 시                |
| `OnSLTransitionToStateUpdate`   | 상태 진입 중 (transition 도중) |
| `OnSLStatePostEnter`            | 완전 진입한 첫 프레임            |
| `OnSLStateNoTransitionUpdate`   | 상태 유지 중 (transition 없음) |
| `OnSLStatePreExit`              | 이탈 시작 첫 프레임             |
| `OnSLTransitionFromStateUpdate` | 상태 이탈 중 (transition 도중) |
| `OnSLStateExit`                 | 상태 종료 시                 |

> 각 메서드는 `AnimatorControllerPlayable`을 인자로 받는 오버로드 버전도 있음

---

## SealedSMB 클래스

```csharp
public abstract class SealedSMB : StateMachineBehaviour
```

* 기본 `OnStateEnter`, `OnStateUpdate`, `OnStateExit`를 `sealed`로 재정의
* `SceneLinkedSMB` 이외의 클래스가 직접 상태 처리 로직을 구현하지 못하도록 제한

---

## 사용 예

```csharp
public class PlayerController : MonoBehaviour
{
    void Start()
    {
        SceneLinkedSMB<PlayerController>.Initialise(GetComponent<Animator>(), this);
    }
}

public class PlayerIdleSMB : SceneLinkedSMB<PlayerController>
{
    public override void OnSLStateEnter(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
    {
        m_MonoBehaviour.DoIdleStuff();
    }
}
```

---

## 요약

| 항목  | 설명                                    |
| --- | ------------------------------------- |
| 목적  | FSM 구조를 `MonoBehaviour`와 타입 안정성 있게 연결 |
| 장점  | 컴포넌트 캐싱, 재사용성, 명확한 상태 구분              |
| 이벤트 | 상태 진입/유지/이탈/트랜지션을 정밀하게 분리             |
| 성능  | `GetComponent` 반복 제거로 최적화됨            |
| 확장성 | 서브클래싱으로 상태별 기능 구현 가능                  |
