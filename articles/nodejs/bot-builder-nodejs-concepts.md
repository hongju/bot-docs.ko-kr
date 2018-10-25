---
title: Node.js용 Bot Builder SDK의 주요 개념 | Microsoft Docs
description: Node.js용 Bot Builder SDK에서 사용할 수 있는 대화 봇의 빌드 및 배포와 관련한 주요 개념과 도구를 살펴봅니다.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 43b6f669cdbc91b78094d3d0d9e7a54f97f9884f
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998030"
---
# <a name="key-concepts-in-the-bot-builder-sdk-for-nodejs"></a>Node.js용 Bot Builder SDK의 주요 개념

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-concepts.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-concepts.md)

이 문서에서는 Node.js용 Bot Builder SDK의 주요 개념을 소개합니다. Bot Framework에 대한 소개는 [Bot Framework 개요](../overview-introduction-bot-framework.md)를 참조하세요.

## <a name="connector"></a>커넥터

Bot Framework 커넥터는 Skype, Facebook, Slack 및 SMS와 같은 클라이언트에 해당하는 여러 *채널*에 봇을 연결하는 서비스입니다. 이 커넥터는 봇에서 채널로, 그리고 채널에서 봇으로 메시지를 릴레이하면서 봇과 사용자 간의 통신을 용이하게 합니다. 봇의 논리는 커넥터 서비스를 통해 사용자로부터 메시지를 수신하는 웹 서비스로 호스팅되며, 봇의 회신은 HTTPS POST를 사용하여 커넥터로 전송됩니다. 

Node.js용 Bot Builder SDK는 Bot Framework 커넥터를 통해 메시지를 주고 받도록 봇을 구성하는 데 필요한 [UniversalBot][UniversalBot] 및 [ChatConnector][ChatConnector] 클래스를 제공합니다. `UniversalBot` 클래스는 봇의 두뇌를 구성합니다. 봇이 사용자와 수행하는 모든 대화를 관리해야 합니다. `ChatConnector` 클래스는 봇을 Bot Framework 커넥터 서비스에 연결합니다.
이러한 클래스의 사용을 보여 주는 예제는 [Node.js용 Bot Builder SDK를 사용하여 봇 만들기](bot-builder-nodejs-quickstart.md)를 참조하세요.

또한 이 커넥터는 봇이 채널에 보내는 메시지를 정규화하므로 플랫폼과 상관없는 방식으로 봇을 개발할 수 있습니다. 메시지를 정규화하려면 Bot Framework의 스키마에서 채널의 스키마로 변환해야 합니다. 채널이 프레임워크의 스키마 중 일부를 지원하지 않는 경우 커넥터는 채널이 지원하는 형식으로 메시지를 변환하려고 시도합니다. 예를 들어, 봇이 작업 단추가 있는 카드가 포함된 메시지를 SMS 채널에 보내면 커넥터는 카드를 이미지로 렌더링하고 메시지 텍스트에 링크로 작업을 포함시킬 수 있습니다. [채널 검사기][ChannelInspector]는 커넥터가 다양한 채널에서 메시지를 렌더링하는 방법을 보여 주는 웹 도구입니다.

`ChatConnector`를 사용하려면 봇 내에 API 엔드포인트를 설정해야 합니다. Node.js SDK가 있으면 일반적으로 `restify` Node.js 모듈을 설치하여 이를 수행할 수 있습니다. API 엔드포인트가 필요하지 않은 [ConsoleConnector][ConsoleConnector]를 사용하여 콘솔에 대한 봇을 만들 수도 있습니다.

## <a name="messages"></a>메시지

메시지는 표시할 텍스트, 말할 텍스트, 첨부 파일, 서식 있는 카드 및 제안된 동작으로 구성될 수 있습니다. [session.send][SessionSend] 메서드는 사용자의 메시지에 대한 응답으로 메시지를 보내는 데 사용합니다. 봇은 사용자가 보낸 메시지에 응답하여 원하는 만큼 `send`를 호출할 수 있습니다. 이를 보여 주는 예제는 [사용자 메시지에 응답][RespondMessages]을 참조하세요.

사용자가 작업을 시작하기 위해 클릭하는 대화형 단추가 포함된 서식 있는 그래픽 카드를 보내는 방법을 보여 주는 예제는 [메시지에 서식 있는 카드 추가](bot-builder-nodejs-send-rich-cards.md)를 참조하세요. 첨부 파일을 보내고 받는 방법을 보여 주는 예제는 [첨부 파일 보내기](bot-builder-nodejs-send-receive-attachments.md)를 참조하세요. 음성 지원 채널에서 봇이 말할 텍스트를 지정하는 메시지를 보내는 방법을 보여 주는 예제는 [메시지에 음성 추가](bot-builder-nodejs-text-to-speech.md)를 참조하세요. 제안된 동작을 보내는 방법을 보여 주는 예제는 [제안된 동작 보내기](bot-builder-nodejs-send-suggested-actions.md)를 참조하세요.

## <a name="dialogs"></a>대화 상자
대화 상자는 봇의 대화 논리를 구성하도록 도와주며, [대화 흐름을 설계](../bot-service-design-conversation-flow.md)하는 데 핵심입니다. 대화 상자에 대한 소개는 [대화 상자로 대화 관리](bot-builder-nodejs-dialog-manage-conversation.md)를 참조하세요.

