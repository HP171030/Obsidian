## 프로젝트 핵심 기능: REST API 클라이언트 구현

### 1. 개요 및 설계 목표

본 프로젝트에서는 **안정적이고 효율적인 서버 통신**을 위해 **일반화된(Generic) REST API 클라이언트**를 직접 설계하고 구현했습니다. 대량의 데이터 요청 및 응답, 비동기 처리, 견고한 에러 핸들링이 요구되는 환경에서, 반복적인 API 호출 코드를 최소화하고 시스템의 **유지보수성 및 확장성**에 중점을 두었습니다.

### 2. 주요 구현 기능 및 기술 상세

#### 2.1. API 호출 추상화 (제네릭클래스 확장 메서드)

- **역할:** 클라이언트 코드에서 서버 API 호출을 **간결하게** 수행할 수 있도록 추상화한 핵심 메서드입니다.
    
- **특징:**
    
    - **제네릭 (`<TRequest, TAnswer>`):** 특정 API에 종속되지 않고, 모든 요청(Request) DTO와 응답(Answer) DTO에 유연하게 대응하여 **코드 재사용성**을 극대화했습니다.
        
    - **UniTask 활용:** 비동기 작업을 `await` 키워드를 통해 동기 코드처럼 직관적으로 작성하면서도, 메인 스레드를 블로킹하지 않아 애플리케이션의 **반응성을 유지**합니다.
        
    - **콜백 기반 결과 처리:**
 API 호출 성공 시 액션을 통해 응답 데이터를 전달하여, 비즈니스 로직과 통신 로직을 분리했습니다


```csharp
public static async UniTask<bool> TryGetServerAnswer<TRequest, TAnswer>(
    this TRequest requestDTOClass,
    Func<TRequest, UniTask<ResponseClass<TAnswer>>> api,
    Action<TAnswer> onResponseSuccess = null)
    where TRequest : ServerRequest
    where TAnswer : ServerAnswer
{
    // API 함수 유효성 검사 및 호출
    var _response = await api.Invoke(requestDTOClass);

    // 응답 코드 확인 및 데이터 추출
    if (!_response.TryGetResponseCode(out TAnswer _answer))
    {
        return false; // 오류 발생 시 실패 반환
    }

    onResponseSuccess?.Invoke(_answer); // 성공 시 콜백 실행
    return true;
}
```

#### 2.2. HTTP 요청 구성 및 DTO 직렬화/역직렬화 (`RequestDataFromJson`)

- **역할:** API 엔드포인트, HTTP 메서드, 요청 DTO를 기반으로 실제 HTTP 요청을 구성하고, 서버로부터 받은 JSON 응답을 DTO로 변환합니다
    
- **특징:**
    
    - **DTO 기반 통신:** **`ServerRequest`**와 **`ServerAnswer`** 타입의 DTO 클래스를 사용하여 API 문서에 정의된 데이터 구조를 코드에 그대로 반영, **타입 안정성**과 **가독성**을 확보했습니다.
        
    - **JSON 직렬화/역직렬화:** **Newtonsoft.Json** 라이브러리를 활용하여 DTO 객체를 JSON 문자열로 직렬화하거나, 서버 응답 JSON을 DTO 객체로 역직렬화하여 데이터 교환을 효율적으로 처리합니다. 특히 응답 데이터의 `data` 필드를 `JObject`로 파싱하여 유연하게 DTO로 변환하는 로직을 적용했습니다.
        
    - **인증 토큰 관리:**  로그인 상태를 확인하고, 액세스 토큰 및 리프레시 토큰을 요청 헤더에 자동으로 포함시켜 **토큰 기반 인증**을 구현했습니다. 토큰 유효성 검사 통해 만료 시 재인증 절차를 고려했습니다.


```csharp

private async UniTask<ResponseClass<TAnswer>> RequestDataFromJson<TRequest, TAnswer>(
    string endPoint, TRequest requestDTOClass, E_SERVER_METHOD_TYPE method, E_SERVER_ERROR_TYPE serverErrorType)
    where TRequest : ServerRequest where TAnswer : ServerAnswer
{
    string _toJson = JsonConvert.SerializeObject(requestDTOClass);
    // GET 요청 시 본문 제외 처리
    if (method == E_SERVER_METHOD_TYPE.GET) _toJson = null;

    // 로그인 상태에 따라 토큰 포함 또는 미포함하여 TryConncetion 호출
    if (AccountManager.Instance.LoginAccount == null)
    {
        _value = await TryConncetion<TAnswer>(_url, _toJson, method.ToString(), null);
    }
    else
    {
        // 토큰 유효성 검사 및 처리
        if (CheckToken(ref error) != 0) { /* 에러 처리 */ }
        else
        {
            _value = await TryConncetion<TAnswer>(_url, _toJson, method.ToString(), 
                                            new string[] { _AcessToken, _RefreshToken });
        }
    }
    _value.serverErrorType = serverErrorType; // 에러 타입 지정
    return _value;
}
```

