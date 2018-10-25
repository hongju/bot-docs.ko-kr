---
title: Bot Service에 대한 연속 배포 구성 | Microsoft Docs
description: Bot Service에 대한 소스 제어에서 연속 배포를 설정하는 방법을 알아봅니다.
keywords: 연속 배포, 게시, 배포, azure portal
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
ms.openlocfilehash: 62cbbcc560e049776b8aa891c167b9a6eaba3264
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997800"
---
# <a name="set-up-continuous-deployment"></a>연속 배포 설정
코드가 **GitHub** 또는 **Azure DevOps(이전의 Visual Studio Team Services)** 로 체크 인되는 경우 지속적인 배포를 사용하여 원본 리포지토리에서 Azure로 코드 변경 내용을 자동으로 배포합니다. 이 항목에서는 **GitHub** 및 **Azure DevOps**에 대한 지속적인 배포 설정을 다룹니다.

> [!NOTE]
> 지속적인 배포를 설정하면 Azure Portal의 온라인 코드 편집기는 *읽기 전용*이 됩니다.

## <a name="continuous-deployment-using-github"></a>GitHub를 사용하여 연속 배포

GitHub를 사용하여 연속 배포를 설정하려면 다음을 수행합니다.

1. Azure에 배포하려는 원본 코드를 포함하는 GitHub 리포지토리를 사용합니다. 기존 리포지토리를 [포크](https://help.github.com/articles/fork-a-repo/)하거나 사용자 고유의 리포지토리를 만들고 GitHub 리포지토리로 관련 원본 코드를 업로드할 수 있습니다.
2. [Azure Portal](https://portal.azure.com)에서 봇의 **빌드** 블레이드로 이동하고 **지속적인 배포 구성**을 클릭합니다. 
3. **설치**를 클릭합니다.
   
   ![연속 배포 설정](~/media/azure-bot-build/continuous-deployment-setup.png)

4. **원본 선택**을 클릭하고 **GitHub**를 선택합니다.

   ![GitHub 선택](~/media/azure-bot-build/continuous-deployment-setup-github.png)

5. **권한 부여**를 클릭하고 **권한 부여** 단추를 클릭한 후 지시에 따라 GitHub 계정에 액세스하기 위한 Azure 권한을 부여합니다.

6. **프로젝트 선택**을 클릭하고 프로젝트를 선택합니다.

7. **분기 선택**을 클릭하고 분기를 선택합니다.

8. **확인**을 클릭하여 설치 프로세스를 완료합니다.

이제 GitHub 설치와 함께 지속적인 배포가 완료되었습니다. 원본 코드 리포지토리에 커밋할 때마다 변경 내용이 Azure Bot Service에 자동으로 배포됩니다.

## <a name="continuous-deployment-using-azure-devops"></a>Azure DevOps를 사용하여 지속적인 배포

1. [Azure Portal](https://portal.azure.com)의 봇의 **빌드** 블레이드에서 **지속적인 배포 구성**을 클릭합니다. 
2. **설치**를 클릭합니다.
   
   ![연속 배포 설정](~/media/azure-bot-build/continuous-deployment-setup.png)

3. **원본 선택**을 클릭하고 **Visual Studio Team Services**를 선택합니다. Visual Studio Team Services는 이제 Azure DevOps Services입니다.

   ![Visual Studio Team Services 선택](~/media/azure-bot-build/continuous-deployment-setup-vs.png)

4. **계정 선택**을 클릭하고 계정을 선택합니다.

> [!NOTE]
> 계정이 표시되지 않는 경우 Azure 구독에 연결되어 있는지 확인합니다. 계정을 Azure 구독에 연결하려면 Azure Portal로 이동하고, **Azure DevOps Services 조직(이전의 Team Services)** 을 엽니다. Azure DevOps에 있는 조직의 목록이 표시됩니다. 배포하려는 봇의 원본 코드를 사용하여 하나를 클릭하면 **AAD 연결** 단추가 표시됩니다. 선택하는 조직이 Azure 구독에 연결되지 않은 경우 이 단추가 활성화됩니다. 따라서 이 단추를 클릭하여 연결을 설정합니다. 적용될 때까지 잠시 기다려야 합니다.

5. **프로젝트 선택**을 클릭하고 프로젝트를 선택합니다.

> [!NOTE]
> VSTS Git 프로젝트만 지원됩니다.

6. **분기 선택**을 클릭하고 분기를 선택합니다.
7. **확인**을 클릭하여 설치 프로세스를 완료합니다.

   ![Visual Studio 구성](~/media/azure-bot-build/continuous-deployment-setup-vs-configuration.png)

이제 Azure DevOps 설치와 함께 지속적인 배포가 완료되었습니다. 커밋할 때마다 변경 내용이 Azure에 자동으로 배포됩니다.

## <a name="disable-continuous-deployment"></a>지속적 배포 사용 안 함

봇이 연속 배포용으로 구성된 동안 온라인 코드 편집기를 사용하여 봇을 변경하지 못할 수 있습니다. 온라인 코드 편집기를 사용하려는 경우 연속 배포를 일시적으로 사용하지 않도록 설정할 수 있습니다.

연속 배포를 사용하지 않도록 설정하려면 다음을 수행합니다.

1. 봇의 **빌드** 블레이드에서 **연속 배포 구성**을 클릭합니다. 
2. **연결 끊기**를 클릭하여 연속 배포를 사용하지 않도록 설정합니다. 연속 배포를 다시 사용하도록 설정하려면 위에서 해당 섹션의 단계를 반복합니다.

## <a name="additional-information"></a>추가 정보
- [Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/?view=vsts)
