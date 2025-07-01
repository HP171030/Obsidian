### 1. 더티 플래그(Dirty Flag)

- **의미**: 에셋이 **메모리상에서 변경되었지만, 아직 디스크의 `.asset` 파일에는 저장되지 않았음**을 유니티 에디터에 알리는 내부적인 **표시(Flag)** 임
    
- **필요성**:
    
    - 인스펙터 UI를 통한 변경은 유니티가 자동으로 감지해 플래그를 설정
        
    - **코드로 에셋을 변경할 경우**, 유니티는 자동으로 이를 감지하지 못하므로, 개발자가 **명시적으로 변경을 알려주어야** 함
        

---

### 2. 핵심 함수 및 역할

- **`EditorUtility.SetDirty(Object obj)`**
    
    - `obj`에 해당하는 에셋 인스턴스에 **더티 플래그를 설정**. 유니티 에디터에게 "이 객체는 저장할 필요가 있는 상태가 되었다"고 알려주는 역할
        
    - 이 함수만으로는 디스크에 즉시 저장되지는 않고 메모리 상에서 변경된 오브젝트가 있다라고 플래그만 박는것임
        
- **`AssetDatabase.SaveAssets()`**
    
    - 현재 유니티 프로젝트 내에서 **더티 플래그가 설정된 모든 에셋들을 디스크의 `.asset` 파일에 실제로 저장**함
        
    - `SaveAssets()`가 호출되면, 유니티는 더티 플래그가 있는 각 에셋의 **현재 메모리상에 존재하는 최신 상태를 읽어와서, 해당 에셋의 원본 `.asset` 파일을 통째로 덮어씌움
        

---

### 3. 코드로 `ScriptableObject` 변경 시 사용 플로우

1. **메모리상의 `ScriptableObject` 데이터 변경**:
   ```Csharp
    myScriptableObjectInstance.value = 100;
    ```
    (이 단계에서 메모리상의 인스턴스만 변경되고, `.asset` 파일은 그대로)
    
2. **`EditorUtility.SetDirty()` 호출**:
    ```csharp
    EditorUtility.SetDirty(myScriptableObjectInstance);
    ```
    
    (`myScriptableObjectInstance`가 변경되었으니 저장 대상이라고 알림)
    
3. **`AssetDatabase.SaveAssets()` 호출**:

    ```csharp
    AssetDatabase.SaveAssets();
    ```
    
    (더티 플래그가 설정된 `myScriptableObjectInstance`의 현재 메모리 상태를 디스크의 `.asset` 파일에 덮어씌움)
    

---

### 4. 중요 사항

- `EditorUtility.SetDirty()`와 `AssetDatabase.SaveAssets()`는 **`UnityEditor` 네임스페이스에 속하므로, 빌드된 게임에서는 사용할 수 없고 에디터 환경에서만 사용 가능**. 런타임 시 데이터 저장은 다른 직렬화 방식을 사용해야 함 (Json, XML, 간단한 형식이라면 유니티에서 제공하는 PlayerPrefs라던지 로컬에 저장 가능한 형식을 사용해야 함)