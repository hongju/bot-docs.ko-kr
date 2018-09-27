---
title: 웹 채팅에서 음성 사용 | Microsoft Docs
description: 웹 채팅 채널에 연결된 봇에 대해 웹 채팅 컨트롤에서 음성을 사용하도록 설정하는 방법을 알아봅니다.
keywords: 말하기, 웹 채팅, 음성, 마이크, 오디오
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 563fdd480ac5165d5301faa3ed43af3118d2f7d7
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707479"
---
# <a name="enable-speech-in-web-chat"></a>웹 채팅에서 음성 사용
웹 채팅 컨트롤에서 음성 인터페이스를 사용하도록 설정할 수 있습니다. 웹 채팅 컨트롤에서 마이크를 사용하여 음성 인터페이스로 상호 작용합니다.

![웹 채팅 음성 샘플](~/media/bot-service-channel-webchat/webchat-sample-speech.png)

사용자가 말하는 대신 입력할 경우, 웹 채팅은 음성 기능을 끄고, 봇은 크게 말하는 대신 텍스트 응답만 제공합니다. 음성 응답을 다시 사용하도록 설정하려면 다음 번에 마이크를 사용하여 봇에 응답하면 됩니다. 마이크가 입력을 수락하면 어둡게 표시되거나 채워진 상태로 표시됩니다. 회색으로 표시되면 클릭하여 다시 사용하도록 설정할 수 있습니다.

## <a name="prerequisites"></a>필수 조건

  샘플을 실행하기 전에 웹 채팅 컨트롤을 사용하여 실행하려는 봇에 대한 직접 회선 암호 또는 토큰이 있어야 합니다. 
  * 봇과 연결된 직접 회선 암호를 가져오는 방법에 대한 내용은 [직접 회선에 봇 연결](bot-service-channel-connect-directline.md)을 참조하세요.
  * 토큰에 대한 암호를 교환하는 방법에 대한 내용은 [직접 회선 토큰 생성](rest-api/bot-framework-rest-direct-line-3-0-authentication.md)을 참조하세요.

## <a name="customizing-web-chat-for-speech"></a>음성에 대해 웹 채팅 사용자 지정
웹 채팅에서 음성 기능을 사용하도록 설정하려면 웹 채팅 컨트롤을 호출하는 JavaScript 코드를 사용자 지정해야 합니다. 다음 단계를 사용하여 음성 지원 웹 채팅을 로컬로 사용해볼 수 있습니다.

