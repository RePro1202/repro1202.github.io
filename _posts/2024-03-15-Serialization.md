---
title: "직렬화 (Serialization)"
author: RePro
date: 2024-03-15 10:00:00 +0900
categories: [Programming, C#]
tags: [c#, unity, unreal, cs]
---

# C#/Unity/Unreal의 직렬화 개념 및 비교 총정리

---

## 1. 직렬화란 무엇인가?

**직렬화(Serialization)** 는 메모리 상의 객체나 데이터 구조를 파일, 네트워크, DB 등에 저장하거나 전송하기 위해 **일련의 데이터(문자열 또는 이진 포맷)로 변환하는 과정**이다.  
반대로, 이 데이터를 다시 객체로 되돌리는 과정을 **역직렬화(Deserialization)** 라고 한다.

### 직렬화가 필요한 이유

- 객체의 상태를 **파일로 저장**하거나
- 네트워크를 통해 **전송**하거나
- **딥 카피(복제)** 또는 **디버깅/로그** 목적으로 사용

---

## 2. C#의 직렬화

C#은 직렬화를 언어와 .NET 프레임워크 수준에서 **기본적으로 지원**한다.  
JSON, XML, Binary 등 다양한 형식으로 직렬화가 가능하다.

### 기본 직렬화 방식

| 형식 | 지원 여부 | 대표 클래스 |
|------|-----------|--------------|
| JSON | 기본 지원 | System.Text.Json, Newtonsoft.Json |
| XML | 기본 지원 | XmlSerializer |
| Binary | 비권장 | BinaryFormatter (보안상 권장되지 않음) |

### JSON 직렬화 예제

```csharp
public class Player {
    public string Name { get; set; }
    public int Level { get; set; }
}

var p = new Player { Name = "Alice", Level = 10 };
string json = JsonSerializer.Serialize(p); // 직렬화
Player copy = JsonSerializer.Deserialize<Player>(json); // 역직렬화
```

### XML 직렬화 예제

```csharp
var serializer = new XmlSerializer(typeof(Player));
using (var writer = new StringWriter()) {
    serializer.Serialize(writer, player);
    string xml = writer.ToString();
}
```

### BinaryFormatter 예제 (비권장)

```csharp
[Serializable]
public class Player {
    public string Name;
    public int Level;
}

BinaryFormatter bf = new BinaryFormatter();
using (var fs = new FileStream("save.dat", FileMode.Create)) {
    bf.Serialize(fs, player);
}
```

---

## 3. Unity의 직렬화 시스템

Unity는 에디터와 런타임 모두에서 사용 가능한 자체 직렬화 시스템을 가지고 있다.  
대표적으로 `.meta`, `.prefab`, `.unity`, `.asset` 등의 파일이 YAML 기반으로 직렬화된다.

### .meta 파일

- 모든 에셋 파일에 자동 생성되며, 해당 에셋의 GUID, 임포트 설정 등을 포함한다.
- YAML 포맷으로 저장된다.

```yaml
fileFormatVersion: 2
guid: 7c6f2acb8f7d842438af89b85f6f444d
timeCreated: 1649152920
licenseType: Free
DefaultImporter:
  externalObjects: {}
  userData: 
  assetBundleName: 
  assetBundleVariant: 
```

| 항목 | 설명 |
|------|------|
| guid | 에셋의 고유 식별자 (글로벌 유일) |
| fileFormatVersion | 메타 파일 포맷 버전 |
| DefaultImporter | 에셋 유형별 설정 정보 (텍스처, 모델, 오디오 등) |

### Unity 직렬화 대상

| 조건 | 설명 |
|------|------|
| public 필드 | 자동 직렬화됨 |
| private + [SerializeField] | 직렬화됨 |
| static, const | 직렬화 안됨 |
| 지원 타입 | int, float, string, Vector3 등 기본 타입 및 UnityEngine.Object 파생 클래스 |
| 커스텀 클래스 | [System.Serializable] 필요 |

### 비직렬화 대상 예시

| 제외 항목 | 이유 |
|-----------|------|
| static 변수 | 클래스 전체에 속하므로 저장 불필요 |
| delegate/event | 함수 포인터, 저장 불가 |
| Dictionary | 일부 버전에서만 제한 지원 |

### 직렬화 포맷

| 파일 | 포맷 |
|------|------|
| .meta, .prefab, .unity | YAML (텍스트 직렬화 가능) |
| 빌드 시 | 바이너리 포맷으로 컴파일됨 |

### 런타임 JSON 예시

```csharp
[Serializable]
public class SaveData {
    public int score;
    public string name;
}

SaveData data = new SaveData { score = 100, name = "Alice" };
string json = JsonUtility.ToJson(data);
SaveData copy = JsonUtility.FromJson<SaveData>(json);
```

### 커스텀 직렬화

Unity는 `ISerializationCallbackReceiver` 인터페이스를 통해 직렬화 전/후의 로직 삽입을 지원한다.

```csharp
public class MyData : MonoBehaviour, ISerializationCallbackReceiver {
    public int rawValue;

    public void OnBeforeSerialize() {
        Debug.Log("저장 전 호출");
    }

    public void OnAfterDeserialize() {
        Debug.Log("불러온 후 호출");
    }
}
```

---

## 4. 언리얼 엔진의 직렬화 시스템

언리얼은 C++ 기반이지만, `UCLASS`, `UPROPERTY`, `FArchive` 등으로 구성된 **강력한 리플렉션 + 직렬화 시스템**을 내장하고 있다.

### 자동 직렬화

```cpp
UCLASS()
class UMySaveData : public UObject {
    GENERATED_BODY()

    UPROPERTY()
    int32 Level;

    UPROPERTY()
    FString PlayerName;
};
```

- `UPROPERTY()`가 붙은 필드는 자동으로 에디터/세이브/네트워크 직렬화 대상이 된다.

### 수동 직렬화 (FArchive)

```cpp
void Serialize(FArchive& Ar) override {
    Super::Serialize(Ar);
    Ar << Level;
    Ar << PlayerName;
}
```

- SaveGame, 네트워크 통신, 커스텀 바이너리 저장 등에서 사용

### SaveGame 시스템

```cpp
UCLASS()
class UMySaveGame : public USaveGame {
    GENERATED_BODY()

    UPROPERTY()
    int32 Coins;

    UPROPERTY()
    FVector Location;
};
```

```cpp
UMySaveGame* SaveGame = Cast<UMySaveGame>(
    UGameplayStatics::CreateSaveGameObject(UMySaveGame::StaticClass()));
UGameplayStatics::SaveGameToSlot(SaveGame, TEXT("Slot1"), 0);
```

- `.sav` 파일로 저장되며 자동 직렬화된다.

### JSON 직렬화

```cpp
TSharedPtr<FJsonObject> Json = MakeShareable(new FJsonObject());
Json->SetStringField("Name", "Alice");

FString Output;
TSharedRef<TJsonWriter<>> Writer = TJsonWriterFactory<>::Create(&Output);
FJsonSerializer::Serialize(Json.ToSharedRef(), Writer);
```

---

## 5. Unity vs Unreal 직렬화 비교

| 항목 | Unity | Unreal |
|------|-------|--------|
| 언어 | C# | C++ |
| 리플렉션 | attribute 기반 자동 | UCLASS, UPROPERTY 기반 |
| 직렬화 대상 | 대부분의 클래스 | UObject, UStruct |
| 에셋 저장 포맷 | YAML/텍스트(.meta, .prefab) | 바이너리(.uasset, .umap) |
| JSON 직렬화 | JsonUtility, Newtonsoft.Json | FJsonSerializer |
| 세이브 시스템 | Json, PlayerPrefs, Binary | USaveGame, FArchive |
| 커스텀 제어 | ISerializationCallbackReceiver | Serialize(FArchive&), NetSerialize |

---

## 6. 요약

- 직렬화는 객체 데이터를 파일, 네트워크 등으로 내보내거나 복원하는 데 필수
- C#은 기본 제공, Unity는 강력한 자동 직렬화 시스템 보유
- 언리얼은 C++ 기반에서도 강력한 직렬화 기능을 갖춤 (UCLASS + FArchive)
- JSON은 플랫폼 간 호환에 유리, 바이너리는 성능과 용량 면에서 유리

