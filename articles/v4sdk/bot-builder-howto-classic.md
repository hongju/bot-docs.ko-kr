---
title: SDK V4에서 .NET SDK V3 봇을 실행하는 방법 | Microsoft Docs
description: 클래식 NuGet 패키지를 사용하여 봇을 3.x에서 4.0으로 변환하는 방법을 알아봅니다.
keywords: 마이그레이션, 클래식 봇, v3 변환, v3에서 v4로
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/25/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a808076e4865a181802b85cfc24ce342dbf23cba
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301492"
---
# <a name="how-to-run-net-sdk-v3-bots-in-sdk-40"></a>SDK 4.0에서 .NET SDK V3 봇을 실행하는 방법

**Microsoft.Bot.Builder.Classic** NuGet 패키지는 Microsoft Bot Framework 버전 3.x에서 4.0으로 봇을 간편하게 마이그레이션할 수 있도록 합니다.

**참고:** 이 프로세스는 **ASP.NET 웹 응용 프로그램(.NET Framework)** 봇에서만 작동합니다. **ASP.NET Core 웹 응용 프로그램** 봇에서는 작동하지 않습니다.

## <a name="the-process"></a>프로세스

프로세스는 비교적 간단합니다.

- **Microsoft.Bot.Builder.Classic** NuGet 패키지를 프로젝트에 추가합니다.
    - **Autofac** NuGet 패키지를 업데이트해야 할 수도 있습니다.
- **Microsoft.Bot.Builder.Classic** 네임스페이스를 업데이트합니다.
- 4.0 **ITurnContext** 기반 **Conversation.SendAsync()** 를 사용하여 다이얼로그를 호출합니다.

### <a name="add-the-microsoftbotbuilderclassic-nuget-package"></a>Microsoft.Bot.Builder.Classic NuGet 패키지 추가

**Microsoft.Bot.Builder.Classic** NuGet 패키지를 추가하려면 **NuGet 패키지 관리**를 사용하여 패키지를 추가합니다.

### <a name="update-the-namespaces"></a>네임스페이스 업데이트

네임스페이스를 업데이트하려면 찾은 다음 `using` 문 중 하나를 삭제합니다.

```csharp
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;
using Microsoft.Bot.Builder.FormFlow;
using Microsoft.Bot.Builder.Scorables;
```

그런 후 다음 `using` 문을 해당 위치에 추가합니다.

```csharp
using Microsoft.Bot.Builder.Adapters;
using Microsoft.Bot.Builder.Classic.Dialogs;
using Microsoft.Bot.Builder.Classic.Dialogs.Internals;
using Microsoft.Bot.Builder.Classic.FormFlow;
using Microsoft.Bot.Builder.Classic.Scorables;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Schema;
```

`[BotAuthentication]` 줄이 있는 경우 삭제하거나 주석으로 처리합니다.

### <a name="invoke-your-3x-dialog"></a>3.x 다이얼로그 호출

3.x 다이얼로그를 호출하려면 여전히 `Conversation.SendAsync`를 사용합니다. 현재는 **Activity** 대신 4.0 **ITurnContext**만 사용됩니다.

```csharp
// invoke a Classic V3 IDialog 
await Conversation.SendAsync(turnContext, () => new EchoDialog());
```

**ITurnContext** 개체가 없지만 **Activity** 개체가 있는 경우 다음과 같은 방식으로 **ITurnContext** 개체를 가져올 수 있습니다.

```csharp
BotFrameworkAdapter adapter = new BotFrameworkAdapter("", "");

await adapter.ProcessActivity(this.Request.Headers.Authorization?.Parameter,
        activity,
        async (context) =>
        {
            // Do something with context here. For example, the body of your Post() method may go here.
        });
```

## <a name="fix-assembly-conflicts"></a>어셈블리 충돌 해결

클래식 NuGet 패키지를 사용하여 만든 봇은 해당 어셈블리와 충돌합니다. 이 내용은 빌드 후에에 Visual Studio의 오류 목록 창에 표시됩니다.

### <a name="if-you-see-warning-found-conflicts-between-different-versions-of-the-same-dependent-assembly"></a>“경고: 동일한 종속 어셈블리의 서로 다른 버전 사이에 충돌이 발생했습니다.”가 표시되는 경우

**동일한 종속 어셈블리의 서로 다른 버전 사이에 충돌이 발생했습니다.** 텍스트로 시작되는 경고가 표시되는 경우 다음을 수행합니다.

- 경고 메시지를 두 번 클릭합니다. “응용 프로그램 구성 파일에 바인딩 리디렉션 레코드를 추가하여 충돌을 해결하시겠습니까?”라는 대화 상자가 표시됩니다.
- "예"를 클릭합니다.
- 프로젝트를 다시 빌드합니다.

### <a name="if-you-see-error-missing-method-exception-on-startup"></a>"오류: 시작 시 메서드 누락 예외 발생"이 표시되는 경우

이전 .NET 4.6 프로젝트를 가져와 4.6.1로 업그레이드한 다음, .NET Standard 라이브러리를 사용할 경우 발생할 수 있는 .NET Standard 버그가 있습니다. 기본적으로 동적 교체를 시도하는 2개의 다른 System.Net.Http 어셈블리가 있습니다. 해결 방법은 System.Net.Http용 Web.config에 바인딩 리디렉션을 추가하는 것입니다. 

이 오류가 발생하는 경우 Web.config 파일에 다음을 추가합니다.

```xml
<dependentAssembly>
    <assemblyIdentity name="System.Net.Http" publicKeyToken="B03F5F7F11D50A3A" culture="neutral" />
    <bindingRedirect oldVersion="0.0.0.0-4.2.0.0" newVersion="4.2.0.0" />
</dependentAssembly>
```

이 문제에 대한 자세한 내용은 [MSBuild 도구 #25773에서 복사/로드되는 System.Net.Http v4.2.0.0](https://github.com/dotnet/corefx/issues/25773)를 참조하세요.

## <a name="sample-of-a-converted-bot"></a>변환된 봇 샘플

이미 변환된 봇의 경우 4.0에서 작동하도록 변환된 3.x 봇을 보여 주는 [EchoBot-Classic](https://github.com/Microsoft/botbuilder-dotnet/tree/master/samples/Microsoft.Bot.Samples.EchoBot-Classic)을 확인할 수 있습니다.

## <a name="limitations"></a>제한 사항
Microsoft.Bot.Builder.Classic은 .NET 4.61 라이브러리 전용이며 .NET Core에서는 작동 하지 않습니다.
