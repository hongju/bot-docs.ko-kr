---
title: .NET Core를 사용하여 활동 인증 | Microsoft Docs
description: .NET Core를 사용하여 봇 활동을 인증하는 방법을 알아봅니다.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6df7923caa708ac2b10af37d860dfac317e113a0
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301874"
---
# <a name="authenticating-activities-using-net-core"></a>.NET Core를 사용하여 활동 인증

[.NET Core](/dotnet/core/index)를 사용하여 봇을 개발하려는 경우 [Bot Framework Connector](bot-builder-dotnet-connector.md)를 사용하여 봇에서 [활동](https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html) 메시지를 보내고 받을 수 있습니다. Connector 서비스를 사용하려면 대상으로 지정할 프레임워크 버전에 대해 적절한 인증 모델을 설정해야 합니다.

Bot Framework Connector.AspNetCore는 다음 버전의 ASP.NET을 지원합니다.
* .NET Core v1.1/AspNetCore1.x의 경우 **Microsoft.Bot.Connector.AspNetCore 1.x** 사용
* .NET Core v2.0/AspNetCore2.x의 경우 **Microsoft.Bot.Connector.AspNetCore 2.x** 사용

이 문서에서는 봇이 대상으로 지정하는 특정 프레임워크에 대한 인증 모델을 설정하는 방법을 보여 줍니다.

## <a name="prerequisites"></a>필수 조건

