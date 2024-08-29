Typical exceptions :skull: which should be captured in Azure APIM communications 

```csharp
await Policy
    .Handle<HttpRequestException>()
    .Or<InvalidDataException>()
    .Or<WebException>()
    .Or<OperationCanceledException>()
    .WaitAndRetryAsync()
    .ExecuteAsync();
```

Links:
- [learn.microsoft.com - HttpRequestException](https://learn.microsoft.com/en-us/dotnet/api/system.net.http.httprequestexception?view=net-8.0)
- [learn.microsoft.com - InvalidDataException](https://learn.microsoft.com/en-us/dotnet/api/system.io.invaliddataexception?view=net-8.0)
- [learn.microsoft.com - WebException](https://learn.microsoft.com/en-us/dotnet/api/system.net.webexception?view=net-8.0)
- [learn.microsoft.com - OperationCanceledException](https://learn.microsoft.com/pl-pl/dotnet/api/system.operationcanceledexception?view=net-8.0)
