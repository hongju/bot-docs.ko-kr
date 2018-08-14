---
title: 글로벌 메시지 처리기 구현 | Microsoft Docs
description: 봇이 .NET용 Bot Builder SDK를 사용하여 특정 키워드를 포함하는 사용자 입력을 수신 대기하고 처리하는 방법에 대해 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 00ffc96672ea34cb2702b3b38fad5b98cf124042
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39302514"
---
# <a name="implement-global-message-handlers"></a>글로벌 메시지 처리기 구현

[!INCLUDE [Introduction to global message handlers](../includes/snippet-global-handlers-intro.md)]

## <a name="listen-for-keywords-in-user-input"></a>사용자 입력에서 키워드 수신 대기

다음 연습에서는 .NET용 Bot Builder SDK를 사용하여 글로벌 메시지 처리기를 구현하는 방법을 보여 줍니다.

먼저 `Global.asax.cs`는 여기에 표시된 것처럼 `GlobalMessageHandlersBotModule`을 등록합니다. 이 예제에서 모듈은 설정 변경 요청 관리용 하나(`SettingsScorable`)와 취소 요청 관리용 하나(`CancelScoreable`), 총 두 개의 scorable을 등록합니다.

```cs
public class GlobalMessageHandlersBotModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        base.Load(builder);

        builder
            .Register(c => new SettingsScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();

        builder
            .Register(c => new CancelScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();
    }
}
```

`CancelScorable`에는 트리거를 정의하는 `PrepareAsync` 메서드가 포함되어 있습니다. 메시지 텍스트가 "취소"인 경우 이 scorable이 트리거됩니다.

```cs
protected override async Task<string> PrepareAsync(IActivity activity, CancellationToken token)
{
    var message = activity as IMessageActivity;
    if (message != null && !string.IsNullOrWhiteSpace(message.Text))
    {
        if (message.Text.Equals("cancel", StringComparison.InvariantCultureIgnoreCase))
        {
            return message.Text;
        }
    }
    return null;
}
```

"취소" 요청이 수신되면 `CancelScoreable` 내의 `PostAsync` 메서드가 대화 상자 스택을 다시 설정합니다. 

```cs
protected override async Task PostAsync(IActivity item, string state, CancellationToken token)
{
    this.task.Reset();
}
```

"설정 변경" 요청이 수신되면 `SettingsScorable` 내의 `PostAsync` 메서드가 `SettingsDialog`를 호출(해당 대화 상자로 요청 전달)하여 대화 상자 스택 맨 위에 `SettingsDialog`를 추가하고 대화를 제어할 수 있습니다.

```cs
protected override async Task PostAsync(IActivity item, string state, CancellationToken token)
{
    var message = item as IMessageActivity;
    if (message != null)
    {
        var settingsDialog = new SettingsDialog();
        var interruption = settingsDialog.Void<object, IMessageActivity>();
        this.task.Call(interruption, null);
        await this.task.PollAsync(token);
    }
}
```

## <a name="sample-code"></a>샘플 코드

.NET용 Bot Builder SDK를 사용하여 글로벌 메시지 처리기를 구현하는 방법을 보여 주는 전체 샘플은 GitHub의 <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers" target="_blank">글로벌 메시지 처리기 샘플</a>을 참조하세요.

## <a name="additional-resources"></a>추가 리소스

- [대화 흐름 설계 및 제어](../bot-service-design-conversation-flow.md)
- <a href="/dotnet/api/?view=botbuilder-3.12.2.4" target="_blank">.NET용 Bot Builder SDK 참조</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers" target="_blank">글로벌 메시지 처리기 샘플(GitHub)</a>
