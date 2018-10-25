---
title: PowerBI를 사용하여 대화 분석 | Microsoft Docs
description: 엔터프라이즈 봇 템플릿에서 Application Insights를 사용하여 PowerBI를 통해 인사이트를 활성화하는 방법 알아보기
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9c5a156ff949c4014b69638e9cd5abb4b0ede149
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999393"
---
# <a name="enterprise-bot-template---conversational-analytics-using-powerbi-dashboard-and-application-insights"></a>엔터프라이즈 봇 템플릿 - PowerBI 대시보드 및 Application Insights를 사용하여 대화 분석

> [!NOTE]
> 이 항목은 SDK v4 버전에 적용됩니다. 

봇이 배포되고 메시지 처리를 시작하면 리소스 그룹 내에서 Application Insights 인스턴스로 이동하는 원격 분석이 표시됩니다. 

이 원격 분석은 Azure Portal의 Application Insights 블레이드 내에서 Log Analytics를 사용하여 볼 수 있습니다. 또한 PowerBI에서 동일한 원격 분석을 사용하여 봇 사용에 대한 보다 일반적인 비즈니스 인사이트를 제공할 수 있습니다.

예제 PowerBI 대시보드는 만든 프로젝트의 PowerBI 폴더 내에서 제공됩니다. 이는 예제 목적으로 제공되며 사용자 고유의 인사이트를 만들도록 시작할 수 있는 방법을 보여줍니다. 시간이 지남에 따라 이러한 시각화를 강화할 예정입니다. 

## <a name="getting-started"></a>시작하기

- [여기](https://powerbi.microsoft.com/en-us/desktop/)에서 PowerBI Desktop을 다운로드합니다.
 
- 봇에서 사용되는 Application Insights 리소스에 대한 ```Application Id```를 검색합니다. Application Insights Azure 블레이드의 구성 섹션에서 API 액세스 페이지로 이동하여 가져올 수 있습니다.

솔루션의 PowerBI 폴더 내에 있는 제공된 PowerBI 템플릿 파일을 두 번 클릭합니다. 이전 단계에서 검색한 ```Application Id```에 대해 메시지가 표시됩니다. Azure 구독 자격 증명을 사용하라는 메시지가 표시되면 인증을 완료하고, 조직 계정 설정을 클릭하여 로그인해야 합니다.

결과 대시보드는 이제 Application Insights 인스턴스에 연결되며 메시지가 전송 및 수신되는 경우 대시보드 내에 초기 인사이트가 표시됩니다.

>현재 배포 스크립트는 LUIS 모델을 게시할 때 감정을 활성화하지 않으므로 감정 시각화는 데이터를 표시하지 않습니다. LUIS 모델을 [다시 게시](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-how-to-publish-app)하고 감정을 활성화한 경우 작동합니다.

## <a name="middleware-processing"></a>미들웨어 처리

QnAMaker 및 LuisRecognizer 클래스에 대한 원격 분석 래퍼는 시나리오에 관계없이 일관된 원격 분석 출력을 위해 제공되며 각 프로젝트에서 작동하도록 표준 대시보드를 활성화합니다.

```TelemetryLuisRecognizer``` 및 ```TelemetryQnAMaker```는 모두 생성자에서 개발자가 사용자 이름 및 원래 메시지의 로깅을 비활성화하도록 하는 속성을 제공합니다. 그러나 이는 사용 가능한 인사이트의 양을 줄입니다.

## <a name="telemetry-captured"></a>캡처된 원격 분석

4개의 고유한 원격 분석 이벤트는 엔터프라이즈 템플릿에서 기본적으로 활성화되는 ```TelemetryLuisRecognizer``` 및 ```TelemetryQnAMaker```를 사용하여 캡처됩니다. 

프로젝트에서 사용되는 각 LUIS 의도는 LuisIntent로 접두사가 추가됩니다. 대시보드에서 의도를 쉽게 식별할 수 있도록 하려면

```
-BotMessageReceived
    - ActivityId
    - Channel
    - FromId
    - Conversationid
    - ConversationName
    - Locale
    - UserName
    - Text
```
  
```
-BotMessageSent
    - ActivityId,
    - Channel
    - RecipientId
    - Conversationid
    - ConversationName
    - Locale
    - ReceipientName
    - Text
```

```
- LuisIntent.*
    - ActivityId
    - Intent
    - IntentScore
    - SentimentLabel
    - SentimentScore
    - ConversationId
    - Question
```

```
- QnAMaker
    - ActivityId
    - ConversationId
    - OriginalQuestion
    - UserName
    - QnAItemFound
    - Question
    - Answer
    - Score
```