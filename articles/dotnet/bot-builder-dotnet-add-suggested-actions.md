---
title: 메시지에 제안된 동작 추가 | Microsoft Docs
description: .NET용 Bot Framework SDK를 사용하여 메시지에 제안된 작업을 추가하는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/13/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 3898c89371403cf785e5e356ac46a64bd59ffaf9
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405700"
---
# <a name="add-suggested-actions-to-messages"></a>메시지에 제안된 동작 추가

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-suggested-actions.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-send-suggested-actions.md)
> - [REST (영문)](../rest-api/bot-framework-rest-connector-add-suggested-actions.md)

[!INCLUDE [Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)]

> [!TIP]
> [Channel Inspector][channelInspector]를 사용하여 제안된 작업이 다양한 채널에서 어떻게 보이고 작동하는지를 확인할 수 있습니다.

## <a name="send-suggested-actions"></a>제안된 동작 보내기

제안된 작업을 메시지에 추가하려면 활동의 `SuggestedActions` 속성을 사용자에게 제공될 단추를 나타내는 [CardAction][cardAction] 개체 목록으로 설정합니다. 

이 코드 예제에서는 세 가지 제안된 작업을 사용자에게 제공하는 메시지 생성 방법을 보여 줍니다.

[!code-csharp[Add suggested actions](../includes/code/dotnet-add-suggested-actions.cs#addSuggestedActions)]

사용자가 제안된 작업 중 하나를 탭하면 봇은 해당 동작의 `Value`를 포함하는 메시지를 사용자로부터 수신합니다.

## <a name="additional-resources"></a>추가 리소스

- [채널 검사기를 사용하여 기능 미리 보기][inspector]
- [작업 개요](bot-builder-dotnet-activities.md)
- [메시지 만들기](bot-builder-dotnet-create-messages.md)
- <a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">활동 클래스</a>
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">IMessageActivity 인터페이스</a>
- <a href="/dotnet/api/microsoft.bot.connector.cardaction" target="_blank">CardAction 클래스</a>
- <a href="/dotnet/api/microsoft.bot.connector.suggestedactions" target="_blank">SuggestedActions 클래스</a>

[cardAction]: /dotnet/api/microsoft.bot.connector.cardaction

[inspector]: ../bot-service-channel-inspector.md

[channelInspector]: ../bot-service-channel-inspector.md


