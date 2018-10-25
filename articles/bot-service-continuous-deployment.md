---
title: 소스 제어 또는 Visual Studio에서 Bot Service 게시 | Microsoft Docs
description: Visual Studio에서 한 번 또는 소스 제어로부터 지속적으로 Bot Service 봇을 게시하는 방법을 살펴봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
ms.openlocfilehash: 38b26ed5a50409de64518562faabf532f45c857e
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999150"
---
# <a name="publish-a-bot-to-bot-service"></a>Bot Service에 봇 게시

C# 봇 소스 코드를 업데이트한 후 Visual Studio를 사용하여 게시하여 Bot Service에서 봇을 실행할 수 있습니다. 소스 파일을 확인할 때마다 자동으로 IDE(통합 개발 환경)에서 작성된 C# 또는 Node.js 봇 소스 코드를 소스 제어 서비스에 게시할 수도 있습니다.


## <a name="publish-a-bot-on-app-service-plan-from-the-online-code-editor"></a>온라인 코드 편집기에서 App Service 계획에 봇 게시

지속적인 배포를 구성하지 않은 경우 온라인 코드 편집기에서 소스 파일을 수정할 수 있습니다. 수정된 소스를 배포하려면 다음 단계를 따릅니다.

4. 콘솔 열기 아이콘을 클릭합니다.  
    ![콘솔 아이콘](~/media/azure-bot-service-console-icon.png)
2. 콘솔 창에서 **build.cmd**를 입력하고 Enter 키를 누릅니다.


## <a name="publish-c-bot-on-app-service-plan-from-visual-studio"></a>Visual Studio에서 App Service 계획에서 C# 봇 게시 

Visual Studio에서 `.PublishSettings` 파일을 사용하여 게시를 설정하려면 다음 단계를 수행합니다.

1. Azure Portal에서 Bot 서비스를 클릭하고, **BUILD** 탭을 클릭한 다음, **zip 파일 다운로드**를 클릭합니다.
3. 다운로드한 zip 파일 콘텐츠의 압축을 로컬 폴더에 풉니다.
4. 탐색기에서 봇에 대한 Visual Studio Solution(.sln) 파일을 찾아 두 번 클릭합니다.
4. Visual Studio에서 **보기**를 클릭하고 **솔루션 탐색기**를 클릭합니다.
5. 솔루션 탐색기에서 프로젝트를 마우스 오른쪽 단추로 클릭하고 **게시...** 를 클릭합니다. 게시 창이 열립니다. 
6. 게시 창에서 **새 프로필 만들기**를 클릭하고 **프로필 가져오기**를 클릭한 다음, **확인**을 클릭합니다.
7. 프로젝트 폴더로 이동한 다음, **PostDeployScripts** 폴더로 이동하고 `.PublishSettings`로 끝나는 파일을 선택하고 **열기**를 클릭합니다.

이제 이 프로젝트에 대한 게시를 구성했습니다. Bot Service에 로컬 소스 코드를 게시하려면 프로젝트를 마우스 오른쪽 단추로 클릭하고 **게시...** 를 클릭한 다음, **게시** 단추를 클릭합니다. 

## <a name="set-up-continuous-deployment"></a>연속 배포 설정

기본적으로 Bot Service를 사용하면 로컬 편집기나 소스 제어 없이 Azure 편집기를 사용하여 브라우저에서 바로 봇을 개발할 수 있습니다. 그러나 Azure 편집기에서는 응용 프로그램 내 파일을 관리할 수 없습니다(예: 파일 추가, 파일 이름 변경 또는 파일 삭제). 응용 프로그램 내 파일을 관리하는 기능이 필요할 경우 지속적인 배포를 설정하고 원하는 IDE(통합 배포 환경)과 소스 제어 시스템(예: Visual Studio Team, GitHub, Bitbucket)을 사용할 수 있습니다. 지속적인 배포를 구성하면 소스 제어에 대해 수행된 모든 코드 변경 내용이 자동으로 Azure에 배포됩니다. 지속적인 배포를 구성한 후 [봇을 로컬로 디버그](bot-service-debug-bot.md)할 수 있습니다.

