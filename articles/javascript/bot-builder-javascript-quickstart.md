---
title: JavaScript용 Bot Builder SDK를 사용하여 봇 만들기 | Microsoft Docs
description: JavaScript용 Bot Builder SDK를 사용하여 봇을 빠르게 만듭니다.
keywords: 빠른 시작, bot builder sdk, 시작하기
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 07/12/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 21a9aa5b1d108b5d03b108278a81229e16b5bc99
ms.sourcegitcommit: a2f3d87c0f252e876b3e63d75047ad1e7e110b47
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/25/2018
ms.locfileid: "42928129"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-v4-preview-for-javascript"></a>JavaScript용 Bot Builder SDK v4(미리 보기)를 사용하여 봇 만들기

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

이 빠른 시작은 Yeoman Bot Builder 생성기 및 JavaScript용 Yeoman Bot Builder SDK를 사용하여 봇을 빌드한 다음, Bot Framework Emulator로 테스트하는 방법을 안내합니다. [Microsoft Bot Builder SDK v4](https://github.com/Microsoft/botbuilder-js)를 기반으로 합니다.

## <a name="prerequisites"></a>필수 조건

- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.js](https://nodejs.org/en/)
- [Yeoman](http://yeoman.io/), 생성기를 사용하여 사용자를 위한 봇을 만들 수 있음
- [봇 에뮬레이터](https://github.com/Microsoft/BotFramework-Emulator)
- [Restify](http://restify.com/) 및 JavaScript의 비동기 프로그래밍에 대한 정보

> [!NOTE]
> 일부 설치의 경우 Restify에 대한 설치 단계는 node-gyp 관련 오류를 제공합니다.
> 이 경우에는 `npm install -g windows-build-tools`를 실행해 보세요.

JavaScript용 Bot Builder SDK는 특별한 `@preview` 태그를 사용하여 NPM에서 설치될 수 있는 [패키지](https://github.com/Microsoft/botbuilder-js/tree/master/libraries) 시리즈로 구성되어 있습니다.

## <a name="create-a-bot"></a>봇 만들기

관리자 권한 명령 프롬프트를 열고, 디렉터리를 만들고, 봇용 패키지를 초기화합니다.

```bash
md myJsBots
cd myJsBots
```

다음으로, JavaScript용 생성기 및 Yeoman을 설치합니다.

```bash
npm install -g yo
npm install -g generator-botbuilder@preview
```

그런 다음, 생성기를 사용하여 echo 봇을 만듭니다.

```bash
yo botbuilder
```

Yeoman은 봇을 만드는 데 사용할 일부 정보에 대한 메시지를 표시합니다.

- 봇의 이름을 입력합니다.
- 설명을 입력합니다.
- 봇용 언어, `JavaScript` 또는 `TypeScript`를 선택합니다.
- 사용할 템플릿을 선택합니다. 현재 `Echo`는 템플릿에 불과하지만 곧 다른 것이 추가될 예정입니다.

Yeoman은 새 폴더에서 봇을 만듭니다.

## <a name="explore-code"></a>코드 탐색

새로 만든 봇 폴더를 열면 `app.js` 파일이 표시됩니다. 이 `app.js` 파일에는 봇 앱을 실행하는 데 필요한 모든 코드가 포함됩니다. 이 파일에는 무엇을 입력하든 되돌려 보낼 뿐만 아니라 카운터를 증분하는 echo 봇이 포함됩니다.

다음 코드에서 대화 상태 미들웨어는 메모리 내 저장소를 사용합니다. 저장소에 상태 개체를 작성하고 읽습니다. 개수 변수는 봇에 보내는 메시지 수를 추적합니다. 비슷한 기법을 사용하여 설정 사이의 상태를 유지 관리할 수 있습니다.

**app.js**
```javascript
// Packages are installed for you
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID,
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);
```

다음 코드는 들어오는 요청에 대해 수신 대기하고 사용자에게 회신을 보내기 전에 들어오는 활동 유형을 확인합니다.

```javascript
// Listen for incoming requests
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        // This bot is only handling Messages
        if (context.activity.type === 'message') {

            // Get the conversation state
            const state = conversationState.get(context);

            // If state.count is undefined set it to 0, otherwise increment it by 1
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            // Echo back to the user whatever they typed.
            return context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            // Echo back the type of activity the bot detected if not of type message
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

## <a name="start-your-bot"></a>봇 시작

디렉터리를 봇용으로 만든 디렉터리로 변경하고 시작합니다.

```bash
cd <bot directory>
node app.js
```

## <a name="start-the-emulator-and-connect-your-bot"></a>에뮬레이터 시작 및 봇 연결

이때 봇은 로컬에서 실행됩니다. 에뮬레이터를 시작한 다음, 에뮬레이터에서 봇에 연결합니다.

1. 에뮬레이터 “시작” 탭에서 **새 봇 구성 만들기** 링크를 클릭합니다.
1. **봇 이름**을 입력하고 봇 코드에 대한 디렉터리 경로를 입력합니다. 봇 구성 파일은 이 경로에 저장됩니다.
1. **끝점 URL** 필드에 `http://localhost:port-number/api/messages`를 입력합니다. 여기서 *port-number*는 응용 프로그램이 실행되는 브라우저에 표시된 포트 번호와 일치합니다.
1. **연결**을 클릭하여 봇에 연결합니다. **Microsoft 앱 ID** 및 **Microsoft 앱 암호**를 지정하지 않아도 됩니다. 지금은 이러한 필드를 비워 둘 수 있습니다. 봇을 등록하면 나중에 이 정보를 가져올 수 있습니다.

"Hi"를 봇에 보내면 봇은 '0: "Hi"라고 했습니다'로 메시지에 응답합니다.

## <a name="next-steps"></a>다음 단계

다음으로, [Azure에 봇을 배포](../bot-builder-howto-deploy-azure.md)하거나 이제 봇 및 작동 방식을 설명하는 개념으로 이동합니다.

> [!div class="nextstepaction"]
> [기본 봇 개념](../v4sdk/bot-builder-basics.md)
