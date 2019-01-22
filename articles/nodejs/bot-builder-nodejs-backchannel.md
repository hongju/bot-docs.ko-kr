---
title: 웹 컨트롤을 사용하여 정보 교환 | Microsoft Docs
description: Node.js용 Bot Framework SDK를 사용하여 봇과 웹 페이지 간에 정보를 교환하는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 06fcdd26980b719e7f7db76bc585650176c1d84f
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225682"
---
# <a name="use-the-backchannel-mechanism"></a>백채널 메커니즘 사용

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Introduction to backchannel mechanism](../includes/snippet-backchannel.md)]

## <a name="walk-through"></a>방법 설명

오픈 소스 웹 채팅 컨트롤은 <a href="https://github.com/microsoft/botframework-DirectLinejs" target="_blank">DirectLineJS</a>라는 JavaScript 클래스를 사용하여 직접 회선 API에 액세스합니다. 컨트롤은 고유한 직접 회선 인스턴스를 만들거나 호스팅 페이지와 공유할 수 있습니다. 컨트롤이 직접 회선 인스턴스를 호스팅 페이지와 공유하는 경우 컨트롤과 페이지 둘 다에서 활동을 보내고 받을 수 있습니다. 다음 다이어그램은 오픈 소스 웹(채팅) 컨트롤 및 직접 회선 API를 사용하여 봇 기능을 지원하는 웹 사이트의 개괄적인 아키텍처를 보여 줍니다. 

![백채널](../media/designing-bots/patterns/back-channel.png)

### <a name="sample-code"></a>샘플 코드 

이 예제에서 봇과 웹 페이지는 백채널 메커니즘을 사용하여 사용자에게 표시되지 않는 정보를 교환합니다. 봇은 웹 페이지의 배경색을 변경하도록 요청하고, 웹 페이지는 사용자가 페이지의 단추를 클릭할 때 봇에 알립니다. 

> [!NOTE]
> 이 문서의 코드 조각은 <a href="https://github.com/Microsoft/BotFramework-WebChat/blob/master/samples/backchannel/index.html" target="_blank">백채널 샘플</a> 및 <a href="https://github.com/ryanvolum/backChannelBot" target="_blank">백채널 봇</a>에서 가져온 것입니다. 

#### <a name="client-side-code"></a>클라이언트 쪽 코드

먼저 웹 페이지는 **DirectLine** 개체를 만듭니다.

```javascript
var botConnection = new BotChat.DirectLine(...);
```

그런 다음, WebChat 인스턴스를 만들 때 **DirectLine** 개체를 공유합니다.

```javascript
BotChat.App({
    botConnection: botConnection,
    user: user,
    bot: bot
}, document.getElementById("BotChatGoesHere"));
```

사용자가 웹 페이지의 단추를 클릭하면 웹 페이지는 “event” 형식의 활동을 게시하여 단추가 클릭되었음을 봇에 알립니다.

```javascript
const postButtonMessage = () => {
    botConnection
        .postActivity({type: "event", value: "", from: {id: "me" }, name: "buttonClicked"})
        .subscribe(id => console.log("success"));
    }
```

> [!TIP]
> `name` 및 `value` 특성을 사용하여 봇이 이벤트를 제대로 해석하고 응답하기 위해 필요할 수 있는 모든 정보를 전달합니다. 

마지막으로, 웹 페이지는 봇의 특정 이벤트를 수신 대기합니다.
이 예제에서 웹 페이지는 type=“event” 및 name=“changeBackground”인 활동을 수신 대기합니다. 이 형식의 활동을 받으면 웹 페이지의 배경색이 활동에 지정된 `value`로 변경됩니다. 

```javascript
botConnection.activity$
    .filter(activity => activity.type === "event" && activity.name === "changeBackground")
    .subscribe(activity => changeBackgroundColor(activity.value))
```

#### <a name="server-side-code"></a>서버 쪽 코드

<a href="https://github.com/ryanvolum/backChannelBot" target="_blank">백채널 봇</a>은 도우미 함수를 사용하여 이벤트를 만듭니다.

```javascript
var bot = new builder.UniversalBot(connector, 
    function (session) {
        var reply = createEvent("changeBackground", session.message.text, session.message.address);
        session.endDialog(reply);
    }
);

const createEvent = (eventName, value, address) => {
    var msg = new builder.Message().address(address);
    msg.data.type = "event";
    msg.data.name = eventName;
    msg.data.value = value;
    return msg;
}
```

마찬가지로, 봇도 클라이언트의 이벤트를 수신 대기합니다. 이 예제에서는 봇이 `name="buttonClicked"`인 이벤트를 받을 경우 “I see that you clicked a button.”이라는 메시지를 사용자에게 보냅니다.

```javascript
bot.on("event", function (event) {
    var msg = new builder.Message().address(event.address);
    msg.data.textLocale = "en-us";
    if (event.name === "buttonClicked") {
        msg.data.text = "I see that you clicked a button.";
    }
    bot.send(msg);
})
```

## <a name="additional-resources"></a>추가 리소스

- [직접 회선 API][directLineAPI]
- <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">Microsoft Bot Framework WebChat 컨트롤</a>
- <a href="https://aka.ms/v3-js-backchannel-sample" target="_blank">백채널 샘플</a>
- <a href="https://github.com/ryanvolum/backChannelBot" target="_blank">백채널 봇</a>

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle
