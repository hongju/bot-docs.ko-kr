---
title: 점수 지정 가능 개체를 사용하는 전역 메시지 처리기
description: .NET용 Bot Framework SDK에서 점수 지정 가능 개체를 사용하여 보다 유연한 대화 상자를 만듭니다.
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: b01c5ad1506850a1567d471c85cc3dcd080ab3d7
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225568"
---
# <a name="global-message-handlers-using-scorables"></a>점수 지정 가능 개체를 사용하는 전역 메시지 처리기

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

사용자는 대화 중 봇이 다른 응답을 예상할 때 “도움말”, “취소” 또는 “다시 시작”과 같은 단어를 사용하여 봇의 특정 기능에 액세스하려고 합니다. 점수 지정 가능 대화 상자를 사용하여 이러한 요청을 정상적으로 처리하도록 봇을 디자인할 수 있습니다.

점수 지정 가능 대화 상자는 들어오는 메시지를 모두 모니터링하고 메시지가 어떤 방식으로든 실행 가능한지 확인합니다. 점수 지정 가능 대화 상자는 점수 지정 가능 메시지에 [0 - 1] 사이의 점수를 할당합니다. 최고 점수를 결정하는 점수 지정 가능 대화 상자가 대화 상자 스택의 맨 위에 추가되고 사용자에게 응답을 전달합니다. 점수 지정 가능 대화 상자의 실행이 완료된 후 대화가 중단된 위치부터 계속됩니다.

scorables를 사용하면 사용자가 일반적인 대화 상자에서 발견되는 정상적인 대화 흐름을 ‘중단’할 수 있도록 허용하여 보다 유연한 대화를 만들 수 있습니다.

## <a name="create-a-scorable-dialog"></a>점수 지정 가능 대화 상자 만들기

먼저, 새 [대화 상자](bot-builder-dotnet-dialogs.md)를 정의합니다. 다음 코드는 `IDialog` 인터페이스에서 파생된 대화 상자를 사용합니다.

```cs
public class SampleDialog : IDialog<object>
{
    public async Task StartAsync(IDialogContext context)
    {
        await context.PostAsync("This is a Sample Dialog which is Scorable. Reply with anything to return to the prior prior dialog.");

        context.Wait(this.MessageReceived);
    }

    private async Task MessageReceived(IDialogContext context, IAwaitable<IMessageActivity> result)
    {
        var message = await result;

        if ((message.Text != null) && (message.Text.Trim().Length > 0))
        {
            context.Done<object>(null);
        }
        else
        {
            context.Fail(new Exception("Message was not a string or was an empty string."));
        }
    }
}
```
점수 지정 가능 대화 상자를 만들려면 `ScorableBase` 추상 클래스에서 상속되는 클래스를 만듭니다. 다음 코드는 `SampleScorable` 클래스를 보여 줍니다.

```cs
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;
using Microsoft.Bot.Builder.Internals.Fibers;
using Microsoft.Bot.Builder.Scorables.Internals;

public class SampleScorable : ScorableBase<IActivity, string, double>
{
    private readonly IDialogTask task;

    public SampleScorable(IDialogTask task)
    {
        SetField.NotNull(out this.task, nameof(task), task);
    }
}
```
`ScorableBase` 추상 클래스는 `IScorable` 인터페이스에서 상속됩니다. 클래스에서 다음 `IScorable` 메서드를 구현해야 합니다.

- `PrepareAsync`는 점수 지정 가능 인스턴스에서 호출되는 첫 번째 메서드입니다. 들어오는 메시지 활동을 수락하고, 대화 상자의 상태를 분석 및 설정합니다. 이 상태는 `IScorable` 인터페이스의 다른 모든 메서드에 전달됩니다.

```cs
protected override async Task<string> PrepareAsync(IActivity item, CancellationToken token)
{
        // TODO: insert your code here
}
```

- `HasScore` 메서드는 상태 속성을 검사하여 점수 지정 가능 대화 상자에서 메시지에 대한 점수를 제공해야 하는지 확인합니다. false가 반환되면 점수 지정 가능 대화 상자에서 메시지를 무시합니다.

```cs
protected override bool HasScore(IActivity item, string state)
{
        // TODO: insert your code here
}
```

- `GetScore`는 `HasScore`에서 true가 반환되는 경우에만 트리거됩니다. 이 메서드에 논리를 프로비전하여 0 - 1 사이의 메시지 점수를 확인합니다.

```cs
protected override double GetScore(IActivity item, string state)
{
        // TODO: insert your code here
}
```
- `PostAsync` 메서드에서 점수 지정 가능 클래스에 대해 수행할 핵심 작업을 정의합니다. 모든 점수 지정 가능 대화 상자는 들어오는 메시지를 모니터링하고, 점수 지정 가능 개체의 GetScore 메서드를 기준으로 유효한 메시지에 점수를 할당합니다. 그런 다음, 최고 점수(0 - 1.0 사이)를 확인하는 점수 지정 가능 클래스가 해당 점수 지정 가능 개체의 `PostAsync` 메서드를 트리거합니다.

```cs
protected override Task PostAsync(IActivity item, string state, CancellationToken token)
{
        //TODO: insert your code here
}
```

- 점수 지정 프로세스가 완료되면 `DoneAsync`가 호출됩니다. 이 메서드를 사용하여 범위가 지정된 리소스를 삭제합니다.

```cs
protected override Task DoneAsync(IActivity item, string state, CancellationToken token)
{
        //TODO: insert your code here
}
```

## <a name="create-a-module-to-register-the-iscorable-service"></a>IScorable 서비스를 등록할 모듈 만들기

이제 `SampleScorable` 클래스를 구성 요소로 등록할 `Module`을 정의합니다. 이렇게 하면 `IScorable` 서비스가 프로비전됩니다.

```cs
public class GlobalMessageHandlersBotModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        base.Load(builder);

        builder
            .Register(c => new SampleScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();
    }
}
```
## <a name="register-the-module"></a>모듈 등록  

프로세스의 마지막 단계는 봇의 대화 컨테이너에 `SampleScorable`을 적용하는 것입니다. 이렇게 하면 점수 지정 가능 서비스가 봇 프레임워크의 메시지 처리 파이프라인에 등록됩니다. 다음 코드는 **Global.asax.cs**에서 봇 앱의 초기화에 있는 `Conversation.Container`를 업데이트하는 방법을 보여 줍니다.

```cs
public class WebApiApplication : System.Web.HttpApplication
{
    protected void Application_Start()
    {
        this.RegisterBotModules();
        GlobalConfiguration.Configure(WebApiConfig.Register);
    }

    private void RegisterBotModules()
    {
        var builder = new ContainerBuilder();
        builder.RegisterModule(new ReflectionSurrogateModule());

        //Register the module within the Conversation container
        builder.RegisterModule<GlobalMessageHandlersBotModule>();

        builder.Update(Conversation.Container);
    }
}
```

## <a name="additional-resources"></a>추가 리소스
* [전역 메시지 처리기 샘플](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers)
* [간단한 점수 지정 가능 봇 샘플](https://github.com/Microsoft/BotFramework-Samples/tree/master/blog-samples/CSharp/ScorableBotSample)
* [대화 상자 개요](bot-builder-dotnet-dialogs.md)
* [AutoFac](https://autofac.org/)
