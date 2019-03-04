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
ms.date: 12/13/2017
ms.openlocfilehash: fef82d27c6dd4fb61c9a0cf864e76a88128847d7
ms.sourcegitcommit: 05ddade244874b7d6e2fc91745131b99cc58b0d6
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/21/2019
ms.locfileid: "56591164"
---
# <a name="manage-a-bot"></a>봇 관리

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

**애플리케이션 설정** 블레이드에는 봇의 환경, 디버그 설정, botFilePath 및 botFileSecret와 같은 애플리케이션 설정 키 등 봇에 대한 자세한 정보가 포함되어 있습니다.

### <a name="microsoftappid-and-microsoftapppassword"></a>MicrosoftAppID 및 MicrosoftAppPassword

**MicrosoftAppID** 및 **MicrosoftAppPassword**는 봇의 `.bot` 파일에 보관됩니다. 이를 가져오려면 봇 파일을 다운로드하고 암호를 해독하세요. ID 및 암호를 사용하여 로컬에서 테스트하는 데 필요할 수 있습니다.

### <a name="bot-file-path-and-secret"></a>봇 파일 경로 및 비밀

**애플리케이션 설정** 블레이드에서 봇의 **botFilePath** 및 **botFileSecret**를 찾을 수 있습니다.

![Microsoft 봇 파일 경로 및 비밀](~/media/azure-manage-a-bot/app-settings.png)

> [!NOTE]
> **봇 채널 등록** 봇 서비스에는 *MicrosoftAppID*가 제공되지만 이 유형의 서비스와 연결된 App Service가 없기 때문에 *MicrosoftAppPassword*를 조회할 수 있는 **애플리케이션 설정** 블레이드가 없습니다. 암호를 가져오려면 암호를 생성해야 합니다. **봇 채널 등록**의 암호를 생성하려면 [봇 채널 등록 암호](bot-service-quickstart-registration.md#bot-channels-registration-password)를 참조하세요.

## <a name="next-steps"></a>다음 단계
Azure Portal에서 Bot Service 블레이드를 살펴보았으므로, 이제 온라인 코드 편집기를 사용하여 봇을 사용자 지정하는 방법에 대해 알아봅니다.
> [!div class="nextstepaction"]
> [온라인 코드 편집기 사용](bot-service-build-online-code-editor.md)
