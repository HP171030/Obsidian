`public class ServerRequest{}`
`public class ServerAnswer{}`

``public class ResponseClass<T>
{
	public T data;
	public E_SERVER_ERROR_TYPE serverErrorType;
	public string statusCode_http;
	public int statuscode_local;
}``


### 통신 시스템

- **DTO 기반 통신 구조 설계 및 구현**  
    `ServerRequest`, `ServerAnswer` 클래스를 **Data Transfer Object (DTO)** 로 정의하여 서버와의 데이터 송수신 구조 분리
    
- **제네릭 응답 처리 클래스 구현**  
    `ResponseClass<T>`를 통해 다양한 응답 타입을 처리할 수 있도록 **제네릭 구조**로 설계. 서버에서 받은 데이터(`data`), 에러 타입(`E_SERVER_ERROR_TYPE`), HTTP 상태 코드(`statusCode_http`), 내부 상태 코드(`statuscode_local`)를 포함하여 응답의 유연성과 디버깅 편의성 확보.
    
- **HTTP 통신 처리**  
    **`HttpWebRequest`**를 이용해 API 문서에 명시된 **EndPoint**에 대해 통신을 수행하며, 요청 객체는 **JSON 포맷으로 직렬화**되어 서버에 전송됨.  
    요청은 **Stream 기반 바이트 배열 전송 방식**으로 처리되어 효율적인 데이터 전송을 구현함.
    
- **토큰 기반 인증 처리**  
    서버로부터 전달받은 **Access Token**을 확인하여 요청의 유효성을 검증하며, 인증 실패 시 예외 처리를 포함한 보안 프로세스 구현.