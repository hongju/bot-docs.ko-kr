---
title: 사전 대응 메시지 보내기 | Microsoft Docs
description: .NET용 Bot Framework SDK를 사용하여 사전 대응 메시지를 보내는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: d86ee290ebf33dbfd13017c3fe882ecfdd9102e4
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225728"
---
# <a name="send-proactive-messages"></a>사전 대응 메시지 보내기

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-proactive-messages.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-proactive-messages.md)

[!INCLUDE [Introduction to proactive messages - part 1](../includes/snippet-proactive-messages-intro-1.md)]

## <a name="types-of-proactive-messages"></a>자동 관리 메시지 형식 

[!INCLUDE [Introduction to proactive messages - part 2](../includes/snippet-proactive-messages-intro-2.md)]

## <a name="send-an-ad-hoc-proactive-message"></a>임시 사전 대응 메시지 보내기

다음 코드 샘플은 .NET용 Bot Framework SDK를 사용하여 임시 사전 대응 메시지를 보내는 방법을 보여줍니다.

사용자에게 임시 메시지를 보낼 수 있으려면, 그 전에 봇이 현재 대화에서 사용자에 대한 정보를 수집하고 저장해야 합니다. 

```cs
public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
{
    var message = await result;
    
    // Extract data from the user's message that the bot will need later to send an ad hoc message to the user. 
    // Store the extracted data in a custom class "ConversationStarter" (not shown here).

    ConversationStarter.toId = message.From.Id;
    ConversationStarter.toName = message.From.Name;
    ConversationStarter.fromId = message.Recipient.Id;
    ConversationStarter.fromName = message.Recipient.Name;
    ConversationStarter.serviceUrl = message.ServiceUrl;
    ConversationStarter.channelId = message.ChannelId;
    ConversationStarter.conversationId = message.Conversation.Id;

    // (Save this information somewhere that it can be accessed later, such as in a database.)

    await context.PostAsync("Hello user, good to meet you! I now know your address and can send you notifications in the future.");
    context.Wait(MessageReceivedAsync);
}
```
> [!NOTE]
> 편의상 이 예에서는 사용자 데이터를 저장하는 방법을 지정하지 않습니다. 나중에 봇이 데이터를 검색할 수만 있으면, 데이터를 저장하는 방식은 중요하지 않습니다.

데이터가 저장되기만 하면 봇이 데이터를 간단히 검색하고, 임시 사전 대응 메시지를 생성하여 보낼 수 있습니다. 

```cs
// Use the data stored previously to create the required objects.
var userAccount = new ChannelAccount(toId,toName);
var botAccount = new ChannelAccount(fromId, fromName);
var connector = new ConnectorClient(new Uri(serviceUrl));

// Create a new message.
IMessageActivity message = Activity.CreateMessageActivity();
if (!string.IsNullOrEmpty(conversationId) && !string.IsNullOrEmpty(channelId))  
{
    // If conversation ID and channel ID was stored previously, use it.
    message.ChannelId = channelId;
}
else
{
    // Conversation ID was not stored previously, so create a conversation. 
    // Note: If the user has an existing conversation in a channel, this will likely create a new conversation window.
    conversationId = (await connector.Conversations.CreateDirectConversationAsync( botAccount, userAccount)).Id;
}

// Set the address-related properties in the message and send the message.
message.From = botAccount;
message.Recipient = userAccount;
message.Conversation = new ConversationAccount(id: conversationId);
message.Text = "Hello, this is a notification";
message.Locale = "en-us";
await connector.Conversations.SendToConversationAsync((Activity)message);
```

> [!NOTE]
> 봇이 이전에 저장된 대화 ID를 지정하면, 클라이언트의 기존 대화 창에서 사용자에게 메시지가 전송될 가능성이 있습니다. 봇이 새 대화 ID를 생성하면 클라이언트가 대화 창을 여러 개 지원하는 경우에 한하여 클라이언트의 새 대화 창에서 사용자에게 메시지가 전달됩니다. 

## <a name="send-a-dialog-based-proactive-message"></a>대화 상자 기반 사전 대응 메시지 보내기

다음 코드 샘플은 .NET용 Bot Framework SDK를 사용하여 다이얼로그 기반 사전 대응 메시지를 보내는 방법을 보여줍니다.

