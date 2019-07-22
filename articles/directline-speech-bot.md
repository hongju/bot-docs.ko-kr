---
title: Direct Line Speech 봇 개발 | Microsoft Docs
description: Direct Line Speech 봇 개발
keywords: Direct Line Speech 봇(음성 봇) 개발
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 07/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 898136303d2c5bbf6a8ce5ea5b87bff0bac0a5c7
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230762"
---
# <a name="use-direct-line-speech-in-your-bot"></a>봇에서 Direct Line Speech 사용 

[!INCLUDE [applies-to-v4](includes/applies-to.md)]

Direct Line Speech는 Bot Framework의 새로운 WebSocket 기반 스트리밍 기능을 사용하여 Direct Line Speech 채널과 봇 사이에 메시지를 교환합니다. Azure Portal에서 Direct Line Speech 채널을 활성화한 후 봇을 수신 대기하고 이 WebSocket 연결을 허용하도록 업데이트해야 합니다. 이 지침은 이 작업을 수행하는 방법을 설명합니다.

## <a name="add-the-nuget-package"></a>Nuget 패키지 추가
Direct Line Speech 미리 보기를 위해 봇에 추가해야 하는 추가 NuGet 패키지가 있습니다. 해당 패키지는 myget.org NuGet 피드에서 호스트됩니다.
1.  Visual Studio에서 봇의 프로젝트를 엽니다.

2.  봇 프로젝트의 속성 아래에 있는 Nuget 패키지 관리로 이동합니다.

3.  `Microsoft.Bot.Builder.StreamingExtensions` 패키지를 추가합니다. 미리 보기 패키지를 보려면 "시험판 포함" 확인란을 선택해야 합니다.

4.  프롬프트를 모두 수락하고 프로젝트에 패키지 추가를 완료합니다.

## <a name="set-the-speak-field-on-activities-you-want-spoken-to-the-user"></a>사용자에게 말하려는 활동에 대한 Speak(대화) 필드 설정
사용자에게 말하려는 봇에서 보내는 Activity의 Speak 필드를 설정해야 합니다. 

```cs
public IActivity Speak(string message)
{
    var activity = MessageFactory.Text(message);
    string body = @"<speak version='1.0' xmlns='https://www.w3.org/2001/10/synthesis' xml:lang='en-US'>
        <voice name='Microsoft Server Speech Text to Speech Voice (en-US, JessaNeural)'>" +
        $"{message}" + "</voice></speak>";
    activity.Speak = body;
    return activity;
}
```

## <a name="option-1-update-your-net-core-bot-code-if-your-bot-has-a-botcontrollercs"></a>옵션 #1: _봇에 BotController.cs가 있는 경우_ .NET Core 봇 코드를 업데이트합니다.
템플릿(예: EchoBot) 중 하나를 사용하여 Azure Portal에서 새 봇을 만들면 단일 POST 엔드포인트를 표시하는 ASP.NET MVC 컨트롤러를 포함하는 봇을 얻습니다. 이 지침은 해당 봇을 GET 엔드포인트인 WebSocket 스트리밍 엔드포인트를 수락하는 엔드포인트도 표시하도록 확장하는 방법을 설명합니다.
1.  솔루션의 Controllers 아래에 있는 BotController.cs를 엽니다.

2.  클래스에서 PostAsync 메서드를 찾고 해당 메서드의 장식을 [HttpPost]에서 [HttpPost, HttpGet]로 업데이트합니다.
```cs
[HttpPost, HttpGet]
public async Task PostAsync()
{ 
    await _adapter.ProcessAsync(Request, Response, _bot);
}
```

3.  BotController.cs를 저장한 후 닫습니다.

4.  솔루션의 루트에서 Startup.cs를 엽니다.

5.  새 네임스페이스를 추가합니다.

```cs
using Microsoft.Bot.Builder.StreamingExtensions;
```

6.  ConfigureServices 메서드에서 AdapterWithErrorHandler의 사용을 해당 services.AddSingleton 호출의 WebSocketEnabledHttpAdapter로 바꿉니다.

```cs
public void ConfigureServices(IServiceCollection services)
{
    ...    

    // Create the Bot Framework Adapter.
    services.AddSingleton<IBotFrameworkHttpAdapter, WebSocketEnabledHttpAdapter>();

    services.AddTransient<IBot, EchoBot>();

    ...
}
```

7. 여전히 Startup.cs에서 Configure 메서드의 맨 아래로 이동합니다. app.UseMvc(); 호출의 앞에 app.UseWebSockets();를 추가합니다(이는 Use 호출의 순서에 관한 사항이므로 중요함). 메서드의 끝이 아래와 같아야 합니다.

```cs
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    ...

    app.UseDefaultFiles();
    app.UseStaticFiles();
    app.UseWebSockets();
    app.UseMvc();

    ...
}
```

8.  봇 코드의 나머지 부분은 동일하게 유지됩니다!

## <a name="option-2-update-your-net-core-bot-code-if-your-bot-uses-addbot-and-usebotframework-instead-of-a-botcontroller"></a>옵션 #2: _봇이 BotController 대신 AddBot 및 UseBotFramework를 사용하는 경우_  .NET Core 봇 코드를 업데이트합니다.

버전 4.3.2 이전의 Bot Builder SDK v4를 사용하여 봇을 만든 경우 해당 봇은 BotController를 포함하지 않는 대신 Startup.cs 파일에 AddBot() 및 UseBotFramework() 메서드를 사용하여 봇이 메시지를 수신하는 POST 엔드포인트를 표시할 가능성이 있습니다. 새 스트리밍 엔드포인트를 표시하려면 BotController를 추가하고 AddBot() 및 UseBotFramework() 메서드를 제거해야 합니다. 이 지침은 수행해야 할 변경을 단계별로 안내합니다.

1.  BotController.cs라는 파일을 추가하여 새 MVC 컨트롤러를 봇 프로젝트에 추가합니다. 이 파일에 컨트롤러 코드를 추가합니다.

```cs
[Route("api/messages")]
[ApiController]
public class BotController : ControllerBase
{
    private readonly IBotFrameworkHttpAdapter _adapter;
    private readonly IBot _bot;

    public BotController(IBotFrameworkHttpAdapter adapter, IBot bot)
    {
        _adapter = adapter;
        _bot = bot;
    }

    [HttpPost, HttpGet]
    public async Task ProcessMessageAsync()
    {
        await _adapter.ProcessAsync(Request, Response, _bot);
    }
}
```
2.  Startup.cs 파일에서 Configure 메서드를 찾습니다. UseBotFramework() 줄을 제거하고 다음 줄이 있는지 확인합니다.

```cs
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    ...

    app.UseDefaultFiles();
    app.UseStaticFiles();
    app.UseWebSockets();
    app.UseMvc();

    ...
}
```

3.  역시 Startup.cs 파일에서 ConfigureServices 메서드를 찾습니다. AddBot() 줄을 제거하고 다음 줄이 있는지 확인합니다.

```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

    services.AddSingleton<ICredentialProvider, ConfigurationCredentialProvider>();

    services.AddSingleton<IChannelProvider, ConfigurationChannelProvider>();

    // Create the Bot Framework Adapter.
    services.AddSingleton<IBotFrameworkHttpAdapter, WebSocketEnabledHttpAdapter>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```
4.  봇 코드의 나머지 부분은 동일하게 유지됩니다!

## <a name="next-steps"></a>다음 단계
> [!div class="nextstepaction"]
> [Direct Line Speech에 봇 연결(미리 보기)](./bot-service-channel-connect-directlinespeech.md)
