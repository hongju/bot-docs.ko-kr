---
title: 새로운 기능 | Microsoft Docs
description: Bot Framework의 새로운 기능에 대해 알아봅니다.
keywords: bot framework, azure bot service
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 07/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1bbea2f12af976a9d967e7c62baf416b8938f8aa
ms.sourcegitcommit: b053c0ca7f2e9e60679f7e82e583c57ae83fcb50
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/19/2019
ms.locfileid: "68336742"
---
# <a name="whats-new-in-bot-framework-july-2019"></a>Bot Framework의 새로운 기능(2019년 7월)

[!INCLUDE[applies-to](includes/applies-to.md)]

Bot Framework SDK v4는 개발자가 원하는 프로그래밍 언어를 사용하여 복잡한 대화를 모델링하고 빌드할 수 있게 해주는 [오픈 소스 SDK][1a]입니다.

이 문서에는 Bot Framework 및 Azure Bot Service의 주요 새 기능과 향상 기능이 요약되어 있습니다.

|   | C#  | JS  | Python |   
|---|:---:|:---:|:------:|
|SDK) |[4.5][1] | [4.5][2] | [4.4.0b2(미리 보기)][3] | 
|Docs | [docs][5] |[docs][5] |  | |
|샘플 |[.NET Core][6], [WebAPI][10] |[Node.js][7] , [TypeScript][8], [es6][9]  | [Python][111] | | 

[1a]:https://github.com/microsoft/botframework-sdk/#readme
[1]:https://github.com/Microsoft/botbuilder-dotnet/#packages
[2]:https://github.com/Microsoft/botbuilder-js#packages
[3]:https://github.com/Microsoft/botbuilder-python#packages
[5]:https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0
[6]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore
[7]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs
[8]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_typescript
[9]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_es6
[10]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_webapi
[111]:https://github.com/Microsoft/botbuilder-python/tree/master/samples


