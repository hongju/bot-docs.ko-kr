---
title: 음성 통화 수행 | Microsoft Docs
description: Node.js를 사용하여 봇에서 Skype로 음성 통화를 수행하는 방법을 알아봅니다.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 147862b0fe684f9312a94fe903326ca737f5a3d6
ms.sourcegitcommit: 697a577d72aaf91a0834d4b4c2ef5aa11291f28f
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/01/2019
ms.locfileid: "67496701"
---
# <a name="support-audio-calls-with-skype"></a>Skype를 사용하여 음성 통화 지원

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Skype는 호출 봇이라는 풍부한 기능을 지원합니다.  이 기능이 사용되도록 설정되면 봇에 음성 통화를 걸고 IVR(대화형 음성 응답)을 사용하여 상호 작용할 수 있습니다.  Node.js용 Bot Builder SDK에는 개발자가 챗봇에 호출 기능을 추가하는 데 사용할 수 있는 특수한 [Calling SDK][calling_sdk]가 포함되어 있습니다.   

Calling SDK는 [Chat SDK][chat_sdk]와 매우 유사합니다. 비슷한 클래스를 포함하고 공통된 구조를 공유하며, Chat SDK를 사용하면 통화 중인 사용자에게 메시지를 보낼 수도 있습니다.  두 SDK는 나란히 실행되도록 디자인되었지만 유사한 반면, 중대한 차이점도 있습니다. 따라서 두 라이브러리의 클래스를 혼합해서 사용하지 않는 것이 좋습니다.  

## <a name="create-a-calling-bot"></a>호출 봇 만들기
다음 예제 코드에서는 호출 봇의 "Hello World"가 일반적인 챗봇과 얼마나 유사한지를 보여 줍니다. 

```javascript
var restify = require('restify');
var calling = require('botbuilder-calling');

// Setup Restify Server
var server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
   console.log(`${server.name} listening to ${server.url}`); 
});

// Create calling bot
var connector = new calling.CallConnector({
    callbackUrl: 'https://<your host>/api/calls',
    appId: '<your bots app id>',
    appPassword: '<your bots app password>'
});
var bot = new calling.UniversalCallBot(connector);
server.post('/api/calls', connector.listen());

// Add root dialog
bot.dialog('/', function (session) {
    session.send('Watson... come here!');
});
```

> [!NOTE]
> 봇의 **AppID** 및 **AppPassword**를 찾으려면 [MicrosoftAppID 및 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)를 참조하세요.

에뮬레이터는 현재 호출 봇의 테스트를 지원하지 않습니다. 봇을 테스트하려면 봇을 게시하는 데 필요한 대부분의 단계를 진행해야 합니다.  또한 Skype 클라이언트를 사용하여 봇과 상호 작용해야 합니다. 

### <a name="enable-the-skype-channel"></a>Skype 채널 사용
[봇을 등록](../bot-service-quickstart-registration.md)하고 Skype 채널을 사용하도록 설정합니다. 봇을 등록할 때 메시징 끝점을 제공해야 합니다. 챗봇의 끝점이 일반적으로 해당 필드에 추가하는 대상이 되도록 챗봇과 호출 봇을 페어링하는 것이 좋습니다.  호출 봇을 등록하기만 할 경우에는 호출 끝점을 해당 필드에 붙여넣기만 하면 됩니다.  

실제 호출 기능을 사용하도록 설정하려면 봇의 Skype 채널로 돌아가서 호출 기능을 설정해야 합니다. 그러면 호출 끝점을 복사하기 위한 필드가 제공됩니다. 호출 끝점의 호스트 부분으로 https ngrok 링크를 사용해야 합니다.

봇의 등록하는 동안 hello world 봇에 대한 커넥터 설정에 붙여넣어야 하는 앱 ID 및 암호가 사용자에게 할당됩니다. 또한 전체 호출 링크를 가져와 callbackUrl 설정에 붙여넣어야 합니다.

### <a name="add-bot-to-contacts"></a>연락처에 봇 추가
개발자 포털의 봇 등록 페이지에서 봇 Skype 채널 옆에는 **Skype에에 추가** 단추가 표시됩니다. 이 단추를 클릭하여 Skype의 연락처 목록에 봇을 추가합니다.  이렇게 하면 사용자(및 참가 링크를 받은 모든 사람)는 봇과 통신할 수 있습니다.

