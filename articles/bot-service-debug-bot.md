---
title: Bot Service를 사용하여 빌드된 봇 디버그 | Microsoft Docs
description: Bot Service를 사용하여 빌드된 봇을 디버그하는 방법에 대해 알아봅니다.
author: v-ducvo
ms.author: v-ducvo
keywords: Bot Builder SDK, 지속적인 배포, 앱 서비스, 에뮬레이터
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 07/13/2018
ms.openlocfilehash: 3312aa3508a7d9e57f1adc3e36c9d0325cdf6df6
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997550"
---
# <a name="debug-a-bot-service-bot"></a>Bot Service 봇 디버그

이 문서에서는 Visual Studio 또는 Visual Studio Code 및 Bot Framework Emulator 같은 통합 개발 환경(IDE)를 사용하여 봇을 디버그하는 방법을 설명합니다. 이러한 메서드를 사용하여 봇을 로컬로 디버그할 수 있는 반면 이 문서에서는 [Bot Service를 사용하여 봇 만들기](bot-service-quickstart.md) 문서를 통해 만든 **EchoBot**을 사용합니다.

## <a name="debug-a-javascript-bot"></a>Javascript 봇 디버그

이 섹션의 단계를 수행하여 JavaScript로 작성된 봇을 디버그합니다.

### <a name="prerequisites"></a>필수 조건

JavaScript 봇을 디버그할 수 있기 전에 이러한 작업을 완료해야 합니다.

- [봇 소스 코드 다운로드](bot-service-build-download-source-code.md)에 설명된 대로 Azure에서 봇에 대한 소스 코드를 다운로드합니다.
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases)를 다운로드하고 설치합니다.
- <a href="https://code.visualstudio.com" target="_blank">Visual Studio Code</a> 같은 코드 편집기를 다운로드하여 설치합니다.

### <a name="debug-a-javascript-bot-using-command-line-and-emulator"></a>명령줄 및 에뮬레이터를 사용하여 JavaScript 봇 디버그

명령줄을 사용하고 봇을 에뮬레이터로 테스트하여 JavaScript 봇을 실행하려면 다음을 수행합니다.
1. 명령줄에서 디렉터리를 봇 프로젝트 디렉터리로 변경합니다.
2. **node app.js** 명령을 실행하여 봇을 시작합니다.
3. 에뮬레이터를 시작하고 봇의 엔드포인트에 연결합니다(예: **http://localhost:3978/api/messages**). 처음으로 봇을 실행하는 경우라면 **파일 > 새 봇**을 클릭하고 화면에 표시된 지시를 따릅니다. 그렇지 않은 경우 **파일 > 봇 열기**를 클릭하여 기존 봇을 엽니다. 이 봇이 컴퓨터에서 로컬로 실행되므로 **MSA 앱 ID** 및 **MSA 앱 암호** 필드를 빈 공간으로 내버려둘 수 있습니다. 자세한 내용은 [에뮬레이터를 사용하여 디버그](bot-service-debug-emulator.md)를 참조합니다.
4. 에뮬레이터에서 봇에게 메시지를 보냅니다(예: "Hi" 메시지 보내기). 
5. 에뮬레이터 창의 오른쪽에서 **검사기** 및 **로그** 패널을 사용하여 봇을 디버그합니다. 예를 들어 메시지 말풍선 중 하나를 클릭하면(예: 아래 스크린샷의 "Hi" 메시지 말풍선) 해당 메시지의 세부 정보가 **검사기** 패널에 표시됩니다. 메시지가 에뮬레이터와 봇 간에 교환될 때 말풍선을 사용하여 요청 및 응답을 볼 수 있습니다. 또는 **로그** 패널에서 연결된 텍스트 중 하나를 클릭하여 **검사기** 패널에서 세부 정보를 볼 수 있습니다.

   ![에뮬레이터의 검사기 패널](~/media/bot-service-debug-bot/emulator_inspector.png)

### <a name="debug-a-javascript-bot-using-breakpoints-in-visual-studio-code"></a>Visual Studio Code의 중단점을 사용하여 JavaScript 봇 디버그

코드를 단계별로 실행하려면 Visual Studio Code 같은 IDE를 사용하여 중단점을 설정하고 디버그 모드에서 봇을 실행할 수 있습니다. VS Code에서 중단점을 설정하려면 다음을 수행합니다.

1. VS Code를 시작하고 봇 프로젝트 폴더를 엽니다.
2. 메뉴 모음에서 **디버그** 및 **디버깅 시작**을 차례로 클릭합니다. 코드를 실행하려면 런타임 엔진을 선택하라는 메시지가 나타나면 **Node.js**을 선택합니다. 이때 봇은 로컬에서 실행됩니다. 

   > [!NOTE]
   > "값은 null일 수 없습니다"라는 오류가 발생할 경우 **Table Storage** 설정이 유효한지 확인합니다.
   > **EchoBot** 기본값은 **Table Storage** 사용입니다. 봇에서 Table Storage를 사용하려면 테이블 *이름* 및 *키*가 필요합니다. Table service 인스턴스가 준비되지 않은 경우 테스트를 위해 하나를 만들거나, **TableBotDataStore**를 사용하는 코드를 주석으로 처리하고 **InMemoryDataStore**를 사용하는 코드 줄을 주석 처리 제거할 수 있습니다. **InMemoryDataStore**는 테스트 및 프로토타입 생성만을 위한 것입니다.

3. 필요에 따라 중단점을 설정합니다. VS Code에서 줄 번호 왼쪽의 열 위로 마우스를 이동하여 중단점을 설정할 수 있습니다. 작은 빨간색 점이 표시됩니다. 점을 클릭하면 중단점이 설정됩니다. 점을 다시 클릭하면 중단점이 제거됩니다.

   ![VS Code에서 중단점 설정](~/media/bot-service-debug-bot/breakpoint-set.png)

4. Bot Framework Emulator를 시작하고 위의 섹션에 설명된 대로 봇에 연결합니다. 
5. 에뮬레이터에서 봇에게 메시지를 보냅니다(예: "Hi" 메시지 보내기). 중단점을 배치하는 줄에서 실행이 중지됩니다.

   ![VS Code에서 디버그](~/media/bot-service-debug-bot/breakpoint-caught.png)


## <a name="debug-a-c-bot"></a>C# 봇 디버그

이 섹션의 단계를 수행하여 C#으로 작성된 봇을 디버그합니다.


### <a name="prerequisites"></a>필수 조건

웹앱 C# 봇을 디버그할 수 있기 전에 이러한 작업을 완료해야 합니다.

- [봇 소스 코드 다운로드](bot-service-build-download-source-code.md)에 설명된 대로 Azure에서 봇에 대한 소스 코드를 다운로드합니다.
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases)를 다운로드하고 설치합니다.
- <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2017</a>(Community Edition 이상)를 다운로드하고 설치합니다.