사용자에게 대화 상자 기반 사전 대응 메시지를 보낼 수 있으려면 그 전에 봇이 현재 대화에서 정보를 수집하고 저장해야 합니다. 

```cs
public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
{
    var message = await result;
    
    // Store information about this specific point the conversation, so that the bot can resume this conversation later.
    var conversationReference = message.ToConversationReference();
    ConversationStarter.conversationReference = JsonConvert.SerializeObject(conversationReference);

    await context.PostAsync("Greetings, user! I now know how to start a proactive message to you."); 
    context.Wait(MessageReceivedAsync);
}
```

이제 메시지를 보낼 때 봇은 새 대화 상자를 만들고 대화 상자 스택의 맨 위에 추가합니다. 새 대화 상자가 대화를 제어하고, 사전 대응 메시지를 전달하고, 닫힌 다음, 스택의 이전 대화 상자에 제어 권한을 반환합니다. 

```cs
// This will interrupt the conversation and send the user to SurveyDialog, then wait until that's done 
public static async Task Resume() 
{
    // Recreate the message from the conversation reference that was saved previously.
    var message = JsonConvert.DeserializeObject<ConversationReference>(conversationReference).GetPostToBotMessage(); 
    var client = new ConnectorClient(new Uri(message.ServiceUrl));

    // Create a scope that can be used to work with state from bot framework.
    using (var scope = DialogModule.BeginLifetimeScope(Conversation.Container, message))
    {
        var botData = scope.Resolve<IBotData>();
        await botData.LoadAsync(CancellationToken.None);

        // This is our dialog stack.
        var task = scope.Resolve<IDialogTask>();

        // Create the new dialog and add it to the stack.
        var dialog = new SurveyDialog();
        // interrupt the stack. This means that we're stopping whatever conversation that is currently happening with the user
        // Then adding this stack to run and once it's finished, we will be back to the original conversation
        task.Call(dialog.Void<object, IMessageActivity>(), null);
        
        await task.PollAsync(CancellationToken.None);

        // Flush the dialog stack back to its state store.
        await botData.FlushAsync(CancellationToken.None);        
    }
}
```
`SurveyDialog`는 완료될 때까지 대화를 제어합니다. 작업이 끝나면 `context.Done` 호출하여 마무리하고 이전 대화 상자에 컨트롤을 반환합니다. 

```cs
[Serializable]
public class SurveyDialog : IDialog<object>
{
    public async Task StartAsync(IDialogContext context)
    {
        await context.PostAsync("Hello, I'm the survey dialog. I'm interrupting your conversation to ask you a question. Type \"done\" to resume");

        context.Wait(this.MessageReceivedAsync);
    }
    public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
    {
        if ((await result).Text == "done")
        {
            await context.PostAsync("Great, back to the original conversation!");
            context.Done(String.Empty); // Finish this dialog.
        }
        else
        {
            await context.PostAsync("I'm still on the survey until you type \"done\"");
            context.Wait(MessageReceivedAsync); // Not done yet.
        }
    }
}
```

## <a name="sample-code"></a>샘플 코드

.NET용 Bot Framework SDK를 사용하여 사전 대응 메시지를 보내는 방법을 보여주는 전체 샘플은 GitHub의 <a href="https://aka.ms/proactive-messaging-cs-v3 " target="_blank">Proactive Messages sample</a>(사전 대응 메시지 샘플)을 참조하세요. 사전 대응 메시지 샘플 내에서 <a href="https://aka.ms/proactive-sendmessage-cs-v3 " target="_blank">simpleSendMessage</a>는 임시 사전 대응 메시지를 보내는 방법을 보여주고 <a href="https://aka.ms/proactive-newdialog-cs-v3 " target="_blank">startNewDialog</a>는 대화 상자 기반 사전 대응 메시지를 보내는 방법을 보여줍니다. 

## <a name="additional-resources"></a>추가 리소스

- [대화 흐름 설계 및 제어](../bot-service-design-conversation-flow.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">.NET용 Bot Framework SDK 참조</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-proactiveMessages" target="_blank">사전 대응 메시지 샘플(GitHub)</a>