## <a name="bot-framework-channels"></a>Bot Framework 채널
- [Direct Line Speech(공개 미리 보기)](https://aka.ms/streaming-extensions) | [문서](https://docs.microsoft.com/azure/bot-service/directline-speech-bot?view=azure-bot-service-4.0): Bot Framework 및 Microsoft의 Speech Services는 WebSocket을 사용하여 클라이언트와 봇 애플리케이션 간에 양방향으로 스트리밍되는 음성과 텍스트를 가능하게 하는 채널을 제공합니다.  

- [Direct Line App Service 확장(공개 미리 보기)](https://portal.azure.com) | [문서](https://aka.ms/directline-ase): 클라이언트에서 Direct Line API를 사용하여 봇에 직접 연결할 수 있게 하는 Direct Line 버전입니다. 이는 성능 향상과 격리 강화를 포함한 많은 이점을 누릴 수 있습니다. Direct Line App Service 확장은 Azure App Service Environment 내에서 호스팅되는 서비스를 포함하여 모든 Azure App Services에서 사용할 수 있습니다. Azure App Service Environment는 격리를 제공하며 VNet 내에서 작동하는 데 적합합니다. VNet을 사용하면 Azure에서 사용자 고유의 프라이빗 공간을 만들 수 있으며, 격리, 조각화 및 기타 주요 이점을 누릴 수 있으므로 클라우드 네트워크에 매우 중요합니다. 

## <a name="bot-framework-sdk"></a>Bot Framework SDK
- [적응형 대화(SDK v4.6 미리 보기)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog#readme) | [문서](https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/docs) | [C# 샘플](https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/csharp_dotnetcore): 이제 적응형 대화를 통해 개발자는 컨텍스트와 이벤트에 따라 대화 흐름을 동적으로 업데이트할 수 있습니다. 이는 대화 도중의 대화 컨텍스트 전환 및 중단을 처리할 때 특히 유용합니다. 
  
- [Bot Framework Python SDK(미리 보기 2)](https://github.com/microsoft/botbuilder-python) | [샘플](https://github.com/Microsoft/botbuilder-python/tree/master/samples): Python SDK는 이제 OAuth, Prompts, CosmosDB를 지원하며, SDK 4.5의 모든 주요 기능을 포함합니다. 또한 SDK의 새로운 기능을 알아보는 데 도움이 되는 샘플도 제공합니다.

## <a name="bot-framework-testing"></a>Bot Framework 테스트
- [단위 테스트](http://aka.ms/bot-test-package) | [문서](https://aka.ms/testing-framework) | [C# 샘플](https://aka.ms/corebot-test) | [JS 샘플](https://aka.ms/js-core-test-sample): 더 나은 테스트 도구에 대한 고객과 개발자의 요청을 처리하기 위해 SDK의 7월 버전에는 새로운 단위 테스트 기능이 도입되었습니다. Microsoft.Bot.Builder.testing 패키지는 봇의 단위 테스트 대화를 간소화합니다. 

- [채널 테스트](https://github.com/Microsoft/BotFramework-Emulator/releases): Microsoft Build 2019에서 도입된 봇 검사기는 Microsoft Teams, Slack, Cortana 등과 같은 채널에서 봇을 디버그하고 테스트할 수 있는 Bot Framework Emulator의 새로운 기능입니다. 특정 채널에서 봇을 사용하면 봇에서 받은 메시지 데이터를 검사할 수 있는 Bot Framework Emulator로 메시지가 미러링됩니다. 또한 특정 순서에 채널과 봇 간에 지정된 턴에 대한 봇 메모리 상태의 스냅샷도 렌더링됩니다.

## <a name="web-chat"></a>웹 채팅
- 기업 고객의 요청에 따라 봇을 사용하여 엔터프라이즈 앱의 리소스에 액세스할 수 있는 권한을 사용자에게 부여하는 방법을 보여 주는 [웹 채팅 샘플](https://github.com/microsoft/BotFramework-WebChat/tree/master/samples/19.a.single-sign-on-for-enterprise-apps#single-sign-on-demo-for-enterprise-apps-using-oauth)이 추가되었습니다. OAuth와 Microsoft Graph 및 GitHub API의 상호 운용성을 보여 주기 위해 두 가지 종류의 리소스가 사용됩니다.

## <a name="solutions"></a>솔루션
- [가상 도우미 솔루션 가속기](https://github.com/Microsoft/botframework-solutions#readme) : 정교한 대화형 환경을 구축하는 데 도움이 되는 일단의 템플릿, 솔루션 가속기 및 기술을 제공합니다. Direct Line Speech 및 가상 도우미와 통합되어 디바이스 클라이언트에서 가상 도우미와 상호 작용하고 적응형 카드를 렌더링하는 방법을 보여 주는 새로운 가상 도우미용 Android 앱 클라이언트입니다. 업데이트에는 Direct Line Speech 및 Microsoft Teams 지원도 포함되어 있습니다.
  
- [고객 서비스용 Dynamics 365 가상 에이전트(공개 미리 보기)](https://dynamics.microsoft.com/en-us/ai/virtual-agent-for-customer-service/): 공개 미리 보기를 사용하면 적응 가능한 인텔리전트 가상 에이전트를 통해 뛰어난 고객 서비스를 제공할 수 있습니다. 고객 서비스 전문가는 AI 기반 인사이트를 사용하여 봇을 쉽게 만들고 향상시킬 수 있습니다.
  
- [Dynamics 365용 채팅](https://www.powerobjects.com/powerpacks/powerchat/): Dynamics 365용 채팅은 지원 담당자와 최종 사용자가 효과적으로 상호 작용하고 높은 생산성을 유지할 수 있도록 몇 가지 기능을 제공합니다. Microsoft Dynamics 365의 웹 사이트에 대한 방문자의 라이브 채팅 및 대화를 추적합니다.

## <a name="additional-information"></a>추가 정보
- 이전 공지 사항은 [여기](what-is-new-archive.md)서 확인할 수 있습니다.
