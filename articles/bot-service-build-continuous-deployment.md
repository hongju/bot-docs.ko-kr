---
redirect_url: /bot-framework/bot-builder-deploy-az-cli
ms.openlocfilehash: 8149471553658df6778e2983bae114e80c846c9b
ms.sourcegitcommit: 8183bcb34cecbc17b356eadc425e9d3212547e27
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/09/2019
ms.locfileid: "55971431"
---
<a name="--"></a><!--
---
제목: Bot Service에 대한 지속적인 배포 구성 | Microsoft Docs 설명: Bot Service에 대한 소스 제어에서 연속 배포를 설정하는 방법을 알아봅니다. 키워드: 지속적인 배포, 게시, 배포, Azure Portal 작성자: ivorb ms.author: v-ivorb 관리자: kamrani ms.topic: article ms.service: bot-service ms.date: 12/06/2018
---

# <a name="set-up-continuous-deployment"></a>연속 배포 설정
코드가 **GitHub** 또는 **Azure DevOps(이전의 Visual Studio Team Services)** 로 체크 인되는 경우 지속적인 배포를 사용하여 원본 리포지토리에서 Azure로 코드 변경 내용을 자동으로 배포합니다. 이 항목에서는 **GitHub** 및 **Azure DevOps**에 대한 지속적인 배포 설정을 다룹니다.

> [!NOTE]
> 이 문서에서 다루는 시나리오에서는 봇을 Azure에 배포했다고 가정하고, 이제는 해당 봇에 대한 지속적인 배포를 사용하도록 설정하려고 합니다. 또한 지속적인 배포가 설정되면 Azure Portal의 온라인 코드 편집기가 읽기 전용이 됩니다.

## <a name="continuous-deployment-using-github"></a>GitHub를 사용하여 연속 배포

Azure에 배포하려는 소스 코드가 포함된 GitHub 리포지토리를 사용하여 지속적인 배포를 설정하려면 다음을 수행합니다.

1. [Azure Portal](https://portal.azure.com)에서 봇의 **모든 앱 서비스 설정** 블레이드로 이동하여 **배포 옵션(클래식)** 을 클릭합니다. 

1. **원본 선택**을 클릭하고 **GitHub**를 선택합니다.

   ![GitHub 선택](~/media/azure-bot-build/continuous-deployment-setup-github.png)

1. **권한 부여**를 클릭하고 **권한 부여** 단추를 클릭한 후 지시에 따라 GitHub 계정에 액세스하기 위한 Azure 권한을 부여합니다.

1. **프로젝트 선택**을 클릭하고 프로젝트를 선택합니다.

1. **분기 선택**을 클릭하고 분기를 선택합니다.

1. **확인**을 클릭하여 설치 프로세스를 완료합니다.

이제 GitHub 설치와 함께 지속적인 배포가 완료되었습니다. 원본 코드 리포지토리에 커밋할 때마다 변경 내용이 Azure Bot Service에 자동으로 배포됩니다.

## <a name="continuous-deployment-using-azure-devops"></a>Azure DevOps를 사용하여 지속적인 배포

1. [Azure Portal](https://portal.azure.com)에서 봇의 **모든 앱 서비스 설정** 블레이드로 이동하여 **배포 옵션(클래식)** 을 클릭합니다. 
2. **원본 선택**을 클릭하고 **Visual Studio Team Services**를 선택합니다. Visual Studio Team Services는 이제 Azure DevOps Services입니다.

   ![Visual Studio Team Services 선택](~/media/azure-bot-build/continuous-deployment-setup-vs.png)

3. **계정 선택**을 클릭하고 계정을 선택합니다.

> [!NOTE]
> 계정이 표시되지 않으면 [계정을 Azure 구독에 연결](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/connect-organization-to-azure-ad?view=vsts&tabs=new-nav)해야 합니다. VSTS Git 프로젝트만 지원됩니다.

4. **프로젝트 선택**을 클릭하고 프로젝트를 선택합니다.
5. **분기 선택**을 클릭하고 분기를 선택합니다.
6. **확인**을 클릭하여 설치 프로세스를 완료합니다.

   ![Visual Studio 구성](~/media/azure-bot-build/continuous-deployment-setup-vs-configuration.png)

이제 Azure DevOps 설치와 함께 지속적인 배포가 완료되었습니다. 커밋할 때마다 변경 내용이 Azure에 자동으로 배포됩니다.

## <a name="disable-continuous-deployment"></a>지속적 배포 사용 안 함

봇이 연속 배포용으로 구성된 동안 온라인 코드 편집기를 사용하여 봇을 변경하지 못할 수 있습니다. 온라인 코드 편집기를 사용하려는 경우 연속 배포를 일시적으로 사용하지 않도록 설정할 수 있습니다.

연속 배포를 사용하지 않도록 설정하려면 다음을 수행합니다.
1. [Azure Portal](https://portal.azure.com)에서 봇의 **모든 앱 서비스 설정** 블레이드로 이동하여 **배포 옵션(클래식)** 을 클릭합니다. 
2. **연결 끊기**를 클릭하여 연속 배포를 사용하지 않도록 설정합니다. 연속 배포를 다시 사용하도록 설정하려면 위에서 해당 섹션의 단계를 반복합니다.

## <a name="additional-information"></a>추가 정보
- Visual Studio Team Services는 이제 [Azure DevOps Services](https://docs.microsoft.com/en-us/azure/devops/?view=vsts)입니다.


-->
