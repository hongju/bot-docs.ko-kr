---
title: 봇 문제 해결 | Microsoft Docs
description: 기술 관련 질문과 대답을 통해 봇 개발에서의 일반적인 문제를 해결합니다.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/26/2019
ms.openlocfilehash: 48a0a42d193b0e561a484330222217c18a611e8d
ms.sourcegitcommit: cf3786c6e092adec5409d852849927dc1428e8a2
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/01/2019
ms.locfileid: "57224951"
---
# <a name="troubleshooting-general-problems"></a>일반 문제 해결
이 질문과 대답은 일반 봇 개발 또는 운영 문제를 해결할 수 있습니다.

## <a name="how-can-i-troubleshoot-issues-with-my-bot"></a>봇과 관련된 문제는 어떻게 해결할 수 있나요?

1. [Visual Studio Code](debug-bots-locally-vscode.md) 또는 [Visual Studio](https://docs.microsoft.com/en-us/visualstudio/debugger/navigating-through-code-with-the-debugger?view=vs-2017)를 사용하여 봇 소스 코드를 디버그합니다.
1. 클라우드에 배포하기 전에 [에뮬레이터](bot-service-debug-emulator.md)를 사용하여 테스트합니다.
1. Azure와 같은 플랫폼을 호스팅하는 클라우드에 봇을 배포한 다음, <a href="https://dev.botframework.com" target="_blank">Bot Framework Portal</a>의 봇 대시보드에서 기본 제공 웹 챗 제어를 사용하여 봇 연결을 테스트합니다. Azure에 봇을 배포한 후 문제가 발생하면 블로그 문서 [Azure 문제 해결 및 지원 이해](https://azure.microsoft.com/en-us/blog/understanding-azure-troubleshooting-and-support/)를 사용하는 것이 좋을 수 있습니다.
1. [인증][TroubleshootingAuth]은 가능한 문제에서 배제합니다.
1. Skype에서 봇을 테스트합니다. 이렇게 하면 통합형 사용자 환경의 유효성을 검사하는 데 도움이 됩니다.
1. Direct Line 또는 Web Chat 같은 추가 인증 요구 사항이 있는 채널에서 봇을 테스트하는 것이 좋습니다.
1. [봇을 디버그하는 방법](bot-service-debug-bot.md) 및 해당 섹션의 다른 디버깅 문서를 검토하세요.

## <a name="how-can-i-troubleshoot-authentication-issues"></a>인증 문제는 어떻게 해결하나요?

봇을 사용한 인증 문제 해결에 대한 자세한 내용은 Bot Framework 인증 [문제 해결][TroubleshootingAuth]을 참조하세요.

## <a name="im-using-the-bot-framework-sdk-for-net-how-can-i-troubleshoot-issues-with-my-bot"></a>.NET용 Bot Framework SDK를 사용하고 있습니다. 봇과 관련된 문제는 어떻게 해결할 수 있나요?

**예외를 찾습니다.**  
Visual Studio 2017에서 **디버그** > **Windows** > **예외 설정**으로 이동합니다. **예외 설정** 창에서 **공용 언어 런타임 예외** 옆의 **Break When Thrown**을 선택합니다. Throw되었거나 처리되지 않은 예외가 있으면 출력 창에도 진단 출력이 표시될 수 있습니다.

**호출 스택을 확인합니다.**  
Visual Studio에서 [내 코드만](https://msdn.microsoft.com/en-us/library/dn457346.aspx) 디버그할지 여부를 선택할 수 있습니다. 전체 호출 스택을 검사하면 문제에 대한 추가 정보를 제공할 수 있습니다.

**모든 대화 메서드가 다음 메시지 처리를 위한 계획으로 종료되는지 확인합니다.**  
모든 대화 상자 단계는 폭포의 다음 단계로 제공되거나 현재 대화 상자를 종료하고 스택에 팝업 메시지로 표시되어야 합니다. 단계가 올바르게 처리되지 않으면 대화가 예상대로 진행되지 않습니다. 대화 상자에 대한 자세한 내용을 보려면 [대화 상자](v4sdk/bot-builder-concept-dialog.md)에 대한 개념 문서를 살펴보세요.

## <a name="why-doesnt-the-typing-activity-do-anything"></a>입력 작업이 아무 것도 수행하지 않는 이유는 무엇인가요?
일부 채널은 클라이언트에서 일시적 입력 업데이트를 지원하지 않습니다.

## <a name="what-is-the-difference-between-the-connector-library-and-builder-library-in-the-sdk"></a>SDK에서 Connector 라이브러리와 Builder 라이브러리의 차이는 무엇인가요?

Connector 라이브러리는 REST API의 표시입니다. Builder 라이브러리는 프롬프트, 프로그래밍 모델 및 프롬프트, Watefall, 체인 및 안내가 있는 양식 완성 같은 대화형 대화 프로그래밍 모델 및 기타 기능을 추가합니다. Builder 라이브러리는 LUIS 같은 Cognitive Services에 대한 액세스도 제공합니다.

## <a name="what-causes-an-error-with-http-status-code-429-too-many-requests"></a>HTTP 상태 코드 429 "요청이 너무 많음"이 발생하는 이유는 무엇인가요?

HTTP 상태 코드 429가 있는 오류 응답은 일정 시간 동안 실행된 요청이 너무 많음을 의미합니다. 응답 본문에 문제에 대한 설명이 포함되며 요청 사이의 최소 필요 간격도 지정할 수 있습니다. 이 오류의 가능한 원인 중 하나는 [ngrok](https://ngrok.com/)입니다. 무료 요금제를 사용 중이며 ngrok를 한도까지 실행한 경우 해당 웹 사이트의 가격 책정 및 한도 페이지로 이동하여 자세한 [옵션](https://ngrok.com/product#pricing)을 참조하세요. 

## <a name="why-arent-my-bot-messages-getting-received-by-the-user"></a>사용자가 내 봇 메시지를 수신하지 못하는 이유는 무엇인가요?

응답에서 생성된 메시지 활동의 주소는 올바르게 지정되어야 하며, 그렇지 않으면 원하는 대상에 도착하지 않습니다. 대부분의 경우 이를 명시적으로 처리할 필요가 없습니다. SDK는 메시지 활동의 주소 지정 작업을 자동으로 처리합니다. 

활동의 주소를 올바르게 지정하려면 적절한 *대화 참조* 세부 정보와 발신자 및 수신자에 대한 세부 정보를 포함해야 합니다. 대부분의 경우 메시지 활동은 도착한 활동에 대한 응답으로 전송됩니다. 따라서 인바운드 활동에서 주소 지정 정보를 가져올 수 있습니다. 

추적 또는 감사 로그를 살펴보면 메시지의 주소가 올바르게 지정되었는지 확인할 수 있습니다. 올바르게 지정되지 않은 경우 봇에 중단점을 설정하고, 메시지에 ID가 설정되는지 확인하세요.

## <a name="how-can-i-run-background-tasks-in-aspnet"></a>ASP.NET에서 백그라운드 작업은 어떻게 실행하나요? 

일부 경우 몇 초 동안 대기한 다음, 특정 코드를 실행하여 사용자 프로필을 지우거나 대화/대화 상태를 재설정하는 비동기 작업을 시작하고자 할 수 있습니다. 이 작업을 수행하는 자세한 방법은 [ASP.NET에서 백그라운드 작업을 실행하는 방법](https://www.hanselman.com/blog/HowToRunBackgroundTasksInASPNET.aspx)을 참조하세요. 특히 [HostingEnvironment.QueueBackgroundWorkItem](https://msdn.microsoft.com/en-us/library/dn636893(v=vs.110).aspx)을 사용하는 것이 좋습니다. 


## <a name="how-do-user-messages-relate-to-https-method-calls"></a>HTTPS 메서드 호출과 사용자의 메시지는 어떤 관계가 있나요?

사용자가 채널을 통해 메시지를 보내면 Bot Framework 웹 서비스가 봇의 웹 서비스 엔드포인트에 대해 HTTPS POST를 실행합니다. 봇은 이 채널에서 보내는 각각의 메시지에 대해 Bot Framework에 별도의 HTTPS POST를 실행하여 0, 1 또는 여러 메시지를 사용자에게 돌려보낼 수 있습니다.

## <a name="my-bot-is-slow-to-respond-to-the-first-message-it-receives-how-can-i-make-it-faster"></a>내 봇이 처음 받는 메시지에 느리게 응답합니다. 어떻게 더 빠르게 할 수 있나요?

봇은 웹 서비스이며, Azure 등의 일부 호스팅 플랫폼에서는 일정 시간 동안 트래픽이 수신되지 않으면 서비스를 절전 상태로 지정합니다. 봇에 이런 상황이 발생할 경우 다음번에 메시지를 수신했을 때 처음부터 다시 시작해야 하며 이로 인해 이미 실행 중일 때보다 응답이 훨씬 느려집니다.

일부 호스팅 플랫폼을 사용하면 절전 상태가 되지 않게 서비스를 구성할 수 있습니다. Azure에서 이렇게 하려면 [Azure Portal](https://portal.azure.com)에서 봇의 서비스로 이동하고 **애플리케이션 설정**을 선택한 다음,**항상 실행**을 선택합니다. 이 옵션은 대부분의 서비스 계획에서 사용할 수 있으나 전부는 아닙니다.

## <a name="how-can-i-guarantee-message-delivery-order"></a>메시지 배달 순서는 어떻게 보장하나요?

Bot Framework는 가능한 메시지를 순서대로 유지합니다. 예를 들어 메시지 A를 보내고 다른 HTTP 작업이 메시지 B를 보내기 전에 HTTP 작업이 완료될 때까지 대기할 경우, Bot Framework는 자동으로 메시지 A가 메시지 B보다 우선임을 이해합니다. 그러나 일반적으로 채널이 메시지 배달을 최종적으로 담당하고 메시지 순서를 변경할 수 있기 때문에 메시지 배달 순서는 보장되지 않습니다. 메시지가 잘못된 순서로 배달되는 위험을 완화하기 위해 메시지 간의 시간 지연을 구현할 수 있습니다.

## <a name="how-can-i-intercept-all-messages-between-the-user-and-my-bot"></a>사용자와 내 봇 간의 모든 메시지를 어떻게 가로챌 수 있나요?

.NET용 Bot Framework SDK를 사용하여 `IPostToBot` 및 `IBotToUser` 인터페이스의 구현을 `Autofac` 종속성 주입 컨테이너에 제공할 수 있습니다. 임의 언어용 Bot Framework SDK를 사용하여 거의 같은 용도로 미들웨어를 사용할 수 있습니다. [BotBuilder-Azure](https://github.com/Microsoft/BotBuilder-Azure) 리포지토리에는 이 데이터를 Azure 테이블에 기록하는 C# 및 Node.js 라이브러리가 포함되어 있습니다.

## <a name="why-are-parts-of-my-message-text-being-dropped"></a>메시지 텍스트의 일부가 삭제되는 이유는 무엇인가요?

Bot Framework 및 많은 채널은 텍스트가 [Markdown](https://en.wikipedia.org/wiki/Markdown)으로 서식화된 것처럼 해석합니다. 텍스트에 Markdown 구문으로 해석할 수 있는 문자가 포함되었는지 호가인합니다.

## <a name="how-can-i-support-multiple-bots-at-the-same-bot-service-endpoint"></a>여러 봇을 같은 봇 서비스 엔드포인트에서 지원하는 방법은 무엇인가요? 

이 [샘플](https://github.com/Microsoft/BotBuilder/issues/2258#issuecomment-280506334)에서는 올바른 `MicrosoftAppCredentials`를 통해 `Conversation.Container`를 구성하고 간단한 `MultiCredentialProvider`를 사용하여 여러 앱 ID와 암호를 인증하는 방법을 보여 줍니다.

## <a name="identifiers"></a>식별자

## <a name="how-do-identifiers-work-in-the-bot-framework"></a>식별자는 Bot Framework에서 어떻게 작동하나요?

Bot Framework의 식별자에 대한 자세한 내용은 Bot Framework [식별자 가이드][BotFrameworkIDGuide]를 참조하세요.

## <a name="how-can-i-get-access-to-the-user-id"></a>사용자 ID에 대한 액세스는 어떻게 얻나요?

SMS 및 이메일 메시지가 `from.Id` 속성에서 원시 사용자 ID를 제공합니다. Skype는 메시지의 경우 `from.Id` 속성에 사용자의 Skype ID와 다른 사용자 고유 ID가 포함됩니다. 기존 계정에 연결해야 할 경우 로그인 카드를 사용하고 자체 OAuth 흐름을 구현하여 사용자 ID를 자체 서비스의 사용자 ID에 연결할 수 있습니다.

## <a name="why-are-my-facebook-user-names-not-showing-anymore"></a>왜 내 Facebook 사용자 이름이 더 이상 표시되지 않나요?

Facebook 암호를 변경했을 수 있습니다. 이렇게 하면 액세스 토큰이 무효화되며 <a href="https://dev.botframework.com" target="_blank">Bot Framework 포털</a>에서 Facebook Messenger 채널에 대한 봇의 구성 설정을 업데이트해야 합니다.

## <a name="why-is-my-kik-bot-replying-im-sorry-i-cant-talk-right-now"></a>내 Kik 봇이 "I'm sorry, I can't talk right now"(미안하지만 지금은 대화할 수 없다)라고 답하는 이유는 무엇인가요?

Kik 개발에서는 봇에 50명의 구독자가 허용됩니다. 고유 사용자 50명이 봇과 상호 작용한 후에는 해당 봇과의 채팅을 시도하는 모든 새 사용자가 "I'm sorry, I can't talk right now"(미안하지만 지금은 대화할 수 없다)라는 메시지를 받게 됩니다. 자세한 내용은 [Kik 설명서](https://botsupport.kik.com/hc/en-us/articles/225764648-How-can-I-share-my-bot-with-Kik-users-while-in-development-)를 참조하세요.

## <a name="how-can-i-use-authenticated-services-from-my-bot"></a>봇에서 인증된 서비스는 어떻게 사용하나요?

Azure Active Directory 인증은 [V3](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-authentication?view=azure-bot-service-3.0&tabs=csharp) | [V4](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-authentication?view=azure-bot-service-4.0&tabs=csharp) 인증 추가를 참조하세요. 

> [!NOTE] 
> 봇에 인증 및 보안 기능을 추가할 경우 코드에서 구현한 패턴이 애플리케이션에 적합한 보안 표준을 준수해야 합니다.

## <a name="how-can-i-limit-access-to-my-bot-to-a-pre-determined-list-of-users"></a>미리 결정된 사용자 목록에 대한 봇의 액세스는 어떻게 제한하나요?

SMS 및 이메일 같은 일부 채널은 범위가 지정되지 않는 주소를 제공합니다. 이러한 경우 사용자의 메시지는 `from.Id` 속성에 원시 사용자 ID를 포함하게 됩니다.

Skype, Facebook, Slack 같은 다른 채널은 봇이 미리 사용자 ID를 예측하지 않도록 범위 또는 테넌트가 지정된 주소를 제공합니다. 이러한 경우에는 사용자가 봇을 사용할 권한이 있는지 판단하기 위해 로그인 링크나 공유 비밀을 통해 사용자를 인증해야 합니다.

## <a name="why-does-my-direct-line-11-conversation-start-over-after-every-message"></a>모든 메시지 이후 Direct Line 1.1 대화가 시작되는 이유는 무엇인가요?

모든 메시지 이후 Direct Line 대화가 시작되는 것처럼 보인다면 `from` 속성이 누락되었거나 Direct Line 클라이언트가 봇에 보낸 메시지가 `null`일 수 있습니다. Direct Line 클라이언트가 누락되었거나 `null`인 `from` 속성의 메시지를 보낼 경우, Direct Line 서비스는 자동으로 ID를 할당하므로 클라이언트가 보내는 모든 메시지가 다른 새 사용자에게서 온 것처럼 보입니다.

이 문제를 해결하려면 Direct Line 클라이언트가 보내는 각 메시지의 `from` 속성을, 메시지를 보내는 사용자를 고유하게 나타내는 안정적인 값으로 설정합니다. 예를 들어 사용자가 이미 웹 페이지나 앱에 로그인한 경우 사용자가 보내는 메시지에서 기존 사용자 ID를 `from` 속성 값으로 사용할 수 있습니다. 또는 페이지 로드 또는 애플리케이션 로드 시 임의의 사용자 ID를 생성하여 쿠키나 장치 상태에 저장하고, 이 ID를 사용자가 보내는 메시지에서 `from` 속성 값으로 사용할 수 있습니다.

## <a name="what-causes-the-direct-line-30-service-to-respond-with-http-status-code-502-bad-gateway"></a>Direct Line 3.0 서비스가 HTTP 상태 코드 502 "잘못된 게이트웨이"라 답하는 이유는 무엇인가요?
Direct Line 3.0은 봇 연결을 시도했으나 요청이 성공적으로 완료되지 않으면 HTTP 상태 코드 502를 반환합니다. 이 오류는 봇이 오류를 반환했거나 요청된 시간이 초과되었음을 표시합니다. 봇이 생성하는 오류에 대한 자세한 내용은 <a href="https://dev.botframework.com" target="_blank">Bot Framework 포털</a> 안에서 봇의 대시보드로 이동하고 영향을 받는 채널에 대한 "문제" 링크를 클릭합니다. 봇에 대해 Application Insights를 구성한 경우 거기서도 상세 오류 정보를 제공합니다. 

::: moniker range="azure-bot-service-3.0"

## <a name="what-is-the-idialogstackforward-method-in-the-bot-framework-sdk-for-net"></a>.NET용 Bot Framework SDK의 IDialogStack.Forward 메서드는 무엇인가요?

`IDialogStack.Forward`의 주 목적은 자식 대화(`IDialog.StartAsync`)가 일부 `ResumeAfter` 처리기가 있는 `T` 개체를 대기하는 "사후"인 경우가 많은 기존 자식 대화를 재사용하는 것입니다. 특히 `IMessageActivity` `T`를 대기하는 자식 대화가 있는 경우 `IDialogStack.Forward` 메서드를 사용하여 들어오는 `IMessageActivity`(이미 일부 부모 대화에서 받음)를 전달할 수 있습니다. 예를 들어 들어오는 `IMessageActivity`를 `LuisDialog`에 전달하려면 `IDialogStack.Forward`를 호출하여 대화 스택에 `LuisDialog`를 푸시하고 다음 메시지 대기를 예약할 때까지 `LuisDialog.StartAsync`에서 코드를 실행한 다음, 즉시 전달된 `IMessageActivity`를 통해 해당 대기를 만족합니다.

`IDialog.StartAsync`이 보통 이러한 작업 유형을 대기하도록 생성되기 때문에 `T`는 일반적으로 `IMessageActivity`를 사용합니다. `IDialogStack.Forward`를 `LuisDialog`에 메커니즘으로 사용하여 메시지를 기존 `LuisDialog`에 전달하기 전에 일부 처리를 위해 사용자로부터 메시지를 가로챌 수 있습니다.  이 목적으로 `DispatchDialog`와 `ContinueToNextGroup`을 사용할 수도 있습니다.

`StartAsync`에서 예약한 첫 번째 `ResumeAfter` 처리기(예: `LuisDialog.MessageReceived`)에서 전달된 항목을 찾게 됩니다.

::: moniker-end

## <a name="what-is-the-difference-between-proactive-and-reactive"></a>"사전"과 "사후"는 어떻게 다른가요?

봇의 관점에서 "사후"는 사용자가 봇에게 메시지를 보내 대화를 시작하고 봇이 이 메시지에 답해 반응한다는 뜻입니다. 반대로 "사전"은 봇이 먼저 사용자에게 메시지를 보내 대화를 시작하는 것입니다. 예를 들어, 봇이 타이머 만료나 이벤트 발생을 사용자에게 알리기 위해 사전 메시지를 보낼 수 있습니다.

## <a name="how-can-i-send-proactive-messages-to-the-user"></a>사용자에게 사전 메시지를 보내려면 어떻게 해야 하나요?

사전 메시지를 보내는 방법을 보여 주는 예제는 GitHub의 BotBuilder-Samples 리포지토리 내의 [C# V4 샘플](https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/16.proactive-messages) 및 [Node.js V4 샘플](https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/16.proactive-messages)을 참조하세요.

## <a name="how-can-i-reference-non-serializable-services-from-my-c-dialogs"></a>C# 대화에서 직렬화할 수 없는 서비스는 어떻게 참조하나요?

다음과 같은 몇 가지 옵션이 있습니다.

* `Autofac` 및 `FiberModule.Key_DoNotSerialize`를 통해 종속성을 해결합니다. 이것이 가장 깔끔한 솔루션입니다.
* [NonSerialized](https://msdn.microsoft.com/en-us/library/system.nonserializedattribute(v=vs.110).aspx) 및 [OnDeserialized](https://msdn.microsoft.com/en-us/library/system.runtime.serialization.ondeserializedattribute(v=vs.110).aspx) 특성을 사용하여 역직렬화의 종속성을 복원합니다. 이것이 가장 간단한 솔루션입니다.
* 종속성을 저장하지 않으므로 직렬화되지 않습니다. 이 솔루션은 기술적으로 가능하지만 권장되지는 않습니다.
* 리플렉션 직렬화 서로게이트를 사용합니다. 이 솔루션은 일부 경우 불가능하며 직렬화가 너무 많이 발생할 위험이 있습니다.

::: moniker range="azure-bot-service-3.0"

## <a name="where-is-conversation-state-stored"></a>대화 상태는 어디에 저장되나요?

사용자, 대화 및 개인 대화 속성 모음의 데이터는 Connector의 `IBotState` 인터페이스를 통해 저장됩니다. 각 속성 모음은 봇의 ID로 범위가 지정됩니다. 사용자 속성 모음은 사용자 ID로, 대화 속성 모음은 대화 ID로, 개인 대화 속성 모음은 사용자 ID 및 대화 ID로 키가 지정됩니다. 

.NET용 Bot Framework SDK 또는 Node.js용 Bot Framework SDK로 봇을 빌드하는 경우 대화 스택 및 대화 데이터는 모두 개인 대화 속성 모음의 항목으로 자동 저장됩니다. C# 구현에서는 이진 직렬화를, Node.js 구현에서는 JSON 직렬화를 사용합니다.

`IBotState` REST 인터페이스는 두 서비스로 구현됩니다.

* Bot Framework Connector는 이 인터페이스를 구현하고 Azure에 데이터를 저장하는 클라우드 서비스를 제공합니다.  이 데이터는 사용하지 않게 암호화되며 의도적으로 만료되지 않습니다.
* Bot Framework Emulator는 봇 디버그를 위해 이 인터페이스의 메모리 내 구현을 제공합니다. 이 데이터에는 에뮬레이터 프로세스가 종료되면 만료됩니다.

데이터 센터 내에 이 데이터를 저장할 경우 상태 서비스의 사용자 지정 구현을 제공할 수 있습니다. 이 작업은 최소한 두 가지 방법으로 수행할 수 있습니다.

* REST 계층을 사용하여 사용자 지정 `IBotState` 서비스를 제공합니다.
* 언어(Node.js 또는 C#) 계층에서 Builder 인터페이스를 사용합니다.

> [!IMPORTANT]
> Bot Framework 상태 서비스 API는 프로덕션 환경에는 권장되지 않으며 이후 릴리스에서 사용되지 않을 수 있습니다. 테스트 용도로 메모리 내 저장소를 사용하거나 프로덕션 봇용으로 **Azure 확장**을 사용하도록 봇 코드를 업데이트하는 것이 좋습니다. 자세한 내용은 [.NET](~/dotnet/bot-builder-dotnet-state.md) 또는 [Node](~/nodejs/bot-builder-nodejs-state.md) 구현에 대한 **상태 데이터 관리** 항목을 참조하세요.

::: moniker-end

## <a name="what-is-an-etag--how-does-it-relate-to-bot-data-bag-storage"></a>ETag란?  봇 데이터 저장소와는 어떤 관계가 있나요?

[ETag](https://en.wikipedia.org/wiki/HTTP_ETag)는 [낙관적 동시성 제어](https://en.wikipedia.org/wiki/Optimistic_concurrency_control)를 위한 메커니즘입니다. 이 봇 데이터 모음은 데이터 업데이트 충돌을 방지하기 위해 ETag를 사용합니다. HTTP 상태 코드 412 "전제 조건 실패"가 있는 ETag 오류는 해당 봇 데이터 모음에 대해 여러 "읽기-수정-쓰기" 시퀀스가 동시에 실행되고 있음을 의미합니다.

대화 스택 및 상태는 봇 데이터 모음에 저장됩니다. 예를 들어, 봇이 해당 대화에 대해 새 메시지를 수신했을 때 여전히 이전 메시지를 처리 중이라면 "전제 조건 실패" ETag 오류가 표시될 수 있습니다.

## <a name="what-causes-an-error-with-http-status-code-412-precondition-failed-or-http-status-code-409-conflict"></a>HTTP 상태 코드 412 "전제 조건 실패" 또는 HTTP 상태 코드 409 "충돌"이 발생하는 이유는 무엇인가요?

커넥터의 `IBotState` 서비스는 봇 데이터 모음(즉, 개인 봇 데이터 모음이 대화 스택 "제어 흐름" 상태를 포함하는 경우 사용자, 대화, 개인 봇 데이터 모음)을 저장하는 데 사용됩니다. `IBotState` 서비스의 동시성 제어는 ETag를 통해 낙관적 동시성으로 관리됩니다. "읽기-수정-쓰기" 시퀀스 중에 (단일 봇 데이터 백에 대한 동시 업데이트로 인해) 업데이트 충돌이 발생할 경우

* Etag가 유지될 경우 `IBotState` 서비스에서 HTTP 상태 코드 412 "전제 조건 실패" 오류가 Throw됩니다. 이것이 .NET용 Bot Framework SDK의 기본 동작입니다.
* Etag가 유지되지 않는 경우(즉, ETag가 `\*`로 설정됨) "전제 조건 실패" 오류는 방지하지만 데이터 손실의 위험이 있는 "마지막 쓰기 우선" 정책이 적용됩니다. 이것이 Node.js용 Bot Framework SDK의 기본 동작입니다.

## <a name="how-can-i-fix-precondition-failed-412-or-conflict-409-errors"></a>"전제 조건 실패"(412) 또는 "충돌" (409) 오류는 어떻게 해결하나요?

이러한 오류는 봇이 한 번에 동일한 대화에 대해 여러 메시지를 처리했음을 의미합니다. 봇이 정확하게 정렬된 메시지를 요구하는 서비스에 연결된 경우 메시지가 병렬로 처리되지 않도록 대화 상태를 잠그는 것이 좋습니다. 

::: moniker range="azure-bot-service-3.0"

.NET용 Bot Framework SDK는 메모리 내 세마포를 통해 단일 대화의 처리를 비관적으로 직렬화하는 메커니즘(`IScope`를 구현하는 `LocalMutualExclusion` 클래스)을 제공합니다. 이 구현을 대화 주소를 통해 범위가 지정된 Redis 임대를 사용하도록 확장할 수 있습니다.

봇이 외부 서비스에 연결되어 있지 않거나 동일한 대화로부터의 병렬 처리가 용인될 수 있는 경우, 이 코드를 추가하여 Bot State API에서 발생하는 충돌을 무시할 수 있습니다. 이렇게 하면 마지막 회신이 대화 상태를 설정할 수 있습니다.

```cs
var builder = new ContainerBuilder();
builder
    .Register(c => new CachingBotDataStore(c.Resolve<ConnectorStore>(), CachingBotDataStoreConsistencyPolicy.LastWriteWins))
    .As<IBotDataStore<BotData>>()
    .AsSelf()
    .InstancePerLifetimeScope();
builder.Update(Conversation.Container);
```
::: moniker-end

## <a name="is-there-a-limit-on-the-amount-of-data-i-can-store-using-the-state-api"></a>State API를 사용하여 저장할 수 있는 데이터의 크기에 제한이 있나요?

예, 각 상태 저장소(즉, 사용자, 대화 및 개인 봇 데이터 모음)는 최대 64KB의 데이터를 포함할 수 있습니다. 자세한 내용은 [상태 데이터 관리][StateAPI]를 참조하세요.

::: moniker range="azure-bot-service-3.0"

## <a name="how-do-i-version-the-bot-data-stored-through-the-state-api"></a>State API를 통해 저장된 봇 데이터 버전은 어떻게 지정하나요?

> [!IMPORTANT]
> Bot Framework 상태 서비스 API는 프로덕션 환경 또는 v4 봇에는 권장되지 않으며 이후 릴리스에서 전혀 사용되지 않을 수 있습니다. 테스트 용도로 메모리 내 저장소를 사용하거나 프로덕션 봇용으로 **Azure 확장**을 사용하도록 봇 코드를 업데이트하는 것이 좋습니다. 자세한 내용은 [상태 데이터 관리](v4sdk/bot-builder-howto-v4-state.md) 항목을 참조하세요.

상태 서비스를 사용하면 사용자가 나중에 위치를 잃지 않고 대화로 돌아올 수 있게 대화의 대화 상자에서 진행 상황을 유지할 수 있습니다. 이렇게 유지하기 위해, 봇의 코드를 수정할 때 State API를 통해 저장된 봇 데이터 속성 모음이 자동으로 지워지지 않습니다.  수정된 코드가 이전 데이터 버전과 호환 가능한지 여부에 따라 봇 데이터를 지울지 여부를 결정해야 합니다. 

* 봇 개발 중에 수동으로 대화의 대화 스택과 상태를 재설정하려면 ` /deleteprofile` 명령을 사용하여 상태 데이터를 삭제할 수 있습니다. 채널이 해석하지 않도록 명령에 선행 공백을 포함해야 합니다.
* 봇을 프로덕션에 배포한 후에는 버전을 범프하면 연결된 상태 데이터가 지워지도록 봇 데이터 버전을 지정할 수 있습니다. Node.js용 Bot Framework SDK에서는 미들웨어를 통해, .NET용 Bot Framework SDK에서는 `IPostToBot` 구현을 통해 이를 수행할 수 있습니다.

> [!NOTE]
> 코드가 너무 많이 변경되었거나 직렬화 형식이 변경되어 대화 스택이 올바르게 역직렬화될 수 없으면 대화 상태가 재설정됩니다.

::: moniker-end

## <a name="what-are-the-possible-machine-readable-resolutions-of-the-luis-built-in-date-time-duration-and-set-entities"></a>LUIS 기본 제공 날짜, 시간, 기간, 설정 엔터티의 머신이 읽을 수 있는 가능한 해결책은 무엇인가요?

예제 목록은 LUIS 문서의 [미리 작성된 엔터티 섹션][LUISPreBuiltEntities]을 참조하세요.

## <a name="how-can-i-use-more-than-the-maximum-number-of-luis-intents"></a>최대 LUIS 의도보다 더 사용하려면 어떻게 해야 하나요?

모델을 분할하거나, 일련 또는 병렬로 LUIS 서비스를 호출할 수 있습니다.

## <a name="how-can-i-use-more-than-one-luis-model"></a>여러 LUIS 모델은 어떻게 사용하나요?

Node.js용 Bot Framework SDK와 .NET용 Bot Framework SDK 모두 단일 LUIS 의도 대화에서 여러 LUIS 모델 호출을 지원합니다. 다음 사항을 주의하세요.

* 여러 LUIS 모델을 사용할 때는 LUIS 모델에 겹치지 않는 의도 집합이 있다고 가정합니다.
* 여러 LUIS 모델을 사용할 때는 서로 다른 모델의 점수를 비교하여 여러 모델 중에서 "가장 일치하는 의도"를 선택할 수 있다고 가정합니다.
* 여러 LUIS 모델을 사용하는 것은 의도가 한 모델과 일치하면 다른 모델의 "없음" 의도와도 강력하게 일치한다는 뜻입니다. 이런 상황에서 "없음" 의도 선택을 방지할 수 있습니다. Node.js용 Bot Framework SDK는 이런 문제를 방지하기 위해 "없음" 의도에 대한 점수를 자동으로 줄입니다.

## <a name="where-can-i-get-more-help-on-luis"></a>LUIS에 대한 자세한 도움말은 어디서 얻을 수 있나요?

* [LUIS(Language Understanding) 소개 - Microsoft Cognitive Services](https://www.youtube.com/watch?v=jWeLajon9M8) (동영상)
* [LUIS(Language Understanding)에 대한 고급 교육 세션](https://www.youtube.com/watch?v=39L0Gv2EcSk)(비디오)
* [LUIS 설명서](/azure/cognitive-services/LUIS/Home)
* [Language Understanding 포럼](https://social.msdn.microsoft.com/forums/azure/en-US/home?forum=LUIS) 


## <a name="what-are-some-community-authored-dialogs"></a>커뮤니티에서 작성된 대화에는 어떤 것이 있나요?

* [BotAuth](https://www.nuget.org/packages/BotAuth) - Azure Active Directory 인증
* [BestMatchDialog](http://www.garypretty.co.uk/2016/08/01/bestmatchdialog-for-microsoft-bot-framework-now-available-via-nuget/) - 대화 메서드에 대한 사용자 텍스트의 정규식 기반 디스패치

## <a name="what-are-some-community-authored-templates"></a>커뮤니티에서 작성된 템플릿에는 어떤 것이 있나요?

* [ES6 BotBuilder](https://github.com/brene/botbuilder-es6-template) - ES6 Bot Builder 템플릿

## <a name="why-do-i-get-an-authorizationrequestdenied-exception-when-creating-a-bot"></a>봇을 만들 때 Authorization_RequestDenied 예외가 발생하는 이유는 무엇인가요?

Azure Bot Service 봇을 만들 수 있는 권한은 AAD(Azure Active Directory)를 통해 관리됩니다. [AAD 포털](http://aad.portal.azure.com)에서 권한이 올바르게 구성되지 않은 경우 봇 서비스를 만들 때 **Authorization_RequestDenied** 예외가 발생합니다.

먼저 디렉터리의 "게스트"인지 확인합니다.

1. [Azure Portal](http://portal.azure.com)에 로그인합니다.
2. **모든 서비스**를 선택하고 *active*를 검색합니다.
3. **Azure Active Directory**를 선택합니다.
4. **사용자**를 클릭합니다.
5. 목록에서 사용자를 찾고 **사용자 유형**이 **게스트**가 아닌지 확인합니다.

![Azure Active Directory 사용자 유형](~/media/azure-active-directory/user_type.png)

**게스트**가 아닌지 확인한 다음, Active Directory 내 사용자가 봇 서비스를 만들 수 있는지 확인하고 디렉터리 관리자가 다음 설정을 구성해야 합니다.

1. [AAD 포털](http://aad.portal.azure.com)에 로그인합니다. **사용자 및 그룹**으로 이동하고 **사용자 설정**을 선택합니다.
2. **앱 등록** 섹션에서 **사용자가 애플리케이션을 등록할 수 있음**을 **예**로 설정합니다. 이렇게 하면 디렉터리의 사용자가 봇 서비스를 만들 수 있습니다.
3. **외부 사용자** 섹션에서 **게스트 사용자 권한이 제한됨**을 **아니요**로 설정합니다. 이렇게 하면 디렉터리의 게스트 사용자가 봇 서비스를 만들 수 있습니다.

![Azure Active Directory 관리 센터](~/media/azure-active-directory/admin_center.png)

## <a name="why-cant-i-migrate-my-bot"></a>내 봇을 마이그레이션할 수 없는 이유는 무엇인가요?

봇을 마이그레이션 하는 데 문제가 있는 경우 봇이 기본 디렉토리가 아닌 디렉터리에 속하기 때문일 수 있습니다. 다음 단계를 시도합니다.

1. 대상 디렉터리에서 기본 디렉터리의 멤버가 아닌 새 사용자(이메일 주소를 통해)를 추가하고, 마이그레이션 대상인 구독에서 사용자 기여자 역할을 부여합니다.

2. [Dev Portal](https://dev.botframework.com)에서 사용자의 이메일 주소를 마이그레이션할 봇의 공동 소유자로 추가합니다. 그런 다음, 로그아웃합니다.

3. 새 사용자로 [Dev Portal](https://dev.botframework.com)에 로그인하고 봇 마이그레이션을 진행합니다.

## <a name="where-can-i-get-more-help"></a>자세한 도움말은 어디서 얻을 수 있나요?

* 이전에 [스택 오버플로](https://stackoverflow.com/questions/tagged/botframework)에 관해 답변한 질문에서 정보를 활용하거나 `botframework` 태그를 사용하여 질문을 게시합니다. 스택 오버플로에는 설명을 포함하는 제목, 완전하고 간결한 문제 설명 및 문제를 재현하기 위한 충분한 세부 정보 등의 지침이 포함되어 있습니다. 기능 요청 또는 너무 광범위한 질문은 주제를 벗어납니다. 자세한 내용을 원하는 새 사용자는 [스택 오버플로 도움말 센터](https://stackoverflow.com/help/how-to-ask)를 방문해야 합니다.
* GitHub의 [BotBuilder 문제](https://github.com/Microsoft/BotBuilder/issues)에서 Bot Framework SDK의 알려진 문제를 참조하거나 새 문제를 보고합니다.
* [Gitter](https://gitter.im/Microsoft/BotBuilder)의 BotBuilder 커뮤니티 토론에 있는 정보를 활용합니다.




[LUISPreBuiltEntities]: /azure/cognitive-services/luis/pre-builtentities
[BotFrameworkIDGuide]: bot-service-resources-identifiers-guide.md
[StateAPI]: ~/rest-api/bot-framework-rest-state.md
[TroubleshootingAuth]: bot-service-troubleshoot-authentication-problems.md

