# REST API 헬퍼
## 개요
C# HttpClient을 통해 REAT API 호출을 처리하는 헬퍼 라이브러리

## 구조
![](./images/http_helper.png)

## 사용 예시
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