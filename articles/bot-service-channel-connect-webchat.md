---
title: 웹 채팅 채널에 봇 연결 | Microsoft Docs
description: 웹 채팅 채널에 연결된 봇에 대해 웹 페이지의 웹 채팅 컨트롤을 사용하는 방법을 알아봅니다.
keywords: 웹 채팅, 봇 채널, 웹 페이지, 비밀 키, iframe, HTML
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: b5a9d20c058fe425d727bf2e39597e7dd29ec077
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389632"
---
# <a name="connect-a-bot-to-web-chat"></a>웹 채팅에 봇 연결
Bot Service를 사용하여 [봇을 만들 경우](bot-service-quickstart.md) 웹 채팅 채널이 자동으로 구성됩니다. 웹 채팅 채널에는 웹 페이지에서 직접 봇으로 상호 작용할 수 있는 기능을 제공하는 웹 채팅 컨트롤이 포함되어 있습니다.

![웹 채팅 샘플](~/media/bot-service-channel-webchat/webchat-sample.png)

Bot Framework 포털의 웹 채팅 채널에는 웹 페이지에 웹 채팅 컨트롤을 포함하는 데 필요한 모든 항목이 들어 있습니다. 웹 채팅 컨트롤을 사용하기 위해서는 봇의 비밀 키를 가져오고 웹 페이지에 컨트롤을 포함하기만 하면 됩니다.

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

## <a id="step-1"></a> 봇 비밀 키 가져오기

1. [Azure Portal](http://portal.azure.com)에서 봇을 열고 **채널** 블레이드를 클릭합니다.

2. **웹 채팅** 채널에 대해 **편집**을 클릭합니다.  
![웹 채팅 채널](~/media/bot-service-channel-webchat/bot-service-channel-list.png)

3. **비밀 키** 아래에서 첫 번째 키에 대해 **표시**를 클릭합니다.  
![비밀 키](~/media/bot-service-channel-webchat/secret-key.png)

4. **비밀 키** 및 **Embed 태그**를 복사합니다.

5. **Done**을 클릭합니다.

## <a name="embed-the-web-chat-control-in-your-website"></a>웹 사이트에 웹 채팅 컨트롤 포함

두 옵션 중 하나를 사용하여 웹 사이트에 웹 채팅 컨트롤을 포함할 수 있습니다.

### <a name="option-1---keep-your-secret-hidden-exchange-your-secret-for-a-token-and-generate-the-embed"></a>옵션 1 - 비밀을 숨겨진 상태로 유지하고, 암호를 토큰으로 바꾸고, embed 생성

서버 간 요청을 실행하여 웹 채팅 암호를 임시 토큰으로 바꿀 수 있고, 다른 개발자가 해당 웹 사이트에 사용자의 봇을 포함하기 어렵게 만들려면 이 옵션을 사용합니다. 이 옵션을 사용한다고 해서 다른 개발자가 웹 사이트에 사용자의 봇을 포함하는 것을 완전히 방지하지는 못하지만 어렵게 만들 수는 있습니다.

암호를 토큰으로 바꾸고, embed를 생성하려면

1. `https://webchat.botframework.com/api/tokens`에 대한 **GET** 요청을 실행하고 `Authorization` 헤더를 통해 웹 채팅 비밀을 전달합니다. `Authorization` 헤더는 아래의 예제 요청과 같이 `BotConnector` 체계를 사용하고 암호를 포함합니다.

2. **GET** 요청에 대한 응답에는 **iframe** 내에서 웹 채팅 컨트롤을 렌더링하여 대화를 시작하는 데 사용할 수 있는 토큰(따옴표로 묶음)이 포함됩니다. 토큰은 하나의 대화에서만 유효하므로 다른 대화를 시작하려면 새 토큰을 생성해야 합니다.

3. Bot Framework 포털 내의 웹 채팅 채널에서 복사한 `iframe` **Embed 태그** 내에서(위의 [봇 비밀 키 가져오기](#step-1)에서 설명한 대로) `s=` 매개 변수를 `t=`로 변경하고 "YOUR_SECRET_HERE"를 해당 토큰으로 바꿉니다.

> [!NOTE]
> 토큰은 만료되기 전에 자동으로 갱신됩니다. 

##### <a name="example-request"></a>요청 예

```requestGET https://webchat.botframework.com/api/tokens Authorization: BotConnector YOUR_SECRET_HERE
```

##### Example response 

```response
"IIbSpLnn8sA.dBB.MQBhAFMAZwBXAHoANgBQAGcAZABKAEcAMwB2ADQASABjAFMAegBuAHYANwA.bbguxyOv0gE.cccJjH-TFDs.ruXQyivVZIcgvosGaFs_4jRj1AyPnDt1wk1HMBb5Fuw"
```

##### <a name="example-iframe-using-token"></a>예제 iframe(토큰 사용)

```html
<iframe src="https://webchat.botframework.com/embed/YOUR_BOT_ID?t=YOUR_TOKEN_HERE"></iframe>
```

##### <a name="example-html-code"></a>예제 html 코드
```html
<!DOCTYPE html>
<html>
<body>
  <iframe id="chat" style="width: 400px; height: 400px;" src=''></iframe>
</body>
<script>

    var xhr = new XMLHttpRequest();
    xhr.open('GET', "https://webchat.botframework.com/api/tokens", true);
    xhr.setRequestHeader('Authorization', 'BotConnector ' + 'YOUR SECRET HERE');
    xhr.send();
    xhr.onreadystatechange = processRequest;

    function processRequest(e) {
      if (xhr.readyState == 4  && xhr.status == 200) {
        var response = JSON.parse(xhr.responseText);
        document.getElementById("chat").src="https://webchat.botframework.com/embed/lucas-direct-line?t="+response
      }
    }

  </script>
</html>
```

### <a id="option-2"></a> 옵션 2 - 암호를 사용하여 웹 사이트에 웹 채팅 컨트롤 포함

다른 개발자가 웹 사이트 봇을 쉽게 포함할 수 있도록 하려는 경우 이 옵션을 사용합니다. 

> [!WARNING]
> 이 옵션을 사용하는 경우 다른 개발자가 간단히 embed 태그를 복사하여 봇을 웹 사이트에 포함할 수 있습니다.

`iframe` 태그 내에 암호를 지정하여 웹 사이트에 봇을 포함하려면

1. Bot Framework 포털 내의 웹 채팅 채널에서 `iframe` **Embed**를 복사합니다(위의 [봇 비밀 키 가져오기](#step-1)에 설명한 대로).

2. 해당 **Embed 태그** 내에서 "YOUR_SECRET_HERE"를 같은 페이지에서 복사한 **비밀 키**로 바꿉니다.

##### <a name="example-iframe-using-secret"></a>예제 iframe(암호 사용)

```html
<iframe src="https://webchat.botframework.com/embed/YOUR_BOT_ID?s=YOUR_SECRET_HERE"></iframe>
```

## <a name="style-the-web-chat-control"></a>웹 채팅 컨트롤 스타일 지정

`iframe`의 `style` 특성에서 `height` 및 `width`를 지정하여 웹 채팅 컨트롤의 크기를 변경할 수 있습니다.

```html
<iframe style="height:480px; width:402px" src="... SEE ABOVE ..."></iframe>
```

![채팅 컨트롤 클라이언트](~/media/chatwidget-client.png)

## <a name="additional-resources"></a>추가 리소스

GitHub에서 웹 채팅 컨트롤에 대한 [소스 코드를 다운로드](https://github.com/Microsoft/BotFramework-WebChat)할 수 있습니다.
