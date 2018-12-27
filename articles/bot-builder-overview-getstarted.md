---
title: Bot Builder를 사용하여 봇 빌드 시작 | Microsoft Docs
description: Bot Builder를 사용하여 강력한 봇의 빌드를 시작합니다.
author: robstand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 3ee7843e64dfa95427ebcb132740eab3db281ffc
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904016"
---
# <a name="develop-bots-with-bot-builder"></a>Bot Builder를 사용하여 봇 개발



Bot Builder는 봇을 빌드하고 디버그하는 데 도움이 되는 SDK, 라이브러리, 샘플 및 도구를 제공합니다. Bot Service를 사용하여 봇을 빌드할 경우 봇은 Bot Builder SDK에서 지원됩니다. Bot Builder SDK를 사용하여 C# 또는 Node.js를 통해 처음부터 봇을 만들 수도 있습니다. Bot Builder에는 봇을 테스트하기 위한 Bot Framework Emulator 및 다른 채널에서 봇의 사용자 환경을 미리 보기 위한 채널 검사기가 포함됩니다.

원하는 프로그래밍 언어를 사용하여 봇을 빌드하려면 REST API를 사용하면 됩니다.

## <a name="bot-builder-sdk-for-net"></a>.NET용 Bot Builder SDK
.NET용 Bot Builder SDK는 C#을 사용하여 .NET 개발자가 봇을 작성할 수 있는 친숙한 방법을 제공합니다. 자유 형식 상호 작용 및 사용자가 가능한 값에서 선택하는 자세한 단계별 대화를 둘 다 처리할 수 있는 봇을 생성하기 위한 강력한 프레임워크입니다. 

[.NET 빠른 시작](~/dotnet/bot-builder-dotnet-quickstart.md)에서는 .Net용 Bot Builder SDK를 사용하여 봇을 만드는 과정을 안내합니다.

.NET용 Bot Builder SDK는 [NuGet 패키지](https://www.nuget.org/packages/Microsoft.Bot.Builder/)로 제공됩니다.

> [!IMPORTANT]
> .NET용 Bot Builder SDK에는 .NET Framework 4.6 이상이 필요합니다. 하위 버전의 .NET Framework를 대상으로 하는 기존 프로젝트에 SDK를 추가하는 경우 먼저 .NET Framework 4.6을 대상으로 지정하도록 프로젝트를 업데이트해야 합니다.

기존 Visual Studio 프로젝트에 SDK를 설치하려면 다음 단계를 수행합니다.

1. **솔루션 탐색기**에서 프로젝트 이름을 마우스 오른쪽 단추로 클릭하고 **NuGet 패키지 관리...** 를 선택합니다.
2. **찾아보기** 탭에서 검색 상자에 “Microsoft.Bot.Builder”를 입력합니다.
3. 결과 목록에서 **Microsoft.Bot.Builder**를 클릭하고, **설치**를 클릭한 다음, 변경 내용을 적용합니다.

GitHub에서 .NET용 Bot Builder SDK [원본 코드](https://github.com/Microsoft/BotBuilder/tree/master/CSharp)를 다운로드할 수도 있습니다.

### <a name="visual-studio-project-templates"></a>Visual Studio 프로젝트 템플릿
Visual Studio 프로젝트 템플릿을 다운로드하여 봇을 빠르게 개발합니다.

* C#을 사용하여 봇을 개발하기 위한 [Visual Studio용 봇 템플릿][bot-template]
* C#을 사용하여 Cortana 기술을 개발하기 위한 [Visual Studio용 Cortana 기술 템플릿][cortana-template]

> [!TIP]
> Visual Studio 2017 프로젝트 템플릿을 설치하는 방법을 <a href="/visualstudio/ide/how-to-locate-and-organize-project-and-item-templates" target="_blank">자세히 알아봅니다</a>.

## <a name="bot-builder-sdk-for-nodejs"></a>Node.js용 Bot Builder SDK
Node.js용 Bot Builder SDK는 Node.js 개발자가 봇을 작성할 수 있는 친숙한 방법을 제공합니다. 이를 사용하여 간단한 프롬프트에서 자유 형식 대화까지 다양한 대화형 사용자 인터페이스를 빌드할 수 있습니다. Node.js용 Bot Builder SDK는 웹 서비스를 빌드하기 위한 인기 있는 프레임워크인 restify를 사용하여 봇의 웹 서버를 만듭니다. 이 SDK는 Express와도 호환됩니다. 

[Node.js 빠른 시작](~/nodejs/bot-builder-nodejs-quickstart.md)에서는 Node.js용 Bot Builder SDK를 사용하여 봇을 만드는 과정을 안내합니다. 

Node.js용 Bot Builder SDK는 npm 패키지로 제공됩니다. Node.js용 Bot Builder SDK와 관련 종속 항목을 설치하려면 먼저 봇에 대한 폴더를 만들고 해당 폴더로 이동하여 다음 **npm** 명령을 실행합니다.

```nodejs
npm init
```

그러고 나서 다음 **npm** 명령을 실행하여 Node.js용 Bot Builder SDK 및 <a href="http://restify.com/" target="_blank">Restify</a> 모듈을 설치합니다.

```nodejs
npm install --save botbuilder
npm install --save restify
```

GitHub에서 Node.js용 Bot Builder SDK [원본 코드](https://github.com/Microsoft/BotBuilder/tree/master/Node)를 다운로드할 수도 있습니다.

## <a name="rest-api"></a>REST API

Bot Framework REST API를 사용하여 모든 프로그래밍 언어로 봇을 만들 수 있습니다. [REST 빠른 시작](rest-api/bot-framework-rest-connector-quickstart.md)에서는 REST를 사용하여 봇을 만드는 과정을 안내합니다.

Bot Framework에는 다음 세 가지 REST API가 있습니다.

 - [Bot Connector REST API][connectorAPI]를 사용하면 봇이 [Bot Framework 포털](https://dev.botframework.com/)에 구성된 채널에 메시지를 보내고 받을 수 있습니다. 

- [Bot State REST API][stateAPI]를 사용하면 봇이 Bot Connector REST API를 통해 수행된 대화와 연결된 상태를 저장하고 검색할 수 있습니다.

- [직접 회선 REST API][directLineAPI]를 사용하면 클라이언트 응용 프로그램, 웹 채팅 컨트롤 또는 모바일 앱과 같은 고유한 응용 프로그램을 단일 봇에 직접 연결할 수 있습니다.

## <a name="bot-framework-emulator"></a>Bot Framework Emulator
Bot Framework Emulator는 로컬 또는 원격으로 봇을 테스트 및 디버그할 수 있는 데스크톱 애플리케이션입니다. 에뮬레이터를 사용하면 봇과 채팅하고 봇이 보내고 받는 메시지를 검사할 수 있습니다. [Bot Framework Emulator를 자세히 알아보고](~/bot-service-debug-emulator.md) 사용 중인 플랫폼용 [에뮬레이터를 다운로드](http://emulator.botframework.com)하세요.

## <a name="bot-framework-channel-inspector"></a>Bot Framework 채널 검사기
Bot Framework [채널 검사기](bot-service-channel-inspector.md)를 사용하여 다양한 채널에서 봇 기능이 나타나는 모양을 빠르게 확인합니다.

[bot-template]: http://aka.ms/bf-bc-vstemplate
[cortana-template]: https://aka.ms/bf-cortanaskill-template


[connectorAPI]: https://docs.botframework.com/en-us/restapi/connector/#navtitle
 
[stateAPI]: https://docs.botframework.com/en-us/restapi/state/#navtitle

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle
