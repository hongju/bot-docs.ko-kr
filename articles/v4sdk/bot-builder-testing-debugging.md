---
title: 테스트 및 디버깅 지침 | Microsoft Docs
description: 봇을 테스트 및 디버그하는 방법을 살펴봅니다.
keywords: 테스트 원칙, 모의 요소, faq, 테스트 수준
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3e1ebc07c73dcd7033a6b9a22c94379593c5890e
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215258"
---
# <a name="testing-and-debugging-guidelines"></a>테스트 및 디버깅 지침

[!INCLUDE[applies-to](../includes/applies-to.md)]

봇은 수많은 다양한 파트가 함께 작동하는 복잡한 앱입니다. 다른 복잡한 앱의 경우처럼 흥미로운 버그가 발생하거나 봇이 예상과 다르게 동작할 수 있습니다.

경우에 따라 봇을 테스트한 후 디버그하는 것은 어려운 작업일 수 있습니다. 모든 개발자에게는 이런 작업을 수행하는 선호하는 방법이 있으며, 아래에 제공되는 지침은 대부분의 봇에 적용할 수 있는 제안 사항입니다.

## <a name="testing-your-bot"></a>봇 테스트

아래 지침은 세 가지 **수준**으로 제공됩니다.  수준에 따라 복잡성과 기능이 테스트에 추가되므로 한 수준에 만족한 후 다음 수준으로 이동하는 것이 좋습니다. 이렇게 하면 먼저 가장 낮은 수준 문제를 격리하고 수정한 후 복잡성을 추가할 수 있습니다.

테스트 모범 사례에서는 적용할 수 있는 모든 다양한 측면을 다룹니다. 이 모범 사례에는 보안, 통합, 잘못된 형식의 URL, 유효성 검사 악용, HTTP 상태 코드, JSON 페이로드, null 값 등이 포함됩니다. 봇이 사용자 개인 정보에 영향을 주는 정보를 처리할 경우 이 모범 사례가 특히 중요합니다.

### <a name="level-1-use-mock-elements"></a>수준 1: 모의 요소 사용

앱 또는 이 경우 봇의 각 작은 조각이 정확히 예상대로 작동하는지 확인하는 것이 테스트의 첫 번째 수준입니다. 이 작업을 수행하기 위해 현재 테스트하지 않는 항목에 대한 모의 요소를 사용할 수 있습니다. 참조를 위해 이 수준은 일반적으로 단위 또는 통합 테스트로 생각할 수 있습니다.

#### <a name="use-mock-elements-to-test-individual-sections"></a>모의 요소를 사용하여 개별 섹션 테스트

테스트 중인 조각을 더 효과적으로 격리하기 위해 가능한 만큼 많은 모의 요소를 만들 수 있습니다. 모의 요소의 후보에는 저장소, 어댑터, 미들웨어, 활동 파이프라인, 채널 및 직접적으로 봇의 파트가 아닌 모든 것이 포함됩니다. 이 작업으로 인해 각 조각을 격리하기 위해 테스트 중인 봇의 파트에 포함되지 않은 미들웨어와 같은 특정 측면이 일시적으로 제거될 수도 있습니다. 그러나 미들웨어를 테스트 중이라면 모의 봇을 만들 수 있습니다.

모의 요소를 만드는 데는 요소를 다른 알려진 개체로 바꾸는 것부터 기본 hello world 기능을 구현하는 것까지 몇 가지 방법이 사용될 수 있습니다. 단순히 필요하지 않은 요소를 제거하는 방법을 사용하거나 단순히 아무 작업도 수행하지 않도록 요소를 적용할 수도 있습니다. 

이 수준은 봇 내에서 개별 메서드 및 함수를 실행해야 합니다. 기본 제공 단위 테스트를 사용하거나(권장 방법), 고유한 테스트 앱이나 테스트 도구 모음을 사용하거나, 수동으로 IDE 내에서 작업을 수행하여 개별 메서드를 테스트할 수 있습니다. 

#### <a name="use-mock-elements-to-test-larger-features"></a>모의 요소를 사용하여 더 큰 기능 테스트

각 메서드의 동작 방식에 만족하면 이러한 모의 요소를 사용하여 봇에서 더 완전한 기능을 테스트합니다. 여기서는 사용자와 대화하기 위해 몇몇 레이어가 함께 어떻게 작동하는지를 보여 줍니다. 

이 작업을 돕기 위해 몇 가지 도구가 제공됩니다. 예를 들어, [Azure Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator)는 봇과 통신하기 위한 에뮬레이트된 채널을 제공합니다. 에뮬레이터를 사용하면 단위 및 통합 테스트보다 더 복잡한 상황에 접근할 수 있으므로 다음 테스트 수준으로 진행할 수 있습니다.

### <a name="level-2-use-a-direct-line-client"></a>수준 2: Direct Line 클라이언트 사용

