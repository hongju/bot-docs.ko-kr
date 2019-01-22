---
title: .bot 파일을 사용하여 리소스 관리 | Microsoft Docs
description: 봇 파일의 목적 및 사용 방법을 설명합니다.
keywords: 봇 파일, .bot, .bot 파일, msbot, 봇 리소스, 봇 리소스 관리
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: fdf0b16cc89b322ffa9d36b5516b09b0338ce9dc
ms.sourcegitcommit: b94361234816e6b95459f142add936732fc40344
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/15/2019
ms.locfileid: "54317573"
---
# <a name="manage-resources-with-a-bot-file"></a>.bot 파일을 사용하여 리소스 관리

봇은 일반적으로 [LUIS.ai](https://luis.ai) 또는 [QnaMaker.ai](https://qnamaker.ai)와 같은 다양한 서비스를 사용합니다. 봇을 개발하는 경우 사용되는 서비스에 대한 메타데이터를 저장할 균일한 장소가 없습니다.  따라서 봇을 전체적으로 보여주는 도구를 빌드하지 못합니다.

이 문제를 해결하기 위해 **.bot 파일**을 만들어서 도구를 활용할 수 있도록 한 곳에 모든 서비스 참조를 모으는 역할을 합니다.  예를 들어, Bot Framework 에뮬레이터([V4](https://aka.ms/Emulator-wiki-getting-started))는 .bot 파일을 사용하여 봇이 사용하는 연결된 서비스를 통해 통합 보기를 만듭니다.  

.bot 파일을 사용하여 다음과 같은 서비스를 등록할 수 있습니다.

* **Localhost** 로컬 디버거 엔드포인트
* [**Azure Bot Service**](https://azure.microsoft.com/en-us/services/bot-service/) Azure Bot Service 등록
* [**LUIS.AI**](https://www.luis.ai/) LUIS는 봇이 자연어를 사용하여 사용자와 통신할 수 있는 기능을 제공합니다... 
* [**QnA Maker**](https://qnamaker.ai/) 몇 분 안에 FAQ URL, 구조화된 문서 또는 편집 콘텐츠에 따라 간단한 질문 및 대답 봇을 빌드하고, 학습하고, 게시합니다.
* [**디스패치**](https://github.com/Microsoft/botbuilder-tools/tree/master/Dispatch) 여러 서비스에서 디스패치하기 위한 모델입니다.
* [**Azure Application Insights**](https://azure.microsoft.com/en-us/services/application-insights/) 인사이트 및 봇 분석을 위한 기능입니다.
* [**Azure Blob Storage**](https://azure.microsoft.com/en-us/services/storage/blobs/) 봇 상태 지속성을 위한 기능입니다. 
* [**Azure Cosmos DB**](https://azure.microsoft.com/en-us/services/cosmos-db/) 봇 상태를 유지하기 위해 전역적으로 배포된 다중 모델 데이터베이스 서비스입니다.

이외에도 봇은 다른 사용자 지정 서비스를 사용할 수 있습니다. [일반 서비스](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/add-services.md) 기능을 활용하여 일반 서비스 구성을 연결할 수 있습니다.

## <a name="when-is-a-bot-file-created"></a>.bot 파일이 생성되는 경우 
- [Azure Bot Service](https://ms.portal.azure.com/#blade/Microsoft_Azure_Marketplace/GalleryResultsListBlade/selectedSubMenuItemId/%7B%22menuItemId%22%3A%22gallery%2FCognitiveServices_MP%2FBotService%22%2C%22resourceGroupId%22%3A%22%22%2C%22resourceGroupLocation%22%3A%22%22%2C%22dontDiscardJourney%22%3Afalse%2C%22launchingContext%22%3A%7B%22source%22%3A%5B%22GalleryFeaturedMenuItemPart%22%5D%2C%22menuItemId%22%3A%22CognitiveServices_MP%22%2C%22subMenuItemId%22%3A%22BotService%22%7D%7D)를 사용하는 봇을 만들면 .bot 파일이 프로비전된 연결된 서비스 목록에 자동으로 생성됩니다. .bot은 기본적으로 암호화되어 있습니다.
- Bot Framework V4 SDK [Template](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4) for Visual Studio 또는 Bot Builder [Yeoman Generator](https://www.npmjs.com/package/generator-botbuilder)를 사용하여 봇을 만들면 .bot 파일이 자동으로 만들어집니다. 연결된 서비스가 이 흐름에서 프로비전지 않고 봇 파일이 암호화되지 않습니다.
- [BotBuilder-samples](https://github.com/Microsoft/botbuilder-samples)를 시작하는 경우 Bot Framework V4 SDK의 모든 샘플 및 .bot 파일이 암호화되지 않습니다. 
- [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md) 도구를 사용하여 봇 파일을 만들 수도 있습니다.

## <a name="what-does-a-bot-file-look-like"></a>봇 파일의 모양 
샘플 [.bot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/sample-bot-file.json) 파일을 살펴보세요.
.bot 파일을 암호화하고 암호를 해독하는 방법을 자세히 알아보려면 [봇 비밀](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/bot-file-encryption.md)을 참조하세요.

## <a name="why-do-i-need-a-bot-file"></a>.bot 파일이 필요한 이유는 무엇인가요?

.bot 파일은 Bot Framework SDK를 사용하여 봇을 빌드하기 위한 요구 사항이 **아닙니다**. appsettings.json, web.config, env, keyvault 또는 봇에서 사용하는 서비스 참조 및 키를 추적하는 데 적합한 것으로 보이는 메커니즘을 계속 사용할 수 있습니다. 그러나 에뮬레이터를 사용하여 봇을 테스트하려면 .bot 파일이 필요합니다. 에뮬레이터가 테스트용으로 .bot 파일을 만들 수 있습니다. 이렇게 하려면 에뮬레이터를 시작하고, 시작 페이지에서 **새 봇 구성 만들기** 링크를 클릭합니다. 나타나는 대화 상자에 **봇 이름** 및 **엔드포인트 URL**을 입력합니다. 그런 다음, 연결합니다.

.bot 파일을 사용하는 이점은 다음과 같습니다.
- 사용할 언어/플랫폼에 관계없이 리소스를 저장하는 표준 방법을 제공합니다.   
- Bot Framework 에뮬레이터 및 CLI 도구는 일관된 형식(.bot 파일)으로 추적 중인 연결된 서비스를 사용하고 작동시킵니다. 
- 잘 정의된 스키마(.bot 파일) 없이 서비스를 생성하고 관리하는 세련된 도구 솔루션을 사용하기 어렵습니다.  


## <a name="using-bot-file-in-your-bot-framework-sdk-bot"></a>Bot Framework SDK 봇에서 .bot 파일 사용

봇의 코드에서 서비스 구성 정보를 가져오는 데 .bot 파일을 사용할 수 있습니다. [ C# ](https://www.nuget.org/packages/Microsoft.Bot.Configuration) 및 [JS](https://www.npmjs.com/package/botframework-config)에 사용 가능한 BotFramework-Configuration 라이브러리를 통해 봇 파일을 로드하고 여러 메서드를 지원하여 적절한 서비스 구성 정보를 쿼리하고 가져올 수 있습니다.

## <a name="additional-resources"></a>추가 리소스
봇 파일을 사용하는 방법에 대한 자세한 내용은 [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md) 추가 정보 파일을 참조하세요.