## <a name="actions"></a>작업
대화 흐름 중 언제든지 취소나 도움 요청과 같은 중단을 처리할 수 있도록 봇을 설계해야 합니다. Node.js용 Bot Builder SDK는 취소 또는 다른 대화 상자 호출과 같은 작업을 트리거하는 글로벌 메시지 처리기를 제공합니다. [triggerAction][triggerAction] 처리기를 사용하는 방법에 대한 예제는 [사용자 작업 처리](bot-builder-nodejs-dialog-actions.md)를 참조하세요.
<!--[Handling cancel](bot-builder-nodejs-manage-conversation-flow.md#handling-cancel), [Confirming interruptions](bot-builder-nodejs-manage-conversation-flow.md#confirming-interruptions) and-->


## <a name="recognizers"></a>인식기
사용자가 "도움" 또는 "뉴스 찾기"와 같은 무언가를 봇에 요청하면 봇은 사용자가 요청한 내용을 이해해고 적절한 조치를 취해야 합니다. 사용자의 입력을 기반으로 의도를 인식하고 해당 의도를 작업과 연결하도록 봇을 설계할 수 있습니다. 

Bot Builder SDK가 제공하는 기본 제공 정규식 인식기를 사용하거나, LUIS API와 같은 외부 서비스를 호출하거나, 사용자 지정 인식기를 구현하여 사용자의 의도를 확인할 수 있습니다. 봇에 인식기를 추가하고 이를 사용하여 작업을 트리거하는 방법을 보여 주는 예제는 [사용자 의도 인식](bot-builder-nodejs-recognize-intent-messages.md)을 참조하세요.


## <a name="saving-state"></a>상태 저장

좋은 봇 설계의 핵심은 대화의 컨텍스트를 추적하여 사용자가 물어본 마지막 질문과 같은 것을 기억하도록 하는 것입니다. Bot Builder SDK를 사용하여 빌드된 봇은 상태 비저장으로 설계되어 여러 계산 노드에서 실행되도록 손쉽게 확장할 수 있습니다. Bot Framework는 봇 데이터를 저장하는 저장소 시스템을 제공하므로 봇 웹 서비스를 확장할 수 있습니다. 따라서 일반적으로 글로벌 변수 또는 함수 closure를 사용하여 상태를 저장하지 않아야 합니다. 이렇게 하면 봇을 확장하려고 할 때 문제가 발생합니다. 대신, 봇의 [세션][Session] 개체의 다음 속성을 사용하여 사용자 또는 대화와 관련된 데이터를 저장합니다.

* **userData**는 모든 대화에서 사용자의 대한 정보를 글로벌하게 저장합니다.
* **conversationData**는 단일 대화에 대한 정보를 글로벌하게 저장합니다. 이 데이터는 대화 내 모든 사용자에게 표시되므로 이 속성에 데이터를 저장할 때는 신중을 기해야 합니다. 기본적으로 사용하도록 설정되어 있으며, 봇의 [persistConversationData][PersistConversationData] 설정을 사용하여 비활성화할 수 있습니다.
* **privateConversationData**는 단일 대화에 대해 글로벌하게 정보를 저장하지만 현재 사용자에 관련된 개인 데이터입니다. 이 데이터는 모든 대화 상자에 걸쳐 있으므로 대화가 종료되면 정리할 임시 상태를 저장하는 데 유용합니다.
* **dialogData**는 단일 대화 상자 인스턴스에 대한 정보를 유지합니다. 이는 대화 상자에서 [폭포](bot-builder-nodejs-dialog-waterfall.md)의 단계 사이에 임시 정보를 저장하는 데 필수적입니다.

이러한 속성을 사용하여 데이터를 저장하고 검색하는 방법을 보여 주는 예제는 [상태 데이터 관리](bot-builder-nodejs-state.md)를 참조하세요.

## <a name="natural-language-understanding"></a>자연어 인식

Bot Builder는 LUIS를 사용하여 [LuisRecognizer][LuisRecognizer] 클래스를 통해 봇에 자연어 인식을 추가합니다. 게시된 언어 모델을 참조하는 **LuisRecognizer**의 인스턴스를 추가한 다음, 사용자의 표현에 응답하여 작업을 수행하는 처리기를 추가할 수 있습니다. 실행 중인 LUIS를 보려면 다음 10분 자습서를 참조하세요.

* [Microsoft LUIS 자습서][LUISVideo](비디오)

## <a name="next-steps"></a>다음 단계
> [!div class="nextstepaction"]
> [대화 상자 개요](bot-builder-nodejs-dialog-overview.md)



[PersistConversationData]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iuniversalbotsettings.html#persistconversationdata
[UniversalBot]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html
[ChatConnector]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
[ConsoleConnector]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.consoleconnector.html

[ChannelInspector]: ../bot-service-channel-inspector.md

[Session]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html
[SessionSend]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#send

[triggerAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction
[waterfall]: bot-builder-nodejs-prompts.md

[RespondMessages]:bot-builder-nodejs-use-default-message-handler.md

[LUISRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer
[LUISVideo]: https://vimeo.com/145499419
