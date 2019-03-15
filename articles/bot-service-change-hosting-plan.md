---
title: 사용 계획에서 App Service 계획으로 C# 봇 마이그레이션 | Microsoft Docs
description: 사용 호스팅 계획에서 App Service 호스팅 계획으로 Bot Service C# 봇을 마이그레이션합니다.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
ms.openlocfilehash: e463b272385b97e630d4087908aa82e23a70fea9
ms.sourcegitcommit: b2245df2f0a18c5a66a836ab24a573fd70c7d272
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/07/2019
ms.locfileid: "57568190"
---
# <a name="change-the-hosting-plan-for-your-bot-service"></a>Bot Service에 대한 호스팅 계획 변경

이 항목에서는 사용 계획을 사용하는 C# 스크립트 봇을 App Service 계획을 사용하는 C# 봇으로 마이그레이션하는 방법을 설명합니다. 

## <a name="advantages-of-a-bot-on-an-app-service-plan"></a>App Service 계획에서 봇의 장점

App Service 계획의 봇은 Azure 웹앱으로 실행됩니다. 웹앱 봇은 사용 계획 봇이 할 수 없는 다음과 같은 여러 작업을 수행할 수 있습니다.

- 웹앱 봇은 사용자 지정 경로 정의를 추가할 수 있습니다.
- 웹앱 봇은 Websocket 서버를 사용하도록 설정할 수 있습니다. 
- 사용 호스팅 계획을 사용하는 봇에는 Azure Functions에서 실행되는 모든 코드와 동일한 제한 사항이 적용됩니다. 자세한 내용은 <a target='_blank' href='/azure/azure-functions/functions-scale'>Azure Functions 사용 계획 및 App Service 계획</a>을 참조하세요.

## <a name="download-your-existing-bot-source"></a>기존 봇 소스 다운로드

기존 봇의 소스 코드를 다운로드하려면 다음 단계를 수행합니다.

1. Azure 봇 내에서 **설정** 탭을 클릭하고 **연속 배포** 섹션을 확장합니다.  
2. 파란색 단추를 클릭하여 봇의 소스 코드를 포함하는 zip 파일을 다운로드합니다.  
    1. [!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]
    ![봇 zip 파일 다운로드](~/media/continuous-deployment-consumption-download.png)
3. 다운로드한 zip 파일 콘텐츠의 압축을 로컬 폴더에 풉니다. 


## <a name="create-a-bot-template"></a>봇 템플릿 만들기

Bot Service에는 사용 계획 및 App Service 계획 봇을 위한 동일한 템플릿이 들어 있습니다. 사용 계획 봇에서 마이그레이션하려면 동일한 템플릿을 기준으로 Bot Service에서 새로운 App Service 계획 봇을 만듭니다. 기본 코드는 동일한 템플릿에 대한 두 호스팅 형식 간에 다를 수 있지만, 새 웹앱은 기존 봇이 사용하는 것과 유사한 구조 및 구성 기능을 유지합니다.

## <a name="download-the-new-bot-source"></a>새 봇 소스 다운로드

새 봇의 소스 코드를 다운로드하려면 다음 단계를 수행합니다.

1. Azure 봇 내에서 **빌드** 탭을 클릭하고 **소스 코드 다운로드** 섹션을 찾은 후 **zip 파일 다운로드**를 클릭합니다. 
2. 다운로드한 zip 파일 콘텐츠의 압축을 로컬 폴더에 풉니다.

## <a name="add-source-files-to-new-solution"></a>새 솔루션에 소스 파일 추가

일부 .csx 파일은 새 솔루션에서 .cs 파일로  컴파일되고 실행될 수 있습니다. 솔루션의 각 .csx 파일에 대해 .cs 파일을 만듭니다(`run.csx` 제외). `run.csx` 논리는 직접 마이그레이션할 수 있습니다. .cs 파일에서 클래스 선언 및 선택적 네임스페이스 선언을 추가해야 할 수 있습니다.

## <a name="migrate-runcsx-logic-into-your-project"></a>run.csx 논리를 프로젝트로 마이그레이션

C# 스크립트 프로젝트에는 여러 다른 `ActivityTypes` 값이 처리되는 `Run` 메서드가 있습니다. 활동 처리 논리를 `MessageController.cs`의 `MessageController.Post` 메서드로 가져옵니다.

## <a name="remove-compiler-keywords"></a>컴파일러 키워드 제거

C# 스크립트 파일에는 `#r` 키워드를 사용하는 참조 모듈이 포함될 수 있습니다. 이러한 줄을 제거한 후 Visual Studio 프로젝트에 대한 참조로 추가합니다. 또한 파일 컴파일 시 다른 소스 코드 파일을 삽입하는 `#load` 키워드도 제거합니다. 대신, 프로젝트와 관련된 모든 `.csx` 파일을 `.cs` 소스 코드로 추가합니다.

## <a name="add-references-from-projectjson"></a>project.json에서 참조 추가

소비 계획 봇이 해당 `project.json` 파일에 NuGet 참조를 추가하는 경우 솔루션 탐색기 창에서 프로젝트를 마우스 오른쪽 단추로 클릭하고 **참조 추가**를 클릭하여 새 Visual Studio 솔루션에 이러한 참조를 추가합니다.

### <a name="add-references-that-were-implicit"></a>암시적 참조 추가

사용 계획의 Bot Service 봇은 모든 .csx 소스 파일에 이러한 참조를 암시적으로 포함합니다. .cs 소스 파일로 마이그레이션된 소스는 다음 클래스에 대한 명시적 참조를 추가해야 할 수 있습니다.

- `Microsoft.Azure.WebJobs.Host` 네임스페이스 형식을 제공하는 NuGet 패키지 `Microsoft.Azure.WebJobs`의 `TraceWriter` 
- NuGet 패키지 `Microsoft.Azure.WebJobs.Extensions`의 타이머 트리거
- `Newtonsoft.Json`, `Microsoft.ServiceBus` 및 기타 자동으로 참조된 어셈블리
- `System.Threading.Tasks` 및 기타 자동으로 가져온 네임스페이스

추가 지침에 대해서는 <a target='_blank' href='https://blogs.msdn.microsoft.com/appserviceteam/2017/03/16/publishing-a-net-class-library-as-a-function-app/'>.NET 클래스 라이브러리를 Function App으로 게시</a>에서 *클래스 파일로 변환*을 참조하세요.

## <a name="debug-your-new-bot"></a>새 봇 디버그

App Service 계획의 봇은 사용 계획의 봇에 비해 로컬로 디버그하기가 훨씬 더 쉽습니다. [에뮬레이터](bot-service-debug-emulator.md)를 사용하여 마이그레이션된 코드를 로컬로 디버그할 수 있습니다.

## <a name="publish-from-visual-studio-or-set-up-continuous-deployment"></a>Visual Studio에서 게시 또는 연속 배포 설정

마지막으로 해당 `.PublishSettings` 파일을 가져오고 **게시**를 클릭하거나 [연속 배포를 설정](bot-service-debug-bot.md)하여 마이그레이션된 소스 코드를 Bot Service에 게시합니다.
