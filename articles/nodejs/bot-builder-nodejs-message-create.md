---
title: 메시지 만들기 | Microsoft Docs
description: Node.js용 Bot Builder SDK를 사용하여 메시지를 만드는 방법에 대해 알아봅니다.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/7/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 804081e52a03a27da418f549f0fadb6dc8bca52b
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905677"
---
# <a name="create-messages"></a>메시지 만들기

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

봇과 사용자는 메시지를 통해 통신합니다. 봇은 사용자에게 정보를 전달하도록 메시지 작업을 보내고 마찬가지로 사용자에게서 메시지 작업을 받습니다. 일부 메시지는 단순히 일반 텍스트로 구성될 수 있지만 기타 메시지에는 음성화된 텍스트, 제안된 작업, 미디어 첨부 파일, 다양한 카드 및 채널 관련 데이터와 같은 다양한 콘텐츠가 포함될 수 있습니다.

이 문서에서는 사용자 환경을 개선하는 데 사용할 수 있는 일반적으로 통용되는 메시지 메서드 중 일부를 설명합니다.

## <a name="default-message-handler"></a>기본 메시지 처리기

Node.js용 Bot Builder SDK는 [`session`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html)개체에 기본 제공되는 기본 메시지 처리기와 함께 제공됩니다. 이 메시지 처리기를 사용하면 봇과 사용자 간에 텍스트 메시지를 주고받을 수 있습니다.

### <a name="send-a-text-message"></a>텍스트 메시지 보내기

기본 메시지 처리기를 사용하여 텍스트 메시지를 보내는 것은 간단합니다. `session.send`를 호출하고 **문자열**에 전달합니다.

이 샘플에서는 사용자를 반기기 위해 텍스트 메시지를 보낼 수 있는 방법을 보여줍니다.
```javascript
session.send("Good morning.");
```

이 샘플에서는 JavaScript 문자열 템플릿을 사용하여 문자 메시지를 보낼 수 있는 방법을 보여줍니다.
```javascript
var msg = `You ordered: ${order.Description} for a total of $${order.Price}.`;
session.send(msg); //msg: "You ordered: Potato Salad for a total of $5.99."
```

### <a name="receive-a-text-message"></a>텍스트 메시지 수신

사용자가 봇에 메시지를 보낼 경우 봇은 `session.message` 속성을 통해 메시지를 받습니다.

이 샘플에서는 사용자의 메시지에 액세스하는 방법을 보여줍니다.
```javascript
var userMessage = session.message.text;
```

## <a name="customizing-a-message"></a>메시지 사용자 지정

메시지의 텍스트 서식에 대한 제어를 강화하려면 사용자 지정 [`message`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html) 개체를 만들고 사용자에게 보내기 전에 필요한 속성을 설정할 수 있습니다.

이 샘플에서는 사용자 지정 `message` 개체를 설정하고 `text`, `textFormat` 및 `textLocale` 속성을 설정하는 방법을 보여줍니다

```javascript
var customMessage = new builder.Message(session)
    .text("Hello!")
    .textFormat("plain")
    .textLocale("en-us");
session.send(customMessage);
```

범위 내에 `session` 개체가 없는 경우에는 `bot.send` 메서드를 사용하여 사용자에게 서식이 지정된 메시지를 보낼 수 있습니다.

메시지의 `textFormat` 속성은 텍스트의 서식을 지정하는 데 사용할 수 있습니다. `textFormat` 속성은 **일반**, **Markdown** 또는 **xml**로 설정할 수 있습니다. `textFormat`의 기본값은 **Markdown**입니다. 

