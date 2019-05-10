---
title: 봇 관리 | Microsoft Docs
description: 봇 서비스 포털을 통해 봇을 관리하는 방법에 대해 알아봅니다.
keywords: Azure Portal, 봇 관리, 웹 채팅 테스트, MicrosoftAppID, MicrosoftAppPassword, 애플리케이션 설정
author: v-ducvo
ms.author: rstand
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 4/13/2019
ms.openlocfilehash: 17d80fe4d4730ed294b770fd05bc5d7ea3d114af
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033044"
---
# <a name="manage-a-bot"></a>봇 관리

[!INCLUDE [applies-to-both](includes/applies-to-both.md)]

이 항목에서는 Azure Portal을 사용하여 봇을 관리하는 방법을 설명합니다.

## <a name="bot-settings-overview"></a>봇 설정 개요

![봇 설정 개요](~/media/azure-manage-a-bot/overview.png)

**개요** 블레이드에서 봇에 대한 대략적인 정보를 찾을 수 있습니다. 예를 들어, 봇의 **구독 ID**, **가격 책정 계층** 및 **메시징 엔드포인트**를 볼 수 있습니다.

## <a name="bot-management"></a>봇 관리

 **봇 관리** 섹션에서 대부분의 봇의 관리 옵션을 찾을 수 있습니다. 다음은 봇을 관리하는 데 도움이 되는 옵션 목록입니다.

![봇 관리](~/media/azure-manage-a-bot/bot-management.png)

| 옵션 |  설명 |
| ---- | ---- |
| **빌드** | 빌드 탭에서는 봇을 변경하는 옵션을 제공합니다. 이 옵션은 **등록 전용 봇**에는 사용할 수 없습니다. |
| **웹 채팅에서 테스트** | 통합 웹 채팅 컨트롤을 사용하여 봇을 신속하게 테스트할 수 있습니다. |
| **분석** | 봇에 대한 분석 기능이 켜지면 Application Insights에서 봇에 대해 수집한 분석 데이터를 볼 수 있습니다. |
| **Channels** | 봇이 사용자와 통신하는 데 사용하는 채널을 구성합니다. |
| **설정** | 표시 이름, 분석 및 메시징 엔드포인트와 같은 다양한 봇 프로필 설정을 관리합니다. |
| **음성 초기화** | LUIS 앱과 Bing Speech 서비스 간의 연결을 관리합니다. |
| **Bot Service 가격 책정** | Bot Service의 가격 책정 계층을 관리합니다. |

## <a name="app-service-settings"></a>App Service 설정

![App Service 설정](~/media/azure-manage-a-bot/app-service-settings.png)

**애플리케이션 설정** 블레이드에는 봇의 환경, 디버그 설정 및 애플리케이션 설정 키와 같은 봇에 대한 자세한 정보가 포함되어 있습니다.

### <a name="microsoftappid-and-microsoftapppassword"></a>MicrosoftAppID 및 MicrosoftAppPassword

**MicrosoftAppID** 및 **MicrosoftAppPassword**는 봇의 구성 파일 또는 Azure Key Vault 내에 유지됩니다. 검색하려면 봇의 설정 또는 구성 파일을 다운로드하거나 Azure Key Vault에 액세스합니다. ID와 암호를 사용하여 로컬에서 테스트해야 할 수도 있습니다.

> [!NOTE]
> **봇 채널 등록** 봇 서비스에는 *MicrosoftAppID*가 제공되지만 이 유형의 서비스와 연결된 App Service가 없기 때문에 *MicrosoftAppPassword*를 조회할 수 있는 **애플리케이션 설정** 블레이드가 없습니다. 암호를 가져오려면 암호를 생성해야 합니다. **봇 채널 등록**의 암호를 생성하려면 [봇 채널 등록 암호](bot-service-quickstart-registration.md#bot-channels-registration-password)를 참조하세요.

## <a name="next-steps"></a>다음 단계
Azure Portal에서 Bot Service 블레이드를 살펴보았으므로, 이제 온라인 코드 편집기를 사용하여 봇을 사용자 지정하는 방법에 대해 알아봅니다.
> [!div class="nextstepaction"]
> [온라인 코드 편집기 사용](bot-service-build-online-code-editor.md)
