---
title: 엔터프라이즈 봇 새 프로젝트 만들기 | Microsoft Docs
description: 엔터프라이즈 봇 템플릿을 기반으로 하여 새 봇을 만드는 방법을 알아봅니다.
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: fc7b168cc8c509b81539682a3717b54ed5c2109c
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997120"
---
# <a name="enterprise-bot-template---creating-a-new-project"></a>엔터프라이즈 봇 템플릿 - 새 프로젝트 만들기

> [!NOTE]
> 이 항목은 SDK v4 버전에 적용됩니다. 

엔터프라이즈 봇 템플릿은 대화형 환경 구축을 통해 식별한 모범 사례와 지원 구성 요소를 모두 제공합니다. 템플릿을 사용할 수 있는 Bot Builder SDK 플랫폼은 다음과 같습니다.

- .NET
- Node.js(출시 예정)

## <a name="net"></a>.NET

엔터프라이즈 봇 템플릿은 SDK **V4** 버전을 대상으로 하는 .NET에서 사용할 수 있습니다. 이 템플릿은 [VSIX](https://docs.microsoft.com/en-us/visualstudio/extensibility/anatomy-of-a-vsix-package) 패키지로 제공됩니다. 다운로드하려면 다음 링크를 클릭하세요.

- [Bot Builder SDK V4 엔터프라이즈 봇 템플릿](https://aka.ms/GetEnterpriseBotTemplate)

#### <a name="prerequisites"></a>필수 조건

- [Visual Studio 2017 이상](https://www.visualstudio.com/downloads/)
- [Azure 계정](https://azure.microsoft.com/en-us/free/)
- [Azure PowerShell](https://docs.microsoft.com/en-us/powershell/azure/overview?view=azurermps-6.8.1)

### <a name="install-the-template"></a>템플릿 설치

저장된 디렉터리에서 VSIX 패키지를 열기만 하면 엔터프라이즈 봇 템플릿이 Visual Studio에 설치되며, 다음에 템플릿을 열 때 사용할 수 있게 됩니다.

템플릿을 사용하여 새 봇 프로젝트를 만들려면 Visual Studio를 열고, **파일** > **새로 만들기** > **프로젝트**를 차례로 선택한 다음, Visual C#에서 **Bot Framework** > Enterprise Bot Template을 차례로 선택합니다. 이렇게 하면 원하는 대로 편집할 수 있는 새 봇 프로젝트가 로컬로 만들어집니다. 

![새 프로젝트 템플릿 파일](media/enterprise-template/EnterpriseBot-NewProject.png)

## <a name="deploy-your-bot"></a>봇 배포

이제 프로젝트가 만들어졌으므로 다음 단계에서는 봇을 즉시 사용할 수 있도록 지원되는 Azure 인프라를 만들고 구성/배포를 수행합니다. [봇 배포](bot-builder-enterprise-template-deployment.md)로 계속 진행하세요.

> 이 단계는 반드시 실행해야 합니다. 그렇지 않으면 봇 초기화(AppInsights) 및 LUIS 종속성을 사용할 수 없게 됩니다.
## <a name="customize-your-bot"></a>봇 사용자 지정

즉시 사용할 수 있는 봇을 성공적으로 배포했는지 확인한 후 시나리오 및 필요에 맞게 봇을 사용자 지정할 수 있습니다. [봇 사용자 지정](bot-builder-enterprise-template-customize.md)으로 계속 진행하세요.
