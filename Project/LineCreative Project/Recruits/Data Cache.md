
## 2. 대량 데이터 로딩 및 캐싱 시스템 (데이터 매니저)

프로젝트 내 **방대한 양의 설정 및 마스터 데이터**를 효율적으로 관리하고 빠르게 접근하기 위한 **데이터 로딩 및 캐싱 시스템**을 구축했습니다.

### 주요 구현 특징

- **범용 데이터 파싱 및 캐싱:**
    
    - **제네릭(`TKey`, `TData`)을 활용하여 어떤 데이터 타입이라도 파싱하고 `Dictionary`에 캐싱할 수 있도록 설계했습니다.
        
    - 이를 통해 CSV와 JSON 등 **다양한 파일 형식을 동일한 로직으로 처리**할 수 있는 **유연한 데이터 로딩 인터페이스**를 제공했습니다.
        
    - 데이터는 메모리상에 `Dictionary` 형태로 캐싱되어 런타임 중 **$O(1)$에 가까운 속도로 조회**할 수 있도록 최적화되었습니다.
    


#### 3. 프로젝트 기여 및 성과 (강조할 점)

- **파서 플러그인 아키텍처 :** 새로운 데이터 형식(예: XML)이 추가될 때, 인터페이스와 어트리뷰트를 활용하여 동적으로 파서타입을 등록합니다.  클래스나 호출 로직을 수정할 필요없어 유지보수 비용이 낮아 확장성 좋고 유지보수에 용이합니다
    

#### 4. 설계 선택의 고려사항 및 트레이드오프

자동화된 플러그인 아키텍처는 유지보수와 확장성에 유리하지만, 모든 설계에는 트레이드오프가 존재합니다. 본 시스템 설계 시 다음과 같은 점들을 인지했습니다

- **컴파일 타임 명시성 감소와 런타임 검증 의존:**
    
    - 파서 클래스들이 명시적인 코드 참조 대신 어트리뷰트와 리플렉션을 통해 동적으로 등록되므로, IDE의 "Find Usages"와 같은 기능으로 직접적인 사용처를 추적하기 어려울 수 있습니다.
        
    - 이는 새로운 개발자가 시스템의 전체적인 흐름을 파악하는 데 초기 학습 곡선을 증가시킬 수 있으며, 특성 사용의 오류(예: 오타, 누락)는 컴파일 시점이 아닌 **런타임에 가서야 발견되는 오류**가 될 수 있습니다.
    
- **리플렉션의 초기 오버헤드:**
    
    - 유니티를 통해 애플리케이션 시작 시 한 번의 어셈블리 스캔이 발생합니다. 미미한 수준이지만, 프로젝트 내 클래스의 수가 극단적으로 많아지면 초기 로딩 시간에 영향을 줄 수도 있습니다
    

**이러한 고려사항에도 불구하고, 본 프로젝트에서는 새로운 데이터 형식의 추가 및 관리에 필요한 개발자의 수동 작업을 최소화하고, 시스템의 확장성을 확보하는 것이 가장 큰 가치라고 판단하여 이와 같은 설계를 선택했습니다. 감수할 수 있는 초기 오버헤드와 디버깅의 간접성보다, 얻게 되는 자동화된 유연성의 이점이 훨씬 크다고 보았습니다.**


### 기술적 성과

본 시스템을 통해 초기 로딩 시간을 단축하고, 게임 내에서 대량의 데이터를 **매우 빠르고 안정적으로 활용**할 수 있게 되었습니다. 제네릭을 통한 유연한 설계로 새로운 데이터가 추가되거나 변경되어도 **코드 수정 없이 쉽게 확장**할 수 있습니다.




```csharp

public static class ReadableParserRegist
{
    static readonly Dictionary<E_DATA_TABLE_TYPE, IReadableData> _parsers = new();
    [RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.BeforeSceneLoad)]
     static void RegistParser()
    {
        foreach (var type in Assembly.GetExecutingAssembly().GetTypes())
        {
            if (typeof(IReadableData).IsAssignableFrom(type) && !type.IsAbstract && !type.IsInterface)
            {
                var attribute = type.GetCustomAttribute<DataParserAttribute>();
                if (attribute != null)
                {
                    E_DATA_TABLE_TYPE parserType = attribute.ParseType;
                    IReadableData parserInstance = (IReadableData)Activator.CreateInstance(type);
                    _parsers.Add(parserType, parserInstance);
                }
            }
        }
    }

    public static IReadableData GetParser(this E_DATA_TABLE_TYPE type)
    {
        if (_parsers.TryGetValue(type, out IReadableData data))
            return data;
        throw new Exception($"{type.ToString()}은 지원하지 않는 파서 타입임");
    }

}

public interface IReadableData
{
    List<TData> Parse<TData>(string resourcePath, string fileName) where TData : new();
}

  

[DataParser(E_DATA_TABLE_TYPE.CSV)]
public class CSVData : IReadableData
{
    public List<TData> Parse<TData>(string resourcePath, string fileName) where TData : new()
    {
        return CSVParser.Read<TData>(resourcePath, fileName, true);
    }
}

[DataParser(E_DATA_TABLE_TYPE.JSON)]
public class JsonData : IReadableData
{
    public List<TData> Parse<TData>(string resourcePath, string fileName) where TData : new()
    {
        return JsonParser.LoadJsonFromResources<List<TData>>(resourcePath, fileName);
    }
}

  

[AttributeUsage(AttributeTargets.Class, Inherited = false, AllowMultiple = false)]
public sealed class DataParserAttribute : Attribute
{
    public E_DATA_TABLE_TYPE ParseType { get; }
    public DataParserAttribute(E_DATA_TABLE_TYPE type)
    {
        ParseType = type;
    }
}

  

        private void ParseDataTableFromResources<TKey, TData>(E_DATA_TABLE_TYPE dataTable, Dictionary<TKey, TData> dataDic, System.Func<TData, TKey> getKey, System.Action<TData> onRegist = null, string fileName = null, bool reDefinePath = false, string filePathReName = null) where TData : new()
        {
            List<TData> _dataList = null;
            string filePath = JSON_RESOURCES_PATH + "/DataTable";

            if (fileName == null)
            {
                fileName = typeof(TData).Name; ;
            }
            if (reDefinePath)
                filePath = filePathReName;

            _dataList = dataTable.GetParser().Parse<TData>(filePath, fileName);
            foreach (var _data in _dataList)
            {
                if (dataDic.ContainsKey(getKey.Invoke(_data)))
                    continue;
                else
                {
                    dataDic.Add(getKey.Invoke(_data), _data);
                    onRegist?.Invoke(_data);
                }
            }
        }
```