#### 2.3. 저수준 HTTP 통신 처리 및 스레드 관리 (`SendingInformation`)

- **역할:** 실제 네트워크 통신을 담당하며, `HttpWebRequest`와 `HttpWebResponse` 객체를 직접 사용하여 HTTP 요청을 전송하고 응답을 수신합니다.
    
- **특징:**
    
    - **`HttpWebRequest` 직접 제어:** HTTP 메서드, `Content-Type`, `Content-Length`, `Timeout` 등 요청의 모든 부분을 세밀하게 설정합니다
        
    - **비동기 및 스레드 풀 활용:** 비동기로 호출하여 실제 네트워크 요청을 메인 스레드가 아닌 별도의 스레드 풀에서 실행함으로써, **장시간이 소요될 수 있는 UI Blocking을 방지**했습니다.
        
    - **세분화된 응답 처리:** 응답 헤더를 파싱하여 활용할 수 있도록 저장하고, 응답 스트림으로부터 본문 데이터를 읽어 최종 결과를 반환합니다.
        
    - **타임아웃 및 기본적인 예외 처리:** 네트워크 지연 상황에 대비한 타임아웃 설정을 포함하며, 통신 중 발생할 수 있는 예외에 대한 기본적인 처리를 포함합니다.


```csharp

public async UniTask<string> SendingInformation(string Url, string Method, string Jsonbody = null, string[] Token = null)

    if (caninternet == false) return "999"; // 네트워크 없음 에러 코드

    HttpWebRequest httpWebRequest = null;
    try
    {
        // POST/PUT/PATCH/DELETE 요청 구성
        if (Method == "POST" || Method == "PUT" || Method == "PATCH" || Method == "DELETE")
        {
            byte[] sendData = UTF8Encoding.UTF8.GetBytes(Jsonbody);
            httpWebRequest = (HttpWebRequest)WebRequest.Create(Url);
            httpWebRequest.ContentType = "application/json";
            httpWebRequest.Method = Method;
            httpWebRequest.Timeout = TimeOutTime;
            httpWebRequest.ContentLength = sendData.Length;
            SetHeadToken(Token, ref httpWebRequest); // 인증 토큰 헤더 설정
            using (Stream requestStream = httpWebRequest.GetRequestStream())
            {
                requestStream.Write(sendData, 0, sendData.Length);
            }
        }
        // GET 요청 구성
        else if (Method == "GET")
        {
            httpWebRequest = (HttpWebRequest)WebRequest.Create(Url);
            httpWebRequest.Timeout = TimeOutTime;
            httpWebRequest.Method = Method;
            SetHeadToken(Token, ref httpWebRequest);
        }

        // 응답 수신 및 처리
        using (HttpWebResponse httpWebResponse = (HttpWebResponse)httpWebRequest.GetResponse())
        {
            // 응답 헤더 파싱
            for (int i = 0; i < httpWebResponse.Headers.Count; i++)
                _ResponeHeader.Add(httpWebResponse.Headers.Keys[i], httpWebResponse.Headers[i]);

            // 응답 본문 읽기
            using (StreamReader _StreamReader = new StreamReader(httpWebResponse.GetResponseStream(), Encoding.GetEncoding("UTF-8")))
            {
                return _StreamReader.ReadToEnd();
            }
        }
    }
    catch (InvalidOperationException ex) { /* ... */ } // 예외 처리
    // ... (WebException 등..)
}
```


### 3. 프로젝트 기여 및 성과

- **생산성 및 개발 효율성 증대:** 제네릭 및 추상화된 API 호출 메서드를 통해 반복적인 네트워크 통신 로직에 신경 쓰지 않고 비즈니스 로직에 집중할 수 있도록 하여 **개발 생산성을 크게 향상**시켰습니다.
    
- **애플리케이션 성능 및 UX 최적화:** 모든 네트워크 통신을 비동기적으로 처리하고 별도의 스레드 풀에서 실행함으로써, 메인 스레드 블로킹을 방지하여 UX를 고려했습니다
    
- **확장 가능한 통신 인터페이스:**  DTO 기반의 타입 안전한 통신, 토큰 기반 인증 처리, 그리고 저수준에서 디테일하게 HTTP 제어하여 어떤 REST API 서비스와도 연동 가능한 **유연한  통신 인터페이스**로 사용할 수 있도록 만들었습니다