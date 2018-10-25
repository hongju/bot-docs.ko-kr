---
title: Azure Bot Service 소개 | Microsoft Docs
description: 봇의 빌드, 연결, 테스트, 배포, 모니터링 및 관리를 위한 Bot Service에 대해 알아봅니다.
keywords: 개요, 소개, SDK, 개요
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 10/08/2018
ms.openlocfilehash: 750f348437f4e729ecc24317f1849b96c47d743a
ms.sourcegitcommit: a3833dd173a30f3f0c1baa807e690a5778862d64
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/08/2018
ms.locfileid: "48864028"
---
::: moniker range="azure-bot-service-3.0"

# <a name="about-azure-bot-service"></a>Azure Bot Service 정보

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Azure Bot Service는 지능형 봇을 모두 한 번에 빌드, 테스트, 배포 및 관리할 수 있는 도구를 제공합니다. 개발자는 SDK에서 제공하는 확장 가능한 모듈식 프레임워크를 통해 템플릿을 활용하여 음성, 언어 이해, 질문 및 답변 등을 제공하는 봇을 만들 수 있습니다.  

## <a name="what-is-a-bot"></a>봇이란?
봇은 사용자가 텍스트, 그래픽(카드) 또는 음성을 사용하여 대화형 방식으로 상호 작용하는 앱입니다. 간단한 질문 및 답변 대화 상자일 수도 있고, 패턴 일치, 상태 추적 및 기존 비즈니스 서비스와 잘 통합된 인공 지능 기술을 사용하여 지능형 방식으로 사람들이 서비스와 상호 작용할 수 있도록 하는 정교한 봇일 수도 있습니다. 

## <a name="building-a-bot"></a>봇 빌드 
C# 또는 Node.js에서 봇을 만들 때 즐겨 찾는 개발 환경이나 명령줄 도구를 사용할지 선택할 수 있습니다. 시작하기 위해 봇을 빌드하는 데 사용할 수 있는 봇 개발의 다양한 단계에 대한 도구를 제공합니다.    

![봇 개요](media/bot-service-overview.png) 

## <a name="plan"></a>계획 
코드를 작성하기 전에 모범 사례에 대한 봇 [디자인 지침](bot-service-design-principles.md) 을 검토하고 봇에 필요한 요구 사항을 확인합니다. 간단한 봇을 만들거나 음성, 언어 이해, QnA 또는 다양한 출처에서 정보를 추출하여 지능형 응답을 제공하는 기능 등 더 복잡한 기능을 포함시킬 수 있습니다.  

