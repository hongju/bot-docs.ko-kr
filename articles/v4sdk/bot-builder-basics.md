---
title: 봇 작동 방식 | Microsoft Docs
description: Bot Framework SDK 내부에서 작업 및 http의 작동 방식을 설명합니다.
keywords: 대화 흐름, 순서, 봇 대화, 대화 상자, 프롬프트, 폭포, 대화 상자 집합
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 187a8427fd8627b0ce6b812ce8ee857e62b0394d
ms.sourcegitcommit: a47183f5d1c2b2454c4a06c0f292d7c075612cdd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/19/2019
ms.locfileid: "67252683"
---
# <a name="how-bots-work"></a>봇 작동 방식

[!INCLUDE[applies-to](../includes/applies-to.md)]

봇은 사용자가 텍스트, 그래픽(예: 카드 또는 이미지) 또는 음성을 사용하여 대화형 방식으로 상호 작용하는 앱입니다. 사용자와 봇 간에 상호 작용이 이루어질 때마다 *활동*이 생성됩니다. Azure Bot Service의 구성 요소인 Bot Framework Service는 사용자의 봇에 연결된 앱(예: Facebook, Skype, Slack 등을 말하며, *채널*이라고 함)과 봇 간에 정보를 전송합니다. 각 채널이 전송하는 활동에 추가 정보가 포함될 수 있습니다. 봇을 만들기 전에 봇이 활동 개체를 사용하여 사용자와 소통하는 방법을 이해하는 것이 중요합니다. 먼저 간단한 에코 봇을 실행할 때 주고받는 활동을 살펴보겠습니다. 

![활동 다이어그램](media/bot-builder-activity.png)

여기에 설명된 두 가지 활동은 *대화 업데이트*와 *메시지*입니다.

대화 상대가 대화에 참여하면 Bot Framework Service에서 대화 업데이트를 보낼 수 있습니다. 예를 들어 Bot Framework Emulator와의 대화를 시작하면 두 가지 대화 업데이트 활동(사용자 대화 참여 활동과 봇 참여 활동)이 표시됩니다. 이러한 대화 업데이트 활동을 구분하려면 *members added* 속성에 봇 아닌 멤버가 포함되어 있는지 확인하세요. 

메시지 활동은 대화 상대 간에 대화 정보를 전달합니다. 에코 봇 예에서는 메시지 활동이 간단한 텍스트를 전달하고 채널이 이 텍스트를 렌더링합니다. 또는 메시지 활동이 말할 텍스트, 제안된 작업 또는 표시할 카드를 전달할 수 있습니다.

이 예에서는 봇이 수신한 인바운드 메시지 활동에 대한 응답으로 메시지 활동을 만들고 전송했습니다. 하지만 봇은 수신된 메시지 활동에 다른 방식으로 응답할 수 있습니다. 봇이 메시지 활동으로 환영 텍스트를 보내 대화 업데이트 활동에 응답하는 것은 일반적이지 않습니다. 자세한 내용은 [사용자 환영](bot-builder-welcome-user.md)에서 찾을 수 있습니다.

### <a name="http-details"></a>HTTP 세부 정보

활동은 HTTP POST 요청을 통해 Bot Framework Service에서 봇에 도착합니다. 봇은 HTTP 상태 코드 200으로 인바운드 POST 요청에 응답합니다. 봇에서 채널로 전송된 활동은 별도의 HTTP POST에서 Bot Framework Service로 전송됩니다. 이는 다시 말해 200 HTTP 상태 코드로 승인이 요청됩니다.

프로토콜은 이러한 POST 요청과 승인 요청이 수행되는 순서를 지정하지 않습니다. 그러나 일반적으로 이러한 요청은 일반적인 HTTP 서비스 프레임워크에 맞추기 위해 중첩됩니다. 즉, 인바운드 HTTP 요청의 범위 내에서 봇이 아웃바운드 HTTP 요청을 수행합니다. 이 패턴은 위의 다이어그램에 나와 있습니다. 두 개의 개별적인 HTTP 연결이 연달아 수행되므로 이에 대한 보안 모델이 제공되어야 합니다.

### <a name="defining-a-turn"></a>순서 정의

