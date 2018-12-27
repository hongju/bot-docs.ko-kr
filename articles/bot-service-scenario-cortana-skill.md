---
title: Cortana 스킬 봇 시나리오 | Microsoft Docs
description: Bot Framework를 사용하여 Cortana 스킬 봇 시나리오를 살펴봅니다.
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 7676b7bd75a45130b62c1a691499095d6ba07291
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000230"
---
# <a name="cortana-skills-bot-scenario"></a>Cortana 스킬 봇 시나리오

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Cortana 스킬 봇은 Cortana를 확장하여 일정에서 컨텍스트 음성을 사용하여 모바일 자동 유지 관리 약속을 편리하게 예약할 수 있도록 합니다.

Cortana에 개인 비서입니다. 음성 및 사용자 지정 Cortana 스킬 봇의 자연스러운 인터페이스를 사용하여 Cortana에 자동차 매장과 같은 회사에 약속을 잡아주도록 요청할 수 있습니다. 이 서비스는 서비스, 사용 가능한 시간 및 기간 목록을 제공할 수 있습니다. Cortana는 일정을 확인하여 충돌하는 시간에 포함된 약속이 있는지 검토하고 이러한 경우 약속을 새로 만들고 일정에 추가할 수 있습니다.

![Cortana 스킬 봇 다이어그램](~/media/scenarios/bot-service-scenario-cortana-skill.png)

자동차 매장에 대한 Cortana 스킬 봇의 논리 흐름은 다음과 같습니다.

1. 사용자가 PC 또는 모바일 디바이스에서 Cortana에 액세스합니다.
2. 사용자가 텍스트 또는 음성 명령을 사용하여 자동차 유지 관리 약속을 요청합니다.
3. 봇은 Cortana에 통합되어 있으므로 사용자의 일정에 대한 액세스 권한이 있으며 요청에 논리를 적용합니다.
4. 봇이 해당 정보를 사용하여 자동차 서비스에 유효한 약속을 쿼리할 수 있습니다.
5. 상황에 맞는 옵션이 제공되므로 사용자가 약속을 예약할 수 있습니다.
6. Application Insights가 봇 성능과 사용을 통한 개발에 도움이 되도록 런타임 원격 분석을 수집합니다.

## <a name="sample-bot"></a>샘플 봇
Cortana 스킬 봇은 주로 개인 컨텍스트와 관련이 있습니다. Cortana에서 "Bob's Mobile Maintenance"라고 말하여 사용자의 위치를 기준으로 자동차에 작동하게 합니다. 봇은 Cortana를 통해 노출된 개인 정보를 사용하여 사용자가 봇에게 말할 때 있던 위치를 기준으로 해당 위치를 확인할 수 있습니다.

이 샘플 봇의 소스 코드는 [일반적인 Bot Framework 시나리오에 대한 샘플](https://aka.ms/bot/scenarios)에서 다운로드하거나 복제할 수 있습니다.

## <a name="components-youll-use"></a>사용할 구성 요소
Cortana 봇은 다음 구성 요소를 사용합니다.
-   Cortana
-   Application Insights

### <a name="cortana"></a>Cortana
이제 Cortana 스킬을 만들어 봇에 지원을 추가할 수 있습니다. Cortana 스킬 키트를 사용하여 Cortana에 대한 새로운 기능(스킬이라고 함)을 빌드합니다. 스킬은 Cortana가 더 많은 작업을 수행하도록 하는 구문입니다. 스킬을 Bot과 통합되도록 빌드하여 Cortana를 사용하여 태스크를 완료하여 작업을 수행할 수 있습니다. 호출 프로세스의 일부로, Cortana는 (사용자의 동의를 사용하여) 런타임에 사용자에 대한 정보를 스킬에 전달하므로, 스킬은 해당 환경을 그에 따라 사용자 지정할 수 있습니다. Cortana의 상황에 맞는 지식을 사용하여 봇을 더 유용하고 똑똑하게 만들 수 있습니다. 일단 호출되면 특정 유형의 스킬이 Cortana의 인터페이스는 조작하여 스킬과 최종 사용자 간에 대화가 이루어지도록 할 수 있습니다. 게시된 후에는 Windows 10 1주년 업데이트 이상(Desktop 및 Mobile), iOS 및 Android용 Cortana에서 스킬을 확인하고 사용할 수 있습니다.

### <a name="application-insights"></a>Application Insights
Application Insights는 APM(애플리케이션 성능 관리) 및 즉각적인 분석을 통해 실행 가능한 통찰력을 가져오도록 도와줍니다. 기본적으로 다양한 성능 모니터링, 강력한 경고 및 사용이 간편한 대시보드를 통해 봇의 가용성과 올바른 작동을 보장할 수 있습니다. 문제가 있는지 신속하게 확인한 다음, 근본 원인을 분석하여 문제를 찾아 해결할 수 있습니다.

## <a name="next-steps"></a>다음 단계
다음으로, 엔터프라이즈 생산성 봇 시나리오에 대해 알아봅니다.

> [!div class="nextstepaction"]
> [엔터프라이즈 생산성 봇 시나리오](bot-service-scenario-enterprise-productivity.md)