> [!TIP]
> [Azure](https://portal.azure.com) 계정을 만듭니다. 

## <a name="build-your-bot"></a>봇 빌드 
봇은 대화형 인터페이스를 구현하고 Bot Service와 통신하는 웹 서비스입니다. 이 솔루션은 여러 환경 및 언어에서 만들 수 있으며, Visual Studio 또는 Yeoman이나 Azure Portal 내에서 직접 손쉬운 시작 도구를 제공합니다. 사용할 수 있는 도구 및 서비스 중 일부가 아래에 나와 있습니다.

> [!TIP]
> [Azure Portal](bot-service-quickstart.md)을 사용하여 봇을 만듭니다. 필요한 경우 다음과 같은 구성 요소를 추가합니다. 
> - [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/home)(Language Understanding) 
> - 사용자 질문에 대답하기 위한 [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/home) 기술 자료  

## <a name="test-your-bot"></a>봇 테스트 
봇은 수많은 다양한 파트가 함께 작동하는 복잡한 앱입니다. 다른 복잡한 앱과 마찬가지로 흥미로운 버그가 발생하거나 봇이 예상과 다르게 동작할 수 있습니다. 게시하기 전에 봇을 테스트합니다.

> [!TIP] 
> - [웹 채팅](bot-service-manage-test-webchat.md)을 사용하여 봇을 테스트하거나 [에뮬레이터](bot-service-debug-emulator.md)를 사용하여 로컬로 봇을 테스트합니다.

## <a name="publish"></a>게시 
준비되면 봇을 Azure 또는 사용자 고유의 웹 서비스 또는 데이터 센터에 게시합니다. 봇을 로컬로 개발할 수 있도록 지속적인 배포를 설정할 수 있습니다. 이는 봇이 GitHub 또는 Visual Studio Team Services와 같은 소스 제어에 체크 인하는 경우에 유용합니다. 변경 내용을 소스 리포지토리로 다시 체크 인하면 변경 내용이 자동으로 Azure에 배포됩니다.

> [!Tip]
> - [코드 다운로드 및 Azure에 재배포](bot-service-build-download-source-code.md)

## <a name="connect"></a>연결          
Facebook, Messenger, Kik, Skype, Slack, Microsoft Teams, Telegram, 텍스트/SMS, Twilio, Cortana 및 Skype와 같은 채널에 봇을 연결하여 상호 작용을 늘리고 더 많은 고객과 연결합니다.  
  
> [!TIP]
> - [추가할 채널 선택](bot-service-manage-channels.md)


## <a name="evaluate"></a>평가 
Azure Portal에서 수집된 데이터를 사용하여 봇의 성능과 기능을 향상시킬 기회를 확인합니다. 트래픽, 대기 시간 및 통합과 같은 서비스 수준 및 계측 데이터를 가져올 수 있습니다. 또한 분석은 사용자에 대한 대화 수준 보고, 메시지 및 채널 데이터를 제공합니다.

> [!Tip]
> - [분석 수집](bot-service-manage-analytics.md) 


::: moniker-end

::: moniker range="azure-bot-service-4.0"

# <a name="about-azure-bot-service"></a>Azure Bot Service 정보

[!INCLUDE [pre-release-label](includes/pre-release-label.md)]

Azure Bot Service는 지능형 봇을 모두 한 번에 빌드, 테스트, 배포 및 관리할 수 있는 도구를 제공합니다. 개발자는 SDK, 도구, 템플릿 및 AI 서비스에서 제공하는 모듈 및 확장 가능한 프레임워크를 사용하여 음성을 사용하고, 자연어를 이해하고, 질문 및 답변 처리 등을 수행하는 봇을 만들 수 있습니다.

## <a name="what-is-a-bot"></a>봇이란?
봇은 컴퓨터를 사용하는 것보다는 사용자 또는 적어도 지능형 로봇을 다루는 것과 같은 환경을 제공합니다. 직접적인 사용자의 개입을 더 이상 요구할 수 없는 자동화된 시스템으로 저녁 식사를 예약하거나 프로필 정보를 수집하는 등 간단한 반복 작업을 이동할 수 없습니다. 사용자는 텍스트, 대화형 카드 및 음성을 사용하여 봇과 대화합니다. 빠른 질문 및 대답이거나 지능적으로 서비스에 대한 액세스를 제공하는 복잡한 대화를 통해 봇과 상호 작용할 수 있습니다.

봇은 인터넷에 연결되어 최신 웹 응용 프로그램과 비슷하고 API를 사용하여 메시지를 주고 받습니다. 봇의 기능은 봇의 종류에 따라 크게 다릅니다. 최신 봇 소프트웨어는 많은 기술 및 도구를 사용하여 다양한 플랫폼에서 더 복잡한 환경을 제공합니다. 그러나 간단한 봇은 메시지를 수신하고 적은 코드를 사용하여 사용자에게 다시 보내기만 할 수 있습니다. 

봇은 파일을 읽고 쓰며, 데이터베이스 및 API를 사용하고, 정규 계산 작업을 수행하는 등 다른 형식의 소프트웨어가 수행하는 동일한 작업을 수행할 수 있습니다. 봇이 고유한 이유는 일반적으로 사용자 간 통신에 유보된 메커니즘을 사용한다는 점입니다. 

봇은 보통 다음 구성 요소로 구성됩니다.
* 대부분의 경우 공용 인터넷에서 사용할 수 있는 웹 서버
* 봇을 개발하는 인터페이스를 제공하는 Bot Builder SDK 및 Bot Builder 도구
* Azure Cognitive Service 
* Azure Storage

## <a name="building-a-bot"></a>봇 빌드 

Azure Bot Service는 프로세스를 용이하게 하는 일련의 통합된 도구 및 서비스를 제공합니다. C#, JavaScript 또는 Typescript로 봇을 만들 때 즐겨 찾는 개발 환경이나 명령줄 도구를 선택합니다. (Java 및 Python도 곧 출시 예정입니다.) 시작하기 위해 봇을 빌드하는 데 사용할 수 있는 봇 개발의 다양한 단계에 대한 도구를 제공합니다.

![봇 개요](media/bot-service-overview.png) 

### <a name="plan"></a>계획
봇을 성공적으로 만드는 프로세스에서는 소프트웨어의 형식에 관계 없이 목표, 프로세스 및 사용자 요구 사항을 철저하게 파악하는 것이 중요합니다. 코드를 작성하기 전에 모범 사례에 대한 봇 [디자인 지침](bot-service-design-principles.md) 을 검토하고 봇에 필요한 요구 사항을 확인합니다. 간단한 봇을 만들거나 음성, 자연어 이해 또는 질문 답변과 같은 보다 정교한 기능을 포함할 수 있습니다.

### <a name="build"></a>빌드
봇은 대화형 인터페이스를 구현하고 Bot Framework Service와 통신하여 메시지 및 이벤트를 주고 받는 웹 서비스입니다. 다양한 환경 및 언어로 봇을 만들 수 있습니다. [Azure Portal](bot-service-quickstart.md)에서 봇 개발을 시작하거나 로컬 개발을 위한 [[C#](dotnet/bot-builder-dotnet-sdk-quickstart.md) | [JavaScript](javascript/bot-builder-javascript-quickstart.md)] 템플릿을 사용할 수 있습니다.

Azure Bot Service의 일부로 봇의 기능을 확장하는 데 사용할 수 있는 추가 구성 요소를 제공합니다.

| 기능 | 설명 | 링크 |
| --- | --- | --- |
| 자연어 처리 추가 | 봇이 자연어를 이해하고, 맞춤법 오류를 이해하고, 음성 기능을 사용하고, 사용자의 의도를 인식할 수 있게 해줍니다. | [LUIS](~/v4sdk/bot-builder-howto-v4-luis.md) 사용 방법 
| 질문에 답변 | 더 자연스럽고 대화하는 방식으로 사용자 질문에 답변하기 위해 기술 자료 추가 | [QnA Maker](~/v4sdk/bot-builder-howto-qna.md) 사용 방법 
| 여러 모델 관리 | LUIS 및 QnA Maker와 같은 둘 이상의 모델을 사용하는 경우, 봇 대화 중에 어느 모델을 사용해야 하는지 지능적으로 결정합니다. | [Dispatch](~/v4sdk/bot-builder-tutorial-dispatch.md) 도구|
| 카드 및 단추 추가 | 그래픽, 메뉴 및 카드와 같은 텍스트 이외의 다른 미디어를 사용하여 사용자 환경 향상 | [카드 추가](v4sdk/bot-builder-howto-add-media-attachments.md) 방법 |

> [!NOTE]
> 위의 표는 포괄적인 목록은 아닙니다. 왼쪽 문서를 [메시지 보내기](~/v4sdk/bot-builder-howto-send-messages.md)부터 탐색하여 더 많은 봇 기능에 대해 알아봅니다.

또한 봇 자산을 생성, 관리 및 테스트할 수 있는 명령줄 도구를 제공합니다. 이러한 도구는 봇 구성 파일을 관리하고, LUIS 앱을 구성하고, QnA 기술 자료를 빌드하고, 대화를 모의할 수 있습니다. 자세한 내용은 명령줄 도구 [추가 정보](https://aka.ms/botbuilder-tools-readme)에서 볼 수 있습니다.

SDK를 통해 사용할 수 있는 다양한 기능을 소개하는 [샘플](https://github.com/microsoft/botbuilder-samples)에 액세스할 수도 있습니다. 다양한 기능 시작점을 찾는 개발자에게 적합합니다.

### <a name="test"></a>테스트 
봇은 수많은 다양한 파트가 함께 작동하는 복잡한 앱입니다. 다른 복잡한 앱과 마찬가지로 흥미로운 버그가 발생하거나 봇이 예상과 다르게 동작할 수 있습니다. 게시하기 전에 봇을 테스트합니다. 봇을 사용하도록 출시하기 전에 테스트하는 여러 방법을 제공합니다.

- [에뮬레이터](bot-service-debug-emulator.md)를 사용하여 로컬로 봇을 테스트합니다. Bot Framework Emulator는 채팅 인터페이스를 제공하는 독립 실행형 앱일 뿐만 아니라 봇이 수행하는 방법과 이유, 내용을 이해하기 위한 디버깅 및 질문 도구입니다.  에뮬레이터는 개발용 봇 응용 프로그램과 함께 로컬로 실행할 수 있습니다. 
 
- [웹](bot-service-manage-test-webchat.md)에서 봇을 테스트합니다. Azure Portal을 통해 봇을 구성하면 웹 채팅 인터페이스를 통해 액세스할 수도 있습니다. 웹 채팅 인터페이스는 테스터 및 봇의 실행 코드에 대한 직접 액세스 권한이 없는 다른 사용자에게 액세스 권한을 부여할 수 있는 좋은 방법입니다.

### <a name="publish"></a>게시 
웹에서 봇을 사용할 준비가 되면 [Azure](bot-builder-howto-deploy-azure.md) 또는 고유한 웹 서비스 또는 데이터 센터에 봇을 게시합니다. 사이트 또는 채팅 채널 내에서 봇을 사용하는 첫 번째 단계는 공용 인터넷에서 주소를 유지하는 것입니다.

### <a name="connect"></a>연결          
Facebook, Messenger, Kik, Skype, Slack, Microsoft Teams, Telegram, 텍스트/SMS, Twilio, Cortana 및 Skype와 같은 채널에 봇을 연결합니다. Bot Framework에서는 이러한 다양한 플랫폼에서 메시지를 송수신하는 데 필요한 대부분의 작업을 수행합니다. 봇 응용 프로그램은 연결된 채널의 수와 형식에 관계 없이 통합되고 정규화된 메시지 스트림을 수신합니다. 채널을 추가하는 정보는 [채널](bot-service-manage-channels.md) 토픽을 참조하세요.

### <a name="evaluate"></a>평가 
Azure Portal에서 수집된 데이터를 사용하여 봇의 성능과 기능을 향상시킬 기회를 확인합니다. 트래픽, 대기 시간 및 통합과 같은 서비스 수준 및 계측 데이터를 가져올 수 있습니다. 또한 분석은 사용자에 대한 대화 수준 보고, 메시지 및 채널 데이터를 제공합니다. 자세한 내용은 [분석을 수집하는 방법](bot-service-manage-analytics.md)을 참조하세요.


## <a name="next-steps"></a>다음 단계
이러한 봇의 [사례 연구](https://azure.microsoft.com/services/bot-service/)를 체크 아웃하거나 아래 링크를 클릭하여 봇을 만듭니다.
> [!div class="nextstepaction"]
> [봇 만들기](bot-service-quickstart.md)

::: moniker-end
