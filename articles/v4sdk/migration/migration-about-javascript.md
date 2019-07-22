---
title: v3 및 v4 NodeJS SDK 간의 차이점 | Microsoft Docs
description: v3 및 v4 NodeJS SDK 간의 차이점에 대해 설명합니다.
keywords: 봇 마이그레이션, 대화, 상태
author: mmiele
ms.author: v-mimiel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 07/08/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 24725df2f109e73df142194c05ee27c2d8ba5da4
ms.sourcegitcommit: 4f78e68507fa3594971bfcbb13231c5bfd2ba555
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/17/2019
ms.locfileid: "68292213"
---
# <a name="differences-between-the-v3-and-v4-javascript-sdk"></a>v3 및 v4 JavaScript SDK 간의 차이점

Bot Framework SDK 버전 4는 버전 3과 동일한 기본 Bot Framework Service를 지원합니다. 그러나 v4는 이전 버전의 SDK를 리팩터링하여 개발자가 자신의 봇을 더 유연하게 제어할 수 있도록 합니다. SDK의 주요 변경 내용은 다음과 같습니다.

- 봇 어댑터 소개
  - 활동 처리 스택의 일부입니다.
  - Bot Framework 인증을 처리합니다.
  - 채널과 봇의 턴 처리기 간에 들어오고 나가는 트래픽을 관리하여 Bot Framework 커넥터에 대한 호출을 캡슐화합니다.
  - 각 턴에 대한 컨텍스트를 초기화합니다.
  - 자세한 내용은 [봇 작동 방식](../bot-builder-basics.md)을 참조하세요.
- 리팩터링된 상태 관리
  - 상태 데이터는 더 이상 봇 내에서 자동으로 사용할 수 없습니다.
  - 이제 상태 관리 개체 및 속성 접근자를 통해 관리됩니다.
  - 자세한 내용은 [상태 관리](../bot-builder-concept-state.md)를 참조하세요.
- 새 Dialogs 라이브러리
  - v3 대화는 새 대화 라이브러리에 맞게 다시 작성해야 합니다.
  - Scorable 클래스가 더 이상 존재하지 않습니다. 대화 상자에 제어를 전달하기 전에 "global" 명령을 확인할 수 있습니다. v4 봇을 디자인하는 방법에 따라 메시지 처리기 또는 부모 대화 상자에서 이 작업을 할 수 있습니다. 예를 들어 [사용자 중단을 처리하는 방법](../bot-builder-howto-handle-user-interrupt.md)을 참조하세요.
  - 자세한 내용은 [Dialogs 라이브러리](../bot-builder-concept-dialog.md)를 참조하세요.

## <a name="activity-processing"></a>작업 처리

봇 어댑터를 만들 때 채널과 사용자로부터 들어오는 작업을 받을 메시지 처리기 대리자도 제공합니다. 어댑터는 들어오는 각 작업에 대한 턴 컨텍스트 개체를 만듭니다. 턴 컨텍스트 개체를 봇의 턴 처리기에 전달한 다음, 턴이 완료되면 개체를 삭제합니다.

