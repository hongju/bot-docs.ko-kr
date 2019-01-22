---
ms.openlocfilehash: 8eb125b2d0f0c9981cafb763ba809c7d042d8f77
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225988"
---
Bot Service는 봇에 대한 두 가지 호스팅 계획을 제공합니다. 봇 소스 코드를 한 가지 계획에서 다른 계획으로 변환하는 프로세스는 수동입니다.   

## <a name="app-service-plan"></a>App Service 계획

App Service 계획을 사용하는 봇은 예측 가능한 비용과 규모로 미리 정의된 용량을 할당하도록 설정할 수 있는 표준 Azure 웹앱입니다. 이런 호스팅 계획을 사용하는 봇을 사용하면:

* 고급 브라우저 내 코드 편집기를 사용하여 온라인에서 봇 소스 코드를 편집할 수 있습니다.
* Visual Studio를 사용하여 C# 봇을 다운로드, 디버그 및 다시 게시할 수 있습니다.
* Visual Studio Online 및 Github용 지속적인 배포를 손쉽게 설정할 수 있습니다.
* Bot Framework SDK용으로 준비된 샘플 코드를 사용할 수 있습니다.

## <a name="consumption-plan"></a>소비 계획

소비 계획을 사용하는 봇은 <a href="http://go.microsoft.com/fwlink/?linkID=747839" target="_blank">Azure Functions</a>에서 실행되고 종량 요금제(실행당) Azure Functions 가격 책정을 사용하는 서버리스 봇입니다. 이 호스팅 계획을 사용하는 봇은 거대한 트래픽 급증을 처리하도록 확장할 수 있습니다. 기본 브라우저 내 코드 편집기를 사용하여 온라인에서 봇 소스 코드를 편집할 수 있습니다. 소비 계획 봇의 런타임 환경에 대한 자세한 내용은 <a target='_blank' href='/azure/azure-functions/functions-scale'>Azure Functions 소비 및 App Service 계획</a>을 참조하세요.