일반적으로 지원되는 텍스트 서식 지정 목록은 [텍스트 서식 지정](../bot-service-channel-inspector.md#text-formatting)을 참조하세요. 대상 채널이 사용하려는 기능을 지원하는지 확인하려면 [채널 검사기](../bot-service-channel-inspector.md)를 사용하여 기능을 미리 보기합니다.

## <a name="message-property"></a>메시지 속성

[`Message`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html) 개체에는 송신할 메시지를 관리하는 데 사용되는 내부 **데이터** 속성이 있습니다. 설정한 다른 속성은 이 개체가 공개하는 다양한 방법을 통해 설정됩니다. 

## <a name="message-methods"></a>메시지 메서드

메시지 속성은 개체의 메서드를 통해 설정되고 검색됩니다. 아래 테이블에서는 다른 **메시지** 속성을 설정하기/가져오기 위해 호출할 수 있는 메서드 목록을 제공합니다.

| 방법 | 설명 |
| ---- | ---- | 
| [`addAttachment(attachment:AttachmentType)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#addattachment) | 메시지에 첨부 파일 추가|
| [`addEntity(obj:Object)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#addentity) | 메시지에 엔터티를 추가합니다. |
| [`address(adr:IAddress)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#address) | 메시지에 대한 라우팅 정보 주소를 지정합니다. 사용자에게 사전 대응 메시지를 보내려면 메시지의 주소를 userData 모음에 저장합니다. |
| [`attachmentLayout(style:string)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#attachmentlayout) | 클라이언트가 여러 첨부 파일을 레이아웃해야 하는 방법에 대한 힌트입니다. 기본값은 '목록'입니다. |
| [`attachments(list:AttachmentType)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#attachments) | 사용자에게 보낼 이미지 또는 카드의 목록입니다. |
| [`compose(prompts:string[], ...args:any[])`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#compose) | 사용자에게 보낼 복잡하고 임의적인 응답을 작성합니다. |
| [`entities(list:Object[])`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#entities) | 봇 또는 사용자에게 전달된 구조화된 개체입니다. |
| [`inputHint(hint:string)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#inputhint) | 봇이 추가 입력이 필요한지 여부를 알리며 사용자에게 보낸 힌트입니다. 기본 제공 프롬프트는 보내는 메시지에 대한 이 값을 자동으로 채웁니다. |
| [`localTimeStamp((optional)time:string)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#localtimestamp) | 메시지를 보낸 로컬 시간(클라이언트 또는 봇에서 설정, 예: 2016-09-23T13:07:49.4714686-07:00.) |
| [`originalEvent(event:any)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#originalevent) | 들어오는 메시지에 대한 채널의 원래/원시 형식의 메시지입니다. |
| [`sourceEvent(map:ISourceEventMap)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#sourceevent) | 보내는 메시지는 사용자 지정 첨부 파일과 같이 원본 관련 이벤트 데이터에 전달하는 데 사용할 수 있습니다. |
| [`speak(ssml:TextType, ...args:any[])`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#speak) | 메시지의 음성 필드를 *SSML(Speech Synthesis Markup Language)* 로 설정합니다. 이는 지원되는 장치에서 사용자에게 음성으로 전달됩니다. |
| [`suggestedActions(suggestions:ISuggestedActions `&#124;` IIsSuggestedActions)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#suggestedactions) | 사용자에게 보낼 제안된 선택적 작업입니다. 제안된 작업은 제안된 작업을 지원하는 채널에만 표시됩니다. |
| [`summary(text:TextType, ...argus:any[])`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#summary) | 메시지 콘텐츠에 대한 간단한 설명 및 대체로 표시될 텍스트입니다(예: 최근 대화 목록.) |
| [`text(text:TextType, ...args:any[])`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#text) | 메시지 텍스트를 설정합니다. |
| [`textFormat(style:string)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#textformat) | 텍스트 형식을 설정합니다. 기본 형식은 **markdown**입니다. |
| [`textLocale(locale:string)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#textlocale) | 메시지의 대상 언어를 설정합니다. |
| [`toMessage()`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#tomessage) | 메시지에 대한 JSON을 가져옵니다. |
| [`composePrompt(session:Session, prompts:string[], args?:any[])`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#composeprompt-1) | 프롬프트 배열을 단일 지역화된 프롬프트에 결합한 다음, 전달된 인수를 사용하여 필요에 따라 프롬프트 템플릿 슬롯을 채웁니다. |
| [`randomPrompt(prompts:TextType)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#randomprompt) | 전달되는 **프롬프트*의 배열에서 임의의 프롬프트를 가져옵니다. |

## <a name="next-step"></a>다음 단계

> [!div class="nextstepaction"]
> [첨부 파일 보내기 및 받기](bot-builder-nodejs-send-receive-attachments.md)

