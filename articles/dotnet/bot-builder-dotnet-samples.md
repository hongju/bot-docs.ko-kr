---
title: .NET용 Bot Builder SDK의 샘플 봇 | Microsoft Docs
description: .NET용 Bot Builder SDK를 사용하여 봇 개발을 시작하는 데 도움이 되는 샘플 봇을 탐색합니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/13/2018
ms.openlocfilehash: 0ee2f062b66db8a09be844a6a2af33bb12a1d7fd
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000250"
---
# <a name="bot-builder-sdk-for-net-samples"></a>.NET용 Bot Builder SDK 샘플

::: moniker range="azure-bot-service-3.0"

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

이 샘플은 .NET용 Bot Builder SDK의 기능을 활용하는 방법을 보여주는 작업에 중점을 두고 있는 봇을 설명합니다. 샘플을 사용하여 다양한 기능이 포함된 유용한 봇 빌드를 빠르게 시작할 수 있습니다.

## <a name="get-the-samples"></a>샘플 가져오기
샘플을 얻으려면 Git를 사용하여 [BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) GitHub 리포지토리를 복제합니다.

```cmd
git clone https://github.com/Microsoft/BotBuilder-Samples.git
cd BotBuilder-Samples
```

.NET용 Bot Builder SDK로 작성한 샘플 봇은 **CSharp** 디렉터리에 정리되어 있습니다.

GitHub에서 샘플을 살펴보고 Azure에 직접 배포할 수도 있습니다.

## <a name="core"></a>코어
이러한 샘플은 다양한 기능을 제공하는 봇을 빌드하는 기본적인 기술을 보여줍니다.

샘플 | 설명
------------ | ------------- 
[첨부 파일 보내기](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-SendAttachment) | 사용자에게 간단한 미디어 첨부 파일(이미지)을 보내는 샘플 봇. 
[첨부 파일 받기](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-ReceiveAttachment) | 사용자가 보낸 첨부 파일을 받아서 다운로드하는 샘플 봇. 
[새 대화 만들기](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-CreateNewConversation)  | 이전에 저장된 사용자 주소를 사용하여 새로운 대화를 시작하는 샘플 봇.
[대화 멤버 가져오기](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GetConversationMembers) | 대화의 멤버 목록을 검색하고 변경 내용을 검색하는 샘플 봇. 
[Direct Line](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-DirectLine) | Direct Line API를 사용하여 서로 통신하는 샘플 봇과 사용자 지정 클라이언트. 
[Direct Line(WebSockets)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-DirectLineWebSockets) | Direct Line API + WebSocket을 사용하여 서로 통신하는 샘플 봇과 사용자 지정 클라이언트. 
[다중 대화 상자](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-MultiDialogs) | 다양한 대화 상자를 보여주는 샘플 봇.
[상태 API](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-State) | 대화의 컨텍스트를 추적하는 상태 비저장 샘플 봇.
[사용자 지정 상태 API](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-CustomState) | 사용자 지정 저장소 공급자를 사용하여 대화의 컨텍스트를 추적하는 상태 비저장 샘플 봇.
[ChannelData](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-ChannelData) | ChannelData를 사용하여 Facebook에 기본 메타데이터를 전송하는 샘플 봇.
[AppInsights](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-AppInsights) | Application Insights 인스턴스에 원격 분석 데이터를 기록하는 샘플 봇.

## <a name="search"></a>검색
이 샘플은 데이터 기반 봇에서 Azure Search를 활용하는 방법을 보여줍니다.

샘플 | 설명
------------ | -------------
[Azure Search](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-Search) | 사용자가 대량의 콘텐츠를 탐색하도록 도와주는 두 개의 샘플 봇.


## <a name="cards"></a>카드
이 샘플은 Bot Framework에서 다양한 카드를 보내는 방법을 보여줍니다.

샘플 | 설명
------------ | -------------
[서식 있는 카드](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/cards-RichCards) | 여러 종류의 서식 있는 카드를 보내는 샘플 봇.
[회전식 카드](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/cards-CarouselCards) | 회전식 레이아웃을 사용하여 단일 메시지에 여러 서식 있는 카드를 보내는 샘플 봇.

## <a name="intelligence"></a>인텔리전스
이 샘플은 Bing 및 Microsoft Cognitive Services API를 사용하여 봇에 인공 지능 기능을 추가하는 방법을 보여줍니다.

샘플 | 설명
------------ | -------------
[LUIS](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-LUIS) | LuisDialog를 사용하여 LUIS.ai 애플리케이션과 통합하는 샘플 봇.
[이미지 캡션](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-ImageCaption) | Microsoft Cognitive Services Vision API를 사용하여 이미지 캡션을 가져오는 샘플 봇.
[음성을 텍스트로 변환](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-SpeechToText)  | Bing Speech API를 사용하여 오디오에서 텍스트를 가져오는 샘플 봇.
[유사한 제품](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-SimilarProducts) | Bing Image Search API를 사용하여 시각적으로 유사한 제품을 찾는 샘플 봇. 
[Zummer](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-Zummer) | Bing Search API를 사용하여 wikipedia 문서를 찾는 샘플 봇.

## <a name="reference-implementation"></a>참조 구현
이 샘플은 통합형 시나리오를 보여주도록 설계되었습니다. 봇에서 좀 더 복잡한 기능을 구현하려는 분들에게 매우 유용한 코드 조각 소스입니다.


샘플 | 설명
------------ | -------------
[Contoso Flowers](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-ContosoFlowers) | Bot Framework의 여러 기능을 사용하는 샘플 봇.

::: moniker-end

::: moniker range="azure-bot-service-4.0"

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Bot Builder 샘플 리포지토리의 샘플은 .NET용 SDK에 제공되는 기능의 활용 방법을 보여주는 작업 중심 봇에 대해 설명합니다. 샘플을 사용하여 다양한 기능이 포함된 유용한 봇 빌드를 빠르게 시작할 수 있습니다. 샘플 목록 및 추가 정보는 [readme](https://github.com/Microsoft/BotBuilder-Samples/blob/master/README.md) 파일을 참조하세요.

샘플을 얻으려면 Git를 사용하여 [botbuilder-samples](https://github.com/Microsoft/botbuilder-samples) GitHub 리포지토리를 복제합니다.
```cmd
git clone https://github.com/Microsoft/botbuilder-dotnet.git
```

::: moniker-end

