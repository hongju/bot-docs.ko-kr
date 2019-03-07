---
title: 기록 파일을 사용하여 봇 디버그 | Microsoft Docs
description: 기록 파일을 사용하여 봇을 디버그하는 방법을 알아봅니다.
keywords: 디버깅, faq, 기록 파일, 에뮬레이터
author: DanDev33
ms.author: v-dashel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservices: sdk
ms.date: 2/26/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 997ad82e15a0fcd67d47b2fd6495c8e88a5ea127
ms.sourcegitcommit: cf3786c6e092adec5409d852849927dc1428e8a2
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/01/2019
ms.locfileid: "57224821"
---
# <a name="debug-your-bot-using-transcript-files"></a>기록 파일을 사용하여 봇 디버그
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

봇을 성공적으로 테스트 및 디버깅하기 위한 요소 중 하나는 봇을 실행하는 동안 발생하는 조건 집합을 기록하고 검사하는 기능입니다. 이 문서에서는 봇 기록 파일을 만들어서 테스트 및 디버깅을 위한 구체적인 사용자 상호 작용 및 봇 응답 집합을 제공하는 방법을 설명합니다.

## <a name="the-bot-transcript-file"></a>봇 기록 파일
봇 기록 파일은 사용자와 봇 간의 상호 작용을 유지하는 특수 JSON 파일입니다. 기록 파일은 메시지의 콘텐츠뿐만 아니라 사용자 id, 채널 id, 채널 형식, 채널 기능, 상호 작용 시간 등의 상호 작용 세부 정보를 유지합니다. 이 모든 정보는 봇을 테스트 또는 디버깅할 때 문제를 찾아서 해결하는 데 사용할 수 있습니다. 

## <a name="creatingstoring-a-bot-transcript-file"></a>봇 기록 파일 만들기/저장
이 문서에서는 Microsoft의 [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator)를 사용하여 봇 기록 파일을 만드는 방법을 보여줍니다. 기록 파일을 프로그래밍 방식으로 만들 수도 있습니다. 이 방법에 대한 자세한 내용은 [여기](./bot-builder-howto-v4-storage.md#blob-transcript-storage)서 확인할 수 있습니다. 이 문서에서는 사용자의 이름 및 전화 번호를 요청하도록 수정된 [EchoBot with Counter](https://aka.ms/EchoBot-With-Counter)용 Bot Framework 샘플 코드를 사용하지만, Microsoft의 Bot Framework Emulator를 사용하여 액세스할 수 있는 아무 코드를 사용하여 기록 파일을 만들 수 있습니다.

이 프로세스를 시작하려면 테스트하려는 봇 코드가 개발 환경 내에서 실행되고 있는지 확인해야 합니다. Bot Framework Emulator를 시작하고, _Bot 열기_ 단추를 선택하고, 아래 이미지처럼 에뮬레이터를 코드의 _봇 구성_ 파일에 연결합니다.

![코드에 에뮬레이터 연결](./media/emulator_open_bot_configuration.png)

에뮬레이터를 실행 중인 코드에 연결한 후에는 시뮬레이션된 사용자 상호 작용을 봇에 전송하여 코드를 테스트합니다. 이 예제에서는 사용자 이름과 전화 번호를 전달했습니다. 유지하려는 사용자 상호 작용을 모두 입력한 후에는 Bot Framework Emulator를 사용하여 이 대화를 포함하는 기록 파일을 만들고 저장합니다. 

아래처럼 _실시간 채팅_ 탭 내에서 _기록 저장_ 단추를 선택합니다. 

![기록 저장 선택](./media/emulator_transcript_save.png)

기록 파일의 위치와 이름을 선택한 다음, [저장] 단추를 선택합니다.

![기록을 ursula로 저장](./media/emulator_transcript_saveas_ursula.png)

에뮬레이터로 코드를 테스트하기 위해 입력한 모든 사용자 상호 작용 및 봇 응답이 기록 파일에 저장되었으며, 나중에 이 기록 파일을 다시 로드하여 사용자와 봇 간의 상호 작용을 디버그하는 데 사용할 수 있습니다.

## <a name="retrieving-a-bot-transcript-file"></a>봇 기록 파일 검색
Bot Framework Emulator를 사용하여 봇 기록 파일을 검색하려면 아래와 같이 에뮬레이터의 왼쪽 위 모서리에 있는 _리소스_ 섹션의 _기록_ 목록 컨트롤을 선택합니다. 다음으로, 검색하려는 기록 파일을 선택합니다. 이 예제에서는 "Ursula_User_transcript"라는 기록 파일을 검색합니다. 기록 파일을 선택하면 유지된 전체 대화가 자동으로 _기록_이라는 제목이 새 탭에 로드됩니다.

![저장된 기록 검색](./media/emulator_transcript_retrieve.png)

## <a name="debug-using-transcript-file"></a>기록 파일을 사용하여 디버그
기록 파일이 로드되었으니, 사용자와 봇 간에 캡처한 상호 작용을 디버그할 준비가 완료되었습니다. 상호 작용을 디버그하려면 에뮬레이터의 오른쪽 아래 영역에 보이는 것처럼 _로그_ 섹션에 기록된 이벤트 또는 작업을 클릭합니다. 아래 예에서는 사용자가 "Hello" 메시지를 보낼 때의 첫 번째 상호 작용을 선택했습니다. 이렇게 할 때 이 상호 작용을 포함하는 기록 파일의 모든 정보가 에뮬레이터의 _검사기_ 창에 JSON 형식으로 표시됩니다. 아래에서 위쪽 방향으로 이러한 값 중 일부를 살펴보면, 다음과 같은 정보가 보입니다.
* 상호 작용 유형은 _message_였습니다.
* 메시지가 전송된 시간.
* 일반 텍스트로 전송되었으며 "Hello"가 포함되었습니다.
* 메시지가 봇으로 전송되었습니다.
* 사용자 id 및 정보.
* 채널 id, 기능 및 정보.

![기록을 사용하여 디버깅](./media/emulator_transcript_debug.png)

이 자세한 정보를 통해 사용자 입력과 봇 응답 간의 단계별 상호 작용을 추적할 수 있으며, 봇이 예상 방식대로 응답하지 않거나 사용자에게 전혀 응답하지 않는 상황을 디버깅하는 데 도움이 됩니다. 이러한 값과 실패한 상호 작용까지 이어지는 단계별 레코드를 모두 이용하여 코드를 살펴보고, 봇이 예상대로 응답하지 않는 위치를 찾고, 문제를 해결할 수 있습니다.

Bot Framework Emulator와 함께 기록 파일을 사용하는 것은 봇의 코드 및 사용자 상호 작용을 테스트하고 디버그하는 데 사용할 수 있는 여러 도구 중 하나일 뿐입니다. 봇을 테스트 및 디버그하는 방법을 더 보려면 아래에 나열된 추가 리소스를 참조하세요.

## <a name="additional-resources"></a>추가 리소스

테스트 및 디버깅에 대한 추가 정보는 다음 항목을 참조하세요.

* [봇 테스트 및 디버깅 지침](./bot-builder-testing-debugging.md)
* [Bot Framework Emulator를 사용하여 디버그](../bot-service-debug-emulator.md)
* [일반 문제 해결](../bot-service-troubleshoot-bot-configuration.md) 및 해당 섹션의 다른 문제 해결 문서
* [Visual Studio의 디버깅](https://docs.microsoft.com/en-us/visualstudio/debugger/index)
