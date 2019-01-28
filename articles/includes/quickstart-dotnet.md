---
ms.openlocfilehash: 2096aa342fe954b9f9f1d128bc080c0e0e6efdce
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360803"
---
## <a name="prerequisites"></a>필수 조건
- Visual Studio [2017](https://www.visualstudio.com/downloads)
- [C#](https://aka.ms/bot-vsix)용 Bot Framework SDK v4 템플릿
- Bot Framework [Emulator](https://aka.ms/Emulator-wiki-getting-started)
- [ASP.Net Core](https://docs.microsoft.com/aspnet/core/) 및 [C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/index)의 비동기 프로그래밍에 대한 정보

## <a name="create-a-bot"></a>봇 만들기
필수 조건 섹션에서 다운로드한 BotBuilderVSIX.vsix 템플릿을 설치합니다.

Visual Studio에서 Bot Framework Echo Bot V4 템플릿을 사용하여 새 봇 프로젝트를 만듭니다.

![Visual Studio 프로젝트](~/media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> 필요한 경우 프로젝트 빌드 형식을 ``.Net Core 2.1``로 변경합니다. 또한 필요한 경우 `Microsoft.Bot.Builder` [NuGet 패키지](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio)를 업데이트합니다.

템플릿 덕분에 프로젝트에는 이 빠른 시작에서 봇을 만드는 데 필요한 모든 코드가 포함되어 있습니다. 실제로 추가 코드를 작성할 필요가 없습니다.

## <a name="start-your-bot-in-visual-studio"></a>Visual Studio에서 봇 시작

실행 단추를 클릭하면 Visual Studio는 애플리케이션을 빌드하고, localhost로 배포하고, 웹 브라우저를 시작하여 애플리케이션의 `default.htm` 페이지를 표시합니다. 이때 봇은 로컬에서 실행됩니다.

## <a name="start-the-emulator-and-connect-your-bot"></a>에뮬레이터 시작 및 봇 연결

에뮬레이터를 시작한 다음, 에뮬레이터에서 봇에 연결합니다.

1. 에뮬레이터 "시작" 탭에서 **봇 열기** 링크를 클릭합니다. 
2. Visual Studio 솔루션을 만든 디렉터리에서 .bot 파일을 선택합니다.

## <a name="interact-with-your-bot"></a>봇과의 상호 작용

봇에 메시지를 보내면 봇이 메시지를 통해 응답합니다.

![에뮬레이터 실행](~/media/emulator-v4/emulator-running.png)

> [!NOTE]
> 해당 메시지를 전송할 수 없는 경우 ngrok가 아직 시스템에 대한 필요한 권한을 얻지 못했기 때문에 컴퓨터를 다시 시작해야 할 수 있습니다(한 번만 수행해야 함).