---
title: 봇 설정 구성 | Microsoft Docs
description: Azure Portal을 사용하여 봇의 다양한 옵션을 구성하는 방법을 알아봅니다.
keywords: 봇 설정 구성, 표시 이름, 아이콘, Application Insights, 설정 블레이드
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 531e37f2186de2e315f11362dcefcc30a2ab6879
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301859"
---
# <a name="configure-bot-settings"></a>봇 설정 구성

표시 이름, 아이콘 및 Application Insights와 같은 봇 설정은 해당 **설정** 블레이드에서 보고 수정할 수 있습니다.

![봇 설정 블레이드](~/media/bot-service-portal-configure-settings/bot-settings-blade.png)

다음은 **설정** 블레이드의 필드 목록입니다.

| 필드 | 설명 |
| :---  | :---        |
| 아이콘 | 채널의 봇을 시각적으로 식별하기 위한 사용자 지정 아이콘 및 Skype, Cortana, 기타 서비스에 대한 아이콘입니다. 이 아이콘은 PNG 형식이어야 하며 크기는 30K 이하여야 합니다. 이 값은 언제든지 변경할 수 있습니다. |
| 표시 이름 | 채널 및 디렉터리에서 봇의 이름입니다. 이 값은 언제든지 변경할 수 있습니다. 35자로 제한됩니다. |
| 봇 핸들 | 봇의 고유 식별자입니다. Bot Service를 사용하여 봇을 만든 후에는 이 값을 변경할 수 없습니다. |
| 메시징 끝점 | 봇과 통신하기 위한 끝점입니다. |
| Microsoft 앱 ID | 봇의 고유 식별자입니다. 이 값은 변경할 수 없습니다. **관리** 링크를 클릭하여 새 암호를 생성할 수 있습니다. |
| Application Insights 계측 키 | 봇 원격 분석을 위한 고유 키입니다. 이 봇에 대한 봇 원격 분석을 수신하려면 Azure Application Insights 키를 이 필드에 복사합니다. 이 값은 선택 사항입니다. Azure Portal에서 만든 봇의 경우 이 키가 생성됩니다. 이 필드에 대한 자세한 내용은 [Application Insights 키](~/bot-service-resources-app-insights-keys.md)를 참조하세요. |
| Application Insights API 키 | 봇 분석을 위한 고유 키입니다. 대시보드에서 봇에 대한 분석을 보려는 경우 Azure Application Insights API 키를 이 필드에 복사합니다. 이 값은 선택 사항입니다. 이 필드에 대한 자세한 내용은 [Application Insights 키](~/bot-service-resources-app-insights-keys.md)를 참조하세요. |
| Application Insights 응용 프로그램 ID | 봇 분석을 위한 고유 키입니다. 대시보드에서 봇에 대한 분석을 보려는 경우 Azure Insights 응용 프로그램 ID 키를 이 필드에 복사합니다. 이 값은 선택 사항입니다. Azure Portal에서 만든 봇의 경우 이 키가 생성됩니다. 이 필드에 대한 자세한 내용은 [Application Insights 키](~/bot-service-resources-app-insights-keys.md)를 참조하세요. |

> [!NOTE]
> 봇에 대한 설정을 변경한 후에 블레이드 맨 위에 있는 **저장** 단추를 클릭하여 새 봇 설정을 저장합니다.

## <a name="next-steps"></a>다음 단계
지금까지 Bot Service에 대한 설정을 구성하는 방법을 배웠으므로 음성 초기화를 구성하는 방법을 알아봅니다.
> [!div class="nextstepaction"]
> [음성 초기화](bot-service-manage-speech-priming.md)