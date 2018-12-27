---
title: .NET용 Bot Builder SDK를 사용하여 봇 만들기 | Microsoft Docs
description: 강력한 봇 생성 프레임워크인 .NET용 Bot Builder SDK를 사용하여 봇을 만듭니다.
keywords: Bot Builder SDK, 봇 만들기, 빠른 시작, 시작
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 04/27/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 16ca658dddbba987545b7aa18c1e8f63f13983f1
ms.sourcegitcommit: 97bb24f15041caccef4ca5736aa14f144881e0c6
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/06/2018
ms.locfileid: "39567572"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-net"></a>.NET용 Bot Builder SDK를 사용하여 봇 만들기

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-quickstart.md)
> - [Node.js](../nodejs/bot-builder-nodejs-quickstart.md)
> - [Bot Service](../bot-service-quickstart.md)
> - [REST](../rest-api/bot-framework-rest-connector-quickstart.md)

<a href="https://github.com/Microsoft/BotBuilder" target="_blank">.NET용 Bot Builder SDK</a>는 Visual Studio 및 Windows를 사용한 봇 개발에 사용되는 편리한 프레임워크입니다. SDK는 C#을 사용하여 .NET 개발자가 강력한 봇을 만들 수 있는 친숙한 방법을 제공합니다.


이 자습서는 봇 애플리케이션 템플릿 및 .NET용 Bot Builder SDK를 사용하여 봇을 빌드한 다음, Bot Framework Emulator로 테스트하는 방법을 안내합니다.