봇이 원하는 대로 작동하는지 확인한 후 다음 단계에서 봇을 채널에 연결합니다. 이렇게 하려면 봇을 준비 서버에 배포하고 봇을 연결할 고유한 직접 회선 클라이언트를 만들면 됩니다.
<!--IBTODO [Direct Line client](bot-builder-howto-direct-line.md)-->

고유한 클라이언트를 만들면 채널의 내부 작업을 정의할 뿐만 아니라 특히 봇이 특정 활동 교환에 어떻게 응답하는지 테스트할 수 있습니다. 클라이언트에 연결된 후 테스트를 실행하여 봇 상태를 설정하고 기능을 확인합니다. 봇이 음성 같은 기능을 사용하는 경우, 이러한 채널을 사용하면 해당 기능을 확인할 수 있습니다.

여기서 Azure Portal 통해 Emulator와 웹 채팅을 둘 다 사용하면 다른 채널과 상호 작용하는 동안 봇의 작동 방식에 대한 인사이트를 더 자세히 제공할 수 있습니다.

### <a name="level-3-channel-tests"></a>수준 3: 채널 테스트

봇의 독립적인 성능을 자신할 수 있으면 봇을 사용할 수 있는 다양한 채널에서 봇이 어떻게 작동하는지 확인해야 합니다. 

