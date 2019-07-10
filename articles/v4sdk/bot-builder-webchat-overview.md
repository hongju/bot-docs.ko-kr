---
title: 웹 채팅 개요 | Microsoft Docs
description: Bot Framework 웹 채팅을 구성하는 방법을 알아봅니다.
keywords: bot framework, 웹 채팅, 채팅, 샘플, react, 참조
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 06/07/2019
ms.openlocfilehash: 4dab0a8f3dab2a87184a03fe25b2a9abeaff67a2
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464650"
---
# <a name="web-chat-overview"></a>웹 채팅 개요

이 문서에는 [Bot Framework 웹 채팅](https://github.com/microsoft/BotFramework-WebChat) 구성 요소에 대한 세부 정보가 포함되어 있습니다. Bot Framework 웹 채팅 구성 요소는 Bot Framework V4 SDK용 웹 기반 클라이언트로, 세부적으로 사용자 지정할 수 있습니다. Bot Framework SDK v4를 사용하면 개발자가 대화를 모델링하고 정교한 봇 애플리케이션을 빌드할 수 있습니다.

웹 채팅 v3에서 v4로 마이그레이션하려는 경우 [마이그레이션 섹션](#migrating-from-web-chat-v3-to-v4)으로 이동합니다.

## <a name="how-to-use"></a>사용 방법

> [!NOTE]
> 이전 버전의 웹 채팅(v3)을 사용하는 경우 [웹 채팅 v3 분기](https://github.com/Microsoft/BotFramework-WebChat/tree/v3)로 이동합니다.

먼저 [Azure Bot Service](https://azure.microsoft.com/services/bot-service/)를 사용하여 봇을 만듭니다.
봇이 생성되면 Azure Portal에서 [봇의 웹 채팅 비밀을 얻어야](../bot-service-channel-connect-webchat.md#step-1) 합니다. 그런 다음, 이 비밀을 사용하여 [토큰을 생성](../rest-api/bot-framework-rest-direct-line-3-0-authentication.md)하고 웹 채팅에 전달합니다.

다음은 웹 사이트에 웹 채팅 컨트롤을 추가하는 방법입니다.

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  token: 'YOUR_DIRECT_LINE_TOKEN'
               }),
               userID: 'YOUR_USER_ID',
               username: 'Web Chat User',
               locale: 'en-US',
               botAvatarInitials: 'WC',
               userAvatarInitials: 'WW'
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

> `userID`, `username`, `locale`, `botAvatarInitials` 및 `userAvatarInitials`는 모두 `renderWebChat` 메서드에 전달할 선택적 매개 변수입니다. 웹 채팅 속성에 대해 자세히 알아보려면 이 `README`의 [웹 채팅 API 참조](#web-chat-api-reference) 섹션을 참조하세요.
> ![웹 채팅 스크린샷](https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/weatherquery.png.jpg)

### <a name="integrate-with-javascript"></a>JavaScript와 통합

웹 채팅은 JavaScript 또는 React를 사용하여 기존 웹 사이트와 통합하도록 설계되었습니다. JavaScript와 통합하면 보통 수준의 스타일 지정 및 사용자 지정 기능이 제공됩니다.

가장 일반적으로 사용되는 기능이 포함된 일반적인 전체 웹 채팅 패키지를 사용할 수 있습니다.

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  token: 'YOUR_DIRECT_LINE_TOKEN'
               }),
               userID: 'YOUR_USER_ID'
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

[전체 웹 채팅 번들](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.a.getting-started-full-bundle)의 작업 샘플을 참조하세요.

### <a name="integrate-with-react"></a>React와 통합

전체 사용자 지정을 위해 React를 사용하여 웹 채팅의 구성 요소를 재구성할 수 있습니다.

NPM에서 프로덕션 빌드를 설치하려면 `npm install botframework-webchat`를 실행합니다.

```jsx
import { DirectLine } from 'botframework-directlinejs';
import React from 'react';
import ReactWebChat from 'botframework-webchat';

export default class extends React.Component {
  constructor(props) {
    super(props);

    this.directLine = new DirectLine({ token: 'YOUR_DIRECT_LINE_TOKEN' });
  }

  render() {
    return (
      <ReactWebChat directLine={ this.directLine } userID='YOUR_USER_ID' />
      element
    );
  }
}
```

> `npm install botframework-webchat@master`를 실행하여 웹 채팅의 GitHub `master` 분기와 동기화된 개발 빌드를 설치할 수도 있습니다.

[React를 통해 렌더링된 웹 채팅](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/03.a.host-with-react/)의 작업 샘플을 참조하세요.

## <a name="customize-web-chat-ui"></a>웹 채팅 UI 사용자 지정

웹 채팅은 소스 코드를 포크하지 않고 사용자 지정할 수 있도록 설계되었습니다. 아래 표에서는 다양한 방법으로 웹 채팅을 가져올 때 사용할 수 있는 사용자 지정 종류를 간략하게 설명합니다. 이 목록은 전체 목록이 아닙니다.

|                               | CDN 번들         | React              |
| ----------------------------- | ------------------ | ------------------ |
| 색 변경                 | :heavy_check_mark: | :heavy_check_mark: |
| 크기 변경                  | :heavy_check_mark: | :heavy_check_mark: |
| CSS 스타일 업데이트/바꾸기     | :heavy_check_mark: | :heavy_check_mark: |
| 이벤트 수신 대기              | :heavy_check_mark: | :heavy_check_mark: |
| 호스팅 웹 페이지와 상호 작용 | :heavy_check_mark: | :heavy_check_mark: |
| 활동 사용자 지정 렌더링      |                    | :heavy_check_mark: |
| 첨부 파일 사용자 지정 렌더링     |                    | :heavy_check_mark: |
| 새 UI 구성 요소 추가         |                    | :heavy_check_mark: |
| 전체 UI 재구성        |                    | :heavy_check_mark: |

사용자 지정에 대해 자세히 알아보려면 [웹 채팅 사용자 지정](https://github.com/Microsoft/BotFramework-WebChat/blob/master/SAMPLES.md)을 참조하세요.

## <a name="migrating-from-web-chat-v3-to-v4"></a>웹 채팅 v3에서 v4로 마이그레이션

v3에서 v4로 마이그레이션할 때 사용할 수 있는 세 가지 경로가 있습니다. 먼저 시작 시나리오를 비교합니다.

### <a name="my-current-website-integrates-web-chat-using-an-iframe-element-obtained-from-azure-bot-services-i-want-to-upgrade-to-v4"></a>현재 웹 사이트가 Azure Bot Services에서 가져온 `<iframe>` 요소를 사용하여 웹 채팅을 통합합니다. v4로 업그레이드하려고 합니다.

2019년 5월부터, `<iframe>` 요소를 사용하여 웹 채팅을 통합하는 웹 사이트에 v4를 배포하고 있습니다. `<iframe>`을 사용하여 웹 채팅을 통합하는 방법에 대한 자세한 내용은 [포함 문서](https://github.com/Microsoft/BotFramework-WebChat/tree/master/packages/embed)를 참조하세요.

### <a name="my-website-is-integrated-with-web-chat-v3-and-uses-customization-options-provided-by-web-chat-no-customization-at-all-or-very-little-of-my-own-customization-that-was-not-available-with-web-chat"></a>웹 사이트가 웹 채팅 v3와 통합되어 있으며, 웹 채팅에서 제공하는 사용자 지정 옵션을 사용합니다. 웹 채팅에서 사용할 수 없는 사용자 지정이 전혀 또는 거의 없었습니다.

웹 페이지를 웹 채팅 v3에서 v4로 변환하려면 샘플 [`01.c.getting-started-migration`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.c.getting-started-migration) 구현을 따르세요.

### <a name="my-website-is-integrated-with-a-fork-of-web-chat-v3-i-have-implemented-a-lot-of-customization-in-my-version-of-web-chat-and-i-am-concerned-v4-is-not-compatible-with-my-needs"></a>웹 사이트가 웹 채팅 v3 포크와 통합되어 있습니다. 웹 채팅 버전에서 많은 사용자 지정을 구현했는데, v4가 내 요구 사항과 호환되지 않을까 염려가 됩니다.

웹 채팅 v4에서 팀이 가장 좋아하는 사항 중 하나는 **웹 채팅을 포크하지 않고도** 사용자 지정을 추가할 수 있다는 것입니다. 이로 인해 이전에 웹 채팅을 포크한 v3 사용자에게 추가 오버헤드가 발생하긴 하지만, 곤란을 겪는 고객을 지원하기 위해 최선을 다하겠습니다. 다음 제안 사항을 사용하세요.

-  샘플 [`01.c.getting-started-migration`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.c.getting-started-migration) 구현을 살펴봅니다. 웹 채팅을 시작하고 실행하는 데 이 샘플을 활용하면 좋습니다.
-  그런 다음, [샘플 목록](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples)을 검토하여 사용자 지정 요구 사항을 웹 채팅에서 이미 지원하는 사항과 비교합니다. 해당 샘플은 웹 채팅에 대해 일반적으로 요청된 기능으로 이루어져 있습니다.
-  샘플에서 하나 이상의 기능을 사용할 수 없는 경우, [미해결 문제 및 종결된 문제](https://github.com/Microsoft/BotFramework-WebChat/issues?utf8=%E2%9C%93&q=is%3Aissue+), [샘플 레이블](https://github.com/Microsoft/BotFramework-WebChat/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+label%3ASample) 및 [마이그레이션 지원 레이블](https://github.com/Microsoft/BotFramework-WebChat/issues?q=is%3Aissue+migrate+label%3A%22Migration+Support%22)을 살펴보고 원하는 기능에 대한 샘플 요청 및/또는 사용자 지정 지원을 검색합니다. 미해결 문제에 주석을 추가하면 팀이 수요가 많은 요청에 우선 순위를 지정하는 데 도움이 되며, 커뮤니티에도 참여해 보시기 바랍니다.
-  미해결 요청 목록에서 원하는 기능을 찾지 못한 경우 언제든지 [요청을 직접 제출](https://github.com/Microsoft/BotFramework-WebChat/issues/new)하시면 됩니다. 위의 항목과 마찬가지로, 미해결 문제에 설명을 추가하는 다른 고객도 웹 채팅 사용자에게 가장 일반적으로 필요한 기능의 우선 순위를 지정하는 데 도움이 됩니다.
-  마지막으로, 가능한 한 빨리 기능이 필요한 경우 웹 채팅에 대한 [끌어오기 요청](https://github.com/Microsoft/BotFramework-WebChat/compare)을 실행합니다. 기능을 직접 구현할 수 있는 코딩 경험이 있는 경우 추가적인 지원을 부탁드립니다. 기능을 직접 만들면 웹 채팅에서 보다 신속하게 사용할 수 있으며, 동일하거나 유사한 기능을 원하는 다른 고객도 이 기능을 활용할 수 있습니다.
-  v4에 대해 자세히 알아보려면 이 `README`의 나머지 부분을 확인하세요.

## <a name="samples-list"></a>샘플 목록

| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;샘플&nbsp;이름&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | 설명                                                                                                                                                                                                                         | 링크                                                                                                                                |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| [`01.a.getting-started-full-bundle`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.a.getting-started-full-bundle)                                                                                       | CDN의 웹 채팅 포함을 소개하고, 간단하면서도 모든 기능을 갖춘 웹 채팅을 보여 줍니다. 여기에는 적응형 카드, Cognitive Services, Markdown-It 종속성 등이 포함됩니다.                                                            | [전체 번들 데모](https://microsoft.github.io/BotFramework-WebChat/01.a.getting-started-full-bundle)                               |
| [`01.b.getting-started-es5-bundle`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.b.getting-started-es5-bundle)                                                                                         | 웹 채팅의 ES5 포니필을 사용하는 ES5 브라우저의 이전 버전과 호환성이 있는 모든 기능을 갖춘 웹 채팅 포함을 소개합니다.                                                                                                                | [ES5 번들 데모](https://microsoft.github.io/BotFramework-WebChat/01.b.getting-started-es5-bundle)                                 |
| [`01.c.getting-started-migration`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.c.getting-started-migration)                                                                                           | 웹 채팅 v3 봇에서 v4로 마이그레이션하는 방법을 보여 줍니다.                                                                                                                                                                        | [마이그레이션 데모](https://microsoft.github.io/BotFramework-WebChat/01.c.getting-started-migration)                                   |
| [`02.a.getting-started-minimal-bundle`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/02.a.getting-started-minimal-bundle)                                                                                 | 기본 종속성만 있는 최소화된 CDN을 소개합니다. 여기에는 적응형 카드, Cognitive Services 종속성 또는 Markdown-It 종속성이 포함되지 않습니다.                                                                      | [최소 번들 데모](https://microsoft.github.io/BotFramework-WebChat/02.a.getting-started-minimal-bundle)                         |
| [`02.b.getting-started-minimal-markdown`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/02.b.getting-started-minimal-markdown)                                                                             | 최소 번들에 Markdown-It 종속성 CDN을 추가하는 방법을 보여 줍니다.                                                                                                                                            | [Markdown 포함 최소 번들 데모](https://microsoft.github.io/BotFramework-WebChat/02.b.getting-started-minimal-markdown):                |
| [`03.a.host-with-react`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/03.a.host-with-react)                                                                                                               | 완전한 기능을 갖춘 웹 채팅을 호스트하는 React 구성 요소를 만드는 방법을 보여 줍니다.                                                                                                                                                 | [React로 호스트 데모](https://microsoft.github.io/BotFramework-WebChat/03.a.host-with-react)                                       |
| [`03.b.host-with-Angular`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/03.b.host-with-angular)                                                                                                           | 완전한 기능을 갖춘 웹 채팅을 호스트하는 Angular 구성 요소를 만드는 방법을 보여 줍니다.                                                                                                                                              | [Angular로 호스트 데모](https://stackblitz.com/github/omarsourour/ng-webchat-example)                                              |
| [`04.a.display-user-bot-initials-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/04.a.display-user-bot-initials-styling)                                                                           | 양쪽 웹 채팅 참가자의 이니셜을 모두 표시하는 방법을 보여 줍니다.                                                                                                                                                                | [봇 이니셜 데모](https://github.com/Microsoft/BotFramework-WebChat/blob/master/samples/04.a.display-user-bot-initials-styling/)  |
| [`04.b.display-user-bot-images-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/04.b.display-user-bot-images-styling)                                                                               | 양쪽 웹 채팅 참가자의 이미지와 이니셜을 모두 표시하는 방법을 보여 줍니다.                                                                                                                                                     | [사용자 이미지 데모](https://microsoft.github.io/BotFramework-WebChat/04.b.display-user-bot-images-styling)                           |
| [`05.a.branding-webchat-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/05.a.branding-webchat-styling)                                                                                             | 브랜드에 맞게 웹 채팅의 스타일을 지정하는 기능을 소개합니다. 이 사용자 지정 스타일 방법은 웹 채팅 업데이트 시 중단되지 않습니다.                                                                                                   | [웹 채팅 브랜딩 데모](https://microsoft.github.io/BotFramework-WebChat/05.a.branding-webchat-styling)                            |
| [`05.b.idiosyncratic-manual-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/05.b.idiosyncratic-manual-styling/)                                                                                    | 수동으로 스타일을 변경하는 방법을 소개하며, 웹 채팅의 스타일을 사용자 지정하는 더 복잡하고 시간이 오래 걸리는 방법입니다. 수동 스타일은 웹 채팅 업데이트 시 중단될 수 있습니다.                                                | [독특한 스타일 지정 데모](https://microsoft.github.io/BotFramework-WebChat/05.b.idiosyncratic-manual-styling)                    |
| [`05.c.presentation-mode-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/05.c.presentation-mode-styling)                                                                                           | 채팅 기록만 표시되고 보내기 상자가 표시되지 않으며 적응형 카드의 대화형 작업을 사용할 수 없는 프레젠테이션 모드를 설정하는 방법을 보여 줍니다.                                                                         | [프레젠테이션 모드 데모](https://microsoft.github.io/BotFramework-WebChat/05.c.presentation-mode-styling)                           |
| [`05.d.hide-upload-button-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/05.d.hide-upload-button-styling)                                                                                         | 스타일 지정을 통해 파일 업로드 단추를 숨기는 방법을 보여 줍니다.                                                                                                                                                                            | [업로드 단추 숨기기 데모](https://microsoft.github.io/BotFramework-WebChat/05.d.hide-upload-button-styling)                         |
| [`06.a.cognitive-services-bing-speech-js`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.a.cognitive-services-bing-speech-js)                                                                           | Cognitive Services Bing Speech API(사용되지 않음)와 JavaScript를 사용한 음성 텍스트 변환 및 텍스트 음성 변환 기능을 소개합니다.                                                                                                      | [JS를 사용한 Bing Speech 데모](https://microsoft.github.io/BotFramework-WebChat/06.a.cognitive-services-bing-speech-js)                 |
| [`06.b.cognitive-services-bing-speech-react`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.b.cognitive-services-bing-speech-react)                                                                     | Cognitive Services Bing Speech API(사용되지 않음)와 React를 사용한 음성 텍스트 변환 및 텍스트 음성 변환 기능을 소개합니다.                                                                                                           | [React를 사용한 Bing Speech 데모](https://microsoft.github.io/BotFramework-WebChat/06.b.cognitive-services-bing-speech-react)           |
| [`06.c.cognitive-services-speech-services-js`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.c.cognitive-services-speech-services-js)                                                                   | Cognitive Services Speech Services API를 사용한 음성 텍스트 변환 및 텍스트 음성 변환 기능을 소개합니다.                                                                                                                                  | [JS를 사용한 Speech Services 데모](https://microsoft.github.io/BotFramework-WebChat/06.c.cognitive-services-speech-services-js)         |
| [`06.d.speech-web-browser`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.d.speech-web-browser)                                                                                                         | 웹 채팅의 브라우저 기반 Web Search API를 사용하여 텍스트 음성 변환을 구현하는 방법을 보여 줍니다. (샘플에서 W3C 표준으로 연결됨)                                                                                                    | [Web Speech API 데모](https://microsoft.github.io/BotFramework-WebChat/06.d.speech-web-browser)                                     |
| [`06.e.cognitive-services-speech-services-with-lexical-result`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.e.cognitive-services-speech-services-with-lexical-result)                                 | Cognitive Services Speech Services API의 어휘 결과를 사용하는 방법을 보여 줍니다.                                                                                                                                                 | [어휘 결과 데모](https://microsoft.github.io/BotFramework-WebChat/06.e.cognitive-services-speech-services-with-lexical-result) |
| [`06.f.hybrid-speech`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.f.hybrid-speech)                                                                                                                   | 음성 텍스트 변환에 브라우저 기반 Web Speech API를 사용하고, 텍스트 음성 변환에 Cognitive Services Speech Services API를 사용하는 방법을 보여 줍니다.                                                                                        | [하이브리드 음성 데모](https://microsoft.github.io/BotFramework-WebChat/06.f.hybrid-speech)                                           |
| [`07.a.customization-timestamp-grouping`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/07.a.customization-timestamp-grouping)                                                                             | 타임스탬프를 표시하거나 숨기고 시간별 메시지 그룹화를 변경하여 타임스탬프를 사용자 지정하는 방법을 보여 줍니다.                                                                                                             | [타임스탬프 그룹화 데모](https://microsoft.github.io/BotFramework-WebChat/07.a.customization-timestamp-grouping)                   |
| [`07.b.customization-send-typing-indicator`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/07.b.customization-send-typing-indicator)                                                                       | 사용자가 보내기 상자에서 입력을 시작할 때 입력 활동을 보내는 방법을 보여 줍니다.                                                                                                                                                | [사용자 입력 표시기 데모](https://microsoft.github.io/BotFramework-WebChat/07.b.customization-send-typing-indicator)             |
| [`08.customization-user-highlighting`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/08.customization-user-highlighting)                                                                                   | 메시지가 사용자 또는 봇이 보낸 것인지에 따라 활동의 스타일을 사용자 지정하는 방법을 보여 줍니다.                                                                                                                      | [사용자 강조 표시 데모](https://microsoft.github.io/BotFramework-WebChat/08.customization-user-highlighting)                       |
| [`09.customization-reaction-buttons`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/09.customization-reaction-buttons/)                                                                                    | 봇의 요구 사항에 고유한 사용자 지정 웹 채팅 구성 요소를 만드는 기능을 소개합니다. 이 자습서에서는 :thumbsup:, :thumbsdown: 등의 반응 이모지를 대화 활동에 추가하는 기능을 보여 줍니다. | [반응 단추 데모](https://microsoft.github.io/BotFramework-WebChat/09.customization-reaction-buttons)                         |
| [`10.a.customization-card-components`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/10.a.customization-card-components)                                                                                   | 사용자 지정 활동 카드 첨부 파일(데모에서는 GitHub 리포지토리 카드)을 만드는 방법을 보여 줍니다.                                                                                                                                  | [카드 구성 요소 데모](https://microsoft.github.io/BotFramework-WebChat/10.a.customization-card-components)                         |
| [`10.b.customization-password-input`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/10.b.customization-password-input)                                                                                     | 암호 입력을 위한 사용자 지정 활동을 만드는 방법을 보여 줍니다.                                                                                                                                                                      | [암호 입력 데모](https://microsoft.github.io/BotFramework-WebChat/10.b.customization-password-input)                           |
| [`11.customization-redux-actions`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/11.customization-redux-actions)                                                                                           | 고급 자습서: 봇을 통해 redux 작업을 전송하여 redux 미들웨어를 웹 채팅 앱에 통합하는 방법을 보여 줍니다. 이 예제에서는 봇과 사용자 간의 활동을 기반으로 하는 수동 스타일 지정을 보여 줍니다.             | [Redux 작업 데모](https://microsoft.github.io/BotFramework-WebChat/11.customization-redux-actions)                               |
| [`12.customization-minimizable-web-chat`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/12.customization-minimizable-web-chat)                                                                             | 고급 자습서: 웹 사이트에 웹 채팅 인터페이스를 최소화 가능 표시/숨기기 채팅 상자로 추가하는 방법을 보여 줍니다.                                                                                                              | [최소화 가능 웹 채팅 데모](https://microsoft.github.io/BotFramework-WebChat/12.customization-minimizable-web-chat)                 |
| [`13.customization-speech-ui`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/13.customization-speech-ui)                                                                                                   | 고급 자습서: 봇의 주요 구성 요소(데모에서는 음성)를 완전히 사용자 지정하여 텍스트 기반 기록 UI를 완전히 대체하고, 대신 봇의 응답과 함께 간단한 음성 단추를 표시하는 방법을 보여 줍니다.      | [음성 UI 데모](https://microsoft.github.io/BotFramework-WebChat/13.customization-speech-ui)                                       |
| [`14.customization-piping-to-redux`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/14.customization-piping-to-redux)                                                                                       | 고급 자습서: 봇 활동을 사용자 고유의 Redux 저장소로 파이프하고, 봇을 사용하여 봇 활동과 Redux를 통해 페이지를 제어하는 방법을 보여 줍니다.                                                                          | [Redux로 파이프 데모](https://microsoft.github.io/BotFramework-WebChat/14.customization-piping-to-redux)                           |
| [`15.a.backchannel-piggyback-on-outgoing-activities`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/15.a.backchannel-piggyback-on-outgoing-activities)                                                     | 고급 자습서: 나가는 모든 활동에 사용자 지정 데이터를 추가하는 방법을 보여 줍니다.                                                                                                                                                | [백채널 피기배킹(Piggybacking) 데모](https://microsoft.github.io/BotFramework-WebChat/15.a.backchannel-piggyback-on-outgoing-activities) |
| [`15.b.incoming-activity-event`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/15.b.incoming-activity-event)                                                                                               | 고급 자습서: 추가 처리를 위해 들어오는 모든 활동을 JavaScript 이벤트로 전달하는 방법을 보여 줍니다.                                                                                                                | [들어오는 활동 데모](https://microsoft.github.io/BotFramework-WebChat/15.b.incoming-activity-event)                             |
| [`15.c.programmatic-post-activity`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/15.c.programmatic-post-activity)                                                                                         | 고급 자습서: 프로그래밍 방식으로 메시지를 보내는 방법을 보여 줍니다.                                                                                                                                                             | [프로그래밍 방식 게시 데모](https://microsoft.github.io/BotFramework-WebChat/15.c.programmatic-post-activity)                       |
| [`15.d.backchannel-send-welcome-event`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/15.d.backchannel-send-welcome-event)                                                                                 | 고급 자습서: 브라우저 언어 등의 클라이언트 기능을 사용하여 시작 이벤트를 보내는 방법을 보여 줍니다.                                                                                                                        | [시작 이벤트 데모](https://microsoft.github.io/BotFramework-WebChat/15.d.backchannel-send-welcome-event)                          |
| [`16.customization-selectable-activity`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/16.customization-selectable-activity)                                                                               | 고급 자습서: 각 활동에 사용자 지정 클릭 동작을 추가하는 방법을 보여 줍니다.                                                                                                                                                  | [선택 가능한 활동 데모](https://microsoft.github.io/BotFramework-WebChat/16.customization-selectable-activity)                   |
| [`17.chat-send-history`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/17.chat-send-history)                                                                                                               | 고급 자습서: 사용자 입력을 저장하고 사용자가 이전에 보낸 메시지를 살펴볼 수 있는 기능을 보여 줍니다.                                                                                                      | [채팅 전송 기록 데모](https://microsoft.github.io/BotFramework-WebChat/17.chat-send-history)                                     |
| [`18.customization-open-url`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/18.customization-open-url)                                                                                                     | 고급 자습서: URL 열기 동작을 사용자 지정하는 방법을 보여 줍니다.                                                                                                                                                             | [URL 열기 사용자 지정 데모](https://microsoft.github.io/BotFramework-WebChat/18.customization-open-url)                               |
| [`19.a.single-sign-on-for-enterprise-apps`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/19.a.single-sign-on-for-enterprise-apps)                                                                         | OAuth를 사용하여 엔터프라이즈 앱에 Single Sign-On을 사용하는 방법을 보여 줍니다.                                                                                                                                                              | [OAuth를 사용한 Single Sign-On 데모](https://microsoft.github.io/BotFramework-WebChat/19.a.single-sign-on-for-enterprise-apps)         |
 

## <a name="web-chat-api-reference"></a>웹 채팅 API 참조

웹 채팅 React 구성 요소(`<ReactWebChat>`) 또는 `renderWebChat()` 메서드에 전달할 수 있는 몇 가지 속성이 있습니다. [`packages/component/src/Composer.js`](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Composer.js#L378)부터 시작해서 소스 코드를 자유롭게 살펴보세요. 다음은 사용 가능한 속성에 대한 간단한 설명입니다.

| 자산                   | 설명                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `activityMiddleware`       | 개발자가 기존 활동 DOM에 새 DOM 구성 요소를 추가할 수 있도록 하는 미들웨어 체인으로, [Redux 미들웨어](https://medium.com/@jacobp100/you-arent-using-redux-middleware-enough-94ffe991e6)를 따라 모델링되었습니다. 미들웨어 서명은 `options => next => card => children => next(card)(children)`입니다.                                                                                                                                                                                                                                           |
| `activityRenderer`         | Redux의 [저장소 강화 도구](https://github.com/reduxjs/redux/blob/master/docs/Glossary.md#store-enhancer) 개념과 유사한 `activityMiddleware`의 “평면화된” 버전입니다.                                                                                                                                                                                                                                                                                                                                                                                                                |
| `adaptiveCardHostConfig`   | 사용자 지정 적응형 카드 호스트 구성을 전달합니다. 사용 중인 적응형 카드 버전으로 호스트 구성을 확인해야 합니다. 자세한 내용은 [사용자 지정 호스트 구성](https://github.com/microsoft/BotFramework-WebChat/issues/2034#issuecomment-501818238)을 참조하세요.                                                                                                                                                                                                                                                                                                                                    |
| `attachmentMiddleware`     | 개발자가 첨부 파일에 고유한 사용자 지정 HTML 요소를 추가할 수 있도록 하는 미들웨어 체인입니다. 서명은 `options => next => card => next(card)`입니다.                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `attachmentRenderer`       | `attachmentMiddleware`의 “평면화된” 버전입니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `cardActionMiddleware`     | 개발자가 적응형 카드, 제안된 작업 등의 카드 작업을 수정할 수 있도록 하는 미들웨어 체인입니다. 미들웨어 서명은 `cardActionMiddleware: () => next => ({ cardAction, getSignInUrl }) => next(cardAction)`입니다.                                                                                                                                                                                                                                                                                                                                           |
| `createDirectLine`         | Direct Line 개체를 인스턴스화하는 팩터리 메서드입니다. Azure Government 사용자는 `createDirectLine({ domain: 'https://directline.botframework.azure.us/v3/directline', token });`을 사용해서 엔드포인트를 변경해야 합니다. 매개 변수의 전체 목록은 `conversationId`, `domain`, `fetch`, `pollingInterval`, `secret`, `streamUrl`, `token`, `watermark` `webSocket`입니다.                                                                                                                                                                                                                         |
| `createStore`              | 개발자가 저장소 작업을 수정할 수 있도록 하는 미들웨어 체인입니다. 미들웨어 서명은 `createStore: ({}, ({ dispatch }) => next => action => next(cardAction)`입니다.                                                                                                                                                                                                                                                                                                                                                                                                |
| `directLine`               | DirectLine 토큰이 있는 DirectLine 개체를 지정합니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `disabled`                 | 웹 채팅 UI(즉, 프레젠테이션 모드 UI)를 사용하지 않도록 설정합니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `grammars`                 | 음성(Bing Speech 또는 Cognitive Services Speech Services)의 문법 목록을 지정합니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `groupTimeStamp`           | 타임스탬프 그룹화의 기본 설정을 변경합니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `locale`                   | 웹 채팅의 기본 언어를 나타냅니다. 네 개의 문자 코드(예: `en-US`)가 권장됩니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `renderMarkdown`           | 기본 Markdown 렌더러 개체를 변경합니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `sendTypingIndicator`      | 사용자의 입력 신호를 봇에 표시하여 사용자가 유휴 상태가 아님을 나타냅니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `store`                    | 봇에 프로그래밍 활동 추가 등에 사용할 사용자 지정 저장소를 지정합니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `styleOptions`             | 웹 채팅의 스타일 지정을 위한 사용자 지정 값을 저장하는 개체입니다. (자주 업데이트되는) 기본 스타일 옵션의 전체 목록은 [defaultStyleOptions.js](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Styles/defaultStyleOptions.js) 파일을 참조하세요.                                                                                                                                                                                                                                                                              |
| `styleSet`                 | 스타일을 재정의하는 권장되지 않는 방법입니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `userID`                   | userID를 지정합니다. `userID`를 지정하는 방법에는 속성에서 지정하거나 토큰 호출(`createDirectLine()`)을 생성할 때 토큰에서 지정하는 두 가지가 있습니다. 두 방법을 모두 사용해서 userID를 지정하는 경우 토큰 userID 속성이 사용되고, 런타임 중에 `console.warn`이 표시됩니다. 속성을 통해 `userID`를 제공하지만 `'dl'`를 앞에 추가하면(예: `'dl_1234'`), 값이 throw되고 새 `ID`가 생성됩니다. `userID`를 지정하지 않으면 기본적으로 임의 사용자 ID로 설정됩니다. 여러 사용자가 동일한 사용자 ID를 공유하는 것은 권장되지 않습니다. 사용자 상태가 공유됩니다. |
| `username`                 | 사용자 이름을 지정합니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `webSpeechPonyFillFactory` | 텍스트 음성 변환 및 음성 텍스트 변환에 사용할 Web Search 개체를 지정합니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
## <a name="browser-compatibility"></a>브라우저 호환성
웹 채팅은 Chrome, Edge 및 FireFox와 같은 최신 브라우저의 최근 버전 2개를 지원합니다.
Internet Explorer 11에서 웹 채팅이 필요한 경우 [ES5 번들 데모](https://microsoft.github.io/BotFramework-WebChat/01.b.getting-started-es5-bundle)를 참조하세요.

그러나 다음 사항에 유의하세요.
- 웹 채팅은 Internet Explorer 11 이전 버전을 지원하지 않습니다.
- 비 ES5 샘플에 표시된 사용자 지정은 Internet Explorer에서 지원되지 않습니다. IE11은 최신 브라우저가 아니기 때문에 ES6를 지원하지 않으며, 화살표 기능과 최신 프라미스를 사용하는 많은 샘플을 ES5로 수동 변환해야 합니다.  많은 앱 사용자 지정이 필요한 경우 Google Chrome 또는 Edge와 같은 최신 브라우저용 앱을 개발하는 것이 좋습니다.
- 웹 채팅은 IE11(ES5)용 샘플을 지원할 계획이 없습니다.
   - IE11에서 작동하도록 다른 샘플을 수동으로 다시 생성하려는 고객의 경우 [`babel`](https://babeljs.io/docs/en/next/babel-standalone.html) 등의 폴리필 및 트랜스파일러를 사용하여 ES6 이상에서 ES5로 코드를 변환하는 것이 좋습니다.

## <a name="how-to-test-with-web-chats-latest-bits"></a>웹 채팅의 최신 기능으로 테스트하는 방법

릴리스되지 않은 기능은 현재 MyGet 패키징을 통해서만 테스트할 수 있습니다. 

아직 릴리스되지 않은 기능 또는 버그 수정을 테스트하려는 경우 웹 채팅 패키지에서 공식적인 npmjs 피드가 아니라 웹 채팅의 일별 피드를 가리키는 것이 좋습니다.

현재, MyGet 피드를 구독하면 웹 채팅의 일별 피드에 액세스할 수 있습니다. 이 작업을 위해 프로젝트에서 레지스트리를 업데이트해야 합니다. **이 변경 내용은 취소할 수 있으며, 공식적인 릴리스 구독으로 되돌리는 방법도 지침에 포함되어 있습니다**.

### <a name="subscribe-to-latest-bits-on-mygetorg"></a>`myget.org`에서 최신 기능 구독

이 작업을 위해 패키지를 추가한 다음, 프로젝트의 레지스트리를 변경해야 할 수도 있습니다.

1. 웹 채팅 이외의 프로젝트 종속성을 추가합니다.
1. 프로젝트의 루트 디렉터리에 `.npmrc` 파일을 만듭니다.
1. 파일에 다음 줄을 추가합니다. `registry=https://botbuilder.myget.org/F/botframework-webchat/npm/`
1. 프로젝트 종속성에 웹 채팅을 추가합니다. `npm i botframework-webchat --save`
1. `package-lock.json`에서 가리키는 레지스트리가 이제 MyGet입니다. 웹 채팅 프로젝트에서 업스트림 소스 프록시를 사용할 수 있으므로, 비 MyGet 패키지가 `npmjs.com`으로 리디렉션됩니다.

### <a name="re-subscribe-to-official-release-on-npmjscom"></a>`npmjs.com`에서 공식적인 릴리스 다시 구독
다시 구독하려면 레지스트리를 다시 설정해야 합니다.

1. `.npmrc file`을 삭제합니다.
1. 루트 `package-lock.json`을 삭제합니다.
1. `node_modules` 디렉터리를 제거합니다.
1. `npm i`를 사용하여 패키지를 다시 설치합니다.
1. `package-lock.json`에서 레지스트리가 다시 https://npmjs.com/ 을 가리키는 것을 확인합니다.


## <a name="contributing"></a>참여

프로젝트를 빌드하는 방법에 대한 자세한 내용 및 끌어오기 요청의 리포지토리 지침은 [기여 페이지](https://github.com/Microsoft/BotFramework-WebChat/tree/master/.github/CONTRIBUTING.md)를 참조하세요.

이 프로젝트에는 [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/)(Microsoft 오픈 소스 준수 사항)이 적용됩니다.
자세한 내용은 [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/)(준수 사항 FAQ)를 참조하거나 [opencode@microsoft.com](mailto:opencode@microsoft.com)에 추가 질문 또는 의견을 알려주세요.

## <a name="reporting-security-issues"></a>보안 문제 보고

보안 문제 및 버그는 메일을 통해 비공개로 [secure@microsoft.com](mailto:secure@microsoft.com)의 MSRC(Microsoft 보안 대응 센터)에 보고해야 합니다. 24시간 이내에 응답을 받게 됩니다. 어떤 이유로든 응답을 받지 못한 경우, 메일을 통해 후속 조치를 취하여 처음에 보낸 메시지를 MSRC가 받았는지 확인합니다. [MSRC PGP](https://technet.microsoft.com/security/dn606155) 키를 비롯한 자세한 내용은 [Security TechCenter](https://technet.microsoft.com/security/default)에서 확인할 수 있습니다.

Copyright (c) Microsoft Corporation. All rights reserved.