대화에서 사람들은 한 번에 한 명씩 순서대로 말을 합니다. 봇을 사용하면 봇은 일반적으로 사용자 입력에 반응합니다. Bot Framework SDK 내에서 _순서_는 사용자로부터 봇으로 들어오는 작업과 봇이 즉각적으로 응답하여 사용자에게 되돌려 보내는 작업으로 구성됩니다. 순서를 특정 작업의 도착과 관련된 프로세싱으로 생각하시면 됩니다.

*순서 컨텍스트* 개체는 발신자 및 수신자, 채널, 활동을 처리하는 데 필요한 기타 데이터와 같은 활동에 대한 정보를 제공합니다. 또한 봇의 다양한 계층에서 순서가 처리되는 동안 정보를 추가할 수 있게 해줍니다.

순서 컨텍스트는 SDK의 가장 중요한 추상화 중 하나입니다. 모든 미들웨어 구성 요소에 대한 인바운드 활동과 애플리케이션 논리를 전달할 뿐만 아니라 미들웨어 구성 요소와 애플리케이션 논리가 아웃바운드 활동을 전송하는 데 사용할 수 있는 메커니즘을 제공합니다.

## <a name="the-activity-processing-stack"></a>활동 처리 스택

이전 다이어그램에서 메시지 활동의 도착을 중점적으로 살펴보도록 하겠습니다.

![활동 처리 스택](media/bot-builder-activity-processing-stack.png)

위의 예제에서 봇은 동일한 텍스트 메시지를 포함하는 다른 메시지 활동을 사용하여 메시지 활동에 응답했습니다. HTTP POST 요청부터 처리되며, 활동 정보는 JSON 페이로드로 전달되고, 웹 서버에 도착합니다. C#에서 이는 일반적으로 ASP.NET 프로젝트가 되며, JavaScript Node.js 프로젝트에서 이는 Express 또는 Restify처럼 인기 있는 프레임워크 중 하나일 가능성이 높습니다.

SDK의 통합 구성 요소인 *어댑터*는 SDK 런타임의 핵심입니다. 작업은 HTTP POST 본문에서 JSON으로 전달됩니다. 이 JSON을 역직렬화하여 작업 개체를 만듭니다. 그런 다음, *처리 작업* 메서드에 대한 호출을 사용하여 어댑터에 전달됩니다. 어댑터가 작업을 수신하면 *순서 컨텍스트*를 만들고 미들웨어를 호출합니다. 

위에서 말했듯이, 순서 컨텍스트는 봇이 주로 인바운드 작업에 대한 응답으로 아웃바운드 작업을 보내는 데 사용하는 메커니즘입니다. 이를 위해 순서 컨텍스트는 _작업 보내기, 업데이트 및 삭제_ 응답 메서드를 제공합니다. 각 응답 메서드는 비동기 프로세스에서 실행됩니다. 

[!INCLUDE [alert-await-send-activity](../includes/alert-await-send-activity.md)]

## <a name="activity-handlers"></a>작업 처리기

봇은 작업을 수신하면 *작업 처리기*로 전달합니다. 내부적으로 *순서 처리기*라는 기본 처리기가 하나 있습니다. 모든 작업은 이곳을 거쳐 라우팅됩니다. 이때 순서 처리기는 어떤 유형의 작업이 수신되든 개별 작업 처리기를 호출합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

예를 들어 봇이 메시지 작업을 수신하면 순서 처리기가 수신 작업을 확인하고 `OnMessageActivityAsync` 작업 처리기로 보냅니다. 

봇을 빌드할 때 메시지 처리 및 응답용 봇 논리는 이 `OnMessageActivityAsync` 처리기로 들어갑니다. 마찬가지로, 대화에 추가되는 멤버를 처리하는 논리는 `OnMembersAddedAsync` 처리기로 들어가고, 대화에 멤버가 추가될 때마다 이 처리기가 호출됩니다.

