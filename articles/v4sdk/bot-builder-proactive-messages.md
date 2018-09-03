---
title: 사전 대응 메시지 | Microsoft Docs
description: 메시지를 자동으로 관리하는 방법을 이해합니다.
keywords: 사용자 환경, 대화 시작
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: f311f70e9bfb72db780546b5e289f09d803589dc
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756712"
---
# <a name="proactive-messages"></a>사전 대응 메시지
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]
<!--
When you think about the exchange of messages between your bot and the user, you're probably thinking about the scenario where the user sends a message to your bot and your bot then replies to the user with a message of its own. We call this _reactive messaging_ and it's by far the most common flow that you should optimize your bot's code for.

It is possible, however, for your bot to initiate a conversation with the user by sending them a message first. We call this _proactive messaging_ and while the code you'll write to send a proactive message is very similar to what you'd write in the reactive case, there are a few differences that are worth exploring.

The first thing to note is that before you can send a proactive message to a user, the user will have to send at least one reactive style message to your bot. There are two reasons for this.

1. You need to get the user's `ConversationReference` and save it somewhere for future use. You can think of the conversation reference as the user's address, as it contains information about the channel they came in on, their user ID, the conversation ID, and even the server that should receive any future messages. This object is simple JSON and should be saved whole without tampering.
2. Most channels by policy won't let a bot initiate conversations with users they've never spoken to before. Depending on the channel the user might need to explicitly add the bot to a conversation or at a minimum send an initial message to the bot.

> ![NOTE]
> This bot currently runs properly only when deployed to Azure. However, you can test the bot without publishing it.

A common case of proactive messaging comes when our bot is performing a time-consuming task. In this case, we send a **typing** activity indicates to the user that the bot is in a *processing* mode, and then follow it up with a proactive message once our processing has completed.
-->

[!include[Introduction to proactive messages - part 1](../includes/snippet-proactive-messages-intro-1.md)] 

## <a name="types-of-proactive-messages"></a>자동 관리 메시지 형식 

[!include[Introduction to proactive messages - part 2](../includes/snippet-proactive-messages-intro-2.md)] 

## <a name="next-steps"></a>다음 단계

이제 활동, 메시지 및 대화 흐름을 숙지했으므로 LUIS를 사용하여 언어 이해를 시작하는 봇의 기타 중요한 측면을 살펴보겠습니다.

> [!div class="nextstepaction"]
> [언어 이해](bot-builder-concept-luis.md)