### <a name="debug-a-c-bot-using-breakpoints-in-visual-studio"></a>Visual Studio의 중단점을 사용하여 C# 봇 디버그

코드를 단계별로 실행하려면 Visual Studio(VS) 같은 IDE를 사용하여 중단점을 설정하고 디버그 모드에서 봇을 실행할 수 있습니다. VS에서 중단점을 설정하려면 다음을 수행합니다.

1. 봇 폴더로 이동하여 **.sln** 파일을 엽니다. 이렇게 하면 VS에서 솔루션이 열립니다.
2. 메뉴 모음에서 **빌드**, **솔루션 빌드**를 차례로 클릭합니다.
3. **솔루션 탐색기**에서 **대화 상자** 폴더를 확장하고 **EchoDialog.cs**를 클릭합니다. 이 파일은 기본 봇 논리를 정의합니다.
4. 메뉴 모음에서 **디버그** 및 **디버깅 시작**을 차례로 클릭합니다. 이때 봇은 로컬에서 실행됩니다. 

   > [!NOTE]
   > "값은 null일 수 없습니다"라는 오류가 발생할 경우 **Table Storage** 설정이 유효한지 확인합니다.
   > **EchoBot** 기본값은 **Table Storage** 사용입니다. 봇에서 Table Storage를 사용하려면 테이블 *이름* 및 *키*가 필요합니다. Table service 인스턴스가 준비되지 않은 경우 테스트를 위해 하나를 만들거나, **TableBotDataStore**를 사용하는 코드를 주석으로 처리하고 **InMemoryDataStore**를 사용하는 코드 줄을 주석 처리 제거할 수 있습니다. **InMemoryDataStore**는 테스트 및 프로토타입 생성만을 위한 것입니다.

5. 필요에 따라 중단점을 설정합니다. VS에서 줄 번호 왼쪽의 열 위로 마우스를 이동하여 중단점을 설정할 수 있습니다. 작은 빨간색 점이 표시됩니다. 점을 클릭하면 중단점이 설정됩니다. 점을 다시 클릭하면 중단점이 제거됩니다.

   ![VS에서 중단점 설정](~/media/bot-service-debug-bot/breakpoint-set-vs.png)

6. Bot Framework Emulator를 시작하고 위의 섹션에 설명된 대로 봇에 연결합니다. 
7. 에뮬레이터에서 봇에게 메시지를 보냅니다(예: "Hi" 메시지 보내기). 중단점을 배치하는 줄에서 실행이 중지됩니다.

   ![VS에서 디버그](~/media/bot-service-debug-bot/breakpoint-caught-vs.png)

::: moniker range="azure-bot-service-3.0" 

## <a id="debug-csharp-serverless"></a> 소비 계획 C\# 함수 봇 디버그

