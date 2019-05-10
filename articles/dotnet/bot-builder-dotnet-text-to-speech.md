---
title: 메시지에 음성 추가 | Microsoft Docs
description: .NET용 Bot Framework SDK를 사용하여 메시지에 음성을 추가하는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f93ab91281cf0f19be10898436dc41a6a1583c9a
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032981"
---
# <a name="add-speech-to-messages"></a>메시지에 음성 추가

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST (영문)](../rest-api/bot-framework-rest-connector-text-to-speech.md)

Cortana와 같은 음성 지원 채널을 위한 봇을 빌드하는 경우 봇의 음성 텍스트를 지정하는 메시지를 구성할 수 있습니다. 봇이 사용자 입력을 수락, 대기 또는 무시할지 여부를 나타내는 [입력 힌트](bot-builder-dotnet-add-input-hints.md)를 지정하여 클라이언트의 마이크 상태에 영향을 미칠 수 있습니다.

## <a name="specify-text-to-be-spoken-by-your-bot"></a>봇의 음성 텍스트 지정

.NET용 Bot Framework SDK를 사용하면 음성 지원 채널에서 봇의 음성 텍스트를 지정하는 여러 가지 방법이 있습니다. [메시지][IMessageActivity]의 `Speak` 속성을 설정하거나 `IDialogContext.SayAsync()` 메서드를 호출하거나 내장 프롬프트를 사용하여 메시지를 보낼 때 프롬프트 옵션 `speak` 및 `retrySpeak`을 지정할 수 있습니다.

### <a id="message-speak"></a> IMessageActivity.Speak

[메시지][IMessageActivity]를 만들어서 개별 속성을 설정하는 경우 메시지의 `Speak` 속성을 설정하여 봇의 음성 텍스트를 지정할 수 있습니다. 다음 코드 예제는 표시할 텍스트와 음성 텍스트를 지정하고 봇이 [사용자 입력을 수락](bot-builder-dotnet-add-input-hints.md)하는 것을 나타내는 메시지를 만듭니다.

[!code-csharp[Set speak property](../includes/code/dotnet-text-to-speech.cs#Speak1)]

### <a id="say-async"></a> IDialogContext.SayAsync()

[대화](bot-builder-dotnet-dialogs.md)를 사용하는 경우 `SayAsync()` 메서드를 호출하여 표시할 텍스트 및 기타 옵션 외에 음성 텍스트를 만들어서 보낼 수 있습니다. 다음 코드 예제는 표시할 텍스트와 음성 텍스트를 만듭니다.

[!code-csharp[Call SayAsync()](../includes/code/dotnet-text-to-speech.cs#Speak2)]

### <a id="prompt-options"></a> 프롬프트 옵션

기본 제공 프롬프트를 사용하면 `speak` 및 `retrySpeak` 옵션을 설정하여 봇의 음성 텍스트를 지정할 수 있습니다. 다음 코드 예제는 표시할 텍스트, 처음에 사용할 음성 텍스트 및 사용자 입력을 잠시 기다린 후 사용할 음성 텍스트를 지정하는 프롬프트를 만듭니다. [SSML](#ssml) 양식을 사용하여 "sure"이라는 단어를 중간 정도의 강조로 말해야 함을 나타냅니다.

[!code-csharp[Set Prompt options](../includes/code/dotnet-text-to-speech.cs#Speak3)]

## <a id="ssml"></a>SSML(Speech Synthesis Markup Language)

봇에서 사용할 텍스트를 지정하려면 SSML(Speech Synthesis Markup Language)로 서식이 지정된 문자열을 지정하면 됩니다. SSML은 음성, 속도, 볼륨, 발음, 피치 등과 같은 봇 음성의 다양한 특성을 제어할 수 있는 XML 기반 태그 언어입니다(따라서 올바른 XML이어야 함). SSML에 대한 자세한 내용은 <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">Speech Synthesis Markup Language 참조</a>를 참조하세요.

SSML 서식 지정 문자열을 제공할 때 외부 SSML 래퍼 요소를 생략할 수 있습니다.

## <a name="input-hints"></a>입력 힌트

음성 지원 채널에서 메시지를 전송하는 경우 봇이 사용자 입력을 수락, 대기 또는 무시할지 여부를 나타내는 입력 힌트를 포함시켜서 클라이언트의 마이크의 상태에 영향을 미칠 수 있습니다. 자세한 내용은 [메시지에 입력 힌트 추가](bot-builder-dotnet-add-input-hints.md)를 참조하세요.

## <a name="sample-code"></a>샘플 코드 

.NET용 Bot Framework SDK를 사용하여 음성 지원 봇을 만드는 방법을 보여주는 전체 샘플은 GitHub의 <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp" target="_blank">롤러 기술 샘플</a>을 참조하세요.

## <a name="additional-resources"></a>추가 리소스

- [메시지 만들기](bot-builder-dotnet-create-messages.md)
- [메시지에 입력 힌트 추가](bot-builder-dotnet-add-input-hints.md)
- <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">SSML(Speech Synthesis Markup Language)</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/demo-RollerSkill" target="_blank">롤러 스킬 샘플(GitHub)</a>
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">활동 클래스</a>
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">IMessageActivity 인터페이스</a>
- <a href="/dotnet/api/microsoft.bot.builder.dialogs.internals.dialogcontext" target="_blank">DialogContext 클래스</a>
- <a href="/dotnet/api/microsoft.bot.builder.dialogs.internals.prompt-2" target="_blank">Prompt 클래스</a>

[IMessageActivity]: /dotnet/api/microsoft.bot.connector.imessageactivity

