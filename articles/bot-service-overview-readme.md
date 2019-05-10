---
title: Bot Service 작동 방법 | Microsoft Docs
description: Bot Service의 특징과 기능에 대해 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 85ef0fde39980bab1b891518e338fddbd56b275a
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563867"
---
# <a name="how-bot-service-works"></a>Bot Service 작동 방법

Bot Service는 봇 개발을 위한 Bot Framework SDK 및 채널에 봇을 연결하기 위한 Bot Framework를 포함하여 봇 만들기의 핵심 구성 요소를 제공합니다. Bot Service는 .NET 및 Node.js를 지원하는 봇을 만들 때 선택할 수 있는 5가지 템플릿을 제공합니다.

> [!IMPORTANT]
> Bot Service를 사용하려면 Microsoft Azure 구독이 있어야 합니다. 아직 구독이 없는 경우 <a href="https://azure.microsoft.com/en-us/free/" target="_blank">체험 계정</a>으로 등록할 수 있습니다.

## <a name="hosting-plans"></a>호스팅 계획
Bot Service는 봇에 대해 두 가지 호스팅 계획을 제공합니다. 요구에 적합한 호스팅 계획을 선택할 수 있습니다.

### <a name="app-service-plan"></a>App Service 계획

App Service 계획을 사용하는 봇은 예측 가능한 비용과 규모로 미리 정의된 용량을 할당하도록 설정할 수 있는 표준 Azure 웹앱입니다. 이런 호스팅 계획을 사용하는 봇을 사용하면:

* 고급 브라우저 내 코드 편집기를 사용하여 온라인에서 봇 소스 코드를 편집할 수 있습니다.
* Visual Studio를 사용하여 C# 봇을 다운로드, 디버그 및 다시 게시할 수 있습니다.
* Visual Studio Online 및 Github용 지속적인 배포를 손쉽게 설정할 수 있습니다.
* Bot Framework SDK용으로 준비된 샘플 코드를 사용할 수 있습니다.

### <a name="consumption-plan"></a>소비 계획
소비 계획을 사용하는 봇은 <a href="http://go.microsoft.com/fwlink/?linkID=747839" target="_blank">Azure Functions</a>에서 실행되고 종량 요금제(실행당) Azure Functions 가격 책정을 사용하는 서버리스 봇입니다. 이 호스팅 계획을 사용하는 봇은 거대한 트래픽 급증을 처리하도록 확장할 수 있습니다. 기본 브라우저 내 코드 편집기를 사용하여 온라인에서 봇 소스 코드를 편집할 수 있습니다. 소비 계획 봇의 런타임 환경에 대한 자세한 내용은 <a target='_blank' href='/azure/azure-functions/functions-scale'>Azure Functions 소비 및 App Service 계획</a>을 참조하세요.

## <a name="templates"></a>템플릿

Bot Service에서는 다섯 가지 템플릿 중 하나를 사용하여 C# 또는 Node.js로 손쉽고 빠르게 봇을 만들 수 있습니다.

[!INCLUDE [Bot Service templates](~/includes/snippet-abs-templates.md)]

다양한 템플릿 및 이 템플릿이 봇을 빌드하는 데 어떻게 도움이 되는지 [자세히 알아보세요](bot-service-concept-templates.md).

## <a name="develop-and-deploy"></a>개발 및 배포

Bot Service를 사용하면 도구 체인이 필요 없이 온라인 코드 편집기를 사용하여 브라우저에서 바로 봇을 개발할 수 있습니다. 

Bot Framework SDK 및 Visual Studio 2017과 같은 IDE를 사용하여 로컬에서 봇을 개발하고 디버그할 수 있습니다. Visual Studio 2017 또는 Azure CLI를 사용하여 봇을 Azure에 직접 게시할 수 있습니다. VSTS나 GitHub와 같은 원하는 소스 제어 시스템으로 [지속적인 배포를 설정](bot-service-continuous-deployment.md)할 수도 있습니다. 지속적인 배포를 구성하면 로컬 컴퓨터의 IDE에서 개발하고 디버그할 수 있으며, 코드를 변경하고 소스 제어에 커밋하면 Azure에 자동으로 배포됩니다.  

> [!TIP]
> 지속적인 배포를 사용하도록 설정한 경우에는 충돌을 피하기 위해 다른 메커니즘이 아닌 지속적인 배포를 통해서만 코드를 수정해야 합니다.

## <a name="manage-your-bot"></a>봇 관리 

Bot Service를 사용하여 봇을 만드는 과정에서 봇의 이름을 지정하고, 호스팅 계획을 설정하고, 기타 설정을 구성합니다. 봇이 만들어지면 봇의 설정을 변경할 수 있고 하나 이상의 채널에서 실행되도록 구성할 수 있으며 웹 채팅에서 테스트할 수 있습니다. 

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [Bot Service를 사용하여 봇 만들기](bot-service-quickstart.md)