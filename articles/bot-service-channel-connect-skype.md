---
title: Skype에 봇 연결 | Microsoft Docs
description: Skype 인터페이스를 통해 액세스하기 위해 봇을 구성하는 방법을 알아봅니다.
keywords: skype, 봇 채널, skype 구성, 게시, 채널에 연결
author: v-ducvo
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/11/2018
ms.openlocfilehash: f52c8fd668299486eccda3920181df971f475a90
ms.sourcegitcommit: 75f32b3325dd0fc4d8128dee6c22ebf91e5785b3
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/09/2018
ms.locfileid: "53120660"
---
# <a name="connect-a-bot-to-skype"></a>Skype에 봇 연결

Skype는 인스턴트 메시징, 휴대폰 및 영상 통화를 통해 사용자와 연결된 상태를 유지합니다. Skype 인터페이스를 통해 사용자가 검색하고 상호 작용할 수 있는 봇을 구축하여 이 기능을 확장할 수 있습니다.

Skype 채널을 추가하려면 [Azure Portal](https://portal.azure.com/)에서 봇을 열고, **채널** 블레이드를 클릭한 다음, **Skype**를 클릭합니다.

![Skype 채널 추가](~/media/channels/skype-addchannel.png)

이렇게 하면 **Skype 구성** 설정 페이지로 이동됩니다.

![Skype 채널 구성](~/media/channels/skype_configure.png)

**웹 컨트롤**, **메시징**, **호출**, **그룹** 및 **게시**에서 설정을 구성해야 합니다. 하나씩 살펴보겠습니다.

## <a name="web-control"></a>웹 컨트롤

봇을 웹 사이트에 포함하려면 **웹 컨트롤** 섹션에서 **포함 코드 가져오기** 단추를 클릭합니다. 그러면 개발자용 Skype 페이지로 이동됩니다. 이 페이지의 지침을 따라 포함 코드를 가져옵니다.

## <a name="messaging"></a>메시징

이 섹션에서는 봇이 Skype에서 메시지를 보내고 받는 방법을 구성합니다.

## <a name="calling"></a>호출

이 섹션에서는 봇에서 Skype의 호출 기능을 구성합니다. 봇에 대해 **호출**이 사용되도록 설정되어 있는지 여부를 지정하고, 사용되도록 설정되어 있으면 IVR 기능 또는 실시간 미디어 기능을 사용할 수 있는지를 지정할 수 있습니다.

## <a name="groups"></a>그룹

이 섹션에서는 봇을 그룹에 추가할 수 있는지 여부, 메시징용 그룹에서 봇이 동작하는 방식 및 봇이 호출 봇에 대한 그룹 영상 통화를 사용하도록 설정하는 데 사용되는지도 구성합니다.

## <a name="publish"></a>게시

이 섹션에서는 봇의 게시 설정을 구성합니다. *로 레이블이 지정된 모든 필드는 필수 필드입니다.

**미리 보기**의 봇은 100개 연락처로 제한됩니다. 연락처가 100개 이상 필요한 경우 봇을 검토용으로 제출합니다. **검토를 위해 제출**을 클릭하면 Skype에서 봇이 자동으로 검색 가능해집니다(허용되는 경우). 요청을 승인할 수 없는 경우 승인을 위해 먼저 변경해야 하는 항목에 대한 알림이 제공됩니다.

> [!TIP]
> 검토를 위해 봇을 제출하려면 먼저 [Skype 인증 검사 목록](https://github.com/Microsoft/skype-dev-bots/blob/master/certification/CHECKLIST.md)이 충족되어야 허용됩니다.

구성을 완료한 후 **저장**을 클릭하고 **서비스 약관**에 동의합니다. 이제 봇에 Skype 채널이 추가됩니다.

## <a name="next-steps"></a>다음 단계

* [비즈니스용 Skype](bot-service-channel-connect-skypeforbusiness.md)
