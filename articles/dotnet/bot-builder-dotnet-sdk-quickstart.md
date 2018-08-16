---
title: .NET용 Bot Builder SDK를 사용하여 봇 만들기 | Microsoft Docs
description: 강력한 봇 생성 프레임워크인 .NET용 Bot Builder SDK를 사용하여 봇을 만듭니다.
keywords: Bot Builder SDK, 봇 만들기, 빠른 시작, .NET, 시작
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 04/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 98c6103f0cd38bf6d3f477735dafcf01952304d5
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304147"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-v4-for-net"></a>.NET용 Bot Builder SDK v4를 사용하여 봇 만들기
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

이 빠른 시작은 봇 응용 프로그램 템플릿 및 .NET용 Bot Builder SDK를 사용하여 봇을 빌드한 다음, Bot Framework Emulator로 테스트하는 방법을 안내합니다. [Microsoft Bot Builder SDK v4](https://github.com/Microsoft/botbuilder-dotnet)를 기반으로 합니다.

## <a name="prerequisites"></a>필수 조건
- Visual Studio [2017](https://www.visualstudio.com/downloads)
- [C#](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4)용 Bot Builder SDK v4 템플릿
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases)
- [ASP.Net Core](https://docs.microsoft.com/aspnet/core/) 및 [C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/index)의 비동기 프로그래밍에 대한 정보

## <a name="create-a-bot"></a>봇 만들기
필수 조건 섹션에서 다운로드한 BotBuilderVSIX.vsix 템플릿을 설치합니다. 

Visual Studio에서 새 프로젝트를 만듭니다.

![Visual Studio 프로젝트](../media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> 필요한 경우 [NuGet 패키지](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio)를 업데이트합니다.

## <a name="explore-code"></a>코드 탐색
Startup.cs 파일을 열어 `ConfigureServices(IServiceCollection services)` 메서드의 코드를 검토합니다. `CatchExceptionMiddleware` 미들웨어가 메시징 파이프라인에 추가됩니다. 이는 다른 미들웨어 또는 OnTurn 메서드에서 throw된 예외를 처리합니다. 

```cs
options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
{
    await context.TraceActivity("EchoBot Exception", exception);
    await context.SendActivity("Sorry, it looks like something went wrong!");
}));
```

대화 상태 미들웨어는 메모리 내 저장소를 사용합니다. 저장소에 EchoState를 작성하고 읽습니다.  EchoState 클래스의 턴 카운트는 봇에 전송된 메시지 수를 추적합니다. 비슷한 기술을 사용하여 턴 사이의 상태를 유지 관리할 수 있습니다.

```cs
 IStorage dataStore = new MemoryStorage();
 options.Middleware.Add(new ConversationState<EchoState>(dataStore));
```

`Configure(IApplicationBuilder app, IHostingEnvironment env)` 메서드는 `.UseBotFramework`를 호출하여 들어오는 작업을 봇 어댑터에 라우팅합니다. 

EchoBot.cs 파일의 `OnTurn(ITurnContext context)` 메서드는 들어오는 작업 형식을 확인하고 사용자에게 회신을 전송하는 데 사용됩니다. 

```cs
public async Task OnTurn(ITurnContext context)
{
    // This bot is only handling Messages
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context
        var state = context.GetConversationState<EchoState>();

        // Bump the turn count. 
        state.TurnCount++;

        // Echo back to the user whatever they typed.
        await context.SendActivity($"Turn {state.TurnCount}: You sent '{context.Activity.Text}'");
    }
}
```
## <a name="start-your-bot"></a>봇 시작

- `default.html` 페이지가 브라우저에 표시됩니다.
- 페이지에 대한 localhost 포트 번호를 적어 둡니다. 이 정보는 봇과 상호 작용하기 위해 필요합니다.

### <a name="start-the-emulator-and-connect-your-bot"></a>에뮬레이터 시작 및 봇 연결

이 시점에서 봇이 로컬로 실행됩니다.
다음으로, 에뮬레이터를 시작한 다음, 에뮬레이터에서 봇에 연결합니다.

1. 에뮬레이터 “시작” 탭에서 **새 봇 구성 만들기** 링크를 클릭합니다. 

2. **봇 이름**을 입력하고 봇 코드에 대한 디렉터리 경로를 입력합니다. 봇 구성 파일은 이 경로에 저장됩니다.

3. **엔드포인트 URL** 필드에 `http://localhost:port-number/api/messages`를 입력합니다. 여기서 *port-number*는 응용 프로그램이 실행되는 브라우저에 표시된 포트 번호와 일치합니다.

4. **연결**을 클릭하여 봇에 연결합니다. **Microsoft 앱 ID** 및 **Microsoft 앱 암호**를 지정하지 않아도 됩니다. 지금은 이러한 필드를 비워 두어도 됩니다. 나중에 봇을 등록할 때 이 정보를 가져올 수 있습니다.

## <a name="interact-with-your-bot"></a>봇과의 상호 작용

봇에 “Hi”라고 보내면 봇은 메시지에 대해 “Turn 1: You sent Hi”라고 응답합니다.

## <a name="next-steps"></a>다음 단계

다음으로, [Azure에 봇을 배포](../bot-builder-howto-deploy-azure.md)하거나 이제 봇 및 작동 방식을 설명하는 개념으로 이동합니다.

> [!div class="nextstepaction"]
> [기본 봇 개념](../v4sdk/bot-builder-basics.md)
