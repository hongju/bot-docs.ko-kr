---
title: Azure에 봇 배포 | Microsoft Docs
description: Azure 클라우드에 봇을 배포합니다.
keywords: 봇 배포, Azure 배포, 봇 채널 등록, visual studio 게시
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 09/19/2018
ms.openlocfilehash: 5f3fc43bed16d50c347d4e07a9154dfd307f0dc0
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999370"
---
# <a name="deploy-your-bot-to-azure"></a>Azure에 봇 배포

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

봇을 생성하고 로컬에서 확인한 후에는 어디서나 액세스할 수 있도록 Azure로 푸시할 수 있습니다. C# 봇의 경우 Visual Studio 또는 Azure CLI를 사용하여 봇을 Azure에 게시할 수 있습니다. 

## <a name="publish-from-visual-studio"></a>Visual Studio에서 게시
먼저 App Service의 Visual Studio에서 Azure에 봇을 배포합니다. 그런 다음, 봇 채널 등록을 사용하여 Azure Bot Service로 봇을 구성합니다.

[솔루션 탐색기] 창에서 프로젝트 노드를 마우스 오른쪽 단추로 클릭하고 [게시]를 선택합니다.

![게시 설정](media/azure-bot-quickstarts/getting-started-publish-setting.png)

2. Pick a publish target(게시 대상 선택) 대화 상자에서 왼쪽에 **App Service**가 선택되어 있고 오른쪽에 **새로 만들기**가 선택되어 있어야 합니다.

3. [게시] 단추를 클릭합니다.

4. 대화 상자의 오른쪽 위에서 Azure 구독에 대한 올바른 사용자 ID가 대화 상자에 표시되는지 확인합니다.

![게시 메인](media/azure-bot-quickstarts/getting-started-publish-main.png)

5. 앱 이름, 구독, 리소스 그룹 및 호스팅 계획 정보를 입력합니다.

6. 준비되면 [만들기]를 클릭합니다. 이 프로세스를 완료하는 데 몇 분이 걸릴 수 있습니다.

7. 완료되면 봇이 공용 URL을 보여주는 웹 브라우저가 열립니다.

8. 이 URL(https://<yourbotname>.azurewebsites.net/와 유사함)을 복사해 둡니다.

> [!NOTE] 
> 봇을 등록할 때 HTTPS 버전의 URL을 사용해야 합니다. Azure는 Azure App Service에서 SSL 지원을 제공합니다.

## <a name="create-your-bot-channels-registration"></a>봇 채널 등록 만들기
봇이 Azure에 배포되면 Azure Bot Service에 등록해야 합니다.

1. https://portal.azure.com에서 Azure Portal에 액세스합니다.

2. 이전에 Visual Studio에서 봇을 게시할 때 사용한 ID와 동일한 ID를 사용하여 로그인합니다.

3. [리소스 만들기]를 클릭합니다.

4. [Marketplace 검색] 필드에 봇 채널 등록을 입력하고 Enter를 누릅니다.

5. 반환된 목록에서 [봇 채널 등록]을 선택합니다.

![게시](media/azure-bot-quickstarts/getting-started-bot-registration.png)

6. 열리는 블레이드에서 [만들기]를 클릭합니다.

7. 봇의 이름을 제공합니다.

8. 봇의 코드를 배포한 구독과 동일한 구독을 선택합니다.

9. 위치를 설정할 기존 리소스 그룹을 선택합니다.

10. 개발 및 테스트를 위해 F0 가격 책정 계층을 선택할 수 있습니다.

11. 봇의 URL을 입력합니다. HTTPS로 시작해야 하고 /api/messages를 추가해야 합니다. 예 https://yourbotname.azurewebsites.net/api/messages

12. 이제 Application Insights를 끕니다.

13. [Microsoft 앱 ID 및 암호]를 클릭합니다.

14. 새 블레이드에서 [새로 만들기]를 클릭합니다.

15. 오른쪽에 열리는 새 블레이드에서 "Create App ID in the App Registration Portal"(앱 등록 포털에 앱 ID 생성)을 클릭하면 새 브라우저 탭이 열립니다.

![봇 MSA](media/azure-bot-quickstarts/getting-started-msa.png)

16. 새 탭에서 앱 ID의 복사본을 만들고 원하는 위치에 저장합니다. 

17. [계속하려면 앱 암호 생성] 단추를 클릭합니다.

18. 브라우저 대화 상자가 열리고 앱의 암호가 제공됩니다. 이때만 암호가 제공됩니다. 이 암호를 복사하여 나중에 사용할 수 있도록 원하는 곳에 저장합니다.

19. 암호가 저장되면 [확인]을 클릭합니다.

20. 브라우저 탭을 닫고 Azure Portal 탭으로 돌아갑니다.

21. 앱 ID와 암호를 올바른 필드에 붙여넣고 [확인]을 클릭합니다.

22. 이제 [만들기]를 클릭하여 채널 등록을 설정합니다. 몇 초에서 몇 분 정도 걸릴 수 있습니다.

## <a name="update-your-bots-application-settings"></a>봇의 응용 프로그램 설정 업데이트
Azure Bot Service로 봇을 인증하려면 Azure App Service에서 봇의 응용 프로그램 설정에 두 가지 설정을 추가해야 합니다. 

1. [App Services]를 클릭합니다. [구독] 텍스트 상자에 봇의 이름을 입력합니다. 다음으로, 목록에서 봇의 이름을 클릭합니다.

![앱 서비스](media/azure-bot-quickstarts/getting-started-app-service.png)

2. 봇 옵션 내에서 왼쪽에 있는 옵션 목록에서 [설정] 섹션의 [응용 프로그램 설정]을 찾아서 클릭합니다.

![봇 ID](media/azure-bot-quickstarts/getting-started-app-settings-1.png)

3. 응용 프로그램 설정 섹션을 찾을 때까지 스크롤합니다.

![봇 MSA](media/azure-bot-quickstarts/getting-started-app-settings-2.png)

4. [새 설정 추가]를 클릭합니다.

5. 이름에 **MicrosoftAppId**을 입력하고 값에 앱 ID를 입력합니다.

6. [새 설정 추가]를 클릭합니다.

7. 이름에 **MicrosoftAppPassword**를 입력하고 값에 암호를 입력합니다.

8. 위쪽의 [저장] 단추를 클릭합니다.

## <a name="test-your-bot-in-production"></a>프로덕션 환경에서 봇 테스트
이 시점에서 기본 제공되는 웹 채팅 클라이언트를 사용하여 Azure에서 봇을 테스트할 수 있습니다.

1. Azure Portal의 리소스 그룹으로 돌아갑니다.

2. 봇을 엽니다.

3. **봇 관리**에서 **웹 채팅에서 테스트**를 선택합니다.

![웹 채팅에서 테스트](media/azure-bot-quickstarts/getting-started-test-webchat.png)

4. `Hi`와 같은 메시지를 입력하고 Enter 키를 누릅니다. 봇이 `Turn 1: You sent Hi`라고 반향합니다.

## <a name="next-steps"></a>다음 단계
> [!div class="nextstepaction"]
> [지속적인 배포 설정](bot-service-build-continuous-deployment.md)
