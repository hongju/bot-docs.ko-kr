---
title: 웹 채팅 사용자 지정 | Microsoft Docs
description: Bot Framework 웹 채팅을 사용자 지정하는 방법을 알아봅니다.
keywords: bot framework, 웹 채팅, 채팅, 샘플, react, 참조
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 06/07/2019
ms.openlocfilehash: 80d16cda7bb51aecdbe0406d4d2b07a6c6275182
ms.sourcegitcommit: abd5b8c89e0928e03e5928b64c160c92cce03d70
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/21/2019
ms.locfileid: "67313186"
---
# <a name="web-chat-customization"></a>웹 채팅 사용자 지정

이 문서에서는 봇에 맞게 웹 채팅 샘플을 사용자 지정하는 방법을 자세히 설명합니다.

## <a name="integrate-web-chat-into-your-website"></a>웹 사이트에 웹 채팅 통합

웹 사이트에 웹 채팅 컨트롤을 통합하려면 [개요 페이지](bot-builder-webchat-overview.md)의 지침을 따르세요.

## <a name="customizing-styles"></a>스타일 사용자 지정

최신 버전의 웹 채팅 컨트롤은 다양한 사용자 지정 옵션을 제공합니다. 색, 크기, 요소 배치를 변경하고, 사용자 지정 요소를 추가하며, 호스팅 웹 페이지를 조작할 수 있습니다. 다음은 웹 채팅 UI의 이러한 요소를 사용자 지정하는 방법의 몇 가지 예입니다.

웹 채팅에서 쉽게 수정할 수 있는 모든 설정의 전체 목록은 [`defaultStyleOptions.js` 파일](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Styles/defaultStyleOptions.js)에서 확인할 수 있습니다.

해당 설정은 [glamor](https://github.com/threepointone/glamor)로 향상된 CSS 규칙 세트인 _스타일 세트_를 생성합니다. 스타일 세트에 생성되는 CSS 스타일의 전체 목록은 [ `createStyleSet.js` 파일](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Styles/createStyleSet.js)에서 확인할 수 있습니다.

## <a name="set-the-size-of-the-web-chat-container"></a>웹 채팅 컨테이너의 크기 설정

이제 `styleSetOptions`를 사용하여 웹 채팅 컨테이너의 크기를 조정할 수 있습니다. 다음 예제에서는 웹 채팅 컨테이너(흰색 배경이 있는 섹션)를 표시하기 위해 `body`의 배경색이 `paleturquoise`입니다.

```js
…
<head>
  <style>
    html, body { height: 100% }
    body {
      margin: 0;
      background-color: paleturquoise;
    }

    #webchat {
      height: 100%;
      width: 100%;
    }
  </style>
</head>
<body>
  <div id="webchat" role="main"></div>
  <script>
    (async function () {
    window.WebChat.renderWebChat({
      directLine: window.WebChat.createDirectLine({ token }),
      styleOptions: {
        rootHeight: '100%',
        rootWidth: '50%'
      }
    }, document.getElementById('webchat'));
    })()
  </script>
…
```

결과는 다음과 같습니다.

<img alt="Web Chat with root height and root width set" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/rootHeightWidth.png" width="600"/>

## <a name="change-font-or-color"></a>글꼴 또는 색 변경

채팅 거품 내에서 사용되는 기본 배경색과 글꼴 대신, 대상 웹 페이지의 스타일에 맞게 배경색과 글꼴을 사용자 지정할 수 있습니다. 아래 코드 조각을 사용하면 사용자 및 봇의 메시지 배경색을 변경할 수 있습니다.

<img alt="Screenshot with custom style options" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-custom-style-options.png" width="396" />

몇 가지 간단한 스타일을 지정해야 하는 경우 `styleOptions`를 통해 설정할 수 있습니다. 스타일 옵션은 직접 수정할 수 있는 미리 정의된 스타일 세트로, 웹 채팅은 스타일 옵션을 기준으로 전체 스타일시트를 컴퓨팅합니다.

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         const styleOptions = {
            bubbleBackground: 'rgba(0, 0, 255, .1)',
            bubbleFromUserBackground: 'rgba(0, 255, 0, .1)'
         };

         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  secret: 'YOUR_BOT_SECRET'
               }),

               // Passing 'styleOptions' when rendering Web Chat
               styleOptions
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

