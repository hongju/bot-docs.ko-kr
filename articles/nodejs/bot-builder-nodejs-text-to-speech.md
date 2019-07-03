---
title: 메시지에 음성 추가 | Microsoft Docs
description: Node.js용 Bot Framework SDK를 사용하여 메시지에 음성을 추가하는 방법을 알아봅니다.
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 14b8bf7aa4e99e3ca97442c2ba57dc8c57138d99
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404691"
---
# <a name="add-speech-to-messages"></a>메시지에 음성 추가

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST (영문)](../rest-api/bot-framework-rest-connector-text-to-speech.md)

Cortana와 같은 음성 지원 채널을 위한 봇을 빌드하는 경우 봇의 음성 텍스트를 지정하는 메시지를 구성할 수 있습니다. 봇이 사용자 입력을 수락, 대기 또는 무시할지 여부를 나타내는 [입력 힌트](bot-builder-nodejs-send-input-hints.md)를 지정하여 클라이언트의 마이크 상태에 영향을 미칠 수 있습니다.

## <a name="specify-text-to-be-spoken-by-your-bot"></a>봇의 음성 텍스트 지정

Node.js용 Bot Framework SDK를 사용하면 음성 지원 채널에서 봇의 음성 텍스트를 지정하는 여러 가지 방법이 있습니다. `IMessage.speak` 속성을 설정하여 `session.send()` 메서드를 사용하여 메시지를 보내고, `session.say()` 메서드를 사용하여 메시지를 보내거나(표시 텍스트, 음성 텍스트, 옵션을 지정하는 매개 변수 전달), 기본 제공 프롬프트(`speak` 및 `retrySpeak` 옵션 지정)를 사용하여 메시지를 보낼 수 있습니다.

### <a id="message-speak"></a> IMessage.speak

`session.send()` 메서드를 사용하여 보낼 메시지를 만들 경우 `speak` 속성을 설정하여 봇이 이야기할 텍스트를 지정합니다. 다음 코드 예제는 이야기할 텍스트를 지정하고 봇이 [사용자 입력을 수락](bot-builder-nodejs-send-input-hints.md)하는 것을 나타내는 메시지를 만듭니다.

[!code-javascript[IMessage.speak](../includes/code/node-text-to-speech.js#IMessageSpeak)]

### <a id="session-say"></a> session.say()

`session.send()`를 사용하는 대신 `session.say()` 메서드를 호출하여 표시할 텍스트 및 기타 옵션 외에 음성 텍스트를 만들어서 보낼 수 있습니다. 메서드는 다음과 같이 정의됩니다.

`session.say(displayText: string, speechText: string, options?: object)`

| 매개 변수 | 설명 |
|----|----|
| `displayText` | 표시될 텍스트입니다. |
| `speechText` | 이야기할 텍스트(일반 텍스트 또는 <a href="https://msdn.microsoft.com/library/hh378377(v=office.14).aspx" target="_blank">SSML</a> 형식)입니다. |
| `options` | 첨부 파일이나 [입력 힌트](bot-builder-nodejs-send-input-hints.md)를 포함할 수 있는 `IMessage` 개체입니다. |

다음 코드 예제는 표시할 텍스트와 이야기할 텍스트를 지정하고 봇이 [사용자 입력을 무시](bot-builder-nodejs-send-input-hints.md)하는 것을 나타내는 메시지를 만듭니다.

[!code-javascript[Session.say()](../includes/code/node-text-to-speech.js#SessionSay)]

### <a id="prompt-options"></a> 프롬프트 옵션

기본 제공 프롬프트를 사용하면 `speak` 및 `retrySpeak` 옵션을 설정하여 봇의 음성 텍스트를 지정할 수 있습니다. 다음 코드 예제는 표시할 텍스트, 처음에 사용할 음성 텍스트 및 사용자 입력을 잠시 기다린 후 사용할 음성 텍스트를 지정하는 프롬프트를 만듭니다. 봇이 [사용자 입력을 대기 중이고](bot-builder-nodejs-send-input-hints.md) [SSML](#ssml) 형식을 사용하여 "sure"라는 단어를 중간 수준의 어조로 이야기하도록 지정합니다.

[!code-javascript[Prompt](../includes/code/node-text-to-speech.js#Prompt)]

## <a id="ssml"></a>SSML(Speech Synthesis Markup Language)

봇의 음성 텍스트를 지정하기 위해 일반 텍스트 문자열 또는 SSML(Speech Synthesis Markup Language) 형식의 문자열을 사용할 수 있습니다. SSML은 목소리, 속도, 음량, 발음, 음색 등과 같은 봇의 음성에 대한 다양한 특징을 제어할 수 있는 XML 기반 마크업 언어입니다. SSML에 대한 자세한 내용은 <a href="https://msdn.microsoft.com/library/hh378377(v=office.14).aspx" target="_blank">Speech Synthesis Markup Language 참조</a>를 참조하세요.

> [!TIP]
> <a href="https://www.npmjs.com/search?q=ssml" target="_blank">SSML 라이브러리</a>를 사용하여 형식이 올바르게 지정된 SSML을 만듭니다.

## <a name="input-hints"></a>입력 힌트

음성 지원 채널에서 메시지를 전송하는 경우 봇이 사용자 입력을 수락, 대기 또는 무시할지 여부를 나타내는 입력 힌트를 포함시켜서 클라이언트의 마이크의 상태에 영향을 미칠 수 있습니다. 자세한 내용은 [메시지에 입력 힌트 추가](bot-builder-nodejs-send-input-hints.md)를 참조하세요.

## <a name="sample-code"></a>샘플 코드 

.NET용 Bot Framework SDK를 사용하여 음성 지원 봇을 만드는 방법을 보여주는 전체 샘플은 GitHub의 <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill" target="_blank">롤러 샘플</a>을 참조하세요.

## <a name="additional-resources"></a>추가 리소스

- <a href="https://msdn.microsoft.com/library/hh378377(v=office.14).aspx" target="_blank">SSML(Speech Synthesis Markup Language)</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill" target="_blank">롤러 샘플(GitHub)</a>