> [!NOTE]
> 봇에 지속적인 배포를 사용할 경우 소스 제어 서비스에서 코드 변경 내용을 확인해야 합니다. Azure 편집기에서 다시 코드를 편집하려면 [지속적인 배포를 사용하지 않도록 설정](#disable-continuous-deployment)해야 합니다.

다음 단계를 완료하여 봇 앱에 대해 지속적인 배포를 사용하도록 설정할 수 있습니다.

## <a name="set-up-continuous-deployment-for-a-bot-on-an-app-service-plan"></a>App Service 계획에 대해 봇의 지속적인 배포 설정

이 섹션에서는 App Service 호스팅 계획이 있는 Bot Service를 사용하여 만든 봇에 지속적인 배포를 사용하도록 설정하는 방법을 설명합니다.

1. Azure Portal 안에서 Azure 봇을 찾고 **BUILD** 탭을 클릭하고 **소스 제어에서 지속적인 배포** 섹션을 찾습니다.
2. Visual Studio Online 또는 GitHub에 대해 해당 웹 사이트에서 나에게 발행된 액세스 토큰을 제공합니다. 소스를 Azure에서 소스 리포지토리로 가져옵니다.
3. 다른 소스 제어 시스템의 경우 **기타**를 선택하고 나타나는 단계를 따릅니다. 
3. **사용**을 클릭합니다.  

### <a name="create-an-empty-repository-and-download-bot-source-code"></a>빈 리포지토리를 만들고 봇 소스 코드 다운로드

Visual Studio Online 또는 Github *이외의* 소스 제어 서비스를 사용하려면 이 단계를 따릅니다. Visual Studio Online 및 Github가 Azure에서 봇에 대한 소스 코드를 가져오므로 이 두 서비스 사용자는 이 단계를 건너뛸 수 있습니다.

3. App Service 계획의 봇의 경우 Azure에서 봇 페이지를 찾고 **BUILD** 탭을 클릭한 다음, **소스 코드 다운로드** 섹션을 찾아 **zip 파일 다운로드**를 클릭합니다.
1. Azure에서 지원하는 소스 제어 시스템 중 하나에서 빈 리포지토리를 만듭니다.

    ![소스 제어 시스템](~/media/continuous-integration-sourcecontrolsystem.png)

3. 다운로드한 zip 파일의 콘텐츠 압축을 배포 소스를 동기화하려는 로컬 폴더에 풉니다.
4. **구성**을 클릭하고 나타나는 단계를 따릅니다. 

## <a name="set-up-continuous-deployment-for-a-bot-on-a-consumption-plan"></a>Consumption 계획에 대해 봇의 지속적인 배포 설정 

봇에 대해 배포 소스를 선택하고 리포지토리에 연결합니다. 

1. Azure Poral의 Azure 봇에서 **SETTINGS** 탭을 클릭하고 **구성**을 클릭하여 **지속적인 배포**를 확장합니다.  
2. 단계를 따르고 확인란을 선택하여 준비가 되었음을 확인합니다. 
3. **구성**을 클릭하고 이전에 빈 리포지토리를 만든 소스 제어 시스템에 해당하는 배포 소스를 선택하고 단계를 완료하여 연결합니다.   


## <a name="disable-continuous-deployment"></a>지속적 배포 사용 안 함 

지속적인 배포를 사용하지 않으면 소스 제어 서비스가 계속 작동은 하지만 체크인한 변경 내용이 자동으로 Azure에 게시되지는 않습니다. 지속적인 배포를 사용하지 않으려면 다음 단계를 수행합니다.

1. 봇에 App Service 호스팅 계획이 있는 경우 Azure Portal 안에서 Azure 봇을 찾고 **BUILD** 탭을 클릭하고 **소스 제어에서 지속적인 배포** 섹션, *또는...* 을 찾습니다. 
2. 봇에 Consumption 계획이 있으면 **설정** 탭을 클릭하고 **지속적인 배포** 섹션을 확장한 다음, **구성**을 클릭합니다.
3. **배포** 창에서 지속적인 배포를 사용하도록 설정된 소스 제어 서비스를 선택하고 **연결 끊기**를 클릭합니다.  


## <a name="additional-resources"></a>추가 리소스

지속적인 배포를 구성한 후 로컬로 봇을 디버그하는 방법은 [Bot Service 봇 디버그](bot-service-debug-bot.md)를 참조하세요.

이 문서에서는 Bot Service의 특정 지속적인 배포 기능에 초점을 맞춥니다. 지속적인 배포는 Azure App Services와 관련이 있으므로 자세한 내용은 <a href="https://azure.microsoft.com/en-us/documentation/articles/app-service-continuous-deployment/" target="_blank">Azure App Service에 지속적인 배포</a>를 참조하세요.
