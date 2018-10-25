---
title: 상거래 봇 시나리오 | Microsoft Docs
description: Bot Framework를 사용하여 상거래 봇 시나리오를 살펴봅니다.
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f880bc9a424fd9905f7e4ced25e97e2c37155072
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49996830"
---
# <a name="commerce-bot-scenario"></a>상거래 봇 시나리오

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

[상거래 봇](bot-service-scenario-commerce.md) 시나리오는 사람들이 기존의 호텔 컨시어지 서비스와 가져왔던 기존 이메일 및 전화 통화 상호 작용을 대체하는 봇을 설명합니다. 봇은 Cognitive Services를 활용하여 백 엔드 서비스와의 통합에서 수집된 컨텍스트로 텍스트 및 음성을 통해 고객 요청을 더 잘 처리합니다.

![응용 프로그램 봇 다이어그램](~/media/scenarios/bot-service-scenario-commerce-bot.png)

호텔의 컨시어지 역할을 수행하는 상거래 봇의 논리 흐름은 다음과 같습니다.

1. 고객이 호텔 모바일 앱을 사용합니다.
2. Azure AD B2C를 사용하여 사용자가 인증합니다.
3. 사용자 지정 응용 프로그램 봇을 사용하여 사용자가 정보를 요청합니다. 
4. Cognitive Services가 자연어 요청을 처리하도록 돕습니다.
5. 고객이 응답을 검토하고, 자연스러운 대화로 질문을 구체화할 수 있습니다.
6. 사용자가 결과에 만족하면 응용 프로그램 봇이 고객의 예약을 업데이트합니다.
7. Application Insights가 봇 성능과 사용을 통한 개발에 도움이 되도록 런타임 원격 분석 데이터를 수집합니다.

## <a name="sample-bot"></a>샘플 봇
샘플 상거래 봇은 가상의 호텔 컨시어지 서비스를 중심으로 설계되었습니다. C#으로 작성한 경우, 체인의 멤버 서비스 모바일 앱을 통해 Azure AD B2C를 호텔에 인증하면 고객이 봇에 액세스하게 됩니다. 체인은 SQL Database에 예약을 저장합니다. 고객은 다음과 “숙박용으로 카바나 풀을 빌리는 데 얼마인가요?”같은 자연어 구문을 사용할 수 있습니다. 봇에는 어떤 호텔인지에 대한 컨텍스트와 게스트 숙박 기간에 대한 컨텍스트가 있습니다. 또한 Language Understanding(LUIS) 서비스를 사용하면 "풀 카바나(pool cabana)"와 같은 간단한 문구를 통해서 봇이 쉽게 상황을 파악할 수 있습니다. 봇은 답변을 제공한 다음, 숙박 기간 및 카바나 유형과 관련된 옵션을 제공하면서 카바나를 예약하도록 고객에게 제안합니다. 봇이 필요한 데이터를 모두 확보하면 요청을 예약합니다. 게스트는 음성을 통해서도 동일한 요청을 할 수 있습니다.

이 샘플 봇의 소스 코드는 [일반적인 Bot Framework 시나리오에 대한 샘플](https://aka.ms/bot/scenarios)에서 다운로드하거나 복제할 수 있습니다.

## <a name="components-youll-use"></a>사용할 구성 요소
상거래 봇은 다음 구성 요소를 사용합니다.
-   인증에 Azure AD
-   Cognitive Services: LUIS
-   Application Insights

### <a name="azure-active-directory-azure-ad"></a>Azure AD(Azure Active Directory)
Azure Active Directory(Azure AD)는 Microsoft의 다중 테넌트 클라우드 기반 디렉터리 및 ID 관리 서비스입니다. Azure AD를 사용하면 전 세계 수백만 개의 조직에서 사용하는 세계 정상급 ID 관리 솔루션과 쉽고 빠르고 통합할 수 있기 때문 봇 개발자가 봇을 구축하는 데 집중할 수 있습니다. Azure AD는 B2C 커넥터를 지원하여 Google, Facebook 또는 Microsoft 계정과 같은 외부 ID를 사용하여 개인을 식별할 수 있습니다. Azure AD를 사용하면 사용자의 자격 증명을 관리해야 할 책임이 없어지기 때문에 봇의 사용자를 응용 프로그램에서 노출되는 올바른 데이터와 상호 연결할 수 있다는 점을 알고 봇의 솔루션에 더 집중할 수 있습니다.

### <a name="cognitive-services-luis"></a>Cognitive Services: LUIS
Cognitive Services 제품군 기술에 속하는 Language Understanding(LUIS)를 사용하면 앱에 머신 러닝의 기능을 앱에 적용할 수 있습니다. 현재, LUIS는 사람이 원하는 것을 봇이 이해할 수 있도록 하는 여러 가지 언어를 지원합니다. LUIS와 통합할 때, 의도를 표현하고 봇이 이해하는 엔터티를 정의합니다. 그런 다음, 예제 발언을 통해 봇을 학습시켜서 의도와 엔터티를 이해하도록 봇을 교육합니다. 특정한 대화 요구에 맞게 봇을 최대한 유동적으로 만들 수 있도록 문구 목록과 regex 기능을 사용하여 통합을 조정할 수 있습니다.

### <a name="application-insights"></a>Application Insights
Application Insights는 APM(응용 프로그램 성능 관리) 및 즉각적인 분석을 통해 실행 가능한 통찰력을 가져오도록 도와줍니다. 기본적으로 다양한 성능 모니터링, 강력한 경고 및 사용이 간편한 대시보드를 통해 봇의 가용성과 올바른 작동을 보장할 수 있습니다. 문제가 있는지 신속하게 확인한 다음, 근본 원인을 분석하여 문제를 찾아 해결할 수 있습니다.

## <a name="next-steps"></a>다음 단계
다음은, Cortana 기술 봇 시나리오에 대해 알아봅니다.

> [!div class="nextstepaction"]
> [Cortana 기술 봇 시나리오](bot-service-scenario-cortana-skill.md)