이 작업을 수행하는 방법은 개별적으로 다른 채널 및 브라우저를 사용하는 방법부터 채널을 통해 상호 작용하고 봇의 응답을 스크랩하는 [Selenium](https://docs.seleniumhq.org/) 같은 타사 도구를 사용하는 방법까지 크게 달라질 수 있습니다.

### <a name="other-testing"></a>기타 테스트

다양한 유형의 테스트를 위의 수준과 함께 수행하거나 스트레스 테스트, 성능 테스트 또는 봇 활동 프로파일링과 같은 여러 가지 측면에서 수행할 수 있습니다. Visual Studio는 이 작업을 로컬에서 수행할 수 있는 메서드와 앱 테스트를 위한 [도구 모음](https://azure.microsoft.com/en-us/solutions/dev-test/)을 제공하고 [Azure Portal](https://portal.azure.com)은 봇의 작동 방식에 대한 인사이트를 제공합니다.

## <a name="debugging"></a>디버그

봇 디버그는 중단점을 설정하는 기능이나 직접 실행 창 같은 기능을 통해 다중 스레드 앱과 비슷하게 이루어집니다. 

봇은 이벤트 기반 프로그래밍 패러다임을 따르므로, 이 패러다임에 익숙하지 않을 경우 합리적으로 생각하기 어려울 수 있습니다. 봇이 상태 비저장, 다중 스레드로 설정되고 async/await 호출을 처리하면 예기치 않은 버그가 발생할 수 있습니다. 봇 디버그는 다른 다중 스레드 앱과 비슷하게 작동하지만 도움이 되는 몇 가지 제안 사항, 도구 및 리소스를 살펴보겠습니다.

### <a name="understanding-bot-activities-with-the-emulator"></a>에뮬레이터를 사용하여 봇 활동 이해

봇은 일반 ‘메시지’ 활동 외에도 다양한 유형의 [활동](bot-builder-basics.md#the-activity-processing-stack)을 처리합니다.  [에뮬레이터](../bot-service-debug-emulator.md)를 사용하면 해당 활동이 무엇이고, 언제 수행되고, 어떤 정보를 포함하는지 확인할 수 있습니다. 해당 활동을 파악하면 봇을 효율적으로 코딩할 수 있고 봇이 보내고 받는 활동이 예상한 활동인지 확인할 수 있습니다.

### <a name="saving-and-retrieving-user-interactions-with-transcripts"></a>대본을 통해 사용자 상호 작용 저장 및 검색

Azure Blob 기록 저장소는 사용자 및 봇 간의 상호 작용을 포함하는 [기록을 저장 및 검색](bot-builder-howto-v4-storage.md)을 모두 할 수 있는 특수화된 리소스를 제공합니다.  

또한 사용자 입력 상호 작용이 저장되면 Azure의 "_Storage Explorer_"를 사용하여 Blob 기록 저장소 내에 저장된 기록에 포함된 데이터를 수동으로 확인할 수 있습니다. 다음 예제에서는 "_mynewtestblobstorage_"에 대한 설정에서 "_스토리지 탐색기_"를 엽니다. 저장된 사용자 입력을 열려면    Blob 컨테이너 > ChannelId > TranscriptId > ConversationId를 차례로 선택합니다.

![Examine_stored_transcript_text](./media/examine_transcript_text_in_azure.png)

이는 저장된 사용자 대화 입력을 JSON 형식으로 엽니다. 사용자 입력은 "_텍스트:_ " 키와 함께 유지됩니다.

### <a name="how-middleware-works"></a>미들웨어의 작동 방식

[미들웨어](bot-builder-concept-middleware.md)는 처음 사용할 경우, 특히 실행의 연속 또는 단락 측면에서 사용하기 쉽지 않을 수 있습니다. 미들웨어는 실행이 봇 논리에 전달되는 시기를 지정하는 `next()` 대리자 호출을 통해 회전의 시작 또는 종료 시 실행될 수 있습니다. 

미들웨어의 여러 조각을 사용 중인 경우 대리자는 파이프라인에 지정된 대로 미들웨어의 다른 조각에 실행을 전달할 수 있습니다. [봇 미들웨어 파이프라인](bot-builder-concept-middleware.md#the-bot-middleware-pipeline)에 대한 세부 정보를 사용하여 아이디어를 더 분명하게 만들 수 있습니다.

`next()` 대리자가 호출되지 않는 경우 이를 [단락 라우팅](bot-builder-concept-middleware.md#short-circuiting)이라고 합니다. 미들웨어가 현재 활동에 만족하고 실행을 전달할 필요가 없다고 판단하면 이 경우가 발생합니다. 

미들웨어가 단락되는 시기와 이유를 파악하면 파이프라인에서 최우선으로 고려해야 하는 미들웨어의 조각을 나타낼 수 있습니다. 또한 SDK 또는 다른 개발자에 의해 제공되는 기본 제공 미들웨어의 경우, 예상 결과를 파악하는 것이 중요합니다. 기본 제공 미들웨어를 자세히 살펴보려면 먼저 고유한 미들웨어를 만들어 테스트해 보는 것이 유용할 수 있습니다.

<!-- Snip: QnA was once implemented as middleware.
For example [QnA maker](bot-builder-howto-qna.md) is designed to handle certain interactions and short-circuit the pipeline when it does, which can be confusing when first learning how to use it.
-->

### <a name="understanding-state"></a>상태 이해

상태 추적은, 특히 복잡한 작업의 경우, 봇의 중요한 부분입니다. 일반적으로 가능한 한 빨리 활동을 처리하고 해당 상태가 지속되도록 처리를 완료하는 것이 좋습니다. 활동이 거의 동시에 봇에 전송될 수 있고, 이 경우 비동기 아키텍처 때문에 매우 혼란스러운 버그가 도입될 수 있습니다.

가장 중요한 것은 상태가 예상대로 지속되는지 확인하는 것입니다. 지속된 상태의 위치에 따라 [Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/local-emulator) 및 [Azure Table Storage](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-emulator)용 스토리지 에뮬레이터를 사용하면 프로덕션 스토리지를 사용하기 전에 해당 상태를 확인할 수 있습니다.

### <a name="how-to-use-activity-handlers"></a>활동 처리기를 사용하는 방법

특히 각 활동이 독립적인 스레드(또는 사용자의 언어에 따라 웹 작업자)에서 실행되므로 활동 처리기는 또 다른 계층의 복잡성을 도입할 수 있습니다. 처리기가 수행하는 작업에 따라, 이로 인해 현재 상태가 예상한 상태가 아닌 문제가 발생할 수 있습니다.

기본 제공 상태는 턴의 끝에 기록되지만, 해당 턴에서 생성된 모든 활동은 턴 파이프라인과는 독립적으로 실행됩니다. 대부분 이는 별다른 영향이 없지만 활동 처리기가 상태를 변경하는 경우에는 해당 변경을 포함하는 상태가 기록되어야 합니다. 이 경우, 회전 파이프라인은 완료하기 전에 처리를 마칠 때까지 활동에서 대기할 수 있으므로 해당 회전에 대한 올바른 상태가 기록됩니다.

_활동 보내기_ 메서드와 해당 처리기에는 고유한 문제를 일으킵니다. _활동 보내기 시_ 처리기 내에서 _활동 보내기_를 호출하기만 하면 스레드가 무한히 포크됩니다. 이 문제는 나가는 정보에 추가 메시지를 추가하거나 봇이 충돌하지 않도록 다른 위치(예: 콘솔 또는 파일)에 기록하는 것과 같은 방법으로 해결할 수 있습니다.

## <a name="additional-resources"></a>추가 리소스

* [Visual Studio의 디버깅](https://docs.microsoft.com/en-us/visualstudio/debugger/index)
* Bot Framework에 대한 [디버깅, 추적 및 프로파일링](https://docs.microsoft.com/en-us/dotnet/framework/debug-trace-profile/)
* 프로덕션 코드에 포함하지 않을 메서드에 [ConditionalAttribute](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.conditionalattribute?view=netcore-2.0) 사용
* [Fiddler](https://www.telerik.com/fiddler) 같은 도구를 사용하여 네트워크 트래픽 확인
* [봇 도구 리포지토리](https://github.com/Microsoft/botbuilder-tools)
* 테스트에 [Moq](https://github.com/moq/moq4)와 같은 유용한 프레임워크 사용
* [일반 문제 해결](../bot-service-troubleshoot-bot-configuration.md) 및 해당 섹션의 다른 문제 해결 문서
