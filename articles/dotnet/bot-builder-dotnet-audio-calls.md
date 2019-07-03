---
title: Skype를 사용하여 음성 통화 수행 | Microsoft Docs
description: .NET용 Bot Framework SDK를 사용하여 Skype 음성 통화를 수행하는 방법을 살펴봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 621be0d3fe785cfdd51e9bd5c864b9bc4f60d8ad
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464431"
---
# <a name="conduct-audio-calls-with-skype"></a>Skype를 사용하여 음성 통화 수행

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Introduction to conducting audio calls](../includes/snippet-audio-call-intro.md)]

음성 통화를 지원하는 봇의 아키텍처는 일반 봇의 아키텍처와 매우 유사합니다. 다음 코드 샘플에서는 .NET용 Bot Framework SDK를 사용하여 Skype 음성 통화 지원을 사용하도록 설정하는 방법을 보여줍니다. 

## <a name="enable-support-for-audio-calls"></a>음성 통화 지원을 사용하도록 설정

봇을 음성 통화 지원을 사용하도록 설정하려면 `CallingController`를 정의합니다.

```cs
[BotAuthentication]
[RoutePrefix("api/calling")]
public class CallingController : ApiController
{
    public CallingController() : base()
    {
        CallingConversation.RegisterCallingBot(callingBotService => new IVRBot(callingBotService));
    }

    [Route("callback")]
    public async Task<HttpResponseMessage> ProcessCallingEventAsync()
    {
        return await CallingConversation.SendAsync(this.Request, CallRequestType.CallingEvent);
    }

    [Route("call")]
    public async Task<HttpResponseMessage> ProcessIncomingCallAsync()
    {
        return await CallingConversation.SendAsync(this.Request, CallRequestType.IncomingCall);
    }
}
```

> [!NOTE]
> 음성 통화를 지원하는 `CallingController` 외에 봇도 메시지 지원을 위해 `MessagesController`를 포함할 수 있습니다. 두 옵션을 모두 제공하면 사용자가 자신이 원하는 방식으로 봇과 상호 작용할 수 있습니다. <!-- docs on MessagesController are where? -->

## <a name="answer-the-call"></a>통화 받기

`ProcessIncomingCallAsync` 작업은 사용자가 Skype에서 이 봇 호출을 시작할 때마다 실행됩니다.
생성자가 `incomingCallEvent`에 대해 미리 정의된 처리기인 `IVRBot` 클래스를 등록합니다.

워크플로 내 첫 번째 작업은 봇이 걸려오는 통화에 응답할지 또는 거부할지를 결정하는 것이어야 합니다. 이 워크플로는 봇이 걸려오는 통화에 응답한 다음, 환영 메시지를 재생하도록 지시합니다. 

```cs
private Task OnIncomingCallReceived(IncomingCallEvent incomingCallEvent)
{
    this.callStateMap[incomingCallEvent.IncomingCall.Id] = new CallState(incomingCallEvent.IncomingCall.Participants);

    incomingCallEvent.ResultingWorkflow.Actions = new List<ActionBase>
    {
        new Answer { OperationId = Guid.NewGuid().ToString() },
        GetPromptForText(WelcomeMessage)
    };

    return Task.FromResult(true);
}
```

## <a name="after-the-bot-answers"></a>봇 응답 후

봇이 통화에 응답한 후 워크플로에서 지정한 후속 작업은 **Skype Bot Platform for Calling**이 프롬프트를 재생하고, 오디오를 녹화하며, 음성을 인식하거나 숫자 패드에서 숫자를 수집하도록 지시합니다. 워크플로의 최종 작업은 통화 종료가 될 수 있습니다. 

이 코드 샘플에서는 환영 메시지 완료 후 메뉴를 설정하는 처리기를 정의합니다.

```cs
private Task OnPlayPromptCompleted(PlayPromptOutcomeEvent playPromptOutcomeEvent)
{
    var callState = this.callStateMap[playPromptOutcomeEvent.ConversationResult.Id];
    SetupInitialMenu(playPromptOutcomeEvent.ResultingWorkflow);
    return Task.FromResult(true);
}
```

`CreateIvrOptions` 메서드는 사용자에게 표시되는 메뉴를 정의합니다.

```cs
private static Recognize CreateIvrOptions(string textToBeRead, int numberOfOptions, bool includeBack)
{
    if (numberOfOptions > 9)
    {
        throw new Exception("too many options specified");
    }

    var choices = new List<RecognitionOption>();

    for (int i = 1; i <= numberOfOptions; i++)
    {
        choices.Add(new RecognitionOption { Name = Convert.ToString(i), DtmfVariation = (char)('0' + i) });
    }

    if (includeBack)
    {
        choices.Add(new RecognitionOption { Name = "#", DtmfVariation = '#' });
    }

    var recognize = new Recognize
    {
        OperationId = Guid.NewGuid().ToString(),
        PlayPrompt = GetPromptForText(textToBeRead),
        BargeInAllowed = true,
        Choices = choices
    };

    return recognize;
}
```

`RecognitionOption` 클래스는 음성 답변과 해당하는 DTMF(Dual-Tone Multi-Frequency) 변형을 모두 정의합니다. DTMF를 통해 사용자는 말 대신 키패드에 해당 숫자를 입력하여 응답할 수 있습니다.

`OnRecognizeCompleted` 메서드는 사용자 선택을 처리하고 `recognizeOutcomeEvent` 입력 매개 변수에는 사용자 선택 값이 포함됩니다.

```cs
private Task OnRecognizeCompleted(RecognizeOutcomeEvent recognizeOutcomeEvent)
{
    var callState = this.callStateMap[recognizeOutcomeEvent.ConversationResult.Id];
    ProcessMainMenuSelection(recognizeOutcomeEvent, callState);
    return Task.FromResult(true);
}
```

## <a name="support-natural-language"></a>자연어 지원
자연어 응답을 지원하도록 봇을 설계할 수도 있습니다. **Bing Speech API**를 사용하면 봇이 사용자의 음성 답변에서 단어를 인식할 수 있습니다.

```cs
private async Task OnRecordCompleted(RecordOutcomeEvent recordOutcomeEvent)
{
    recordOutcomeEvent.ResultingWorkflow.Actions = new List<ActionBase>
    {
        GetPromptForText(EndingMessage),
        new Hangup { OperationId = Guid.NewGuid().ToString() }
    };

    // Convert the audio to text
    if (recordOutcomeEvent.RecordOutcome.Outcome == Outcome.Success)
    {
        var record = await recordOutcomeEvent.RecordedContent;
        string text = await this.GetTextFromAudioAsync(record);

        var callState = this.callStateMap[recordOutcomeEvent.ConversationResult.Id];

        await this.SendSTTResultToUser("We detected the following audio: " + text, callState.Participants);
    }

    recordOutcomeEvent.ResultingWorkflow.Links = null;
    this.callStateMap.Remove(recordOutcomeEvent.ConversationResult.Id);
}
```

## <a name="sample-code"></a>샘플 코드

.NET용 Bot Framework SDK를 사용하여 Skype 음성 통화를 지원하는 방법을 보여주는 전체 샘플은 GitHub의 <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/skype-CallingBot" target="_blank">Skype Calling Bot 샘플</a>을 참조하세요.

## <a name="additional-resources"></a>추가 리소스

- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">.NET용 Bot Framework SDK 참조</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/skype-CallingBot" target="_blank">Skype Calling Bot 샘플(GitHub)</a>
