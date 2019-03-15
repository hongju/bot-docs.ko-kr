---
title: 엔터프라이즈 봇 템플릿에 대한 자세한 개요 | Microsoft Docs
description: 엔터프라이즈 봇 템플릿에서 디자인 결정을 알아봅니다.
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 582ec21d36e15fcbaef2d26616a9e55a04d8e5f2
ms.sourcegitcommit: b2245df2f0a18c5a66a836ab24a573fd70c7d272
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/07/2019
ms.locfileid: "57568220"
---
# <a name="enterprise-bot-template---overview"></a>Enterprise Bot Template - 개요

> [!NOTE]
> 이 토픽은 SDK v4 버전에 적용됩니다. 

Enterprise Bot Template은 대화 환경을 만들고, 노력을 줄이고, 품질 수준을 높이는 데 필요한 모범 사례와 서비스의 견고한 기초를 제공합니다. 템플릿에서 [Bot Builder SDK v4](https://github.com/Microsoft/botbuilder) 및 [Bot Builder 도구](https://github.com/Microsoft/botbuilder-tools)를 활용하여 제공하는 기능은 다음과 같습니다.

기능      | 설명 |
------------ | -------------
소개 | 대화 시작 시 [적응형 카드]()가 있는 소개 메시지입니다.
입력 표시기  | 대화 중에 자동화된 시각적 입력 표시기이며, 장기 실행 작업에 대해 반복합니다.
기본 LUIS 모델  | **Cancel**, **Help**, **Escalate** 등과 같은 일반적인 의도를 지원합니다.
기본 대화 | 기본 사용자 정보 및 취소/도움말 의도에 대한 중단 논리를 캡처하는 대화 흐름입니다.
기본 응답  | 기본 의도 및 대화에 대한 텍스트 및 음성 응답입니다.
FAQ | [QnA Maker](https://www.qnamaker.ai)와 통합하여 기술 자료에서 일반적인 질문에 답변합니다. 
잡담 | 일반적인 쿼리에 대한 표준 답변을 제공하는 전문가 잡담 모델입니다([자세한 정보](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/chit-chat-knowledge-base)).
디스패처 | LUIS 또는 QnA Maker에서 지정된 발화를 처리해야 하는지 여부를 식별하는 통합 [Dispatch](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig) 모델입니다.
언어 지원 | 영어, 프랑스어, 이탈리아어, 독일어, 스페인어 및 중국어로 사용할 수 있습니다.
스크립트(Transcript) | Azure Storage에 저장된 모든 대화에 대한 음성 텍스트입니다.
원격 분석  | [Application Insights](https://azure.microsoft.com/en-gb/services/application-insights/)와 통합하여 모든 대화에 대한 원격 분석을 수집합니다.
분석 | 대화 환경에 대한 인사이트를 시작할 수 있는 Power BI 대시보드 예제입니다.
자동화된 배포 | [Bot Builder 도구](https://github.com/Microsoft/botbuilder-tools)를 사용하여 앞에서 언급한 모든 서비스를 쉽게 배포합니다.

# <a name="background"></a>백그라운드

## <a name="introduction-card"></a>소개 카드
소개 카드는 봇의 기능에 대한 개요를 제공하고 쉽게 시작할 수 있도록 발화 샘플을 제공하여 대화를 향상시킵니다. 또한 사용자에게 봇의 성격을 설정합니다.

## <a name="base-luis-model-with-common-intents"></a>일반적인 의도가 포함된 기본 LUIS 모델
템플릿에 포함된 기본 LUIS 모델은 대부분의 봇에서 처리해야 하는 가장 일반적인 의도를 선택합니다. 봇에서 즉시 사용할 수 있는 의도는 다음과 같습니다.

의도       | 발화 샘플 |
-------------|-------------|
취소       |*취소*, *괜찮아*|
Escalate     |*사람에게 이야기할 수 있나요?*|
FinishTask   |*완료*, *모두 완료되었습니다*|
GoBack       |*뒤로 이동*|
도움말         |*도와주실 수 있으세요?*|
Repeat       |*다시 말할 수 있나요?*|
SelectAny    |*이러한 항목 중 하나*|
SelectItem   |*첫 번째 항목*|
SelectNone   |*없음*|
ShowNext     |*자세히 표시*|
ShowPrevious |*이전 표시*|
StartOver    |*restart*|
중지         |*stop*|

## <a name="qna-maker"></a>QnA Maker

[QnA Maker](https://www.qnamaker.ai/)는 개발자가 아닌 사용자가 봇에서 사용할 질문 및 답변 쌍의 모음을 생성할 수 있는 기능을 제공합니다. 이 정보는 FAQ 데이터 원본 및 제품 설명서에서 가져오거나 QnA Maker 포털 내에서 수동으로 만들 수 있습니다.

템플릿에는 두 가지 예제 QnA Maker 모델이 제공됩니다. 하나는 FAQ용 모델이고, 다른 하나는 [전문가 잡담](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/chit-chat-knowledge-base)용 모델입니다. 

## <a name="dispatch"></a>Dispatch

Dispatch 서비스는 각 서비스에서 발화를 추출하고 중앙 디스패치 LUIS 모델을 만들어 여러 LUIS 모델과 QnA Maker 기술 자료 간의 라우팅을 관리하는 데 사용됩니다.

이렇게 하면 봇에서 지정된 발화를 처리해야 하는 구성 요소를 빠르게 식별할 수 있고 QnA Maker 데이터가 계층 구조상 하위 수준이 아니라 상위 수준의 의도 처리에서 고려되도록 할 수 있습니다.

또한 이 Dispatch 도구를 사용하면 서비스 전체에서 겹치는 발화와 불일치를 강조하는 모델을 평가할 수 있습니다.

## <a name="telemetry"></a>원격 분석

봇 대화에 대한 인사이트를 제공하면 사용자 참여 수준, 사용자가 사용하는 기능 및 봇에서 처리할 수 없는 질문을 이해하는 데 도움이 됩니다.

[Application Insights](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)는 봇 서비스에 대한 운영 원격 분석 데이터와 대화 특정 이벤트를 캡처합니다. 이러한 이벤트는 [Power BI](https://powerbi.microsoft.com/en-us/what-is-power-bi/)와 같은 도구를 사용하여 실행 가능한 정보로 집계할 수 있습니다. Power BI 대시보드 예제는 이 기능을 보여 주는 Enterprise Bot Template에서 사용할 수 있습니다.

# <a name="next-steps"></a>다음 단계
엔터프라이즈 봇을 만들고 배포하는 방법에 대한 자세한 내용은 [시작](bot-builder-enterprise-template-getting-started.md)을 참조하세요. 

# <a name="resources"></a>리소스
Enterprise Bot Template의 전체 소스 코드는 [GitHub](https://github.com/Microsoft/AI/tree/master/templates/Enterprise-Template)에서 찾을 수 있습니다.
