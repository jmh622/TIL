# 로깅 시스템

## 개발 환경

- .NET 6
- Visual Studio 2022

# Logger 종류

## Serilog

[github Link](https://github.com/serilog/serilog-aspnetcore)

### Program.cs

```csharp
// Serilog를 로거로 설정
builder.Host.UseSerilog((ctx, lc) => lc.WriteTo.Console()
                                       .ReadFrom.Configuration(ctx.Configuration));

// 로거에 Http 정보 로깅 추가
// 추가적으로 IP 정보 표시
app.UseSerilogRequestLogging(options =>
{
    options.MessageTemplate = "{RemoteIpAddress} {RequestScheme} {RequestMethod} {RequestPath} responded {StatusCode} in {Elapsed:0.0000} ms";
    options.EnrichDiagnosticContext = (diagnosticContext, httpContext) =>
    {
        diagnosticContext.Set("RemoteIpAddress", httpContext.Connection.RemoteIpAddress);
        diagnosticContext.Set("RequestScheme", httpContext.Request.Scheme);
    };
});
```

코드로도 Sink 설정을 할 수 있지만, `ReadFrom.Configuration(ctx.Configuration)`을 통해서 `appsettings.json`의 값을 가져와서 설정하도록 하였다.

그 이유는 배포한 뒤에도 로깅 설정을 자유롭게 변경할 수 있도록 하기 위함.

#### HTTP log customizing

기본 Serilog HTTP log message는 `{RequestMethod} {RequestPath} responded {StatusCode} in {Elapsed:0.0000} ms`이다.

여기에 request한 IP와 scheme을 추가하였다.

`MessageTemplate`를 재정의하고 `EnrichDiagnosticContext` 메서드로 재정의한 변수를 설정해주면 된다.

### appsettings.json

개발용과 운영환경용 분리를 하였다.

- 개발용 (appsettings.Development.json)

```json
{
  "Serilog": {
    "MinimumLevel": {
      "Default": "Debug",
      "Override": {
        "Microsoft": "Warning",
        "Microsoft.Hosting.Lifetime": "Information"
      }
    },
    "WriteTo": [
      {
        "Name": "File",
        "Args": {
          "path": "./logs/log-.txt",
          "rollingInterval": "Day"
        }
      }
    ]
  }
}
```

- 운영용 (appsettings.Production.json)

```json
{
  "Serilog": {
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "Microsoft.Hosting.Lifetime": "Information"
      }
    },
    "WriteTo": [
      {
        "Name": "File",
        "Args": {
          "path": "D:\\Logs\\VppApi\\log-.txt",
          "rollingInterval": "Day"
        }
      }
    ]
  }
}
```