* [Visual Studio 2017](https://www.visualstudio.com/downloads/)
* [.NET Core](https://www.microsoft.com/net/download/windows). 대상으로 지정한 .NET Core 버전 설치(예: .NET Core v1.1 또는 .NET Core v 2.0)을 설치합니다.
* [봇을 등록](~/bot-service-quickstart-registration.md)합니다. 봇을 등록하여 인증 프로세스에 필요한 AppID 및 암호를 획득합니다.

## <a name="create-a-net-core-project"></a>.NET Core 프로젝트 만들기

.NET Core 프로젝트를 만들려면 다음을 수행합니다.

1. Visual Studio 2017를 열고 **파일 > 새로 만들기 > 프로젝트...** 를 클릭합니다.
2. **Visual C#** 노드를 확장하고 **.NET Core**를 클릭합니다.
3. **ASP.NET Core 웹 응용 프로그램** 프로젝트 형식을 선택하고 프로젝트 정보(예: 이름, 위치 및 솔루션 이름 필드)를 입력합니다.
4. **확인**을 클릭합니다.
5. 프로젝트가 원하는 *.NET Core* 및 *ASP.NET Core* 버전을 대상으로 지정하는지 확인합니다. 예를 들어, 아래 스크린샷은 프로젝트가 **.NET Core** 및 **ASP.NET Core 2.0**을 대상으로 지정하고 있음을 보여 줍니다.

![ASP.NET Core v2.0 프로젝트 만들기](~/media/dotnet-core-authentication/create-asp-net-core-2x-project.png)
 
  > [!NOTE]
  > **ASP.NET Core 2.0**을 대상으로 지정하는 경우 시스템에 설치된 버전이 2.x 이상인지 확인합니다.

6. **Web API** 프로젝트 형식을 선택합니다.
7. **확인**을 클릭하여 프로젝트를 만듭니다.

## <a name="download-the-nuget-package"></a>NuGet 패키지 다운로드

**Bot Framework Connector**를 사용하려면 .NET Core 버전에 적합한 NuGet 패키지를 설치합니다. NuGet 패키지를 설치하려면 다음을 수행합니다.

1. **솔루션 탐색기**에서 프로젝트 이름을 마우스 오른쪽 단추로 클릭하고 **NuGet 패키지 관리...** 를 클릭합니다.
2. **찾아보기**를 클릭하고 **Connector.ASPNetCore**를 검색합니다. 
3. 대상으로 지정할 버전을 선택합니다. 예를 들어, 아래 스크린샷은 버전 **2.0.0.3**이 선택되었음을 보여 줍니다. 버전 1.1.3.2를 설치하려면 **버전** 드롭다운을 클릭하고 버전 **1.1.3.2**를 대신 선택합니다.
![ASP Net Core 버전 2.0.0.3용 NuGet 패키지](~/media/dotnet-core-authentication/nuget-package-net-core-version.png)
4. **Install**을 클릭합니다.

## <a name="update-the-appsettingsjson"></a>appsettings.json 업데이트

Bot Framework Connector에서 봇을 인증하려면 AppID 및 암호가 필요합니다. 웹앱 프로젝트의 **appsettings.json**에서 이러한 값을 설정할 수 있습니다.

> [!NOTE]
> 봇의 **AppID** 및 **AppPassword**를 찾으려면 [MicrosoftAppID 및 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)를 참조하세요.

### <a name="appsettingsjson-for-net-core-v11"></a>.NET Core v1.1용 appsettings.json:

```json
{
  "Logging": {
    "IncludeScopes": false,
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "MicrosoftAppId": "<MicrosoftAppId>",
  "MicrosoftAppPassword": "<MicrosoftAppPassword>"
}
```

### <a name="appsettingsjson-for-net-core-v20"></a>.NET Core v2.0용 appsettings.json:

```json
{
  "Logging": {
    "IncludeScopes": false,
    "Debug": {
      "LogLevel": {
        "Default": "Warning"
      }
    },
    "Console": {
      "LogLevel": {
        "Default": "Warning"
      }
    }
  },
  "MicrosoftAppId": "<MicrosoftAppId>",
  "MicrosoftAppPassword": "<MicrosoftAppPassword>"
}
```

## <a name="update-the-startupcs-class"></a>startup.cs 클래스 업데이트

**startup.cs** 클래스를 업데이트합니다. 프로젝트 버전에 따라, 적절한 코드 조각을 업데이트하여 인증이 작동하도록 합니다.

### <a name="startupcs-for-net-core-v11"></a>.NET Core v1.1용 startup.cs

```cs
public class Startup
    {
        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();
            Configuration = builder.Build();
        }

        public IConfigurationRoot Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton(_ => Configuration);

            services.AddSingleton(typeof(ICredentialProvider), new StaticCredentialProvider(
                Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppIdKey)?.Value,
                Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppPasswordKey)?.Value));

            // Add framework services.
            services.AddMvc(options =>
            {
                options.Filters.Add(typeof(TrustServiceUrlAttribute));
            });
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IServiceProvider serviceProvider, IHostingEnvironment env, ILoggerFactory loggerFactory)
        {
            loggerFactory.AddConsole(Configuration.GetSection("Logging"));
            loggerFactory.AddDebug();

            app.UseStaticFiles();

            ICredentialProvider credentialProvider = serviceProvider.GetService<ICredentialProvider>();

            app.UseBotAuthentication(credentialProvider);

            app.UseMvc();
        }
    }
```

### <a name="startupcs-for-net-core-v20"></a>.NET Core v2.0용 startup.cs:

```cs
public class Startup
    {
        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();
            Configuration = builder.Build();
        }

        public IConfiguration Configuration { get; }
        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton(_ => Configuration);

            var credentialProvider = new StaticCredentialProvider(Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppIdKey)?.Value,
                Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppPasswordKey)?.Value);

            services.AddAuthentication(
                    options =>
                    {
                        options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
                        options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
                    }
                )
                .AddBotAuthentication(credentialProvider);

            services.AddSingleton(typeof(ICredentialProvider), credentialProvider);

            services.AddMvc(options =>
            {
                options.Filters.Add(typeof(TrustServiceUrlAttribute));
            });
        }


        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            app.UseStaticFiles();
            app.UseAuthentication();
            app.UseMvc();
        }
    }
```

## <a name="create-the-messagescontrollercs-class"></a>MessagesController.cs 클래스 만들기

**솔루션 탐색기**에서 **MessagesController.cs**라는 새로운 빈 클래스를 추가합니다. **MessagesController.cs** 클래스에서 아래 코드로 **Post** 메서드를 업데이트합니다. 이렇게 하면 봇에서 사용자에 대한 메시지를 보내고 받을 수 있습니다.

```cs
private IConfiguration configuration;

public MessagesController(IConfiguration configuration)
{
    this.configuration = configuration;
}


[Authorize(Roles = "Bot")]
[HttpPost]
public async Task<OkResult> Post([FromBody] Activity activity)
{
    if (activity.Type == ActivityTypes.Message)
    {
        //MicrosoftAppCredentials.TrustServiceUrl(activity.ServiceUrl);
        var appCredentials = new MicrosoftAppCredentials(configuration);
        var connector = new ConnectorClient(new Uri(activity.ServiceUrl), appCredentials);

        // return our reply to the user
        var reply = activity.CreateReply("HelloWorld");
        await connector.Conversations.ReplyToActivityAsync(reply);
    }
    else
    {
        //HandleSystemMessage(activity);
    }
    return Ok();
}
```