턴 처리기는 다양한 유형의 작업을 받을 수 있습니다. 일반적으로 _메시지_ 작업만 봇에 포함된 대화에 전달하려고 합니다. 봇을 `ActivityHandler`에서 파생하는 경우 봇의 턴 처리기가 모든 메시지 작업을 `OnMessage`에 전달합니다. 메시지 처리 논리를 추가하려면 이 메서드를 재정의합니다. 활동 유형에 대한 자세한 내용은 [활동 스키마](https://github.com/Microsoft/botframework-sdk/blob/master/specs/botframework-activity/botframework-activity.md)를 참조하세요.

### <a name="handling-turns"></a>턴 처리

메시지를 처리하는 경우 턴 컨텍스트를 사용하여 들어오는 작업에 대한 정보를 가져오고 사용자에게 작업을 보냅니다.

|작업 |설명 |
|:---|:---|
| 들어오는 작업 가져오기 | 턴 컨텍스트의 `Activity` 속성을 가져옵니다. |
| 작업을 만들어 사용자에게 보내기 | 턴 컨텍스트의 `SendActivity` 메서드를 호출합니다.<br/> 자세한 내용은 [문자 메시지 보내기 및 받기](../../rest-api/bot-framework-rest-direct-line-1-1-receive-messages.md) 및 [메시지에 미디어 추가](../bot-builder-howto-add-media-attachments.md)를 참조하세요. |

`MessageFactory` 클래스는 작업을 만들고 형식을 지정하는 몇 가지 도우미 메서드를 제공합니다.

### <a name="interruptions"></a>중단

이러한 작업은 봇의 메시지 루프에서 처리합니다. v4 대화에서 이 작업을 수행하는 방법에 대한 자세한 내용은 [사용자 중단 처리](../bot-builder-howto-handle-user-interrupt.md)를 참조하세요.

## <a name="state-management"></a>상태 관리

v3의 경우 Bot Framework가 제공한 더 큰 서비스 제품군의 일부로 대화 데이터를 Bot State 서비스에 저장할 수 있었습니다. 그러나 이 서비스는 2018년 3월 31일 이후로 사용되지 않습니다. v4부터 상태 관리 디자인에 대한 고려 사항은 Web App과 동일하며 사용할 수 있는 많은 옵션이 있습니다. 일반적으로 상태를 같은 프로세스에서 메모리에 캐시하는 것이 가장 쉽지만, 프로덕션 앱의 경우 SQL 또는 NoSQL 데이터베이스에 저장하거나 BLOB으로 저장하는 것과 같이 더 영구적으로 저장하는 것이 좋습니다.

v4는 상태를 관리하는 데 `UserData`, `ConversationData` 및 `PrivateConversationData` 속성 및 데이터 모음을 사용하지 않습니다.
상태는 이제 [상태 관리](../bot-builder-concept-state.md)에서 설명한 대로 상태 관리 개체와 속성 접근자를 통해 관리됩니다.

v4는 봇에 대한 상태 데이터를 관리하는 `UserState`, `ConversationState` 및 `PrivateConversationState` 클래스를 정의합니다. 미리 정의된 데이터 모음에서 읽고 쓰는 대신, 유지하려는 각 속성에 대한 상태 속성 접근자를 만들어야 합니다.

### <a name="setting-up-state"></a>상태 설정

상태는 애플리케이션 진입점 파일(일반적으로 NodeJS 애플리케이션의 'index.js' 또는 'app.js')에서 구성해야 합니다. 

1. botbuilder-core에서 제공하는 `Storage` 인터페이스를 구현하는 하나 이상의 개체를 초기화합니다. 이 개체는 봇의 데이터에 대한 백업 저장소를 나타냅니다.
    v4 SDK는 몇 가지 [스토리지 계층](../bot-builder-concept-state.md#storage-layer)을 제공합니다.
    또한 다른 유형의 저장소에 연결하도록 직접 구현할 수도 있습니다.
1. 그런 다음, 필요에 따라 [상태 관리](../bot-builder-concept-state.md#state-management) 개체를 만들어 등록합니다.
    v3과 동일한 범위를 사용할 수 있으며, 필요한 경우 다른 범위를 만들 수 있습니다.
1. 마지막으로, 봇에 필요한 속성에 대한 [상태 속성 접근자](../bot-builder-concept-state.md#state-property-accessors)를 만들어 등록합니다.
    상태 관리 개체 내에서 각 속성 접근자에는 고유한 이름이 필요합니다.

### <a name="using-state"></a>상태 사용

상태 속성 접근자를 사용하여 속성을 가져와서 업데이트하고, 상태 관리 개체를 사용하여 변경 내용을 스토리지에 기록합니다. 동시성 문제를 고려하여 수행해야 하는 몇 가지 일반적인 작업은 다음과 같습니다.

| Task|설명 |
|:---|:---|
| 상태 속성 접근자 만들기 | `BotState` 개체의 `createProperty` 메서드를 호출합니다. <br/>`BotState`는 대화, 프라이빗 대화 및 사용자 상태에 대한 추상 기본 클래스입니다. |
| 속성의 현재 값 가져오기 | `StatePropertyAccessor.get(TurnContext)`를 호출합니다.<br/>이전에 설정한 값이 없으면 출하 시 기본 매개 변수를 사용하여 값을 생성합니다. |
| 스토리지에 대한 상태 변경 내용 유지 | 턴 처리기를 종료하기 전에 상태가 변경된 상태 관리 개체에 대해 `BotState.saveChanges(TurnContext, boolean)`를 호출합니다. |

### <a name="managing-concurrency"></a>동시성 관리

봇에서 상태 동시성을 관리해야 할 수도 있습니다. 자세한 내용은 **상태 관리**의 [상태 저장](../bot-builder-concept-state.md#saving-state) 섹션 및 **스토리지에 직접 작성**의 [eTags를 사용하여 동시성 관리](../bot-builder-howto-v4-storage.md#manage-concurrency-using-etags) 섹션을 참조하세요.

## <a name="dialogs-library"></a>대화 상자 라이브러리

대화의 몇 가지 주요 변경 내용은 다음과 같습니다.

- 이제 Dialogs 라이브러리는 별도의 **botbuilder-dialogs** npm 패키지입니다.
- 대화 상태는 `DialogState` 상태 클래스 및 속성 접근자를 통해 관리됩니다.
  - 대화 상태 속성은 이제 대화 개체 자체와는 달리 턴 간에 유지됩니다.
- _대화 집합_에 대한 대화 컨텍스트를 한 턴 내에 만듭니다.
  - 이 대화 컨텍스트는 대화 스택을 캡슐화합니다. 이 정보는 대화 상태 속성 내에서 유지됩니다.
- 두 버전은 모두 `Dialog` 추상 클래스를 포함하지만, v3에서는 'ActionSet' 클래스를 확장하는 반면 v4에서는 'Object' 클래스를 확장합니다.

### <a name="defining-dialogs"></a>대화 정의

v3는 `Dialog` 클래스를 사용하여 대화를 구현하는 유연한 방법을 제공했지만, 유효성 검사와 같은 기능에 대한 사용자의 고유의 코드를 구현해야 했습니다. v4에서는 이제 사용자 입력을 특정 유형(예: 정수)으로 제한하고 사용자가 올바른 입력을 제공할 때까지 사용자 프롬프트를 다시 표시하여 사용자 입력의 유효성을 자동으로 검사하는 새 프롬프트 클래스가 있으므로 개발자가 유효성 검사를 구현하지 않아도 됩니다. 일반적으로 이렇게 되면 개발자가 작성할 코드가 적어집니다.

이제 대화를 정의하는 방법에 대한 몇 가지 옵션이 있습니다.

|대화 유형| 설명 |
|:---|:---|
| `ComponentDialog` 클래스에서 파생된 구성 요소 대화 상자 | 지정한 이름이 외부 컨텍스트와 충돌하지 않도록 대화 상자 코드를 캡슐화할 수 있습니다. [대화 재사용](../bot-builder-concept-dialog.md)을 참조하세요.|
| `WaterfallDialog` 클래스의 인스턴스인 폭포 대화 상자 | 다양한 유형의 사용자 입력을 요청하고 유효성을 검사하는 프롬프트 대화 상자에서 잘 작동합니다. 폭포는 대부분의 프로세스를 자동화하지만 특정 양식을 대화 코드에 적용합니다. [순차적 대화 흐름](../bot-builder-dialog-manage-conversation-flow.md)을 참조하세요. |
| `Dialog` 추상 클래스에서 파생된 사용자 지정 대화 상자 | 이 옵션은 대화의 작동 방식이 가장 유연하지만 대화 스택의 구현 방식에 대해서도 자세히 알고 있어야 합니다. |

폭포 대화 상자를 만들 때 생성자에 대화 상자의 단계를 정의합니다. 단계가 실행되는 순서는 선언하는 방법을 그대로 따르며 자동으로 한 단계씩 차례로 실행해 나갑니다.

또한 여러 대화를 사용하여 복잡한 제어 흐름을 만들 수도 있습니다. [고급 대화 흐름](../bot-builder-dialog-manage-complex-conversation-flow.md)을 참조하세요.

대화에 액세스하려면 해당 대화의 인스턴스를 _대화 집합_에 배치한 다음, 해당 집합에 대한 _대화 컨텍스트_를 생성해야 합니다. 대화 집합을 만들 때 대화 상태 속성 접근자를 제공해야 합니다. 이렇게 하면 프레임워크가 대화 상태를 한 턴에서 다음 턴으로 유지할 수 있습니다. [상태 관리](../bot-builder-concept-state.md)는 v4에서 상태를 관리하는 방법을 설명합니다.

### <a name="using-dialogs"></a>대화 사용

v3의 일반적인 작업 목록과 폭포 대화 내에서 해당 작업을 수행하는 방법은 다음과 같습니다. 폭포 대화의 각 단계에서 `DialogTurnResult` 값을 반환해야 합니다. 그렇지 않으면 폭포가 일찍 종료될 수 있습니다.

| 작업(Operation) | v3 | v4 |
|:---|:---|:---|
| 대화의 시작 처리 | `session.beginDialog`를 호출하고 대화 ID를 전달합니다. | `DialogContext.beginDialog`에 전화를 겁니다. |
| 활동 보내기 | `session.send`를 호출합니다. | `TurnContext.sendActivity`를 호출합니다.<br/>단계 컨텍스트의 `Context` 속성을 사용하여 턴 컨텍스트(`step.context.sendActivity`)를 가져옵니다.  |
| 사용자 응답 대기 | 폭포 단계 내에서 프롬프트를 호출합니다(예: `builder.Prompts.text(session, 'Please enter your destination')`). 다음 단계에서 응답을 검색합니다. | `TurnContext.prompt`를 기다리는 await를 반환하여 프롬프트 대화를 시작합니다. 그런 다음, 폭포의 다음 단계에서 결과를 검색합니다. |
| 대화의 연속 처리 | 자동 | 폭포 대화에 추가 단계 추가 또는 `Dialog.continueDialog` 구현 |
| 사용자의 다음 메시지가 표시될 때까지 처리 끝 신호 보내기 | `session.endDialog`를 호출합니다. | `Dialog.EndOfTurn`을 반환합니다. |
| 자식 대화 시작 | `session.beginDialog`를 호출합니다. | 단계 컨텍스트의 `beginDialog` 메서드를 기다리는 await를 반환합니다.<br/>자식 대화에서 값을 반환하면 단계 컨텍스트의 `Result` 속성을 통해 폭포의 다음 단계에서 해당 값을 사용할 수 있습니다. |
| 현재 대화를 새 대화로 바꾸기 | `session.replaceDialog`를 호출합니다. | `ITurnContext.replaceDialog`를 기다리는 await를 반환합니다. |
| 현재 대화에 대한 완료 신호 | `session.endDialog`를 호출합니다. | 단계 컨텍스트의 `endDialog` 메서드를 기다리는 await를 반환합니다. |
| 대화 실패 | `session.pruneDialogStack`를 호출합니다. | 봇의 다른 수준에서 포착할 예외를 throw하거나, 단계를 `Cancelled` 상태로 종료하거나, 단계 또는 대화 컨텍스트의 `cancelAllDialogs`를 호출합니다. |

v4 코드에 대한 기타 참고 사항은 다음과 같습니다.

- v4의 다양한 `Prompt` 파생 클래스는 사용자 프롬프트를 별도의 2단계 대화로 구현합니다. [순차적 대화 흐름을 구현하는 방법][sequential-flow]을 참조하세요.
- `DialogSet.createContext`를 사용하여 현재 턴에 대한 대화를 만듭니다.
- 대화 내에서 `DialogContext.context` 속성을 사용하여 현재 턴 컨텍스트를 가져옵니다.
- 폭포 단계에는 `DialogContext`에서 파생된 `WaterfallStepContext` 매개 변수가 있습니다.
- 모든 구체적인 대화 및 프롬프트 클래스는 `Dialog` 추상 클래스에서 파생됩니다.
- 구성 요소 대화를 만들 때 ID를 할당합니다. 대화 집합의 각 대화에는 해당 집합 내에서 고유한 ID가 할당되어야 합니다.

### <a name="passing-state-between-and-within-dialogs"></a>대화 간 및 대화 내 상태 전달

v4의 대화 상태를 관리하는 방법은 **대화 라이브러리** 문서의 [대화 상태 ](../bot-builder-concept-dialog.md#dialog-state), [폭포 단계 컨텍스트 속성](../bot-builder-concept-dialog.md#waterfall-step-context-properties) 및 [대화 사용](../bot-builder-concept-dialog.md#using-dialogs) 섹션에서 설명하고 있습니다.

### <a name="get-user-response"></a>사용자 응답 가져오기

한 턴에 사용자의 작업을 가져오려면 턴 컨텍스트에서 가져옵니다.

사용자에게 프롬프트를 표시하고 프롬프트 결과를 받으려면 다음을 수행합니다.

- 대화 집합에 적절한 프롬프트 인스턴스를 추가합니다.
- 폭포 대화의 한 단계에서 프롬프트를 호출합니다.
- 다음 단계에서 단계 컨텍스트의 `Result` 속성으로부터 결과를 검색합니다.

## <a name="additional-resources"></a>추가 리소스

자세한 내용과 배경 지식은 다음 리소스를 참조하세요.

| 항목 | 설명 |
| :--- | :--- |
|[Javascript SDK v3 봇을 v4로 마이그레이션](https://docs.microsoft.com/en-us/azure/bot-service/migration/conversion-javascript?view=azure-bot-service-4.0)| v3 JavaScript 봇을 v4로 포팅|
| [Bot Framework의 새로운 기능](https://docs.microsoft.com/en-us/azure/bot-service/what-is-new?view=azure-bot-service-4.0) | Bot Framework 및 Azure Bot Service의 주요 기능 및 향상된 기능|
|[봇 작동 방식](../bot-builder-basics.md)|봇의 내부 메커니즘|
|[상태 관리](../bot-builder-concept-state.md)|상태 관리를 더 쉽게 수행하기 위한 추상화|
|[다이얼로그 라이브러리](../bot-builder-concept-dialog.md)| 대화 관리에 대한 중심 개념|
|[문자 메시지 보내기 및 받기](../bot-builder-howto-send-messages.md)|봇에서 사용자와 통신하는 기본 방법|
|[미디어 보내기](../bot-builder-howto-add-media-attachments.md)|이미지, 비디오, 오디오 및 파일과 같은 미디어 첨부 파일| 
|[순차적 대화 흐름](../bot-builder-dialog-manage-conversation-flow.md)| 봇에서 사용자와 상호 작용하는 주요 방법으로 수행하는 질문|
|[사용자 및 대화 데이터 저장](../bot-builder-howto-v4-state.md)|상태 비저장 중인 대화 추적|
|[복잡한 흐름](../bot-builder-dialog-manage-complex-conversation-flow.md)|복잡한 대화 흐름 관리 |
|[대화 재사용](../bot-builder-compositcontrol.md)|특정 시나리오를 처리하는 독립 대화 만들기|
|[중단](../bot-builder-howto-handle-user-interrupt.md)| 강력한 봇을 만들기 위한 중단 처리|
|[활동 스키마](https://aka.ms/botSpecs-activitySchema)|사용자 및 자동화된 소프트웨어에 대한 스키마|