## <a name="change-the-css-manually"></a>수동으로 CSS 변경

색 외에도 메시지를 렌더링하는 데 사용되는 글꼴을 수정할 수 있습니다.

<img alt="Screenshot with custom style set" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-custom-style-set.png" width="396" />

자세한 스타일을 지정하려는 경우 CSS 규칙을 직접 설정하여 스타일 세트를 수동으로 수정할 수도 있습니다.

> CSS 규칙은 DOM 트리 구조에 긴밀하게 결합되어 있으므로, 최신 버전의 웹 채팅에서 사용하기 위해 해당 규칙을 업데이트해야 할 수 있습니다.

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         // "styleSet" is a set of CSS rules which are generated from "styleOptions"
         const styleSet = window.WebChat.createStyleSet({
            bubbleBackground: 'rgba(0, 0, 255, .1)',
            bubbleFromUserBackground: 'rgba(0, 255, 0, .1)'
         });

         // After generated, you can modify the CSS rules
         styleSet.textContent = {
            ...styleSet.textContent,
            fontFamily: "'Comic Sans MS', 'Arial', sans-serif",
            fontWeight: 'bold'
         };

         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  secret: 'YOUR_BOT_SECRET'
               }),

               // Passing 'styleSet' when rendering Web Chat
               styleSet
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

## <a name="change-the-avatar-of-the-bot-within-the-dialog-box"></a>대화 상자 내에서 봇의 아바타 변경

최신 웹 채팅은 아바타를 지원합니다. `botAvatarInitials` 및 `userAvatarInitials` 속성을 사용하여 아바타를 사용자 지정할 수 있습니다.

<img alt="Screenshot with avatar initials" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-avatar-initials.png" width="396" />

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
                  secret: 'YOUR_BOT_SECRET'
               }),

               // Passing avatar initials when rendering Web Chat
               botAvatarInitials: 'BF',
               userAvatarInitials: 'WC'
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

`renderWebChat` 코드 안에 `botAvatarInitials` 및 `userAvatarInitials`를 추가했습니다.

```js
botAvatarInitials: 'BF',
userAvatarInitials: 'WC'
```

`botAvatarInitials`는 아바타 내부 텍스트를 왼쪽에 설정합니다. false 값으로 설정하면 봇 쪽의 아바타가 숨겨집니다. 반면, `userAvatarInitials`는 아바타 텍스트를 오른쪽에 설정합니다.

## <a name="custom-rendering-activity-or-attachment"></a>활동 또는 첨부 파일 사용자 지정 렌더링

