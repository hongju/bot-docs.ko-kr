---
title: Bot Framework SDK 마이그레이션 개요 | Microsoft Docs
description: SDK의 변경 내용과 v3에서 v4로 마이그레이션하는 방법에 대한 개요를 제공합니다.
keywords: 봇 마이그레이션
author: mmiele
ms.author: v-mimiel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 06/11/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b1082e16933da1fb4c20f51d4764ec1774aabdb6
ms.sourcegitcommit: 4f78e68507fa3594971bfcbb13231c5bfd2ba555
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/17/2019
ms.locfileid: "68292193"
---
# <a name="migration-overview"></a>마이그레이션 개요

Bot Framework SDK v4는 이전 SDK의 고객 피드백 및 학습 경험을 기반으로 합니다. 새 버전에서는 적절한 수준의 추상화를 도입하는 한편, 봇 구성 요소의 아키텍처를 유연하게 사용하도록 설정할 수 있습니다. 예를 들어 간단한 봇을 만든 다음, Bot Framework SDK v4의 모듈화 및 확장성을 사용하여 정교하게 확장할 수 있습니다.

> [!NOTE]
> Bot Framework SDK v4는 간단한 작업을 단순하게 유지하려고 하고 복잡한 작업을 가능하게 합니다.

Bot Framework v4 SDK가 커뮤니티의 협력을 통해 빌드될 수 있도록 개방형 접근 방식이 채택되었습니다. 끌어오기 요청을 처음 제출하면 [CLA](https://cla.opensource.microsoft.com/)(기여자 사용권 계약)에서 라이선스가 필요한지 여부를 자동으로 결정합니다. 이 작업은 모든 리포지토리에서 한 번만 수행하면 됩니다. 일반적으로 달성할 일단의 목표를 설정하기 위한 시간 간격이 있습니다.

## <a name="what-happens-to-bots-built-using-sdk-v3"></a>SDK v3을 사용하여 빌드된 봇은 어떻게 되나요?

Bot Framework SDK v3은 사용되지 않지만 기존 V3 봇 워크로드는 중단 없이 계속 실행됩니다. 자세한 내용은 다음을 참조하세요. [Bot Framework SDK 버전 3 평생 지원](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-resources-bot-framework-faq?view=azure-bot-service-4.0#bot-framework-sdk-version-3-lifetime-support)

V3 봇은 V4로 마이그레이션하는 것이 좋습니다. 이 마이그레이션을 지원하기 위해 관련 설명서가 작성되었으며, 표준 채널을 통해 마이그레이션 이니셔티브에 대한 지원을 확장할 예정입니다.

## <a name="advantages"></a>장점

- 더 풍부하고 유연한 개방형 아키텍처: 더 유연한 대화 디자인을 사용하도록 설정
- 가용성: 새 채널 기능을 갖춘 추가 시나리오 소개
- 개발 주기에서 확장된 SME(실무 전문가) 담당자: 비개발자가 새 GUI 디자이너를 통해 대화 디자인에 협업할 수 있음
- 개발 속도: 새 디버깅 및 테스트 개발자 도구
- 성능 인사이트: 대화 품질을 평가하고 향상시키는 새 원격 분석 기능
- 인텔리전스: 향상된 인식 서비스 기능

## <a name="why-migrate"></a>마이그레이션하는 이유
<!-- [!] The declarative model introduced with Adaptive Dialogs would go great here (when ready).  -->
- 유연하고 향상된 대화 관리
  - 활동 처리를 위한 봇 어댑터
  - 리팩터링된 상태 관리
  - 새 대화 라이브러리
  - 구성 및 확장이 가능한 디자인을 위한 미들웨어: 동작을 사용자 정의하기 위한 깨끗하고 일관된 후크
- .NET Core용으로 빌드
  - 성능 향상
  - 플랫폼 간 호환성(Windows/Mac/Linux)
- 여러 프로그래밍 언어에서 일관된 프로그래밍 모델
- 향상된 설명서
- 확장된 디버깅 기능을 제공하는 봇 검사기
- 가상 도우미
  - 기본 대화 의도, 디스패치 통합, QnA Maker, Application Insights 및 자동화된 배포를 통해 봇 만들기를 간소화하는 포괄적인 솔루션입니다.
  - 확장 가능한 Skills: Skills로 알려진 다시 사용할 수 있는 대화형 기능을 모두 연결하여 대화형 환경을 구성합니다.
- 프레임워크 테스트: 새 전송 독립적 어댑터 아키텍처를 사용하여 즉시 테스트할 수 있음
- 원격 분석: 대화형 AI 분석을 사용하여 봇의 상태와 동작에 대한 핵심 인사이트를 얻을 수 있음
- 출시 예정(미리 보기)
  - 적응형 대화: 대화 진행에 따라 동적으로 변경될 수 있는 대화 빌드
  - 언어 생성: 구에 대한 여러 변형 정의
- 미래
  - 추상화 수준을 디자이너에 허용하는 선언적 디자인
  - GUI 대화 디자이너
- Azure Bot Service 
  - Direct Line Speech 채널: Bot Framework와 Microsoft의 음성 서비스를 결합합니다. 이렇게 하면 클라이언트 및 봇 애플리케이션 간에 양방향으로 스트림되는 음성과 텍스트를 사용하도록 설정할 수 있는 채널이 제공됩니다.

## <a name="whats-changed"></a>변경된 내용

Bot Framework SDK v4는 v3과 동일한 기본 Bot Framework Service를 지원합니다. 그러나 v4는 이전 SDK를 리팩터링하여 봇 만들기를 더 유연하게 제어할 수 있습니다. 이 제품에는 다음이 포함됩니다.

- 봇 어댑터 도입
  - 어댑터는 활동 처리 스택의 일부입니다.
  - Bot Framework 인증을 처리하고, 각 턴에 대한 컨텍스트를 초기화합니다.
  - 채널과 봇의 턴 처리기 간에 들어오고 나가는 트래픽을 관리하여 Bot Framework 커넥터 서비스에 대한 호출을 캡슐화합니다.
  - 자세한 내용은 [봇 작동 방식](../bot-builder-basics.md)을 참조하세요.
- 리팩터링된 상태 관리
  - 상태 데이터는 더 이상 봇 내에서 자동으로 사용할 수 없습니다.
  - 상태는 이제 상태 관리 개체 및 속성 접근자를 통해 관리됩니다.
  - 자세한 내용은 [상태 관리](../bot-builder-concept-state.md)를 참조하세요.
- 새 Dialogs 라이브러리 도입
  - v3 대화는 새 대화 라이브러리에 맞게 다시 작성해야 합니다.
  - 자세한 내용은 [Dialogs 라이브러리](../bot-builder-concept-dialog.md)를 참조하세요.

## <a name="whats-involved-in-migration-work"></a>마이그레이션 작업과 관련된 내용

- 설치 논리 업데이트
- 중요한 사용자 상태 포팅
  - 참고: 중요한 사용자 상태는 봇 상태에서 유지할 수 없습니다. 대신 사용자가 제어할 수 있는 별도의 스토리지에 저장하세요.
- 봇 포팅 및 대화 논리(자세한 내용은 언어별 항목 참조)

### <a name="migration-estimation-worksheet"></a>마이그레이션 예측 워크시트

다음 워크시트는 마이그레이션 워크로드를 예측하는 데 도움을 줄 수 있습니다. **발생 수** 열의 *개수*는 실제 숫자 값으로 바꿉니다. **티셔츠** 열에는 예측에 따라 *작음*, *중간*, *큼*과 같은 값을 입력합니다.

단계 | V3 | V4 | 발생 수 | 복잡성 | 티셔츠
-- | -- | -- | -- | -- | --
들어오는 작업 가져오기 | IDialogContext.Activity | ITurnContext.Activity | count | 작음  
작업을 만들어 사용자에게 보내기 | activity.CreateReply(“text”) IDialogContext.PostAsync | MessageFactory.Text(“text”) ITurnContext.SendActivityAsync | count | 작음 |
상태 관리 | UserData, ConversationData 및 PrivateConversationData context.UserData.SetValue context.UserData.TryGetValue botDataStore.LoadAsyn | UserState, ConversationState 및 PrivateConversationState(속성 접근자 포함) | context.UserData.SetValue - 개수 context.UserData.TryGetValue - 개수 botDataStore.LoadAsyn - 개수 | 중간 ~ 큼(사용 가능한 [사용자 상태 관리](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-state?view=azure-bot-service-4.0#state-management) 참조) |
대화의 시작 처리 | IDialog.StartAsync 구현 | 폭포 대화의 첫 번째 단계로 만듭니다. | count | 작음 |  
활동 보내기 | IDialogContext.PostAsync | Call ITurnContext.SendActivityAsync | count | 작음 |  
사용자 응답 대기 | an IAwaitable<IMessageActivity>parameter 및 call IDialogContext.Wait 사용 | await ITurnContext.PromptAsync를 반환하여 프롬프트 대화를 시작합니다. 그런 다음, 폭포의 다음 단계에서 결과를 검색합니다. | count | 중간(흐름에 따라 다름) |  
대화의 연속 처리 | IDialogContext.Wait | 폭포 대화에 추가 단계 추가 또는 Dialog.ContinueDialogAsync 구현 | count | 큰 |  
사용자의 다음 메시지가 표시될 때까지 처리 끝 신호 보내기 | IDialogContext.Wait | Dialog.EndOfTurn 반환 | count | 중간 |  
자식 대화 시작 | IDialogContext.Call | 단계 컨텍스트의 BeginDialogAsyncmethod에 대한 await를 반환합니다. 자식 대화에서 값을 반환하는 경우 해당 값은 단계 컨텍스트의 Resultproperty를 통해 폭포의 다음 단계에서 사용할 수 있습니다. | count | 중간 |  
현재 대화를 새 대화로 바꾸기 | IDialogContext.Forward | await ITurnContext.ReplaceDialogAsync 반환 | count | 큰 |  
현재 대화에 대한 완료 신호 | IDialogContext.Done | 단계 컨텍스트의 EndDialogAsync 메서드에 대한 await 반환 | count | 중간 |  
대화 실패 | IDialogContext.Fail | 봇의 다른 수준에서 포착할 예외를 throw하거나, 단계를 Cancelled(취소됨) 상태로 종료하거나, 단계 또는 대화 컨텍스트의 CancelAllDialogsAsync를 호출합니다. | count | 작음 |  

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

Bot Framework SDK v4는 v3과 동일한 기본 REST API를 기반으로 합니다. 그러나 v4는 이전 버전의 SDK를 리팩터링하여 봇을 더 유연하게 제어할 수 있습니다.

성능이 크게 향상되었으므로 .NET Core로 마이그레이션하는 것이 좋습니다. 그러나 기존 V3 봇 중 일부는 .NET Core에 해당하지 않는 외부 라이브러리를 사용하고 있습니다. 이 경우 .NET Framework 버전 4.6.1 이상에서 Bot Framework SDK v4를 사용할 수 있습니다. 예제는 [corebot](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_webapi) 위치에서 찾을 수 있습니다.

프로젝트를 v3에서 v4로 마이그레이션하는 경우 **.NET Framework**에 맞게 변환 또는 **.NET Core**용 새 프로젝트로 포팅 옵션 중에서 하나를 선택할 수 있습니다.

#### <a name="net-framework"></a>.NET Framework

- NuGet 패키지 업데이트 및 설치
- Global.asax.cs 파일 업데이트
- MessagesController 클래스 업데이트
- 대화 변환

자세한 내용은 [.NET v3 봇을 .NET Framework v4 봇으로 마이그레이션](conversion-framework.md)을 참조하세요.

#### <a name="net-core"></a>.NET Core

- 템플릿을 사용하여 새 프로젝트 만들기
- 필요에 따라 추가 NuGet 패키지 설치
- 봇 맞춤 설정, Startup.cs 파일을 업데이트 및 컨트롤러 클래스 업데이트
- 봇 클래스 업데이트
- 대화와 모델 복사 및 업데이트

자세한 내용은 [.NET v3 봇을 .NET Core v4 봇으로 마이그레이션](conversion-core.md)을 참조하세요.

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**Bot Framework JavaScript SDK v4**는 봇이 작성되는 방식에 대한 몇 가지 기본적인 변경 내용을 소개합니다. 이러한 변경 내용은 특히 봇 개체 만들기, 대화 정의 및 이벤트 처리 논리 코딩과 관련하여 Javascript에서 봇을 개발하기 위한 구문에 영향을 줍니다. Bot Framework SDK v4는 v3과 동일한 기본 REST API를 기반으로 합니다. 그러나 v4는 특히 이전 버전의 SDK를 리팩터링하여 봇을 더 유연하게 제어할 수 있습니다.

- 대화와 봇 인스턴스가 추가로 분리되었습니다. v3에서는 대화 상자가 봇의 생성자에서 직접 등록되었습니다. v4에서는 이제 대화를 인수로 봇 인스턴스에 전달하여 구성 유연성을 향상시킵니다.
- v4는 메시지, 대화 업데이트 및 이벤트 활동과 같은 다양한 유형의 활동을 자동으로 처리하는 데 도움이 되는 `ActivityHandler` 클래스를 제공합니다.
- NodeJS 봇 마이그레이션의 경우 TypeScript 또는 JavaScript에서 새 v4 NodeJS 봇을 만들어야 합니다. 관련 마이그레이션 설명서에서 설명하는 새 v4 구문을 사용하여 봇 논리를 다시 만듭니다.

#### <a name="migrate-from-v3-to-v4"></a>v3에서 v4로 마이그레이션

- 새 프로젝트 만들기 및 종속성 추가
- 진입점 업데이트 및 상수 정의
- 대화 만들기 및 SDK v4를 사용하여 다시 구현
- 대화를 실행하도록 봇 코드 업데이트
- `store.js` 유틸리티 파일 포팅

자세한 내용은 [SDK v3 Javascript 봇을 v4로 마이그레이션](conversion-javascript.md)을 참조하세요.

---

## <a name="additional-resources"></a>추가 리소스

마이그레이션 중에 도움이 되는 추가 정보를 제공하는 추가 리소스는 다음과 같습니다.  

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

<!-- _Mini-TOC with explainer for .NET topics_ -->
다음 항목에서는 .NET v3 및 v4 Bot Framework SDK 간의 차이점, 두 버전 간의 주요 변경 내용 및 봇을 v3에서 v4로 마이그레이션하는 단계에 대해 설명합니다.

| 항목 | 설명 |
| :--- | :--- |
| [v3 및 v4 .NET SDK 간의 차이점](migration-about.md) |v3 및 v4 SDK 간의 일반적인 차이점 |
| [.NET 마이그레이션 빠른 참조](net-migration-quickreference.md) |v3 및 v4 SDK의 주요 변경 내용 |
| [.NET v3 봇을 Framework v4 봇으로 마이그레이션](conversion-framework.md) |동일한 프로젝트 형식을 사용하여 v3을 v4 봇으로 마이그레이션 |
| [.NET v3 봇을 Core v4 봇으로 마이그레이션](conversion-core.md) | 새 .NET Core 프로젝트에서 v3을 v4 봇으로 마이그레이션|

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

<!-- _Mini-TOC with explainer for JavaScript topics_ -->
다음 항목에서는 JavaScript v3 및 v4 Bot Framework SDK 간의 차이점, 두 버전 간의 주요 변경 내용 및 봇을 v3에서 v4로 마이그레이션하는 단계에 대해 설명합니다.

| 항목 | 설명 |
| :--- | :--- |
| [v3 및 v4 JavaScript SDK 간의 차이점](migration-about-javascript.md) | v3 및 v4 SDK 간의 일반적인 차이점 |
| [JavaScript 마이그레이션 빠른 참조](javascript-migration-quickreference.md)| v3 및 v4 SDK의 주요 변경 내용|
| [JavaScript v3 봇을 v4로 마이그레이션](conversion-javascript.md) |v3을 v4 봇으로 마이그레이션 |

---

### <a name="code-samples"></a>코드 샘플

Bot Framework SDK V4를 학습하거나 프로젝트 시작으로 이동하는 데 사용할 수 있는 코드 샘플은 다음과 같습니다.

| 샘플 | 설명 |
| :--- | :--- |
| [V3에서 V4 샘플로 Bot Framework 마이그레이션](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4) <img width="200">| Bot Framework V3 SDK에서 V4 SDK로 샘플 마이그레이션 |
| [Bot Builder .NET 샘플](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore) | Bot Builder C# .NET Core 샘플 |
| [Bot Builder JavaScript 샘플](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs) | Bot Builder JavaScript(node.js) 샘플 |
| [모든 Bot Builder 샘플](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples) | 모든 Bot Builder 샘플 |

### <a name="getting-help"></a>도움말 보기

봇 개발을 위한 추가 정보와 지원을 제공하는 리소스는 다음과 같습니다.

[Bot Framework 추가 리소스](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-resources-links-help?view=azure-bot-service-4.0)

### <a name="references"></a>참조

자세한 내용과 배경 지식은 다음 리소스를 참조하세요.

| 항목 | 설명 |
| :--- | :--- |
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
