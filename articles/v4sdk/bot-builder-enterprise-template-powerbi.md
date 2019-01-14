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
ms.openlocfilehash: 88208a2f5b0eb88d3b2964e63a21585484166d73
ms.sourcegitcommit: 2d84d5d290359ac3cfb8c8f977164f799666f1ab
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/09/2019
ms.locfileid: "54152176"
---
# <a name="enterprise-bot-template---conversational-analytics-using-powerbi-dashboard-and-application-insights"></a>엔터프라이즈 봇 템플릿 - PowerBI 대시보드 및 Application Insights를 사용하여 대화 분석

> [!NOTE]
> 이 토픽은 SDK v4 버전에 적용됩니다. 

봇이 배포되고 메시지 처리를 시작하면 리소스 그룹 내에서 Application Insights 인스턴스로 이동하는 원격 분석이 표시됩니다. 

이 원격 분석은 Azure Portal의 Application Insights 블레이드 내에서 Log Analytics를 사용하여 볼 수 있습니다. 또한 PowerBI에서 동일한 원격 분석을 사용하여 봇 사용에 대한 보다 일반적인 비즈니스 인사이트를 제공할 수 있습니다.

예제 PowerBI 대시보드가 [대화형 AI 원격 분석](https://aka.ms/botPowerBiTemplate)에 제공됩니다. 

이는 예제 목적으로 제공되며 사용자 고유의 인사이트를 만들도록 시작할 수 있는 방법을 보여줍니다. 시간이 지남에 따라 이러한 시각화를 강화할 예정입니다. 


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
    - FromName
    - ConversationId
    - ConversationName
    - Locale
    - Text
    - RecipientId
    - RecipientName
```
  
```
-BotMessageSent
    - ActivityId,
    - Channel
    - RecipientId
    - ConversationId
    - ConversationName
    - Locale
    - RecipientId
    - RecipientName
    - ReplyToId
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
    - DialogId
```

```
- QnAMaker
    - ActivityId
    - ConversationId
    - OriginalQuestion
    - FromName
    - ArticleFound
    - Question
    - Answer
    - Score
```