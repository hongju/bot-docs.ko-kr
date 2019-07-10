---
title: Bot Service를 사용하여 봇 만들기 | Microsoft Docs
description: 통합형 전용 봇 개발 환경인 Bot Service를 사용하여 봇을 만드는 방법을 알아봅니다.
keywords: 빠른 시작, 봇 만들기, 봇 서비스, 웹앱 봇
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 05/31/2019
ms.openlocfilehash: 9ef15b09c8517db9e8b1f13f72172f09f9fb23eb
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464625"
---
# <a name="create-a-bot-with-azure-bot-service"></a>Azure Bot Service로 봇 만들기

::: moniker range="azure-bot-service-3.0"

> [!NOTE]
> **2019년 6월 10일부터 Azure Portal에서 V3 SDK 봇을 만들 수 없습니다. 고객은 앞으로 [V4 SDK](https://docs.microsoft.com/azure/bot-service/bot-service-quickstart?view=azure-bot-service-4.0) 봇을 만드는 것이 좋습니다. V3 SDK 장기 지원에 대한 자세한 내용은 [여기](https://docs.microsoft.com/azure/bot-service/bot-service-resources-bot-framework-faq?view=azure-bot-service-3.0#bot-framework-sdk-version-3-lifetime-support)** 에서 볼 수 있습니다. 

Bot Service는 봇 개발을 위한 Bot Framework SDK 및 채널에 봇을 연결하기 위한 Bot Framework를 포함하여 봇 만들기의 핵심 구성 요소를 제공합니다. Bot Service는 .NET 및 Node.js를 지원하는 봇을 만들 때 선택할 수 있는 5가지 템플릿을 제공합니다. 이 토픽에서는 Bot Service를 통해 Bot Framework SDK를 사용하는 새 봇 만드는 방법을 알아봅니다.

## <a name="log-in-to-azure"></a>Azure에 로그인
[Azure Portal](http://portal.azure.com)에 로그인합니다.

## <a name="create-a-new-bot-service"></a>새 봇 서비스 만들기

1. Azure Portal의 왼쪽 위 모서리에 있는 **새 리소스 만들기** 링크를 클릭하고 **AI + Machine Learning > 웹앱 봇**을 선택합니다. 

2. **웹앱 봇**에 대한 정보가 포함된 새 블레이드가 열립니다.  

3. **Bot Service** 블레이드에서 이미지 아래 표에 지정된 대로 요청된 봇 정보를 제공합니다.  <br/>
   ![웹앱 봇 만들기 블레이드](./media/azure-bot-quickstarts/sdk-create-bot-service-blade.png)

   | 설정 | 제안 값 | 설명 |
   | ---- | ---- | ---- |
   | **봇 이름** | 봇의 표시 이름 | 채널 및 디렉터리에 표시되는 봇의 표시 이름입니다. 이 이름은 언제든지 변경할 수 있습니다. |
   | **구독** | 사용자의 구독 | 사용할 Azure 구독을 선택합니다. |
   | **리소스 그룹** | myResourceGroup | 새 [리소스 그룹](/azure/azure-resource-manager/resource-group-overview#resource-groups)을 만들거나 기존 리소스 그룹을 선택할 수 있습니다. |
   | **위치**: | 기본 위치 | 리소스 그룹의 지리적 위치를 선택합니다. 나열된 모든 위치 중에 선택할 수 있지만, 고객에게 가장 가까운 위치를 선택하는 것이 좋습니다. 봇이 만들어진 후에는 위치를 변경할 수 없습니다. |
   | **가격 책정 계층** | F0 | 가격 책정 계층을 선택합니다. 언제든지 가격 책정 계층을 업데이트할 수 있습니다. 자세한 내용은 [Bot Service 가격](https://azure.microsoft.com/pricing/details/bot-service/)을 참조하세요. |
   | **앱 이름** | 고유한 이름 | 봇의 고유한 URL 이름입니다. 예를 들어, 봇 이름을 *myawesomebot*으로 지정하면 봇 URL은 `http://myawesomebot.azurewebsites.net`입니다. 이름에는 영숫자 및 밑줄 문자만 사용해야 합니다. 이 필드는 35자로 제한됩니다. 봇이 만들어진 후에는 앱 이름을 변경할 수 없습니다. |
   | **봇 템플릿** | Basic | **C#** 또는 **Node.js**를 선택하고, 이 빠른 시작의 **기본** 템플릿을 선택한 다음, **선택**을 클릭합니다. 기본 템플릿은 에코 봇을 만듭니다. 템플릿을 [자세히 알아봅니다](bot-service-concept-templates.md). |
   | **App Service 계획/위치** | App Service 계획  | [App Service 계획](https://azure.microsoft.com/pricing/details/app-service/plans/) 위치를 선택합니다. 나열된 모든 위치 중에 선택할 수 있지만, 고객에게 가장 가까운 위치를 선택하는 것이 좋습니다. (Functions 봇에 사용할 수 없음) |
   | **Application Insights** | 다른 | [Application Insights](/bot-framework/bot-service-manage-analytics)를 **켜기** 또는 **끄기**로 설정할지 결정합니다. **켜기**를 선택하면 지역 위치를 지정해야 합니다. 나열된 모든 위치 중에 선택할 수 있지만, 봇 서비스 위치와 동일한 위치를 선택하는 것이 대체적으로 가장 좋습니다. |
   | **Microsoft 앱 ID 및 암호** | 자동 앱 ID 및 암호 만들기 | Microsoft 앱 ID 및 암호를 수동으로 입력해야 하는 경우 이 옵션을 사용합니다. 그렇지 않으면 봇 만들기 프로세스에서 새 Microsoft 앱 ID 및 암호가 만들어집니다. Bot Service에 사용할 앱 등록을 수동으로 만드는 경우 지원되는 계정 유형을 ‘조직 디렉터리의 계정' 또는 ‘조직 디렉터리의 계정 및 개인 Microsoft 계정(예: Skype, Outlook.com, Xbox 등)’으로 설정해야 합니다.  |

   > [!NOTE]
   > 
   > **Functions 봇**을 만들 경우 **App Service 계획/위치** 필드가 표시되지 않습니다. 대신에 ‘호스팅 플랜’ 필드가 표시됩니다.  이 경우 [호스팅 플랜](bot-service-overview-readme.md#hosting-plans)을 선택합니다.

4. **만들기**를 클릭하여 서비스를 만들고 봇을 클라우드에 배포합니다. 이 프로세스에는 몇 분 정도 걸릴 수 있습니다.

**알림**을 선택하여 봇이 배포되었는지 확인합니다. 알림이 **배포 진행 중...** 에서 **배포 성공**으로 변경됩니다. **리소스로 이동** 단추를 클릭하여 봇의 리소스 블레이드를 엽니다.

 > [!TIP] 
 > 성능상의 이유로 Node.js 템플릿을 실행하는 **Functions 봇**은 *Azure Functions 팩* 도구로 이미 패키지되었습니다. *Azure Functions 팩* 도구는 모든 Node.js 모듈을 사용하고 하나의 *.js 파일로 결합했습니다.
 > 자세한 내용은 [Azure Functions Pack](https://github.com/Azure/azure-functions-pack)(Azure Functions 팩)을 참조하세요.
 
## <a name="test-the-bot"></a>봇 테스트
이제 봇을 만들었으므로 [웹 채팅](bot-service-manage-test-webchat.md)에서 테스트합니다. 메시지를 입력하면 봇이 응답합니다.

## <a name="next-steps"></a>다음 단계

이 항목에서는 Bot Service를 사용하여 **기본** 웹앱 봇/Functions 봇을 만드는 방법을 알아보고 Azure 내에서 기본 제공 웹 채팅 컨트롤을 사용하여 봇의 기능을 확인했습니다. 이제 봇을 관리하는 방법을 알아보고 원본 코드 작업을 시작합니다.

> [!div class="nextstepaction"]
> [봇 관리](bot-service-manage-overview.md)

::: moniker-end

::: moniker range="azure-bot-service-4.0"

[!INCLUDE [applies-to-v4](includes/applies-to.md)]

Azure Bot Service는 봇 개발을 위한 Bot Framework SDK 및 채널에 봇을 연결하기 위한 Bot Service를 포함하여 봇 만들기의 핵심 구성 요소를 제공합니다. 이 토픽에서 .NET 또는 Node.js 템플릿을 선택하여 Bot Framework SDK v4를 사용하는 봇을 만들 수 있습니다.

[!INCLUDE [Azure vs local development](~/includes/snippet-quickstart-paths.md)]

## <a name="prerequisites"></a>필수 조건

- [Azure](http://portal.azure.com) 계정

### <a name="create-a-new-bot-service"></a>새 봇 서비스 만들기

1. [Azure Portal](http://portal.azure.com/)에 로그인합니다.
1. Azure Portal의 왼쪽 위 모서리에 있는 **새 리소스 만들기** 링크를 클릭하고 **AI + 기계 학습** > **웹앱 봇**을 차례로 선택합니다. 

![봇 만들기](~/media/azure-bot-quickstarts/abs-create-blade.png)

2. **웹앱 봇**에 대한 정보가 포함된 *새 블레이드*가 열립니다.  

3. **Bot Service** 블레이드에서 이미지 아래 표에 지정된 대로 요청된 봇 정보를 제공합니다.  <br/>
 ![웹앱 봇 만들기 블레이드](~/media/azure-bot-quickstarts/sdk-create-bot-service-blade.png)

 | 설정 | 제안 값 | 설명 |
 | ---- | ---- | ---- |
 | **봇 이름** | 봇의 표시 이름 | 채널 및 디렉터리에 표시되는 봇의 표시 이름입니다. 이 이름은 언제든지 변경할 수 있습니다. |
 | **구독** | 사용자의 구독 | 사용할 Azure 구독을 선택합니다. |
 | **리소스 그룹** | myResourceGroup | 새 [리소스 그룹](/azure/azure-resource-manager/resource-group-overview#resource-groups)을 만들거나 기존 리소스 그룹을 선택할 수 있습니다. |
 | **위치**: | 기본 위치 | 리소스 그룹의 지리적 위치를 선택합니다. 나열된 모든 위치 중에 선택할 수 있지만, 고객에게 가장 가까운 위치를 선택하는 것이 좋습니다. 봇이 만들어진 후에는 위치를 변경할 수 없습니다. |
 | **가격 책정 계층** | F0 | 가격 책정 계층을 선택합니다. 언제든지 가격 책정 계층을 업데이트할 수 있습니다. 자세한 내용은 [Bot Service 가격](https://azure.microsoft.com/pricing/details/bot-service/)을 참조하세요. |
 | **앱 이름** | 고유한 이름 | 봇의 고유한 URL 이름입니다. 예를 들어, 봇 이름을 *myawesomebot*으로 지정하면 봇 URL은 `http://myawesomebot.azurewebsites.net`입니다. 이름에는 영숫자 및 밑줄 문자만 사용해야 합니다. 이 필드는 35자로 제한됩니다. 봇이 만들어진 후에는 앱 이름을 변경할 수 없습니다. |
 | **봇 템플릿** | Echo 봇 | **SDK v4**를 선택합니다. 이 빠른 시작에 대해 C# 또는 Node.js를 선택한 다음, **선택**을 클릭합니다.  
 | **App Service 계획/위치** | App Service 계획  | [App Service 계획](https://azure.microsoft.com/pricing/details/app-service/plans/) 위치를 선택합니다. 나열된 모든 위치 중에 선택할 수 있지만, 봇 서비스와 동일한 위치를 선택하는 것이 대체적으로 가장 좋습니다. |
 | **Application Insights** | 다른 | [Application Insights](/bot-framework/bot-service-manage-analytics)를 **켜기** 또는 **끄기**로 설정할지 결정합니다. **켜기**를 선택하면 지역 위치를 지정해야 합니다. 나열된 모든 위치 중에 선택할 수 있지만, 봇 서비스와 동일한 위치를 선택하는 것이 대체적으로 가장 좋습니다. |
 | **Microsoft 앱 ID 및 암호** | 자동 앱 ID 및 암호 만들기 | Microsoft 앱 ID 및 암호를 수동으로 입력해야 하는 경우 이 옵션을 사용합니다. 그렇지 않으면 봇 만들기 프로세스에서 새 Microsoft 앱 ID 및 암호가 만들어집니다. Bot Service에 사용할 앱 등록을 수동으로 만드는 경우 지원되는 계정 유형을 ‘조직 디렉터리의 계정' 또는 ‘조직 디렉터리의 계정 및 개인 Microsoft 계정(예: Skype, Outlook.com, Xbox 등)’으로 설정해야 합니다. |

4. **만들기**를 클릭하여 서비스를 만들고 봇을 클라우드에 배포합니다. 이 프로세스에는 몇 분 정도 걸릴 수 있습니다.

**알림**을 선택하여 봇이 배포되었는지 확인합니다. 알림이 **배포 진행 중...** 에서 **배포 성공**으로 변경됩니다. **리소스로 이동** 단추를 클릭하여 봇의 리소스 블레이드를 엽니다.

이제 봇을 만들었으므로 웹 채팅에서 테스트합니다. 

## <a name="test-the-bot"></a>봇 테스트
**봇 관리** 섹션에서 **웹 채팅에서 테스트**를 클릭합니다. Azure Bot Service가 웹 채팅 컨트롤을 로드하고 봇에 연결합니다. 

![Azure 웹 채팅 테스트](./media/azure-bot-quickstarts/azure-webchat-test.png)

메시지를 입력하면 봇이 응답합니다.

## <a name="download-code"></a>코드 다운로드
코드를 다운로드하여 로컬로 작업할 수 있습니다. 
1. **봇 관리** 섹션에서 **빌드**를 클릭합니다. 
1. 오른쪽 창에서 **Bot 소스 코드 다운로드** 링크를 클릭합니다. 
1. 표시되는 메시지에 따라 코드를 다운로드한 다음, 폴더의 압축을 풉니다.
    1. [!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]

## <a name="next-steps"></a>다음 단계
코드가 다운로드되면 머신에서 봇을 로컬로 계속 개발할 수 있습니다. 봇을 테스트하고 봇 코드를 Azure Portal에 업로드할 준비가 되면 [연속 배포 설정](./bot-service-build-continuous-deployment.md) 항목에 나열된 지침에 따라 변경한 후 자동으로 코드를 업데이트합니다.

::: moniker-end