## <a name="prerequisites"></a>필수 조건
1. Visual Studio [2017](https://www.visualstudio.com/).

2. Visual Studio에서 모든 [확장](https://docs.microsoft.com/en-us/visualstudio/extensibility/how-to-update-a-visual-studio-extension)을 최신 버전으로 업데이트합니다.

3. [C#](https://marketplace.visualstudio.com/items?itemName=BotBuilder.BotBuilderV3)을 위한 봇 템플릿.

> [!TIP]
> Visual Studio 2017 프로젝트 템플릿 디렉터리는 일반적으로 `%USERPROFILE%\Documents\Visual Studio 2017\Templates\ProjectTemplates\Visual C#\`에 있으며 항목 템플릿 디렉터리는 `%USERPROFILE%\Documents\Visual Studio 2017\Templates\ItemTemplates\Visual C#\`에 있습니다.

## <a name="create-your-bot"></a>봇 만들기

Visual Studio를 열고 새 C# 프로젝트를 만듭니다. 새 프로젝트에 **간단한 Echo 봇 애플리케이션** 템플릿을 선택합니다.

![Visual Studio 만들기 프로젝트](../media/connector-getstarted-create-project.png)

> [!NOTE]
> Visual Studio에서는 [IIS Express](https://www.microsoft.com/en-us/download/details.aspx?id=48264)를 다운로드하여 설치해야 합니다. 

봇 애플리케이션 템플릿 덕분에 프로젝트에는 이 자습서에서 봇을 만드는 데 필요한 모든 코드가 포함되어 있습니다. 실제로 추가 코드를 작성할 필요가 없습니다. 단, 봇 테스트로 진행하려면 봇 애플리케이션 템플릿이 제공한 코드 몇 가지를 살펴보아야 합니다.

> [!TIP] 
> 필요한 경우 [NuGet 패키지](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio)를 업데이트합니다.

## <a name="explore-the-code"></a>코드 탐색

먼저 **Controllers\MessagesController.cs** 내 `Post` 메서드는 사용자로부터 메시지를 수신하고 루트 대화 상자를 호출합니다.

```csharp
public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
{
    if (activity.GetActivityType() == ActivityTypes.Message)
    {
        await Conversation.SendAsync(activity, () => new Dialogs.RootDialog());
    }
    else
    {
        HandleSystemMessage(activity);
    }
    var response = Request.CreateResponse(HttpStatusCode.OK);
    return response;
}

```

루트 대화 상자는 메시지를 처리하고 응답을 생성합니다. **Dialogs\RootDialog.cs** 내 `MessageReceivedAsync` 메서드는 사용자의 메시지를 다시 에코하는 회신을 전송합니다. 회신은 ‘You sent’ 텍스트를 접두사로 사용하고 ‘which was *##* characters’ 텍스트로 끝나며, 여기서 *##* 은 사용자 메시지의 문자 수를 나타냅니다.

```csharp
public class RootDialog : IDialog<object>
{
    public Task StartAsync(IDialogContext context)
    {
        context.Wait(MessageReceivedAsync);

        return Task.CompletedTask;
    }

    private async Task MessageReceivedAsync(IDialogContext context, IAwaitable<object> result)
    {
        var activity = await result as Activity;

        // Calculate something for us to return
        int length = (activity.Text ?? string.Empty).Length;

        // Return our reply to the user
        await context.PostAsync($"You sent {activity.Text} which was {length} characters");

        context.Wait(MessageReceivedAsync);
    }
}
```

## <a name="test-your-bot"></a>봇 테스트

[!INCLUDE [Get started test your bot](../includes/snippet-getstarted-test-bot.md)]

### <a name="start-your-bot"></a>봇 시작

에뮬레이터를 설치한 후, 브라우저를 애플리케이션 호스트로 사용하여 Visual Studio에서 봇을 시작합니다.
이 Visual Studio 스크린샷은 실행 단추를 클릭하면 봇이 Microsoft Edge에서 시작됨을 보여 줍니다.

![Visual Studio 실행 프로젝트](../media/connector-getstarted-start-bot-locally.png)

실행 단추를 클릭하면 Visual Studio는 애플리케이션을 빌드하고, localhost로 배포하고, 웹 브라우저를 시작하여 애플리케이션의 **default.htm** 페이지를 표시합니다.
예를 들어, 다음은 Microsoft Edge에 표시된 애플리케이션의 **default.htm** 페이지입니다.

![localhost를 실행하는 Visual Studio 봇](../media/connector-getstarted-bot-running-localhost.png)

> [!NOTE]
> 프로젝트 내에 **default.htm** 파일을 수정하여 봇 애플리케이션의 이름 및 설명을 지정할 수 있습니다.

### <a name="start-the-emulator-and-connect-your-bot"></a>에뮬레이터 시작 및 봇 연결

이때 봇은 로컬에서 실행됩니다.
다음으로, 에뮬레이터를 시작한 다음, 에뮬레이터에서 봇에 연결합니다.

1. 새 봇 구성을 만듭니다. 주소 표시줄에 `http://localhost:port-number/api/messages`를 입력합니다. 여기서 *port-number*는 애플리케이션이 실행 중인 브라우저에 표시되는 포트 번호와 일치합니다.

2. **저장 및 연결**을 클릭합니다. **Microsoft 앱 ID** 및 **Microsoft 앱 암호**를 지정하지 않아도 됩니다. 지금은 이러한 필드를 비워 두어도 됩니다. 나중에 [봇을 등록](~/bot-service-quickstart-registration.md)할 때 이 정보를 가져올 수 있습니다.

### <a name="test-your-bot"></a>봇 테스트

봇을 로컬로 실행 중이며 에뮬레이터에 연결되어 있으므로 에뮬레이터에서 몇 가지 메시지를 입력하여 봇을 테스트합니다.
‘You sent’ 텍스트를 접두사로 사용하고 ‘which was *##* characters’ 텍스트로 끝나는 메시지를 다시 에코하여 사용자가 보낸 각 메시지에 봇이 응답하는 것을 확인해야 합니다. 여기서 *##* 은 사용자가 전송한 메시지의 총 문자 수입니다.


> [!TIP]
> 에뮬레이터에서 대화 중 아무 음성 풍선을 클릭합니다. 메시지에 대한 세부 정보는 세부 정보 창에 JSON 형식으로 표시됩니다.

봇 애플리케이션 템플릿 및 .NET용 Bot Builder SDK를 사용하여 봇을 성공적으로 만들었습니다!

## <a name="next-steps"></a>다음 단계

이 빠른 시작에서는 봇 애플리케이션 템플릿 및 .NET용 Bot Builder SDK를 사용하여 간단한 봇을 만들고, Bot Framework Emulator를 사용하여 봇의 기능을 확인했습니다.

다음으로, .NET용 Bot Builder SDK의 주요 개념에 대해 알아봅니다.

> [!div class="nextstepaction"]
> [.NET용 Bot Builder SDK의 주요 개념](bot-builder-dotnet-concepts.md)