### <a name="test-your-bot"></a>봇 테스트
Skype 클라이언트를 사용하여 봇을 테스트할 수 있습니다. 봇 연락처 항목을 클릭하면 호출 아이콘이 켜집니다(검색해야 봇이 표시될 수도 있음).  기존 봇에 호출을 추가한 경우에는 호출 아이콘이 켜지는 데 몇 분 정도 걸릴 수 있습니다.  

호출 단추를 누르면 봇으로 전화를 걸게 되며, “Watson... come here!”라고 말하는 소리가 들립니다. 그러면 끊어야 합니다.

## <a name="calling-basics"></a>호출 기본 사항
[UniversalCallBot](http://docs.botframework.com/node/builder/calling-reference/classes/_botbuilder_d_.universalcallbot) 및 [CallConnector](http://docs.botframework.com/node/builder/calling-reference/classes/_botbuilder_d_.callconnector) 클래스를 사용하여 챗봇의 경우와 같은 방식으로 호출 봇을 작성할 수 있습니다. [채팅 다이얼로그](bot-builder-nodejs-manage-conversation-flow.md)와 기본적으로 동일한 다이얼로그를 봇에 추가합니다. 봇에 [폭포형 패턴](bot-builder-nodejs-prompts.md)을 추가할 수 있습니다. 세션 개체인 [CallSession](http://docs.botframework.com/node/builder/calling-reference/classes/_botbuilder_d_.callsession) 클래스가 있으며, 현재 호출을 관리하기 위한 [answer()](http://docs.botframework.com/node/builder/calling-reference/classes/_botbuilder_d_.callsession#answer), [hangup()](http://docs.botframework.com/node/builder/calling-reference/classes/_botbuilder_d_.callsession#hangup) 및 [reject()](http://docs.botframework.com/node/builder/calling-reference/classes/_botbuilder_d_.callsession#reject) 메서드가 추가되었습니다. 그렇지만 일반적으로 CallSession에는 사용자를 위해 호출을 자동으로 관리하는 논리가 있으므로 이러한 호출 제어 메서드에 대해서는 걱정할 필요가 없습니다. 사용자가 메시지를 전송하거나 기본 제공 프롬프트를 호출하는 것과 같은 작업을 수행하면 세션은 자동으로 호출에 응답합니다. 또한 [endConversation()](http://docs.botframework.com/node/builder/calling-reference/classes/_botbuilder_d_.callsession#endconversation)을 호출하는 경우 세션은 호출을 자동으로 끊거나 거부하고, 사용자가 발신자 질문에 답변을 중지했음을 감지합니다(기본 제공 프롬프트를 호출하지 않음)

호출 봇과 챗봇의 또 다른 차이점은 챗봇은 일반적으로 사용자에게 메시지, 카드 및 키보드를 전송하지만, 호출 봇은 작업 및 결과를 처리한다는 것입니다. Skype 호출 봇은 하나 이상의 [작업](http://docs.botframework.com/node/builder/calling-reference/interfaces/_botbuilder_d_.iaction)으로 구성되는 [워크플로](http://docs.botframework.com/node/builder/calling-reference/interfaces/_botbuilder_d_.iworkflow)를 만드는 데 필요합니다.  Bot Builder 호출 SDK가 이러한 작업을 대부분 관리하므로 실제로는 크게 걱정할 필요가 없습니다. [CallSession.send()](http://docs.botframework.com/node/builder/calling-reference/classes/_botbuilder_d_.callsession#send) 메서드를 사용하여 [PlayPromptActions](http://docs.botframework.com/node/builder/calling-reference/classes/_botbuilder_d_.playpromptaction)로 전환되는 작업 또는 문자열을 제공할 수 있습니다.  세션에는 여러 작업을 호출 서비스로 제출되는 단일 워크플로에 결합하는 자동 일괄 처리 논리가 포함되어 있으므로 send()를 여러 번 호출해도 안전합니다.  또한 사용자는 SDK의 기본 제공 [프롬프트](bot-builder-nodejs-prompts.md)를 사용하여 프롬프트가 모든 결과를 처리할 때 사용자로부터 입력을 수집해야 합니다.  

[calling_sdk]: http://docs.botframework.com/node/builder/calling-reference/modules/_botbuilder_d_
[chat_sdk]: http://docs.botframework.com/node/builder/chat-reference/modules/_botbuilder_d_
