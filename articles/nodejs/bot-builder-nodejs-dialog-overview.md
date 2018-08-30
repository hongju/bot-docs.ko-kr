---
title: 다이얼로그 개요 | Microsoft Docs
description: Node.js용 Bot Builder SDK 내의 다이얼로그를 사용하여 대화를 모델링하고 대화 흐름을 관리하는 방법에 대해 알아봅니다.
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 27e2840680b2c9551915057c1b431afa97236acc
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42906257"
---
# <a name="dialogs-in-the-bot-builder-sdk-for-nodejs"></a>Node.js용 Bot Builder SDK의 다이얼로그

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-dialogs.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-overview.md)

Node.js용 Bot Builder SDK의 다이얼로그를 사용하여 대화를 모델링하고 대화 흐름을 관리할 수 있습니다. 봇은 대화를 통해 사용자와 소통합니다. 대화는 다이얼로그로 구성됩니다. 다이얼로그는 폭포형 단계 및 프롬프트를 포함할 수 있습니다. 사용자가 봇과 상호 작용할 때 봇은 사용자 메시지에 대한 응답으로 다양한 다이얼로그를 시작 및 중지하고 다이얼로그 간을 전환합니다. 다이얼로그의 작동 방식을 이해하는 것이 뛰어난 봇을 성공적으로 디자인하고 만드는 데 핵심적입니다. 

