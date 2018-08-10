---
title: 대화 상자를 사용하여 대화 흐름 관리 | Microsoft Docs
description: 대화 상자 및 .NET용 Bot Builder SDK를 사용하여 대화를 모델링하고 대화 흐름을 관리하는 방법에 대해 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 091da9c1db310491c7e2263d8e7eab51cfcc3787
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39303570"
---
# <a name="manage-conversation-flow-with-dialogs"></a>대화 상자를 사용하여 대화 흐름 관리
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-manage-conversation-flow.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-manage-conversation-flow.md)

[!INCLUDE [Dialog flow example](../includes/snippet-dotnet-manage-conversation-flow-intro.md)]

이 문서에서는 [대화 상자](bot-builder-dotnet-dialogs.md) 및 .NET용 Bot Builder SDK를 사용하여 이 대화 흐름을 모델링하는 방법을 설명합니다. 

## <a name="invoke-the-root-dialog"></a>루트 대화 상자 호출

먼저 봇 컨트롤러가 “루트 대화 상자”를 호출합니다. 다음 예제에서는 기본 HTTP POST 호출을 컨트롤러에 연결하고 나서 루트 대화 상자를 호출하는 방법을 보여 줍니다. 

```cs
public class MessagesController : ApiController
{
    public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
    {
            // Redirect to the root dialog.
            await Conversation.SendAsync(activity, () => new RootDialog()); 
            ...
    }
}
```

## <a name="invoke-the-new-order-dialog"></a>‘새 주문’ 대화 상자 호출

그런 다음, 루트 대화 상자가 ‘새 주문’ 대화 상자를 호출합니다. 

```cs
[Serializable]
public class RootDialog : IDialog<object>
{
    public async Task StartAsync(IDialogContext context)
    {
        // Root dialog initiates and waits for the next message from the user. 
        // When a message arrives, call MessageReceivedAsync.
        context.Wait(this.MessageReceivedAsync); 
    }

    public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
    {
        var message = await result; // We've got a message!
        if (message.Text.ToLower().Contains("order"))
        {
            // User said 'order', so invoke the New Order Dialog and wait for it to finish.
            // Then, call ResumeAfterNewOrderDialog.
            await context.Forward(new NewOrderDialog(), this.ResumeAfterNewOrderDialog, message, CancellationToken.None);
        }
        // User typed something else; for simplicity, ignore this input and wait for the next message.
        context.Wait(this.MessageReceivedAsync);
    }

    private async Task ResumeAfterNewOrderDialog(IDialogContext context, IAwaitable<string> result)
    {
        // Store the value that NewOrderDialog returned. 
        // (At this point, new order dialog has finished and returned some value to use within the root dialog.)
        var resultFromNewOrder = await result;

        await context.PostAsync($"New order dialog just told me this: {resultFromNewOrder}");

        // Again, wait for the next message from the user.
        context.Wait(this.MessageReceivedAsync);
    }
}
```

## <a id="dialog-lifecycle"></a> 대화 상자 수명 주기

대화 상자가 호출되면 이 대화 상자가 대화 흐름을 제어합니다. 모든 새 메시지는 닫히거나 다른 대화 상자로 리디렉션될 때까지 해당 대화 상자에서 처리됩니다. 

C#에서는 `context.Wait()`를 사용하여 다음에 사용자가 메시지를 보낼 때 호출할 콜백을 지정할 수 있습니다. 대화 상자를 닫고 스택에서 제거하려면(이렇게 하면 사용자가 다시 스택의 이전 대화 상자로 돌아감) `context.Done()`를 사용합니다. 모든 대화 상자 메서드는 `context.Wait()`, `context.Fail()`, `context.Done()` 또는 일부 리디렉션 지시문(예: `context.Forward()` 또는 `context.Call()`)을 사용하여 를 종료해야 합니다. 이러한 항목 중 하나를 사용하여 대화 상자 메서드를 종료하지 않으면 오류가 발생합니다(프레임워크가 다음에 사용자가 메시지를 보낼 때 수행할 작업을 알 수 없기 때문).

## <a name="passing-state-between-dialogs"></a>대화 상자 간에 상태 전달

봇 상태에 상태를 저장할 수 있지만 대화 상자 클래스 생성자를 오버로드하여 여러 대화 상자 간에 데이터를 전달할 수도 있습니다.

```cs
[Serializable]
public class AgeDialog : IDialog<int>
{
    private string name;

    public AgeDialog(string name)
    {
        this.name = name;
    }
}
 ```

대화 상자 코드 호출, 사용자의 이름 값 전달.

```cs
private async Task NameDialogResumeAfter(IDialogContext context, IAwaitable<string> result)
{
    try
    {
        this.name = await result;
        context.Call(new AgeDialog(this.name), this.AgeDialogResumeAfter);
    }
    catch (TooManyAttemptsException)
    {
        await context.PostAsync("I'm sorry, I'm having issues understanding you. Let's try again.");
        await this.SendWelcomeMessageAsync(context);
    }
}
```

## <a name="sample-code"></a>샘플 코드 

.NET용 Bot Builder SDK의 대화 상자를 사용하여 대화를 관리하는 방법을 보여 주는 전체 샘플은 GitHub에서 <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-BasicMultiDialog" target="_blank">Basic Multi-Dialog sample</a>(기본 다중 대화 상자 샘플)을 참조하세요.

## <a name="additional-resources"></a>추가 리소스

- [대화 상자](bot-builder-dotnet-dialogs.md)
- [대화 흐름 디자인 및 제어](../bot-service-design-conversation-flow.md)
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-BasicMultiDialog" target="_blank">Basic Multi-Dialog sample (GitHub)</a>(기본 다중 대화 샘플(GitHub))
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">.NET용 Bot Builder SDK 참조</a>
