---
title: 메시지에 음성 추가 | Microsoft Docs
description: .NET용 Bot Builder SDK를 사용하여 메시지에 음성을 추가하는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: dc542c7e85b3a79e1071edebea65d93c99742beb
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000362"
---
# <a name="add-speech-to-messages"></a>메시지에 음성 추가

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST (영문)](../rest-api/bot-framework-rest-connector-text-to-speech.md)

Cortana와 같은 음성 지원 채널을 위한 봇을 빌드하는 경우 봇의 음성 텍스트를 지정하는 메시지를 구성할 수 있습니다. 봇이 사용자 입력을 수락, 대기 또는 무시할지 여부를 나타내는 [입력 힌트](bot-builder-dotnet-add-input-hints.md)를 지정하여 클라이언트의 마이크 상태에 영향을 미칠 수 있습니다.

## <a name="specify-text-to-be-spoken-by-your-bot"></a>봇의 음성 텍스트 지정

.NET용 Bot Builder SDK를 사용하면 음성 지원 채널에서 봇의 음성 텍스트를 지정하는 여러 가지 방법이 있습니다. [메시지][IMessageActivity]의 `Speak` 속성을 설정하거나 `IDialogContext.SayAsync()` 메서드를 호출하거나 내장 프롬프트를 사용하여 메시지를 보낼 때 프롬프트 옵션 `speak` 및 `retrySpeak`을 지정할 수 있습니다.

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

봇의 음성 텍스트를 지정하기 위해 일반 텍스트 문자열 또는 SSML(Speech Synthesis Markup Language) 형식의 문자열을 사용할 수 있습니다. SSML은 목소리, 속도, 음량, 발음, 음색 등과 같은 봇의 음성에 대한 다양한 특징을 제어할 수 있는 XML 기반 마크업 언어입니다. SSML에 대한 자세한 내용은 <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">Speech Synthesis Markup Language 참조</a>를 참조하세요.

## <a name="input-hints"></a>입력 힌트

음성 지원 채널에서 메시지를 전송하는 경우 봇이 사용자 입력을 수락, 대기 또는 무시할지 여부를 나타내는 입력 힌트를 포함시켜서 클라이언트의 마이크의 상태에 영향을 미칠 수 있습니다. 자세한 내용은 [메시지에 입력 힌트 추가](bot-builder-dotnet-add-input-hints.md)를 참조하세요.

## <a name="sample-code"></a>샘플 코드 

.NET용 Bot Builder SDK를 사용하여 음성 지원 봇을 만드는 방법을 보여주는 전체 샘플은 GitHub의 <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp" target="_blank">롤러 스킬 샘플</a>을 참조하세요.

## <a name="additional-resources"></a>추가 리소스

- [메시지 만들기](bot-builder-dotnet-create-messages.md)
- [메시지에 입력 힌트 추가](bot-builder-dotnet-add-input-hints.md)
- <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">SSML(Speech Synthesis Markup Language)</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-RollerSkill" target="_blank">롤러 스킬 샘플(GitHub)</a>
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">활동 클래스</a>
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">IMessageActivity 인터페이스</a>
- <a href="/dotnet/api/microsoft.bot.builder.dialogs.internals.dialogcontext" target="_blank">DialogContext 클래스</a>
- <a href="/dotnet/api/microsoft.bot.builder.dialogs.internals.prompt-2" target="_blank">Prompt 클래스</a>

[IMessageActivity]: /dotnet/api/microsoft.bot.connector.imessageactivity

