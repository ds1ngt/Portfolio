# REST API 헬퍼
- [REST API 헬퍼](#rest-api-헬퍼)
  - [개요](#개요)
  - [기능](#기능)
  - [구조](#구조)
  - [API](#api)
    - [Client](#client)
      - [GET, POST, PUT, DELETE](#get-post-put-delete)
      - [MESSAGE](#message)
      - [Settings](#settings)
      - [Auth](#auth)
      - [Debug](#debug)
    - [HttpRequestBuilder](#httprequestbuilder)
    - [IRequestHandler](#irequesthandler)
    - [Callbacks](#callbacks)
      - [Callbacks](#callbacks-1)
      - [Delegates](#delegates)
  - [Delegate |](#delegate-)
    - [Util](#util)
    - [Struct](#struct)
      - [HttpRequestMessageInfo](#httprequestmessageinfo)
      - [RequestDataSimple](#requestdatasimple)
      - [RequestDataMessage](#requestdatamessage)
    - [Class](#class)
      - [ResponseBase\<T\>](#responsebaset)
      - [ResponseString : ResponseBase\<string\>](#responsestring--responsebasestring)
      - [ResponseStream : ResponseBase\<Stream\>, IAsyncDisposable](#responsestream--responsebasestream-iasyncdisposable)
    - [Enum](#enum)
      - [eAuthorizationType](#eauthorizationtype)
      - [eRequestType](#erequesttype)
  - [사용 예시](#사용-예시)

## 개요
C# HttpClient을 통해 REAT API 호출을 처리하는 헬퍼 라이브러리

## 기능
- .NET HttpClient를 래핑한 클래스 구성  
REST API 요청을 **단일 객체로 관리** 합니다.
- 자주 사용하는 REST API에 대한 랩핑 클래스 제공  
GET, POST, PUT, DELETE 요청에 대한 간단한 요청을 할 수 있는 **랩핑 클래스**를 제공합니다.  
- 상세한 요청에 대한 구성 제공  
요청 할 내용의 Content, ContentType 등을 지정할 수 있는 **HttpRequestBuilder**를 제공합니다.
- 요청의 경량화 및 결과를 스트림으로 수신  
모든 네트워크 요청은 **Header를 우선 요청**하고, 그 후에 **Body를 스트림으로 수신**합니다. 이 때, 데이터는 일정 크기의 **버퍼**만큼 나누어 수신합니다. (네트워크 부하 분산)  
경우에 따라 Header 정보만 수신하고 원하는 시점에 Body를 수신할 수 있습니다.
- 인증 지원  
REST API 요청시 인증을 설정할 수 있도록 합니다. (Basic / Bearer Token)  
한 번 설정 된 인증 상태는 변경하기 전까지 유지 됩니다.
- 콜백 제공  
요청에 대한 과정을 콜백으로 받을 수 있는
- 동시 요청 가능 수 제어
## 구조
![](./images/http_helper.png)
## API
소스코드 전문을 공개할 수 없어 작성된 Public API로 내용을 대체합니다.  
(클래스별 API 분류)

### Client

#### GET, POST, PUT, DELETE
API | 설명
--- | ---
async UniTask\<ResponseString> **RequestStringAsync**(string url, string content = "", CancellationTokenSource cts = null)<td rowspan="4">REST API 요청 (Non- Callback) | 
async UniTask\<ResponseString> **RequestStringAsync**(RequestDataSimple request) |
async UniTask\<ResponseStream> **RequestStreamAsync**(string url, string content = "", CancellationTokenSource cts = null) | 
async UniTask\<ResponseStream> **RequestStreamAsync**(RequestDataSimple request) |
async UniTask\<ResponseBase\<T>> **RequestAsync**\<T>(string url, string content = "", CancellationTokenSource cts = null)<td rowspan="2">Response 오브젝트(T)로 파싱 | 
async UniTask\<ResponseBase\<T>> **RequestAsync**\<T>(RequestDataSimple request) |

#### MESSAGE
API | 설명
--- | ---
async UniTask\<ResponseString> **RequestStringAsync**(HttpRequestMessage request, CancellationTokenSource cts = null, HttpCompletionOption option = HttpCompletionOption.ResponseHeadersRead)<td rowspan="4">REST API 요청 (Non-Callback) | 
async UniTask\<ResponseString> **RequestStringAsync**(RequestDataMessage request) |
async UniTask\<ResponseStream> **RequestStreamAsync**(HttpRequestMessage request, CancellationTokenSource cts = null, HttpCompletionOption option = HttpCompletionOption.ResponseHeadersRead) |
async UniTask\<ResponseStream> **RequestStreamAsync**(RequestDataMessage request) |
async UniTask\<ResponseBase\<T>> Request\<T>(HttpRequestMessage request, CancellationTokenSource cts = null, HttpCompletionOption option = HttpCompletionOption.ResponseHeadersRead)<td rowspan="2">Response 오브젝트 (T)로 파싱|
async UniTask\<ResponseBase\<T>> Request\<T>(RequestDataMessage request) |

#### Settings
API | 설명
--- | ---
void SetBaseAddress(string url) | HttpClient의 BaseAddress 지정
string BaseAddress | HttpClient의 BaseAddress 조회

#### Auth
API | 설명
--- | ---
void SetBasicAuthentication(params (string, string)[] authInfos) | 일반 인증 적용<br>authInfos는 Util의 Make~~~AuthInfo로 생성
void SetTokenAuthentication(params (string, string)[] authInfos) | 토큰 기반 인증 적용

#### Debug
API | 설명
--- | ---
int ConcurrentRequest | 현재 활성화 된 다운로드 수 확인

### HttpRequestBuilder

API | 설명
--- | ---
static HttpRequestBuilder **CreateNew**(Client.eRequestType requestType, string url) | 빌더 생성
HttpRequestBuilder **NewRequest**(Client.eRequestType requestType, string url) | 새 요청 생성 (초기화)
***Content*** | 
HttpRequestBuilder **SetContent**(HttpContent httpContent) | HttpContent
HttpRequestBuilder **SetContent**(string message) | StringContent
HttpRequestBuilder **SetContent**(string message, string mediaType) | StringContent
HttpRequestBuilder **SetContent**(byte[] bytes, int offset = 0, int count = -1) | ByteArrayContent
HttpRequestBuilder **SetContent**(Stream stream, int bufferSize = 0) | StreamContent
HttpRequestBuilder **SetContentType**(string mediaType) | 미디어 타입 지정 (목록)
HttpRequestBuilder **SetContentLength**(long length) | 
***Headers*** | 
HttpRequestBuilder **AddHeader**(string key, string value) |
HttpRequestBuilder **AddHeader**(string key, IEnumerable\<string> values) |
HttpRequestBuilder **RemoveHeader**(string key) |
***Property*** | 
HttpRequestBuilder **AddProperty**(string key, string value) |
HttpRequestBuilder **RemoveProperty**(string key) |
void **ClearAllProperty**() | 지정된 모든 Property 제거
***All-In-One*** | 
static HttpRequestMessage **Generate**(HttpRequestMessageInfo info) | HttpRequestMessageInfo를 바탕으로 HttpRequestMessage 생성

### IRequestHandler

API | 설명
--- | ---
UniTask SendAsync() | 네트워크 요청 보내기
void Cancel() | 취소

### Callbacks
#### Callbacks
Callback | 설명
--- | ---
OnProgress **OnDownloadProgress** | 다운로드 진행 중
OnDownloadComplete **OnComplete** | 성공
OnHttpStatus **OnFailed** | 실패
Action **OnDownloadStart** | 시작
Action **OnFinally** | 네트워크 처리 종료
#### Delegates

Delegate |
---
void **OnProgress**(int currentRead, long totalRead, long totalSize)
void **OnDownloadComplete**(Stream readStream, long totalReadLen)
void **OnHttpStatus**(HttpStatusCode statusCode)

### Util
API | 설명
--- | ---
(string, string)[] **MakeBasicAuthInfo**(string id, string password) | 일반 인증 정보 생성
(string, string)[] **MakeTokenAuthInfo**(string token) | 토큰 기반 인증 정보 생성

### Struct
#### HttpRequestMessageInfo
HttpRequestBuilder.Generate의 인자로 전달되는 요청 메시지 정보 구조체

타입 | 이름 | 설명
--- | --- | ---
Client.eRequestType | **RequestMethod** | 요청 메서드
string | **Url** | 요청 URL
HttpContent | **Content** | 컨텐츠
(string, string)[] | **Headers** | 헤더
(string, string)[] | **Properties** | 속성
string | **ContentType** | 컨텐츠 종류 (기본값: text/plain)
long | **ContentLength** | 컨텐츠 길이 (기본값: 0)

#### RequestDataSimple
타입 | 이름 | 설명
--- | --- | ---
eRequestType | **RequestType**
string | **Url**
string | **Content** |
CancellationTokenSource | **Cts** |
bool | **CanResend** | 토큰 만료시 패킷 재요청 여부

#### RequestDataMessage
타입 | 이름 | 설명
--- | --- | ---
HttpRequestMessage | **Message** |
HttpCompletionOption | **Option** |
CancellationTokenSource | **Cts**
bool | **CanResend** | 토큰 만료시 패킷 재요청 여부

### Class
#### ResponseBase\<T>
**변수**
타입 | 이름 | 설명
--- | --- | ---
public HttpStatusCode | **StatusCode** | Http 응답 코드
public HttpResponseMessage | **Response** | Http 응답 메시지
public T | **Value** | 응답 값

**함수**
API | 설명
--- | ---
public **ResponseBase**(HttpResponseMessage response) | 생성자 (응답 값이 없음)
public **ResponseBase**(T value, HttpResponseMessage response) : this(response) | 생성자 (응답 값 있음)

#### ResponseString : ResponseBase\<string>
API | 설명
--- | ---
public **ResponseString**(HttpResponseMessage response, string value) | 생성자 (Value에 string을 Set)

#### ResponseStream : ResponseBase\<Stream>, IAsyncDisposable
API | 설명
--- | ---
public **ResponseStream**(HttpResponseMessage response, Stream stream) | 생성자 (Value에 Stream을 Set)
public async ValueTask **DisposeAsync**() | Stream 해제

### Enum
#### eAuthorizationType
키 | 설명
--- | ---
**BASIC** | 일반 인증 (ID, Password)
**AUTH_TOKEN** | 토큰 기반 인증 (Bearer 인증)

#### eRequestType
키 | 설명
--- | ---
**GET** <td rowspan="4">REST API 메시지 타입 | 
**POST**
**PUT**
**DELETE**

## 사용 예시
소스코드 전문을 공개할 수 없어 사용 예시로 내용을 대체합니다.

``` csharp
// (GET, POST, PUT, DELETE)
async UniTask SimpleRequestAsync() 
{
    var url = "...";
    var content = "...";
    
    var resultStr = await Client.GET.RequestStringAsync(Url);
    await Client.POST.RequestStringAsync(Url, Content);
    await Client.PUT.RequestStringAsync(Url, Content);
    await Client.DELETE.RequestStringAsync(Url);
}

// HttpRequestMessage
// 1. HttpRequestBuilder.CreateNew
async UniTask SimpleMessageRequest_BuillderAsync()
{
    var url = "...";
    var content = "...";
    var headerKey = "...";
    var headerValue = "...";
    var contentType = "...";
    
    var builder = HttpRequestBuilder.CreateNew(Client.eRequestType.POST, url);
    builder.SetContent(content);
    builder.AddHeader(headerKey, headerValue);
    builder.SetContentType(contentType);
    
    var responseJson = await Client.MESSAGE.RequestStringAsync(builder.Request);
}

// 2. HttpRequestBuilder.Generate
async UniTask SimpleMessageRequest_Builder_AllInOneAsync()
{
    var url = "...";
    var content = "...";
    var headerKey = "..."; var headerValue = "..."; var contentType = "...";
    var request = HttpRequestBuilder.Generate(new HttpRequestMessageInfo 
    {
        RequestMethod = Client.eRequestType.POST, Url = PapagoApiUrl,
        Content = new StringContent(content),
        Headers = new (string, string)[]
        {
            (headerKey, headerValue),
        },
        ContentType = contentType,
    });
    var responseJson = await Client.MESSAGE.RequestStringAsync(request);
}
```