이 문서에서는 다이얼로그 개념을 소개합니다. 이 문서를 읽은 후 [다음 단계](#next-steps) 섹션의 링크에 따라 이러한 개념을 좀 더 심도 깊게 이해할 수 있습니다.

## <a name="conversations-through-dialogs"></a>다이얼로그를 통한 대화

Node.js용 Bot Builder SDK는 하나 이상의 다이얼로그를 통한 봇과 사용자 간의 통신으로 대화를 정의합니다. 가장 기본적인 수준에서 다이얼로그는 작업을 수행하거나 사용자로부터 정보를 수집하는 재사용 가능 모듈입니다. 봇의 복잡한 논리를 재사용 가능 다이얼로그 코드에 캡슐화할 수 있습니다.

대화는 다음과 같은 여러 가지 방법으로 구성하고 변경할 수 있습니다.

- [기본 다이얼로그](#default-dialog)에서 시작될 수 있습니다.
- 하나의 다이얼로그 다른 다이얼로그로 리디렉션될 수 있습니다.
- 다시 시작할 수 있습니다.
- 사용자를 일련의 단계로 안내하는 [폭포형](bot-builder-nodejs-dialog-waterfall.md) 패턴을 따르거나 사용자에게 일련의 질문을 [프롬프트](bot-builder-nodejs-dialog-prompt.md)할 수 있습니다.
- 다른 다이얼로그를 트리거하는 단어 또는 구를 수신하는 [작업](bot-builder-nodejs-dialog-actions.md)을 사용할 수 있습니다. 

대화를 다이얼로그의 부모처럼 생각할 수 있습니다. 이와 같이 대화는 *다이얼로그 스택*을 포함하고, 자체적인 상태 데이터 집합(즉, `conversationData` 및 `privateConversationData`)을 유지 관리리합니다. 한편으로, 다이얼로그는 `dialogData`를 유지 관리리합니다. 상태 데이터에 대한 자세한 내용은 [상태 데이터 관리](bot-builder-nodejs-state.md)를 참조하세요.

## <a name="dialog-stack"></a>다이얼로그 스택

봇은은 다이얼로그 스택에 유지 관리되는 일련의 다이얼로그를 통해 사용자와 상호 작용합니다. 다이얼로그는 대화 과정에서 스택에 푸시되고 갑자기 사라집니다. 이 스택은 일반적인 LIFO 스택처럼 작동합니다. 즉, 완료되기 위해 마지막 다이얼로그가 첫 번째 다이얼로그가 됩니다. 하나의 다이얼로그가 완료되면 제어권이 스택의 이전 다이얼로그로 반환됩니다.

봇 대화가 처음 시작되거나 대화가 끝나면 다이얼로그 스택은 빈 상태가 됩니다. 이때 사용자가 봇으로 메시지를 전송하면 *기본 다이얼로그*로 응답합니다.

## <a name="default-dialog"></a>기본 다이얼로그

Bot Framework 버전 3.5 이전에서는 `/`라는 다이얼로그를 추가하고 URL과 비슷한 명명 규칙을 따라 *루트* 다이얼로그가 정의됩니다. 이 명명 규칙은 다이얼로그 명명에 적절하지 않았습니다. 

> [!NOTE]
> Bot Framework 버전 3.5부터 *기본 다이얼로그*가 [`UniversalBot`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html#constructor)의 구문에 보조 매개 변수로 등록됩니다.  

다음 코드 조각은 `UniversalBot` 개체를 만들 때 기본 다이얼로그를 정의하는 방법을 보여 줍니다.

```javascript
var bot = new builder.UniversalBot(connector, [
    //...Default dialog waterfall steps...
    ]);
```

기본 다이얼로그는 다이얼로그 스택이 비어 있고 다른 다이얼로그가 LUIS 또는 다른 [인식기](bot-builder-nodejs-recognize-intent-messages.md)를 통해 [트리거](bot-builder-nodejs-dialog-actions.md)되지 않을 때 실행됩니다. 기본 다이얼로그는 사용자에 대한 봇의 첫 번째 응답이므로 기본 다이얼로그는 사용 가능한 명령 또는 봇이이 수행할 수 있는 작업의 개요와 같은 일부 컨텍스트 정보를 사용자에게 제공해야 합니다.

## <a name="dialog-handlers"></a>다이얼로그 처리기

다이얼로그 처리기는 대화의 흐름을 관리합니다. 대화를 진행하려면 다이얼로그 처리기가 다이얼로그를 시작하고 종료하여 흐름을 지시합니다. 

## <a name="starting-and-ending-dialogs"></a>다이얼로그 시작 및 종료

새 다이얼로그를 시작(및 스택으로 푸시)하려면 [`session.beginDialog()`](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#begindialog)를 사용합니다. 다이얼로그를 종료(및 스택에서 제거하고 호출 대화 상자로 제어권 반환)하려면 [`session.endDialog()`](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#enddialog) 또는 [`session.endDialogWithResult()`](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#enddialogwithresult)를 사용합니다. 

## <a name="using-waterfalls-and-prompts"></a>폭포형 및 프롬프트 사용

[폭포](bot-builder-nodejs-dialog-waterfall.md)는 대화 흐름을 모델링하고 관리하는 간단한 방법입니다. 폭포에는 단계 시퀀스가 포함됩니다. 각 단계에서 사용자 대신 작업을 완료하거나 사용자에게 정보를 [프롬프트](bot-builder-nodejs-dialog-prompt.md)할 수 있습니다.

폭포는 함수의 컬렉션으로 구성된 다이얼로그를 사용하여 구현됩니다. 각 함수는 폭포의 단계를 정의합니다. 다음 코드 샘플에서는 2단계 폭포를 사용하여 사용자에게 이름을 묻고 이름을 사용해서 인사말을 하는 간단한 대화를 보여 줍니다.

```javascript
// Ask the user for their name and greet them by name.
bot.dialog('greetings', [
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
```

봇이 다이얼로그를 종료하지 않고 폭포 끝에 도달하면 사용자의 다음 메시지에 따라 폭포의 1단계에서 해당 다이얼로그가 다시 시작됩니다. 이 경우 사용자는 루프에 갇힌 것처럼 당황할 수 있습니다. 이러한 상황을 피하려면 대화 또는 다이얼로그가 끝날 때 `endDialog`, `endDialogWithResult` 또는 `endConversation`을 명시적으로 호출하는 것이 가장 좋습니다.

## <a name="next-steps"></a>다음 단계

다이얼로그를 좀 더 깊게 들어가려면 폭포 패턴이 작동하는 방식과 이러한 패턴을 사용하여 사용자에게 프로세스를 안내하는 방법을 이해하는 것이 중요합니다.

> [!div class="nextstepaction"]
> [폭포를 사용하여 대화 단계 정의](bot-builder-nodejs-dialog-waterfall.md)