최신 버전의 웹 채팅에서는 웹 채팅이 기본적으로 지원하지 않는 활동이나 첨부 파일을 렌더링할 수도 있습니다. 활동 및 첨부 파일 렌더링은 [Redux 미들웨어](https://redux.js.org/api/applymiddleware)를 따라 모델링된 사용자 지정 가능한 파이프라인을 통해 전송됩니다. 이 파이프라인은 다음 작업을 쉽게 수행할 수 있을 만큼 유연성이 있습니다.

-  기존 활동/첨부 파일 데코레이트
-  새 활동/첨부 파일 추가
-  기존 활동/첨부 파일 바꾸기(또는 제거)
-  미들웨어 데이지 체인

### <a name="show-github-repository-as-an-attachment"></a>GitHub 리포지토리를 첨부 파일로 표시

GitHub 리포지토리 카드 데크를 표시하려는 경우, GitHub 리포지토리의 새 React 구성 요소를 만들어 첨부 파일의 미들웨어로 추가할 수 있습니다.

<img alt="Screenshot with custom GitHub repository attachment" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-custom-github-repository-attachment.png" width="396" />

```jsx
import ReactWebChat from 'botframework-webchat';
import ReactDOM from 'react-dom';

// Create a new React component that accept render a GitHub repository attachment
const GitHubRepositoryAttachment = props => (
   <div
      style={{
         fontFamily: "'Calibri', 'Helvetica Neue', Arial, sans-serif",
         margin: 20,
         textAlign: 'center'
      }}
   >
      <svg
         height="64"
         viewBox="0 0 16 16"
         version="1.1"
         width="64"
         aria-hidden="true"
      >
         <path
            fillRule="evenodd"
            d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0 0 16 8c0-4.42-3.58-8-8-8z"
         />
      </svg>
      <p>
         <a
            href={`https://github.com/${encodeURI(props.owner)}/${encodeURI(
               props.repo
            )}`}
            target="_blank"
         >
            {props.owner}/<br />
            {props.repo}
         </a>
      </p>
   </div>
);

// Creating a new middleware pipeline that will render <GitHubRepositoryAttachment> for specific type of attachment
const attachmentMiddleware = () => next => card => {
   switch (card.attachment.contentType) {
      case 'application/vnd.microsoft.botframework.samples.github-repository':
         return (
            <GitHubRepositoryAttachment
               owner={card.attachment.content.owner}
               repo={card.attachment.content.repo}
            />
         );

      default:
         return next(card);
   }
};

ReactDOM.render(
   <ReactWebChat
      // Prepending the new middleware pipeline
      attachmentMiddleware={attachmentMiddleware}
      directLine={window.WebChat.createDirectLine({ token })}
   />,
   document.getElementById('webchat')
);
```

전체 샘플은 [customization-card-components 샘플](https://github.com/microsoft/BotFramework-WebChat/tree/master/samples/10.a.customization-card-components)에서 확인할 수 있습니다.

이 샘플에서 `GitHubRepositoryAttachment`라는 새 React 구성 요소를 추가합니다.

```jsx
const GitHubRepositoryAttachment = props => (
   <div
      style={{
         fontFamily: "'Calibri', 'Helvetica Neue', Arial, sans-serif",
         margin: 20,
         textAlign: 'center'
      }}
   >
      <svg
         height="64"
         viewBox="0 0 16 16"
         version="1.1"
         width="64"
         aria-hidden="true"
      >
         <path
            fillRule="evenodd"
            d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0 0 16 8c0-4.42-3.58-8-8-8z"
         />
      </svg>
      <p>
         <a
            href={`https://github.com/${encodeURI(props.owner)}/${encodeURI(
               props.repo
            )}`}
            target="_blank"
         >
            {props.owner}/<br />
            {props.repo}
         </a>
      </p>
   </div>
);
```

그런 다음, 봇이 `application/vnd.microsoft.botframework.samples.github-repository` 콘텐츠 형식의 첨부 파일을 보낼 때 새 React 구성 요소를 렌더링할 미들웨어를 만듭니다. 이 작업을 수행하지 않으면 봇이 `next(card)`를 호출하여 미들웨어를 계속합니다.

```jsx
const attachmentMiddleware = () => next => card => {
   switch (card.attachment.contentType) {
      case 'application/vnd.microsoft.botframework.samples.github-repository':
         return (
            <GitHubRepositoryAttachment
               owner={card.attachment.content.owner}
               repo={card.attachment.content.repo}
            />
         );

      default:
         return next(card);
   }
};
```

봇에서 보내는 활동은 다음과 같습니다.

```json
{
   "type": "message",
   "from": {
      "role": "bot"
   },
   "attachmentLayout": "carousel",
   "attachments": [
      {
         "contentType": "application/vnd.microsoft.botframework.samples.github-repository",
         "content": {
            "owner": "Microsoft",
            "repo": "BotFramework-WebChat"
         }
      },
      {
         "contentType": "application/vnd.microsoft.botframework.samples.github-repository",
         "content": {
            "owner": "Microsoft",
            "repo": "BotFramework-Emulator"
         }
      },
      {
         "contentType": "application/vnd.microsoft.botframework.samples.github-repository",
         "content": {
            "owner": "Microsoft",
            "repo": "BotFramework-DirectLineJS"
         }
      }
   ]
}
```