노드 엔진과 같은 런타임 호스트가 필요하기 때문에 Bot Service에서 소비 계획 서버리스 C\# 환경에는 일반적인 C\# 응용 프로그램보다 Node.js와 공통점이 더 많이 있습니다. Azure에서 런타임은 클라우드의 호스팅 환경에 포함되지만 바탕 화면에서 해당 환경을 로컬로 복제해야 합니다. 

### <a name="prerequisites"></a>필수 조건

소비 계획 C# 봇을 디버그할 수 있기 전에 이러한 작업을 완료해야 합니다.

- [지속적인 배포 설정](bot-service-continuous-deployment.md)에 설명된 대로 Azure에서 봇에 대한 소스 코드를 다운로드합니다.
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases)를 다운로드하고 설치합니다.
- <a href="https://www.npmjs.com/package/azure-functions-cli" target="_blank">Azure Functions CLI</a>를 설치합니다.
- <a href="https://github.com/dotnet/cli" target="_blank">DotNet CLI</a>를 설치합니다.
  
Visual Studio 2017에서 중단점을 사용하여 코드를 디버그할 수 있도록 하려는 경우 이러한 작업을 완료해야 합니다.
  
- <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2017</a>(Community Edition 이상)를 다운로드하고 설치합니다.
- <a href="https://visualstudiogallery.msdn.microsoft.com/e6bf6a3d-7411-4494-8a1e-28c1a8c4ce99" target="_blank">명령 작업 실행기 Visual Studio 확장</a>을 다운로드하고 설치합니다.

> [!NOTE]
> Visual Studio Code는 현재 지원되지 않습니다.

### <a name="debug-a-consumption-plan-c-functions-bot-using-the-emulator"></a>에뮬레이터를 사용하여 소비 계획 C# 함수 봇 디버그

봇을 로컬로 디버그하는 가장 간단한 방법은 봇을 시작한 다음, Bot Framework Emulator에서 봇에 연결하는 것입니다. 
먼저 명령 프롬프트를 열고 리포지토리에서 **project.json** 파일이 위치한 폴더로 이동합니다. 그런 다음, `dotnet restore` 명령을 실행하여 봇에서 참조하는 다양한 패키지를 복원합니다.

> [!NOTE]
> Visual Studio 2017은 Visual Studio에서 종속성을 처리하는 방법을 변경합니다. Visual Studio 2015가 **project.json**을 사용하여 종속성을 처리하는 반면 Visual Studio 2017은 Visual Studio에 로드되는 경우 **.csproj** 모델을 사용합니다. Visual Studio 2017을 사용하는 경우 `dotnet restore` 명령을 실행하기 전에 <a href="https://aka.ms/bf-debug-project">는 이 **.csproj** 파일</a>을 리포지토리의 **/messages** 폴더에 다운로드합니다.

![명령 프롬프트](~/media/bot-service-debug-bot/csharp-azureservice-debug-envconfig.png)

다음으로, `debughost.cmd`를 실행하여 봇을 로드하고 시작합니다. 

![Debughost.cmd를 실행하는 명령 프롬프트](~/media/bot-service-debug-bot/csharp-azureservice-debug-debughost.png)

이때 봇은 로컬에서 실행됩니다. 콘솔 창에서 debughost가 수신 대기하는 엔드포인트를 복사합니다(이 예제에서는 `http://localhost:3978`). 그런 다음, Bot Framework Emulator를 시작하고 엔드포인트를 에뮬레이터의 주소 표시줄에 붙여넣습니다. 예를 들어 `/api/messages`를 엔드포인트에 추가해야 합니다. 로컬 디버깅에 대한 보안이 필요하지 않으므로 **Microsoft 앱 ID** 및 **Microsoft 앱 암호** 필드를 빈 공간으로 내버려둘 수 있습니다. **연결**를 클릭하여 지정된 엔드포인트를 사용하여 봇에 대한 연결을 설정합니다.

![에뮬레이터 구성](~/media/bot-service-debug-bot/mac-azureservice-emulator-config.png)

봇에 에뮬레이터를 연결한 후 에뮬레이터 창의 맨 아래에 있는 텍스트 상자에 일부 텍스트를 입력하여 봇에 메시지를 보냅니다(즉, **메시지 입력...** 이 왼쪽 아래 모서리에 표시됩니다). 에뮬레이터와 봇 간에 메시지가 교환되므로 에뮬레이터 창의 오른쪽에서 **로그** 및 **검사기** 패널을 사용하여 요청 및 응답 메시지를 볼 수 있습니다.

![에뮬레이터를 통한 테스트](~/media/bot-service-debug-bot/mac-azureservice-debug-emulator.png)

또한 콘솔 창에서 로그 세부 정보를 볼 수 있습니다.

![콘솔 창](~/media/bot-service-debug-bot/csharp-azureservice-debug-debughostlogging.png)

::: moniker-end

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [에뮬레이터를 사용하여 디버그합니다](bot-service-debug-emulator.md).