이러한 처리기의 논리를 구현하려면 아래의 [봇 논리](#bot-logic) 섹션에 나와 있는 것처럼 봇의 이러한 메서드를 재정의합니다. 이러한 각 처리기에 대한 기본 구현은 없으므로 재정의할 때 원하는 논리를 추가하기만 하면 됩니다.

순서 종료 시 [상태 저장](bot-builder-concept-state.md) 같은 논리를 추가하여 기본 순서 처리기를 재정의하려는 경우가 있습니다. 이 경우에는 먼저 `await base.OnTurnAsync(turnContext, cancellationToken);`를 호출하여 `OnTurnAsync`의 기본 구현이 추가 코드 전에 실행되도록 하세요. 기본 구현은 `OnMessageActivityAsync`와 같은 나머지 작업 처리기를 호출하는 역할도 합니다.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

예를 들어 봇이 메시지 작업을 수신하면 순서 처리기가 수신 작업을 확인하고 `onMessage` 작업 처리기로 보냅니다.

봇을 빌드할 때 메시지 처리 및 응답용 봇 논리는 이 `onMessage` 처리기로 들어갑니다. 마찬가지로, 대화에 추가되는 멤버를 처리하는 논리는 `onMembersAdded` 처리기로 들어가고, 대화에 멤버가 추가될 때마다 이 처리기가 호출됩니다.

이러한 처리기의 논리를 구현하려면 아래의 [봇 논리](#bot-logic) 섹션에 나와 있는 것처럼 봇의 이러한 메서드를 재정의합니다. 이러한 각 처리기의 봇 논리를 정의한 후 **마지막에 `next()`를 호출**해야 합니다. `next()`를 호출하면 다음 처리기가 실행되도록 보장할 수 있습니다.

기본 순서 처리기를 재정의하는 상황은 자주 없으므로 그렇게 하려는 경우에는 조심하세요. 순서 종료 시 [상태 저장](bot-builder-concept-state.md)과 같은 기능을 수행할 수 있게 `onDialog`라는 특수한 처리기가 있습니다. `onDialog` 처리기는 다른 모든 처리기가 실행된 후 마지막에 실행되며, 특정 작업 유형에 얽매이지 않습니다. 위의 모든 처리기와 마찬가지로 `next()`를 호출하여 나머지 프로세스가 마무리되도록 하세요.

---

## <a name="middleware"></a>미들웨어

미들웨어는 다른 메시징 미들웨어와 유사하게 순서대로 각각 실행되는 선형 구성 요소 집합을 구성하고 각 작업에서 작동할 수 있는 기회를 제공합니다. 미들웨어 파이프라인의 최종 단계는 애플리케이션이 어댑터의 *작업 처리* 메서드에 등록한 봇 클래스의 순서 처리기 함수를 호출하는 콜백입니다. 작업 처리기는 일반적으로 C#의 `OnTurnAsync` 및 JavaScript의 `onTurn`입니다.

순서 처리기는 순서 컨텍스트를 해당 인수로 설정하고, 순서 처리기 함수 내에서 실행 중인 애플리케이션 논리는 일반적으로 인바운드 작업의 콘텐츠를 처리하고, 응답에서 하나 이상의 작업을 생성하여 순서 컨텍스트에서 *send activity* 함수를 사용하여 이러한 항목을 보냅니다. 순서 컨텍스트에서 *send activity*를 호출하면 미들웨어 구성 요소를 아웃바운드 작업에서 호출하게 됩니다. 미들웨어 구성 요소는 봇의 순서 처리기 함수 전후에서 실행됩니다. 실행은 본질적으로 중첩되며 러시아 인형과 같다고도 합니다. 미들웨어에 대한 자세한 내용은 [미들웨어 항목](~/v4sdk/bot-builder-concept-middleware.md)을 참조하세요.

## <a name="bot-structure"></a>봇 구조체

다음 섹션에서는 [**CSharp**](../dotnet/bot-builder-dotnet-sdk-quickstart.md) 또는 [**JavaScript**](../javascript/bot-builder-javascript-quickstart.md)용으로 제공되는 템플릿을 사용하여 손쉽게 만들 수 있는 EchoBot의 _핵심 부분_을 살펴봅니다.

<!--Need to add section calling out the controller in code, and explaining it further-->

봇은 웹 애플리케이션이며, 각 언어용 템플릿이 제공됩니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

VSIX 템플릿은 [ASP.NET MVC Core](https://dotnet.microsoft.com/apps/aspnet/mvc) 웹앱을 생성합니다. [ASP.NET](https://docs.microsoft.com/aspnet/core/fundamentals/index?view=aspnetcore-2.1&tabs=aspnetcore2x) 기본 사항을 보면 **Program.cs** 및 **Startup.cs**와 같은 파일에서 유사한 코드를 확인할 수 있습니다. 이러한 파일은 모든 웹앱에 필요하며 봇 전용이 아닙니다.

### <a name="appsettingsjson-file"></a>appsettings.json 파일

**appsettings.json** 파일은 앱 ID 및 암호 등과 같은 봇의 구성 정보를 지정합니다. 특정 기술을 사용하거나 프로덕션 환경에서 이 봇을 사용하는 경우, 이 구성에 특정 키 또는 URL을 추가해야 합니다. 그러나 이 Echo 봇의 경우에는 이 시점에 아무 작업도 수행할 필요가 없습니다. 이 때 앱 ID와 암호는 정의되지 않은 상태로 남겨둘 수 있습니다.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

<!-- TODO: Update this aka link to point to samples/javascript_nodejs/02.echobot (instead of samples/javascript_nodejs/02.a.echobot) once work-in-progress is merged into master. -->

Yeoman 생성기는 [restify](http://restify.com/) 웹 애플리케이션 형식을 만듭니다. 해당 문서에서 Restify 빠른 시작을 보면 생성된 **index.js** 파일과 유사한 앱이 표시됩니다. 템플릿이 생성하는 주요 파일 몇 가지가 설명되어 있습니다. 일부 파일의 코드는 복사되지 않지만 봇를 실행할 때 표시되고 [Node.js echobot](https://aka.ms/js-echobot-sample) 샘플을 참조할 수 있습니다.

### <a name="packagejson"></a>package.json

**package.json**은 봇의 종속 항목 및 관련 버전을 지정합니다. 이는 모두 템플릿과 시스템에 의해 설정됩니다.

### <a name="env-file"></a>.env 파일

**.env** 파일은 포트 번호, 앱 ID 및 암호 등과 같은 봇의 구성 정보를 지정합니다. 특정 기술을 사용하거나 프로덕션 환경에서 이 봇을 사용하는 경우, 이 구성에 특정 키 또는 URL을 추가해야 합니다. 그러나 이 Echo 봇의 경우에는 이 시점에 아무 작업도 수행할 필요가 없습니다. 이 때 앱 ID와 암호는 정의되지 않은 상태로 남겨둘 수 있습니다.

**.env** 구성 파일을 사용하려면 템플릿에 추가 패키지가 포함되어 있어야 합니다.  먼저 npm에서 `dotenv` 패키지를 가져옵니다.

`npm install dotenv`

---

### <a name="bot-logic"></a>봇 논리

봇 논리는 하나 이상의 채널에서 들어오는 작업을 처리하고, 이에 대한 응답으로 나가는 작업을 생성합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

주 봇 논리는 봇 코드에 정의됩니다(여기서는 `Bots/EchoBot.cs`라고 함). `EchoBot`은 `ActivityHandler`에서 파생되며, 이는 `IBot` 인터페이스에서 파생됩니다. `ActivityHandler`는 각종 작업을 위한 다양한 처리기(예: 여기에 정의된 `OnMessageActivityAsync` 및 `OnMembersAddedAsync`)를 정의합니다. 이러한 메서드는 보호되지만 `ActivityHandler`에서 파생되므로 덮어쓸 수 있습니다.

`ActivityHandler`에 정의된 처리기는 다음과 같습니다.

| 행사 | Handler | 설명 |
| :-- | :-- | :-- |
| 작업 유형이 수신됨 | `OnTurnAsync` | 수신된 작업의 유형에 따라 다른 처리기 중 하나를 호출합니다. |
| 메시지 작업이 수신됨 | `OnMessageActivityAsync` | `Message` 작업을 처리하도록 재정의합니다. |
| 대화 업데이트 작업이 수신됨 | `OnConversationUpdateActivityAsync` | `ConversationUpdate` 작업에서 봇 이외 멤버가 대화에 참가하거나 대화를 나간 경우 처리기를 호출합니다. |
| 봇 이외 멤버가 대화에 참가함 | `OnMembersAddedAsync` | 대화에 참가한 멤버를 처리하도록 재정의합니다. |
| 봇 이외 멤버가 대화를 나감 | `OnMembersRemovedAsync` | 대화를 나간 멤버를 처리하도록 재정의합니다. |
| 이벤트 작업이 수신됨 | `OnEventActivityAsync` | `Event` 작업에서 이벤트 유형과 관련된 처리기를 호출합니다. |
| 토큰-응답 이벤트 작업이 수신됨 | `OnTokenResponseEventAsync` | 토큰 응답 이벤트를 처리하도록 재정의합니다. |
| 토큰-응답 이외 이벤트 작업이 수신됨 | `OnEventAsync` | 다른 유형의 이벤트를 처리하도록 재정의합니다. |
| 다른 작업 유형이 수신됨 | `OnUnrecognizedActivityTypeAsync` | 달리 처리되지 않은 작업 유형을 처리하도록 재정의합니다. |

이러한 여러 처리기에는 수신 작업에 대한 정보를 제공하는 `turnContext`가 있습니다(인바운드 HTTP 요청에 해당). 다양한 작업 유형이 있을 수 있으므로 각 처리기는 순서 컨텍스트 매개 변수에 강력한 형식의 작업을 제공합니다. 대부분의 경우 `OnMessageActivityAsync`는 항상 처리되고, 일반적으로 가장 많이 사용됩니다.

이 프레임워크의 이전 4.x 버전에는 공용 메서드 `OnTurnAsync`를 구현하는 옵션도 있습니다. 현재 이 메서드의 기본 구현은 오류 검사를 처리한 후 수신 작업 유형에 따라 특정 처리기(예: 이 샘플에 정의된 두 처리기)를 각각 호출합니다. 대부분의 경우 해당 메서드를 그대로 두고 개별 처리기를 사용할 수 있지만 `OnTurnAsync`의 사용자 지정 구현이 필요한 상황에서는 그렇게 할 수 있습니다.

> [!IMPORTANT]
> `OnTurnAsync` 메서드를 재정의하는 경우에는 `base.OnTurnAsync`를 호출하여 기본 구현을 가져와서 다른 `On<activity>Async` 처리기를 모두 호출하거나 이러한 처리기를 직접 호출해야 합니다. 그렇지 않을 경우 이러한 처리기가 처리되지 않으며, 해당 코드가 실행되지 않습니다.

이 샘플에서는 `SendActivityAsync` 호출을 사용하여 새 사용자를 환영하거나 사용자가 보낸 메시지를 출력합니다. 아웃바운드 작업은 아웃바운드 HTTP POST 요청에 해당합니다.

```cs
public class MyBot : ActivityHandler
{
    protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
    {
        await turnContext.SendActivityAsync(MessageFactory.Text($"Echo: {turnContext.Activity.Text}"), cancellationToken);
    }

    protected override async Task OnMembersAddedAsync(IList<ChannelAccount> membersAdded, ITurnContext<IConversationUpdateActivity> turnContext, CancellationToken cancellationToken)
    {
        foreach (var member in membersAdded)
        {
            await turnContext.SendActivityAsync(MessageFactory.Text($"welcome {member.Name}"), cancellationToken);
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

주 봇 논리는 봇 코드에 정의됩니다(여기서는 `bots\echoBot.js`라고 함). `EchoBot`은 `ActivityHandler`에서 파생됩니다. `ActivityHandler`는 각종 작업을 위한 다양한 처리기를 정의하며, 여기에 나와 있는 `onMessage` 및 `onConversationUpdate`를 통해 추가 논리를 제공하면 봇의 동작을 수정할 수 있습니다.

`ActivityHandler`에 정의된 처리기는 다음과 같습니다.

| 행사 | Handler | 설명 |
| :-- | :-- | :-- |
| 작업 유형이 수신됨 | `onTurn` | 수신된 작업의 유형에 따라 다른 처리기 중 하나를 호출합니다. |
| 메시지 작업이 수신됨 | `onMessage` | `Message` 작업을 처리할 수 있는 함수를 제공합니다. |
| 대화 업데이트 작업이 수신됨 | `onConversationUpdate` | `ConversationUpdate` 작업에서 봇 이외 멤버가 대화에 참가하거나 대화를 나간 경우 처리기를 호출합니다. |
| 봇 이외 멤버가 대화에 참가함 | `onMembersAdded` | 대화에 참가한 멤버를 처리할 수 있는 함수를 제공합니다. |
| 봇 이외 멤버가 대화를 나감 | `onMembersRemoved` | 대화를 나간 멤버를 처리할 수 있는 함수를 제공합니다. |
| 이벤트 작업이 수신됨 | `onEvent` | `Event` 작업에서 이벤트 유형과 관련된 처리기를 호출합니다. |
| 토큰-응답 이벤트 작업이 수신됨 | `onTokenResponseEvent` | 토큰 응답 이벤트를 처리할 수 있는 함수를 제공합니다. |
| 다른 작업 유형이 수신됨 | `onUnrecognizedActivityType` | 달리 처리되지 않은 작업 유형을 처리할 수 있는 함수를 제공합니다. |
| 작업 처리기가 완료됨 | `onDialog` | 나머지 작업 처리기가 완료된 후 순서 종료 시 수행해야 하는 과정을 처리할 수 있는 함수를 제공합니다. |

각 순서에서는 먼저 봇 메시지를 받았는지 확인합니다. 사용자로부터 메시지가 수신되면 사용자가 보낸 메시지를 출력합니다.

```javascript
const { ActivityHandler } = require('botbuilder');

class MyBot extends ActivityHandler {
    constructor() {
        super();
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        this.onMessage(async (context, next) => {
            await context.sendActivity(`You said '${ context.activity.text }'`);
            // By calling next() you ensure that the next BotHandler is run.
            await next();
        });
        this.onConversationUpdate(async (context, next) => {
            await context.sendActivity('[conversationUpdate event detected]');
            // By calling next() you ensure that the next BotHandler is run.
            await next();
        });
    }
}

module.exports.MyBot = MyBot;
```

---

### <a name="access-the-bot-from-your-app"></a>앱에서 봇에 액세스

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

#### <a name="set-up-services"></a>서비스 설정

`Startup.cs` 파일의 `ConfigureServices` 메서드는 `appsettings.json` 또는 Azure Key Vault(있는 경우)에서 연결된 서비스와 키를 로드하고, 상태를 연결하는 등의 역할을 합니다. 여기에서는 MVC를 추가하고, 서비스의 호환성 버전을 설정한 후 봇 컨트롤러에 종속성을 주입하여 사용할 수 있게 어댑터와 봇을 설정합니다.

<!-- want to explain the singleton vs transient here?-->

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

    // Create the credential provider to be used with the Bot Framework Adapter.
    services.AddSingleton<ICredentialProvider, ConfigurationCredentialProvider>();

    // Create the Bot Framework Adapter.
    services.AddSingleton<IBotFrameworkHttpAdapter, BotFrameworkHttpAdapter>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```

`Configure` 메서드는 MVC 및 몇 가지 다른 파일을 사용하도록 지정하여 앱의 구성을 완료합니다. Bot Framework를 사용하는 다른 봇은 이러한 구성 호출이 필요하지만 봇을 빌드할 때 샘플이나 VSIX 템플릿에 이미 정의되어 있을 것입니다. `ConfigureServices` 및 `Configure`는 앱이 시작될 때 런타임에서 호출합니다.

#### <a name="bot-controller"></a>봇 컨트롤러

표준 MVC 구조를 따르는 이 컨트롤러를 사용하여 메시지 및 HTTP POST 요청의 라우팅을 결정할 수 있습니다. 봇의 경우 위의 [작업 처리 스택](#the-activity-processing-stack) 섹션에 설명된 것처럼 어댑터의 *비동기 작업 처리* 메서드에 수신 요청이 전달됩니다. 이러한 호출에는 봇 및 기타 필요할 수 있는 권한 부여 정보를 지정합니다.

이 컨트롤러는 `ControllerBase`를 구현하고, 설정된 어댑터 및 봇을 `Startup.cs`에 저장하며(종속성 주입을 통해 여기서 사용 가능), HTTP POST를 수신하면 필요한 정보를 봇에 전달합니다.

여기에서는 경로 및 컨트롤러 특성이 처리하는 클래스를 살펴봅니다. 이는 프레임워크가 메시지를 적절히 라우팅하고, 어떤 컨트롤러를 사용해야 할지 파악하도록 도와줍니다. 경로 특정의 값을 변경하면 에뮬레이터 또는 다른 채널이 봇에 액세스할 때 사용하는 엔드포인트가 변경됩니다.

```cs
// This ASP Controller is created to handle a request. Dependency Injection will provide the Adapter and IBot
// implementation at runtime. Multiple different IBot implementations running at different endpoints can be
// achieved by specifying a more specific type for the bot constructor argument.
[Route("api/messages")]
[ApiController]
public class BotController : ControllerBase
{
    private readonly IBotFrameworkHttpAdapter Adapter;
    private readonly IBot Bot;

    public BotController(IBotFrameworkHttpAdapter adapter, IBot bot)
    {
        Adapter = adapter;
        Bot = bot;
    }

    [HttpPost]
    public async Task PostAsync()
    {
        // Delegate the processing of the HTTP POST to the adapter.
        // The adapter will invoke the bot.
        await Adapter.ProcessAsync(Request, Response, Bot);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

#### <a name="indexjs"></a>index.js

`index.js`는 봇 논리로 활동을 전달하는 호스팅 서비스와 봇을 설정합니다.

#### <a name="required-libraries"></a>필수 라이브러리

`index.js` 파일의 맨 위쪽에는 필요한 일련의 모듈 또는 라이브러리가 있습니다. 이러한 모듈은 사용자가 애플리케이션에 포함하는 함수 집합에 액세스할 수 있습니다.

```javascript
const dotenv = require('dotenv');
const path = require('path');
const restify = require('restify');

// Import required bot services.
// See https://aka.ms/bot-services to learn more about the different parts of a bot.
const { BotFrameworkAdapter } = require('botbuilder');

// This bot's main dialog.
const { MyBot } = require('./bot');

// Import required bot configuration.
const ENV_FILE = path.join(__dirname, '.env');
dotenv.config({ path: ENV_FILE });
```

#### <a name="set-up-services"></a>서비스 설정

새로운 부분에서는 봇이 사용자와 통신하고 응답을 보낼 수 있게 해주는 서버와 어댑터를 설정합니다. 서버는 구성 파일의 지정된 포트에서 수신 대기하거나, 에뮬레이터와의 연결을 위해 _3978_로 폴백합니다. 어댑터는 봇의 지휘자 역할을 하며 들어오고 나가는 통신, 인증 등을 지시합니다.

```javascript
// Create HTTP server
const server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, () => {
    console.log(`\n${ server.name } listening to ${ server.url }`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator`);
    console.log(`\nTo talk to your bot, open the emulator select "Open Bot"`);
});

// Create adapter.
// See https://aka.ms/about-bot-adapter to learn more about how bots work.
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});

// Catch-all for errors.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${ error }`);
    // Send a message to the user
    await context.sendActivity(`Oops. Something went wrong!`);
};

// Create the main dialog.
const myBot = new MyBot();
```

#### <a name="forwarding-requests-to-the-bot-logic"></a>봇 논리로 요청 전달

어댑터의 `processActivity`는 수신 활동을 봇 논리로 보냅니다.
`processActivity` 내의 세 번째 매개 변수는 수신된 [작업](#the-activity-processing-stack)이 어댑터에 의해 사전 처리되고 모든 미들웨어를 통해 라우팅된 후 봇의 논리를 수행하기 위해 호출되는 함수 처리기입니다. 함수 처리기에 인수로 전달된 순서 컨텍스트 변수는 수신 활동, 보낸 사람 및 받는 사람, 채널, 대화 등에 대한 정보를 제공하는 데 사용될 수 있습니다. 작업 처리가 봇의 `run` 메서드로 라우팅됩니다. `run`은 `ActivityHandler`에 정의되며, 몇 가지 오류 검사를 수행한 후, 수신된 작업 유형에 따라 봇의 이벤트 처리기를 호출합니다.

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        // Route to main dialog.
        await myBot.run(context);
    });
});
```

---

## <a name="manage-bot-resources"></a>봇 리소스 관리

앱 ID, 암호, 연결된 서비스의 키 또는 비밀과 같은 봇 리소스는 적절히 관리해야 합니다. 이를 수행하는 방법에 대한 자세한 내용은 [봇 리소스 관리](bot-file-basics.md)를 참조하세요.

## <a name="additional-resources"></a>추가 리소스

- 봇 상태의 역할을 이해하려면 [상태 관리](bot-builder-concept-state.md)를 참조하세요.
