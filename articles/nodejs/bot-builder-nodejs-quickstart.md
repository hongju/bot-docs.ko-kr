---
title: Node.js용 Bot Builder SDK를 사용하여 봇 만들기 | Microsoft Docs
description: 강력한 봇 생성 프레임워크인 Node.js용 Bot Builder SDK를 사용하여 봇을 만듭니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6159997ec5ea3dbd3188ba2ea4b6207b5d9db08f
ms.sourcegitcommit: 97bb24f15041caccef4ca5736aa14f144881e0c6
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/06/2018
ms.locfileid: "39567552"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-nodejs"></a>Node.js용 Bot Builder SDK를 사용하여 봇 만들기

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-quickstart.md)
> - [Node.js](../nodejs/bot-builder-nodejs-quickstart.md)
> - [Bot Service](../bot-service-quickstart.md)
> - [REST](../rest-api/bot-framework-rest-connector-quickstart.md)

Node.js용 Bot Builder SDK는 봇 개발을 위한 프레임워크입니다. Express & restify와 같이 JavaScript 개발자에게 봇을 작성하는 친숙한 방법을 제공하기 위한 사용 및 모델링이 편리한 프레임워크입니다.

이 자습서에서는 Node.js용 Bot Builder SDK를 사용하여 봇을 빌드하는 과정을 안내합니다. 콘솔 창 및 Bot Framework Emulator에서 봇을 테스트할 수 있습니다.

## <a name="prerequisites"></a>필수 조건
먼저 다음 필수 태스크를 완료합니다.

1. [Node.js](https://nodejs.org)를 설치합니다.
2. 봇용 폴더를 만듭니다.
3. 명령 프롬프트 또는 터미널에서, 방금 만든 폴더로 이동합니다.
4. 다음 **npm** 명령을 실행합니다.

```nodejs
npm init
```

화면 지시에 따라 봇에 대한 정보를 입력하면 npm이 사용자가 제공한 정보를 포함하는 **package.json** 파일을 만듭니다. 

## <a name="install-the-sdk"></a>SDK 설치
그런 후에 다음 **npm** 명령을 실행하여 Node.js용 Bot Builder SDK를 설치합니다.

```nodejs
npm install --save botbuilder
```

SDK가 설치되어 있으면 첫 번째 봇을 작성할 준비가 이미 끝난 것입니다.

첫 번째 봇에 대해 사용자 입력을 간단히 에코하는 봇을 만듭니다. 봇을 만들려면 다음 단계를 따르세요.

1. 봇에 대해 이전에 만든 폴더에서 **app.js**라는 새 파일을 만듭니다.
2. 원하는 텍스트 편집기 또는 IDE에서 **app.js**를 엽니다. 파일에 다음 코드를 추가합니다. 

   [!code-javascript[consolebot code sample Node.js](../includes/code/node-getstarted.js#consolebot)]

3. 파일을 저장합니다. 이제 봇을 실행하고 테스트할 준비가 되었습니다.

### <a name="start-your-bot"></a>봇 시작

콘솔 창에서 봇 디렉터리로 이동한 후 봇을 시작합니다.

```nodejs
node app.js
```

이제 봇이 로컬에서 실행되고 있습니다. 콘솔 창에서 몇몇 메시지를 입력하여 봇을 테스트합니다.
*"You said:"* 가 접미사로 붙이고 사용자 메시지를 에코하여 사용자가 보낸 각 메시지에 봇이 응답하는 것을 확인할 수 있습니다.

## <a name="install-restify"></a>Restify 설치

콘솔 봇은 적절한 텍스트 기반 클라이언트이지만, Bot Framework 채널을 사용하기 위해(또는 에뮬레이터에서 봇 실행) 봇을 API 끝점에서 실행해야 합니다. 다음 **npm** 명령을 실행하여 <a href="http://restify.com/" target="_blank">restify</a>를 설치합니다.

```nodejs
npm install --save restify
```

Restify를 설치했으면 봇을 변경할 준비가 되었습니다.

## <a name="edit-your-bot"></a>봇 편집

**app.js** 파일을 일부 변경해야 합니다. 

1. `restify` 모듈을 요구하는 줄을 추가합니다.
2. `ChatConnector`를 `ConsoleConnector`로 변경합니다.
3. Microsoft 앱 ID 및 앱 암호를 포함합니다.
4. 커넥터가 API 끝점에서 수신하도록 합니다.

   [!code-javascript[echobot code sample Node.js](../includes/code/node-getstarted.js#echobot)]

5. 파일을 저장합니다. 이제 봇을 실행하고 에뮬레이터에서 테스트할 준비가 되었습니다.

> [!NOTE] 
> Bot Framework Emulator에서 봇을 실행하기 위해 **Microsoft 앱 ID** 또는 **Microsoft 앱 암호**가 필요하지는 않습니다.

## <a name="test-your-bot"></a>봇 테스트
그런 다음, [Bot Framework Emulator](../bot-service-debug-emulator.md)로 봇을 테스트하여 실제로 작동하는 모습을 확인합니다. 에뮬레이터는 localhost에서 또는 터널을 통해 원격으로 실행되는 봇을 테스트하고 디버그하는 데스크톱 애플리케이션입니다.

먼저 에뮬레이터를 [다운로드](https://emulator.botframework.com)하고 설치해야 합니다. 다운로드가 완료된 후 실행 파일을 시작하고 설치 프로세스를 완료합니다.

### <a name="start-your-bot"></a>봇 시작

에뮬레이터를 설치한 후에 콘솔 창에서 봇 디렉터리로 이동하고 봇을 시작합니다.

```nodejs
node app.js
```
   
이제 봇이 로컬에서 실행되고 있습니다.

### <a name="start-the-emulator-and-connect-your-bot"></a>에뮬레이터 시작 및 봇 연결
봇을 시작한 후에 에뮬레이터를 시작하고 봇을 연결합니다.

1. 에뮬레이터 창에서 **새 봇 구성 만들기** 링크를 클릭합니다. 

2. 주소 표시줄에 `http://localhost:port-number/api/messages`를 입력합니다. 여기서 *port-number*는 애플리케이션이 실행 중인 브라우저에 표시되는 포트 번호와 일치합니다.

3. 저장 및 연결을 클릭합니다. Microsoft 앱 ID 및 Microsoft 앱 암호를 지정하지 않아도 됩니다. 지금은 이러한 필드를 비워 둘 수 있습니다. 나중에 봇을 등록할 때 이 정보를 가져올 수 있습니다.

### <a name="try-out-your-bot"></a>봇 사용해보기

봇을 로컬로 실행 중이며 에뮬레이터에 연결되어 있으므로 에뮬레이터에서 몇 가지 메시지를 입력하여 봇을 사용해봅니다.
*"You said:"* 가 접미사로 붙이고 사용자 메시지를 에코하여 사용자가 보낸 각 메시지에 봇이 응답하는 것을 확인할 수 있습니다.

Node.js용 Bot Builder SDK를 사용하여 첫 번째 봇을 만들었습니다.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [Node.js용 Bot Builder SDK](bot-builder-nodejs-overview.md)
