---
title: JavaScript용 Bot Builder SDK를 사용하여 봇 만들기 | Microsoft Docs
description: JavaScript용 Bot Builder SDK를 사용하여 봇을 빠르게 만듭니다.
keywords: 빠른 시작, bot builder sdk, 시작하기
author: jonathanfingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/30/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1804f7bb6a19fbe0cfced49cffa4e9d6d77005eb
ms.sourcegitcommit: f7a8f05fc05ff4a7212a437d540485bf68831604
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/21/2018
ms.locfileid: "53735933"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-javascript"></a>JavaScript용 Bot Builder SDK를 사용하여 봇 만들기

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

이 빠른 시작에서는 Yeoman Bot Builder 생성기 및 JavaScript용 Yeoman Bot Builder SDK를 사용하여 단일 봇을 빌드한 다음, Bot Framework Emulator를 사용하여 테스트하는 방법을 안내합니다.

## <a name="prerequisites"></a>필수 조건

- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.JS](https://nodejs.org/)
- [Yeoman](http://yeoman.io/) - 생성기를 사용하여 사용자를 위한 봇을 만듭니다.
- [git](https://git-scm.com/)
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator)
- [Restify](http://restify.com/) 및 JavaScript의 비동기 프로그래밍에 대한 정보

> [!NOTE]
> 일부 설치의 경우 Restify에 대한 설치 단계는 node-gyp 관련 오류를 제공합니다.
> 이 경우 관리자 권한으로 다음 명령을 실행해 보세요.
> ```bash
> npm install -g windows-build-tools
> ```

## <a name="create-a-bot"></a>봇 만들기

봇을 만들고 해당 패키지를 초기화하려면 다음을 수행합니다.

1. 터미널을 열거나 관리자 권한으로 명령 프롬프트를 엽니다.
1. JavaScript 봇에 대한 디렉터리가 아직 없는 경우 해당 디렉터리를 만든 후 이 디렉터리로 변경합니다. (이 자습서에서는 하나의 봇만 만들지만, 일반적으로는 JavaScript 봇에 대한 디렉터리를 만들고 있습니다.)

   ```bash
   md myJsBots
   cd myJsBots
   ```

1. npm이 최신 버전인지 확인합니다.

   ```bash
   npm install -g npm
   ```

1. 다음으로, JavaScript용 생성기 및 Yeoman을 설치합니다.

   ```bash
   npm install -g yo generator-botbuilder
   ```

1. 그런 다음, 생성기를 사용하여 echo 봇을 만듭니다.

   ```bash
   yo botbuilder
   ```

Yeoman은 봇을 만드는 데 사용할 일부 정보에 대한 메시지를 표시합니다. 이 자습서에서는 기본값을 사용합니다.

- 봇의 이름을 입력합니다. (myChatBot)
- 설명을 입력합니다. (Microsoft Bot Framework의 핵심 기능을 설명합니다.)
- 봇에 대한 언어를 선택합니다. (JavaScript)
- 사용할 템플릿을 선택합니다. (에코)

템플릿 덕분에 프로젝트에는 이 빠른 시작에서 봇을 만드는 데 필요한 모든 코드가 포함되어 있습니다. 실제로 추가 코드를 작성할 필요가 없습니다.

> [!NOTE]
> `Basic` 봇을 만들려면 LUIS 언어 모델이 필요합니다. [luis.ai](https://www.luis.ai)에서 만들 수 있습니다. 모델을 만든 후 .bot 파일을 업데이트합니다. 봇 파일은 [이것](../v4sdk/bot-builder-service-file.md)과 비슷합니다.

## <a name="start-your-bot"></a>봇 시작

터미널 또는 명령 프롬프트에서 디렉터리를 봇에 대해 만든 디렉터리로 변경하고 `npm start`를 사용하여 시작합니다. 이때 봇은 로컬에서 실행됩니다.

## <a name="start-the-emulator-and-connect-your-bot"></a>Emulator 시작 및 봇 연결

1. Bot Framework Emulator를 시작합니다.
2. 에뮬레이터 "시작" 탭에서 **봇 열기** 링크를 클릭합니다.
3. 프로젝트를 만든 디렉터리에 있는 .bot 파일을 선택합니다.

봇에 메시지를 보내면 봇이 메시지를 통해 다시 응답하게 됩니다.
![에뮬레이터 실행](../media/emulator-v4/js-quickstart.png)

## <a name="additional-resources"></a>추가 리소스

원격으로 호스팅되는 봇에 연결하는 방법은 [터널링(ngrok)](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-(ngrok))을 참조하세요.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [봇 작동 방식](../v4sdk/bot-builder-basics.md)
