---
title: Internet of Things 봇 시나리오 | Microsoft Docs
description: Bot Framework를 사용하여 Internet of Things 봇 시나리오를 살펴봅니다.
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 3b65f323427760fa43586f471aefb6811ef3e675
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574527"
---
# <a name="internet-of-things-iot-bot-scenario"></a>IoT(사물 인터넷) 봇 시나리오

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

이 IoT(사물 인터넷) 봇을 통해 음성 또는 대화형 채팅 명령을 사용하여 Philips Hue 조명과 같은 집 주변의 장치를 쉽게 제어할 수 있습니다.

사람들은 사물에게 이야기하는 것을 좋아합니다. 최초의 원격 TV 시절 이래로 사람들은 환경에 영향을 주는 방식으로 전환하는 것을 좋아하지 않았습니다. 이 IoT 봇을 사용하면 간단한 채팅 명령 또는 음성으로 Philips Hue를 관리할 수 있습니다. 또한 채팅을 사용할 경우 선택할 색상과 관련된 시각적 옵션을 제공할 수 있습니다.

![Internet of Things 봇 다이어그램](~/media/scenarios/bot-service-scenario-iot-bot.png)

다음은 IoT 봇의 논리 흐름입니다.

1. 사용자가 Skype에 로그인하고 IoT 봇에 액세스합니다.
2. 음성을 사용하여 사용자가 IoT 장치를 통해 봇에 전등을 켜도록 요청합니다.
3. IoT 장치 네트워크에 대한 액세스 권한이 있는 타사 서비스에 요청이 릴레이됩니다.
4. 명령의 결과가 사용자에게 반환됩니다.
5. Application Insights가 봇 성능과 사용을 통한 개발에 도움이 되도록 런타임 원격 분석을 수집합니다.

## <a name="sample-bot"></a>샘플 봇
IoT 봇을 사용하면 Skype 또는 Slack과 같은 채널의 채팅 명령을 사용하여 Hue를 빠르게 제어할 수 있습니다. 원격 액세스를 용이하게 하기 위해 Hue에 작동하도록 미리 정의된 IFTTT 애플릿을 호출합니다.

이 샘플 봇의 소스 코드는 [일반적인 Bot Framework 시나리오에 대한 샘플](https://aka.ms/bot/scenarios)에서 다운로드하거나 복제할 수 있습니다.

## <a name="components-youll-use"></a>사용할 구성 요소
IoT(사물 인터넷) 봇은 다음 구성 요소를 사용합니다.
-   Philips Hue
-   IFTTT(If This Then That)
-   Application Insights

### <a name="philips-hue"></a>Philips Hue
Philips Hue의 연결된 전구 및 브리지를 사용하여 조명을 완전히 제어할 수 있습니다. 조명에 대해 하려는 모든 작업을 Hue가 해줄 수 있습니다. Hue에는 로컬 네트워크에서 사용할 수 있는 API가 있습니다. 그러나 친숙한 봇 인터페이스를 사용하여 어디에서든지 Hue 제어 장치 및 조명에 액세스하고 싶을 것입니다. 이러한 경우 IFTTT를 통해 Hue에 액세스할 수 있습니다.

### <a name="ifttt"></a>IFTTT
IFTTT는 애플릿이라는 간단한 조건문 체인을 만드는 데 사용할 수 있는 무료 웹 기반 서비스입니다. 봇에서 애플릿을 트리거하여 사용자를 대신해 작업을 수행하도록 할 수 있습니다. 조명을 켜거나 끄고, 장면을 변경하는 등에 사용할 수 있는 미리 정의된 다양한 Hue 애플릿이 있습니다.

### <a name="application-insights"></a>Application Insights
Application Insights는 APM(응용 프로그램 성능 관리) 및 즉각적인 분석을 통해 실행 가능한 통찰력을 가져오도록 도와줍니다. 기본적으로 다양한 성능 모니터링, 강력한 경고 및 사용이 간편한 대시보드를 통해 봇의 가용성과 올바른 작동을 보장할 수 있습니다. 문제가 있는지 신속하게 확인한 다음, 근본 원인을 분석하여 문제를 찾아 해결할 수 있습니다.
