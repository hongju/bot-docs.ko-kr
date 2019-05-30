---
title: Bot Service에 대한 연속 배포 구성 | Microsoft Docs
description: Bot Service에 대한 소스 제어에서 연속 배포를 설정하는 방법을 알아봅니다.
keywords: 연속 배포, 게시, 배포, azure portal
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: dc4b524659df3fb0b91b54fca65b4b7dd36378cd
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/24/2019
ms.locfileid: "66214256"
---
# <a name="set-up-continuous-deployment"></a>연속 배포 설정

[!INCLUDE [applies-to](./includes/applies-to.md)]

이 문서에서는 봇에 대해 지속적인 배포를 구성하는 방법을 보여 줍니다. 지속적인 배포를 사용하여 소스 리포지토리의 코드 변경 내용을 Azure에 자동으로 배포할 수 있습니다. 이 항목에서는 GitHub에 대한 지속적인 배포 설정을 다룹니다. 다른 소스 제어 시스템을 통한 지속적인 배포 설정은 이 페이지 하단의 추가 리소스 섹션을 참조하세요.

## <a name="prerequisites"></a>필수 조건
- Azure 구독이 아직 없는 경우 시작하기 전에 [체험 계정](http://portal.azure.com)을 만듭니다.
- 지속적인 배포를 사용하려면 먼저 [봇을 Azure에 배포](bot-builder-deploy-az-cli.md) **해야 합니다**.

## <a name="prepare-your-repository"></a>리포지토리 준비
리포지토리 루트에 프로젝트의 올바른 파일에 있는지 확인합니다. 이렇게 하면 Azure App Service Kudu 빌드 서버에서 자동 빌드를 가져올 수 있습니다. 

|런타임 | 루트 디렉터리 파일 |
|:-------|:---------------------|
| ASP.NET Core | .sln 또는 .csproj |
| Node.js | server.js, app.js 또는 package.json과 시작 스크립트 |


## <a name="continuous-deployment-using-github"></a>GitHub를 사용하여 연속 배포
GitHub를 통해 지속적인 배포를 사용하도록 설정하려면 Azure Portal에서 **App Service** 페이지로 이동합니다.

**배포 센터** > **GitHub** > **인증**을 클릭합니다.

![지속적인 배포](~/media/azure-bot-build/azure-deployment.png)

열리는 브라우저 창에서 **AzureAppService 권한 부여**를 클릭합니다. 

![Azure Github 권한](~/media/azure-bot-build/azure-deployment-github.png)

**AzureAppService** 권한 부여 후 Azure Portal의 **배포 센터**로 돌아갑니다.

1. **계속**을 클릭합니다. 

1. **App Service 빌드 서비스**를 선택합니다.

1. **계속**을 클릭합니다.

1. **조직**, **리포지토리**, **분기**를 선택합니다.

1. **계속**과 **마침**을 차례로 클릭하여 설정을 완료합니다.

이제 GitHub를 통한 지속적인 배포가 설정되었습니다. 원본 코드 리포지토리에 커밋할 때마다 변경 내용이 Azure Bot Service에 자동으로 배포됩니다.

## <a name="disable-continuous-deployment"></a>지속적 배포 사용 안 함

봇이 연속 배포용으로 구성된 동안 온라인 코드 편집기를 사용하여 봇을 변경하지 못할 수 있습니다. 온라인 코드 편집기를 사용하려는 경우 연속 배포를 일시적으로 사용하지 않도록 설정할 수 있습니다.

연속 배포를 사용하지 않도록 설정하려면 다음을 수행합니다.
1. [Azure Portal](https://portal.azure.com)에서 봇의 **모든 앱 서비스 설정** 블레이드로 이동하여 **배포 센터**를 클릭합니다. 
1. **연결 끊기**를 클릭하여 연속 배포를 사용하지 않도록 설정합니다. 연속 배포를 다시 사용하도록 설정하려면 위에서 해당 섹션의 단계를 반복합니다.

## <a name="additional-resources"></a>추가 리소스
- BitBucket 및 Azure DevOps Services에서 지속적인 배포를 사용하려면 [Azure App Service를 사용한 지속적인 배포](https://docs.microsoft.com/en-us/azure/app-service/deploy-continuous-deployment)를 참조하세요.