1. [샘플 index.html](https://aka.ms/web-chat-speech-sample)을 다운로드합니다. <!-- this aka.ms link needs to be updated if the sample location changes -->
2. 추가하려는 음성 지원 유형에 따라 `index.html`에 코드를 추가합니다. 음성 구현 유형은 [Speech Service 사용](#enable-speech-services)에 설명되어 있습니다. 
3. 웹 서버를 시작합니다. 이렇게 하는 한 가지 방법은 Node.js 명령 프롬프트에서 `npm http-server`를 사용하는 것입니다.

   * 명령줄에서 실행할 수 있게 `http-server`를 전역적으로 설치하려면 다음 명령을 실행합니다.

     ```
     npm install http-server -g
     ```

   * 포트 8000을 사용하여 웹 서버를 시작하려면 `index.html`이 포함된 디렉터리에서 다음 명령을 실행합니다.

     ```
     http-server -p 8000
     ```
4. 브라우저에서 `http://localhost:8000/samples?parameters`를 지정합니다. 예를 들어 `http://localhost:8000/samples?s=YOURDIRECTLINESECRET`는 직접 회선 암호를 사용하여 봇을 호출합니다. 쿼리 문자열에 설정할 수 있는 매개 변수는 다음 표에 나와 있습니다.

   | 매개 변수 | 설명 |
   |-----------|-------------|
   | s | 직접 회선 암호입니다. 직접 회선 암호를 가져오는 방법에 대한 내용은 [직접 회선에 봇 연결](bot-service-channel-connect-directline.md)을 참조하세요. |
   | t | 직접 회선 토큰입니다. 이 토큰을 생성하는 방법에 대한 내용은 [직접 회선 토큰 생성](rest-api/bot-framework-rest-direct-line-3-0-authentication.md)을 참조하세요. |
   | 도메인 | 선택 사항입니다. 대체 직접 회선 끝점의 URL입니다.  |
   | webSocket | 선택 사항입니다. WebSocket을 사용하여 메시지를 수신하려면 'true'로 설정합니다. 기본값은 `false`입니다. |
   | userId | 선택 사항입니다. 봇 사용자의 ID입니다.  |
   | 사용자 이름 | 선택 사항입니다. 봇 사용자의 사용자 이름입니다.  |
   | botid | 선택 사항입니다. 봇의 ID입니다. |
   | botname | 선택 사항입니다. 봇의 이름입니다. |


## <a name="enable-speech-services"></a>Speech Service 사용
사용자 지정을 사용하면 다음 방법 중 하나로 음성 기능을 추가할 수 있습니다.

* **브라우저 제공 음성** - 웹 브라우저에 기본 제공된 음성 기능을 사용합니다. 현재 이 기능은 Chrome 브라우저에서만만 사용할 수 있습니다.
* **Bing Speech Service 사용** - Bing Speech Service를 사용하여 음성 인식 및 합성을 제공할 수 있습니다. 이러한 음성 기능 액세스 방식은 다양한 브라우저에서 지원됩니다. 이 경우 처리는 브라우저가 아닌 서버에서 수행됩니다.
* **Custom Speech Service 만들기** - 사용자 지정 음성 인식 및 음성 합성 구성 요소를 만들 수 있습니다.

### <a name="browser-provided-speech"></a>브라우저 제공 음성

다음 코드는 브라우저에 제공되는 음성 인식기 및 음성 합성 구성 요소를 인스턴스화합니다. 이러한 음성 추가 방법이 모든 브라우저에서 지원되는 것은 아닙니다. 

> [!NOTE] 
> Google Chrome은 브라우저 음성 인식기를 지원합니다. 그러나 Chrome은 다음과 같은 경우 마이크를 차단할 수 있습니다.
> * 웹 채팅을 포함하는 페이지의 URL이 `https://`가 아닌 `http://`로 시작되는 경우
> * URL이 `http://localhost:8000` 대신 `file://` 프로토콜을 사용하는 로컬 파일인 경우

[!code-js[Specify speech options to use in-browser speech (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#BrowserSpeech)]

### <a name="bing-speech-service"></a>Bing Speech Service

다음 코드는 Bing Speech Service를 사용하는 음성 인식기 및 음성 합성 구성 요소를 인스턴스화합니다. 음성의 인식 및 생성은 서버에서 수행됩니다. 이 메커니즘은 여러 브라우저에서 지원됩니다. 

> [!TIP]
> Bing Speech Service를 사용하는 경우 봇의 음성 인식 정확도를 개선하기 위해 음성 인식 초기화를 사용할 수 있습니다. 자세한 내용은 [Speech Support in Bot Framework](https://blog.botframework.com/2017/06/26/Speech-To-Text)(Bot Framework의 음성 지원) 블로그 게시물을 참조하세요.

[!code-js[Specify speech options to use the Bing Speech API (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#BingSpeech)]

#### <a name="use-the-bing-speech-service-with-a-token"></a>Bing Speech Service와 토큰 사용

토큰을 사용하여 Cognitive Services 음성 인식을 설정하기 위한 옵션도 제공됩니다. 토큰은 API 키를 사용하여 보안 백 엔드에서 생성됩니다.

다음 예제 코드에서는 API 키 노출 방지를 위해 보안 백 엔드에서 토큰 페치가 수행되는 방법을 보여 줍니다.

[!code-js[Fetch a token to use with the Bing Speech API (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#FetchToken)]

### <a name="custom-speech-service"></a>Custom Speech Service

ISpeechRecognizer를 구현하는 사용자 지정 음성 인식이나 ISpeechSynthesis를 구현하는 음성 합성을 제공할 수도 있습니다. 

[!code-js[Fetch a token to use with a custom speech service (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#CustomSpeechService)]

### <a name="pass-the-speech-options-to-web-chat"></a>웹 채팅에 음성 옵션 전달

다음 코드는 웹 채팅 컨트롤에 음성 옵션을 전달합니다.

[!code-js[Pass speech options to Web Chat (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#PassSpeechOptionsToWebChat)]

## <a name="next-steps"></a>다음 단계
이제 음성으로 웹 채팅과 상호 작용하도록 설정할 수 있으므로, 봇이 음성 메시지를 생성하고 마이크의 상태를 조정하는 방법을 알아봅니다.
* [메시지에 음성 추가(C#)](dotnet/bot-builder-dotnet-text-to-speech.md)
* [메시지에 음성 추가(Node.js)](nodejs/bot-builder-nodejs-text-to-speech.md)

## <a name="additional-resources"></a>추가 리소스

* GitHub에서 웹 채팅 컨트롤에 대한 [소스 코드를 다운로드](https://github.com/Microsoft/BotFramework-WebChat)할 수 있습니다.
* [Bing Speech API 설명서](https://docs.microsoft.com/azure/cognitive-services/speech/home)는 Bing Speech API에 대해 자세히 설명합니다.

