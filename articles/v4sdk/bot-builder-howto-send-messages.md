---
title: 메시지 보내기 | Microsoft Docs
description: Bot Builder SDK 안에서 메시지를 보내는 방법을 살펴봅니다.
keywords: 메시지 보내기, 메시지 활동, 단문 메시지, 음성, 음성 메시지
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 08/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 075e9b3a41462bfbb1398bd72840cc7e50d3cfca
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707519"
---
# <a name="send-text-and-spoken-messages"></a>텍스트 및 음성 메시지 보내기

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

봇이 사용자와 커뮤니케이션하고 마찬가지로 커뮤니케이션을 수신하는 기본 방법은 **메시지** 활동입니다. 일부 메시지는 단순히 일반 텍스트로 구성되지만 다른 메시지는 카드나 첨부 파일 같이 서식이 더 많은 콘텐츠를 포함할 수 있습니다. 봇의 전환 처리기는 사용자로부터 메시지를 수신하고 거기서부터 사용자에게 응답을 보낼 수 있습니다. [턴 컨텍스트](bot-builder-concept-activity-processing.md#turn-context) 개체는 메시지를 다시 사용자에게 보내기 위한 메서드를 제공합니다. 일반적인 활동 처리에 대한 자세한 내용은 [활동 처리](bot-builder-concept-activity-processing.md)를 참조하세요.

이 문서에서는 단문 및 음성 메시지를 보내는 방법을 설명합니다. 서식이 더 많은 콘텐츠를 보내려면 [서식 있는 미디어 첨부 파일 추가](bot-builder-howto-add-media-attachments.md) 방법을 참조하세요. 프롬프트 개체 사용 방법에 대한 자세한 내용은 [사용자에게 입력할 프롬프트 창 표시](bot-builder-prompts.md)를 참조하세요.

## <a name="send-a-simple-text-message"></a>단문 메시지 보내기

단문 메시지를 보내려면 보낼 문자열을 활동으로 지정합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

봇의 **OnTurn** 메서드에서 전환 컨텍스트 개체의 **SendActivity** 메서드를 사용하여 단문 응답을 보냅니다. 개체의 **SendActivities** 메서드를 사용하여 한 번에 여러 응답을 보낼 수도 있습니다.

```cs
await context.SendActivity("Greetings from sample message.");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

봇의 전환 처리기에서 컨텍스트 개체의 **sendActivity** 메서드를 사용하여 단문 응답을 보냅니다. 개체의 **SendActivities** 메서드를 사용하여 한 번에 여러 응답을 보낼 수도 있습니다.

```javascript
await context.sendActivity("Greetings from sample message.");
```

---

## <a name="send-a-spoken-message"></a>음성 메시지 보내기

특정 채널에서는 음성 가능 봇을 지원하므로 사용자에게 말로 이야기할 수 있습니다. 메시지는 문자 및 음성 콘텐츠를 모두 가질 수 있습니다.

> [!NOTE]
> 음성을 지원하지 않는 채널에서는 음성 콘텐츠가 무시됩니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

선택적 **speak** 매개 변수를 사용하여 응답의 일부로 이야기할 텍스트를 제공합니다.

```cs
await context.SendActivity(
    "This is the text to be displayed.",
    "This is the text to be spoken.");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

음성을 추가하려면 메시지 빌드를 위해 `Microsoft.Bot.Builder.MessageFactory`가 필요합니다. 설명이 더 많은 [서식 있는 미디어](bot-builder-howto-add-media-attachments.md)에는 `MessageFactory`가 더 많이 사용되나 지금 여기서 필요한 만큼만 사용해 보겠습니다.

```javascript
// Require MessageFactory from botbuilder
const {MessageFactory} = require('botbuilder');

const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.');
await context.sendActivity(basicMessage);
```

---
