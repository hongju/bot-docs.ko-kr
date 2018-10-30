---
title: Bot Service 시나리오 개요 | Microsoft Docs
description: Bot Service를 사용하여 빌드된 강력하고 성공적인 봇에 대한 주요 시나리오를 살펴봅니다.
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: e195f83eefd5f162b74f8891f3b174efc8934700
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997970"
---
# <a name="bot-scenarios"></a>봇 시나리오

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

이 항목에서는 Bot Service를 사용하여 빌드된 강력하고 성공적인 봇에 대한 주요 시나리오를 살펴봅니다.

모든 시나리오 봇 샘플의 소스 코드는 [일반적인 Bot Framework 시나리오에 대한 샘플](https://aka.ms/bot/scenarios)에서 다운로드하거나 복제할 수 있습니다.

## <a name="commerce-bot-scenario"></a>상거래 봇 시나리오
[상거래 봇](bot-service-scenario-commerce.md) 시나리오는 사람들이 기존의 호텔 컨시어지 서비스와 가져왔던 기존 이메일 및 전화 통화 상호 작용을 대체하는 봇을 설명합니다. 봇은 Cognitive Services를 활용하여 백 엔드 서비스와의 통합에서 수집된 컨텍스트로 텍스트 및 음성을 통해 고객 요청을 더 잘 처리합니다.

상거래 봇 시나리오에서 고객은 호텔의 컨시어지 서비스를 요청할 수 있습니다. Azure Active Directory v2 인증 엔드포인트를 통해 인증됩니다. 봇은 고객의 예약을 조회하고 다른 서비스 옵션을 제공할 수 있습니다. 예를 들어 고객은 수영장 옆의 카바나를 예약할 수 있습니다. 봇은 LUIS(Language Understanding Intelligent Services)를 사용하여 요청을 구문 분석한 다음, 사용자에게 기존 예약에 대한 카바나 예약의 프로세스를 안내합니다.

## <a name="cortana-skill-bot-scenario"></a>Cortana 기술 봇 시나리오
[Cortana 기술 봇](bot-service-scenario-cortana-skill.md) 시나리오는 Cortana를 활용합니다. 음성 및 사용자 지정 Cortana 기술 봇의 자연스러운 인터페이스를 사용하여 호출을 할 때 있는 위치에 기반하는 약속을 만들 수 있도록 Cortana에 모바일 자동 세부 정보 회사와 같은 조직에게 말하도록 요청할 수 있습니다. 봇은 서비스, 사용 가능한 시간 및 기간 목록을 제공할 수 있습니다.

## <a name="enterprise-productivity-bot-scenario"></a>엔터프라이즈 생산성 봇 시나리오
[엔터프라이즈 생산성 봇](bot-service-scenario-enterprise-productivity.md) 시나리오에서는 생산성 향상을 위해 Office 365 일정 및 다른 서비스와 봇을 통합하는 방법을 보여줍니다.

봇은 더 빠르고 쉽게 다른 사용자와 모임 요청을 만들 수 있도록 Office 365와 통합합니다. 이렇게 하는 동안 Dynamics CRM 등의 추가 서비스에 액세스할 수 있습니다. 이 샘플에서는 Azure Active Directory를 통해 인증을 사용하여 Office 365와 통합하는 데 필요한 코드를 제공합니다. 독자에 대한 연습으로 외부 서비스에 대한 모의 진입점을 제공합니다.

## <a name="information-bot-scenario"></a>정보 봇 시나리오
이 [정보 봇](bot-service-scenario-informational.md)은 Cognitive Services QnA Maker를 사용하여 정보 집합이나 FAQ에 정의된 질문에 답변하고 Azure Search를 사용하여 더 많은 주관식 질문에 답변할 수 있습니다.

종종 정보는 검색을 통해 쉽게 표시될 수 있는 SQL Server와 같은 구조화된 데이터 저장소에 포함되어 있습니다. 간단한 대화형 명령을 사용하여 고객의 주문 상태를 조회한다고 가정합니다. Cognitive Services QnA Maker를 사용하여 고객 조회, 고객의 가장 최근 주문 검토 등과 같은 유효한 검색 옵션의 집합이 표시됩니다. 정의된 QnA 형식을 사용하여 사용자는 SQL Database에 저장된 데이터를 조회할 수 있는 Azure Search에서 지원되는 질문에 쉽게 답할 수 있습니다.

## <a name="iot-bot-scenario"></a>IoT 봇 시나리오
이 [IoT(사물 인터넷) 봇](bot-service-scenario-internet-things.md)을 통해 대화형 채팅 명령을 사용하여 Philips Hue 조명과 같은 집 주변의 장치를 쉽게 제어할 수 있습니다.

이 간단한 봇을 사용하여 무료 IFTTT(If This Then That) 서비스와 함께 Philips Hue 조명을 제어할 수 있습니다. IoT 장치로 Philips Hue는 노출된 API를 통해 로컬로 제어될 수 있습니다. 그러나 이 API는 로컬 네트워크 외부에서 일반 액세스에 대해 노출되지 않습니다. 그러나 IFTTT는 "[Friend of Hue](http://www2.meethue.com/en-us/friends-of-hue/ifttt/)"이므로 조명을 끄고 켜고, 조명 색 또는 조명 강도를 변경하는 등을 발급할 수 있는 여러 컨트롤 명령을 노출했습니다.

## <a name="next-steps"></a>다음 단계
이제 시나리오의 개요가 있으므로 각 시나리오를 자세히 알아봅니다.

> [!div class="nextstepaction"]
> [상거래 봇](bot-service-scenario-commerce.md)
