---
title: JavaScript용 Bot Builder SDK를 사용하여 봇 만들기 | Microsoft Docs
description: JavaScript용 Bot Builder SDK를 사용하여 봇을 빠르게 만듭니다.
keywords: 빠른 시작, bot builder sdk, 시작하기
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/15/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: aa13889cea2a26bf094a919f5d05905d65f7661f
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998862"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-javascript"></a>JavaScript용 Bot Builder SDK를 사용하여 봇 만들기

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

이 빠른 시작은 Yeoman Bot Builder 생성기 및 JavaScript용 Yeoman Bot Builder SDK를 사용하여 봇을 빌드한 다음, Bot Framework Emulator로 테스트하는 방법을 안내합니다. 

## <a name="prerequisites"></a>필수 조건

- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.JS](https://nodejs.org/)
- [Yeoman](http://yeoman.io/), 생성기를 사용하여 사용자를 위한 봇을 만들 수 있음
- [git](https://git-scm.com/)
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator)
- [Restify](http://restify.com/) 및 JavaScript의 비동기 프로그래밍에 대한 정보

> [!NOTE]
> 일부 설치의 경우 Restify에 대한 설치 단계는 node-gyp 관련 오류를 제공합니다.
> 이 경우에는 `npm install -g windows-build-tools`를 실행해 보세요.

## <a name="create-a-bot"></a>봇 만들기

관리자 권한 명령 프롬프트를 열고, 디렉터리를 만들고, 봇용 패키지를 초기화합니다.

```bash
md myJsBots
cd myJsBots
```

npm이 최신 버전인지 확인합니다.
```bash
npm i npm
```

다음으로, JavaScript용 생성기 및 Yeoman을 설치합니다.

```bash
npm install -g yo
npm install -g generator-botbuilder
```

그런 다음, 생성기를 사용하여 echo 봇을 만듭니다.

```bash
yo botbuilder
```

Yeoman은 봇을 만드는 데 사용할 일부 정보에 대한 메시지를 표시합니다.

- 봇의 이름을 입력합니다.
- 설명을 입력합니다.
- 봇용 언어, `JavaScript` 또는 `TypeScript`를 선택합니다.
- `Echo` 템플릿을 선택합니다.

템플릿 덕분에 프로젝트에는 이 빠른 시작에서 봇을 만드는 데 필요한 모든 코드가 포함되어 있습니다. 실제로 추가 코드를 작성할 필요가 없습니다.

> [!NOTE]
> 기본 봇에는 LUIS 언어 모델이 필요합니다. [luis.ai](https://www.luis.ai)에서 만들 수 있습니다. 모델을 만든 후 .bot 파일을 업데이트합니다. 봇 파일은 [이것](../v4sdk/bot-builder-service-file.md)과 비슷합니다. 

## <a name="start-your-bot"></a>봇 시작

Powershell/Bash에서 디렉터리를 봇용으로 만든 디렉터리로 변경하고 `npm start`로 시작합니다. 이때 봇은 로컬에서 실행됩니다.

## <a name="start-the-emulator-and-connect-your-bot"></a>에뮬레이터 시작 및 봇 연결
1. 에뮬레이터를 시작합니다.
2. 에뮬레이터 "시작" 탭에서 **봇 열기** 링크를 클릭합니다.
3. 프로젝트를 만든 디렉터리에 있는 .bot 파일을 선택합니다.

봇에 메시지를 보내면 봇이 메시지를 통해 다시 응답하게 됩니다.
![에뮬레이터 실행](../media/emulator-v4/emulator-running.png)

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [봇 작동 방식](../v4sdk/bot-builder-basics.md) 
