---
title: 문자 메시지 보내기 및 받기 | Microsoft Docs
description: Bot Framework SDK 안에서 문자 메시지를 보내고 받는 방법을 살펴봅니다.
keywords: 메시지 보내기, 메시지 작업, 단문 메시지, 메시지, 문자 메시지, 메시지 받기
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a72c103204384188d509777639c2c90e63431dd8
ms.sourcegitcommit: 697a577d72aaf91a0834d4b4c2ef5aa11291f28f
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/01/2019
ms.locfileid: "67496688"
---
# <a name="send-and-receive-text-message"></a>문자 메시지 보내기 및 받기

[!INCLUDE[applies-to](../includes/applies-to.md)]

봇이 사용자와 커뮤니케이션하고 마찬가지로 커뮤니케이션을 수신하는 기본 방법은 **메시지** 활동입니다. 일부 메시지는 단순히 일반 텍스트로 구성되지만 다른 메시지는 카드나 첨부 파일 같이 서식이 더 많은 콘텐츠를 포함할 수 있습니다. 봇의 전환 처리기는 사용자로부터 메시지를 수신하고 거기서부터 사용자에게 응답을 보낼 수 있습니다. 전환 컨텍스트 개체는 메시지를 다시 사용자에게 보내기 위한 메서드를 제공합니다. 이 문서에서는 단문 메시지를 보내는 방법을 설명합니다.

Markdown은 대부분의 텍스트 필드에 지원되지만 채널별로 지원이 다를 수 있습니다.

실행 중인 봇 전송 및 수신 메시지의 경우, 목차 맨 위에 있는 빠른 시작을 따르거나 [봇 작동 방식에 대한 문서](bot-builder-basics.md#bot-structure)를 확인합니다. 이 문서에는 사용자가 직접 실행할 수 있는 간단한 샘플의 링크도 있습니다.

## <a name="send-a-text-message"></a>텍스트 메시지 보내기

단문 메시지를 보내려면 보낼 문자열을 활동으로 지정합니다.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

봇의 작업 처리기에서 순서 컨텍스트 개체의 `SendActivityAsync` 메서드를 사용하여 단문 메시지 응답을 보냅니다. 개체의 `SendActivitiesAsync` 메서드를 사용하여 한 번에 여러 응답을 보낼 수도 있습니다.

```cs
await turnContext.SendActivityAsync($"Welcome!");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

봇의 작업 처리기에서 순서 컨텍스트 개체의 `sendActivity` 메서드를 사용하여 단문 메시지 응답을 보냅니다. 개체의 `sendActivities` 메서드를 사용하여 한 번에 여러 응답을 보낼 수도 있습니다.

```javascript
await context.sendActivity("Welcome!");
```
---
## <a name="receive-a-text-message"></a>텍스트 메시지 수신

단문 메시지를 수신하려면 *작업* 개체의 *텍스트* 속성을 사용합니다. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

봇의 작업 처리기에서 다음 코드를 사용하여 메시지를 수신합니다. 

```cs
var responseMessage = turnContext.Activity.Text;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

봇의 작업 처리기에서 다음 코드를 사용하여 메시지를 수신합니다.

```javascript
let text = turnContext.activity.text;
```

---

## <a name="additional-resources"></a>추가 리소스

- 일반적인 작업 처리에 대한 자세한 내용은 [작업 처리](~/v4sdk/bot-builder-basics.md#the-activity-processing-stack)를 참조하세요.
- 서식 지정에 대한 자세한 내용은 Bot Framework 작업 스키마의 [메시지 작업 섹션](https://aka.ms/botSpecs-activitySchema#message-activity)을 참조하세요.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [메시지에 미디어 추가](./bot-builder-howto-add-media-attachments.md)
