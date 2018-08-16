---
title: Bot Service에 대한 연속 배포 구성 | Microsoft Docs
description: Bot Service에 대한 소스 제어에서 연속 배포를 설정하는 방법을 알아봅니다.
keywords: 연속 배포, 게시, 배포, azure portal
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/08/2018
ms.openlocfilehash: 596d264c4df72959c71ab353e5038175fc2bed31
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301867"
---
# <a name="set-up-continuous-deployment"></a>연속 배포 설정

연속 배포를 사용하면 봇을 로컬로 개발할 수 있습니다. 연속 배포는 봇이 **GitHub** 또는 **Visual Studio Team Services**와 같은 소스 제어에 체크 인하는 경우에 유용합니다. 변경 내용을 소스 리포지토리로 다시 체크 인하면 변경 내용이 자동으로 Azure에 배포됩니다.

> [!NOTE]
> 연속 배포를 설정하면 [온라인 코드 편집기](bot-service-build-online-code-editor.md)는 *읽기 전용*이 됩니다.

이 항목에서는 **GitHub** 및 **Visual Studio Team Services**에 대해 연속 배포를 설정하는 방법을 보여 줍니다.

## <a name="continuous-deployment-using-github"></a>GitHub를 사용하여 연속 배포

GitHub를 사용하여 연속 배포를 설정하려면 다음을 수행합니다.

1. Azure에 배포하려는 코드를 포함하는 GitHub 리포지토리를 [포크](https://help.github.com/articles/fork-a-repo/)합니다.
2. Azure Portal에서 봇의 **빌드** 블레이드로 이동한 후 **연속 배포 구성**을 클릭합니다. 
3. **설치**를 클릭합니다.
   
   ![연속 배포 설정](~/media/azure-bot-build/continuous-deployment-setup.png)

4. **원본 선택**을 클릭하고 **GitHub**를 선택합니다.

   ![GitHub 선택](~/media/azure-bot-build/continuous-deployment-setup-github.png)

5. **권한 부여**를 클릭하고 **권한 부여** 단추를 클릭한 후 지시에 따라 GitHub 계정에 액세스하기 위한 Azure 권한을 부여합니다.

GitHub 설치와 함께 연속 배포가 완료되었습니다. 커밋할 때마다 변경 내용이 Azure에 자동으로 배포됩니다.

## <a name="continuous-deployment-using-visual-studio"></a>Visual Studio를 사용한 연속 배포

1. 봇의 **빌드** 블레이드에서 **연속 배포 구성**을 클릭합니다. 
2. **설치**를 클릭합니다.
   
   ![연속 배포 설정](~/media/azure-bot-build/continuous-deployment-setup.png)

3. **원본 선택**을 클릭하고 **Visual Studio Team Services**를 선택합니다.

   ![Visual Studio Team Services 선택](~/media/azure-bot-build/continuous-deployment-setup-vs.png)

4. **계정 선택**을 클릭하고 계정을 선택합니다.

> [!NOTE]
> 계정이 표시되지 않는 경우 Azure 구독에 연결되어 있는지 확인합니다.
> 자세한 내용은 [Azure 구독에 VSTS 계정 연결](https://github.com/projectkudu/kudu/wiki/Setting-up-a-VSTS-account-so-it-can-deploy-to-a-Web-App#linking-your-vsts-account-to-your-azure-subscription)을 참조하세요.

5. **프로젝트 선택**을 클릭하고 프로젝트를 선택합니다.

> [!NOTE]
> VSTS Git 프로젝트만 지원됩니다.

6. **분기 선택**을 클릭하고 분기할 분기를 선택합니다.
7. **확인**을 클릭하여 설치 프로세스를 완료합니다.

   ![Visual Studio 구성](~/media/azure-bot-build/continuous-deployment-setup-vs-configuration.png)

Visual Studio Team Services 설치를 사용한 연속 배포가 완료되었습니다. 커밋할 때마다 변경 내용이 Azure에 자동으로 배포됩니다.

## <a name="disable-continuous-deployment"></a>지속적 배포 사용 안 함

봇이 연속 배포용으로 구성된 동안 온라인 코드 편집기를 사용하여 봇을 변경하지 못할 수 있습니다. 온라인 코드 편집기를 사용하려는 경우 연속 배포를 일시적으로 사용하지 않도록 설정할 수 있습니다.

연속 배포를 사용하지 않도록 설정하려면 다음을 수행합니다.

1. 봇의 **빌드** 블레이드에서 **연속 배포 구성**을 클릭합니다. 
2. **연결 끊기**를 클릭하여 연속 배포를 사용하지 않도록 설정합니다. 연속 배포를 다시 사용하도록 설정하려면 위에서 해당 섹션의 단계를 반복합니다.

## <a name="next-steps"></a>다음 단계
이제 봇이 연속 배포용으로 설정되었으므로 온라인 웹 채팅을 사용하여 코드를 테스트합니다.

> [!div class="nextstepaction"]
> [웹 채팅에서 테스트](bot-service-manage-test-webchat.md)
