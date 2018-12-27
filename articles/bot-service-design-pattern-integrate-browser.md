---
title: 웹 브라우저와 봇 통합 | Microsoft Docs
description: 봇에서 웹 브라우저로 원활하게 전환하고 다시 돌아오도록 디자인하는 방법을 알아봅니다.
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: e3833595003edda46a6ffd1d508237262aad94e1
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999230"
---
# <a name="integrate-your-bot-with-a-web-browser"></a>웹 브라우저와 봇 통합

일부 시나리오에서는 요구 사항을 이행하는 봇 이상을 요구합니다. 봇은 웹 브라우저로 사용자를 보내 작업을 완료한 다음, 작업이 완료된 후에 사용자와의 대화를 다시 시작해야 할 수 있습니다. 

## <a name="authentication-and-authorization"></a>인증 및 권한 부여
봇에 Office 365에서 사용자의 일정을 읽거나 해당 사용자 대신 약속을 하는 기능이 필요한 경우 사용자는 먼저 Microsoft Azure Active Directory에서 인증을 받고 봇에게 사용자 일정 데이터에 액세스할 수 있는 권한을 부여합니다. 봇은 사용자를 웹 브라우저로 리디렉션하여 인증 및 권한 부여 작업을 완료한 후, 사용자와의 대화를 다시 시작합니다. 

## <a name="security-and-compliance"></a>보안 및 규정 준수
보안 및 규정 준수 요구 사항은 종종 봇이 사용자와 교환할 수 있는 정보 유형을 제한합니다. 경우에 따라 사용자가 현재 대화 밖에서 데이터를 송/수신해야 할 수도 있습니다. 예를 들어, 사용자가 타사 지급업체를 사용하여 결제를 진행하려는 경우 대화 컨텍스트 내에서 신용 카드 번호를 지정하면 안 됩니다. 대신, 봇은 사용자를 웹 브라우저로 리디렉션하여 결제 프로세스를 완료한 후, 사용자와의 대화를 다시 시작합니다.

이 문서에서는 봇에서 웹 브라우저로 전환했다가 다시 돌아오는 작업을 편리하게 진행하는 프로세스를 알아봅니다. 

> [!NOTE]
> 채팅에서 웹 브라우저로 전환했다가 다시 돌아올 경우 애플리케이션 간을 전환하는 과정에서 사용자를 혼동하기 쉬우므로 이상적인 방식이 아닙니다. 더 나은 환경을 제공하기 위해 많은 채널은 봇이 애플리케이션을 제공하기 위해 사용할 수 있는 기본 제공 HTML 창을 제공합니다(이 경우가 아니면 애플리케이션이 웹 브라우저에 제공됨). 이 기술을 사용하면 대화를 벗어나지 않으면서 외부 리소스에 액세스할 수 있습니다. 이 방법은 포함된 웹 보기 내에서 OAuth를 사용하여 권한 부여 흐름을 관리하는 모바일 애플리케이션과 개념적으로 유사합니다.

## <a name="bot-to-web-browser-and-back-again"></a>봇에서 웹 브라우저로 전환 및 다시 돌아오기

이 다이어그램에서는 봇과 웹 브라우저 간의 통합에 대한 간단한 흐름을 보여 줍니다. 

![봇-웹 상호 작용](~/media/bot-service-design-pattern-integrate-browser/bot-to-web1.png)

흐름의 각 단계를 고려합니다.

1. <a id="generate-hyperlink"></a>봇이 사용자를 웹 사이트로 리디렉션하는 하이퍼링크를 생성하고 표시합니다. 
   하이퍼링크는 일반적으로 채널의 대화 ID, 채널 ID 및 사용자 ID와 같은 현재 대화의 컨텍스트에 대한 정보를 지정하는 쿼리 문자열 매개 변수를 통해 대상 URL에 데이터를 포함합니다. 

2. 사용자는 하이퍼링크를 클릭하면 웹 브라우저 내의 대상 URL로 리디렉션됩니다. 

3. 봇은 웹 사이트 흐름이 완료되었음을 나타내는 웹 사이트의 통신을 대기하는 상태가 됩니다.  
   > [!TIP]
   > 사용자가 웹 사이트 흐름을 절대 완료하지 않을 경우 봇이 영구적으로 ‘대기’ 상태를 유지하는 일이 없도록 이 흐름을 디자인합니다. 즉, 사용자가 웹 브라우저를 중단했다가 봇과의 통신을 다시 시작하면 봇은 해당 입력을 [무시](~/bot-service-design-navigation.md#the-mysterious-bot)하지 않고 승인합니다.

4. 사용자는 웹 브라우저를 통해 필요한 작업을 완료합니다. 
   이것은 일반적인 시나리오에 필요한 OAuth 흐름 또는 이벤트 시퀀스일 수 있습니다. 

5. <a id="generate-magic-number"></a>사용자가 웹 사이트 흐름을 완료하면 웹 사이트는 '[매직 넘버](#verify-identity)'를 생성하고 사용자에게 해당 값을 복사한 후 봇과의 대화에 붙여넣도록 지시합니다. 

6. <a id="signal-to-bot"></a>웹 사이트는 사용자가 웹 사이트 흐름을 완료했다는 사실을을 [봇에게 신호로 알립니다](#website-signal-to-bot). 
   봇에 '매직 넘버'를 전달하고 기타 관련 데이터를 제공합니다.
   예를 들어, OAuth 흐름의 경우 웹 사이트는 봇에 액세스 토큰을 제공해야야 합니다.

7. 사용자는 봇으로 돌아가 채팅에 '매직 넘버'를 붙여넣습니다. 
   봇은 사용자가 제공하는 '매직 넘버'가 예상되는 값과 일치하는지 확인하고, 현재 사용자가 이전에 웹 사이트 흐름을 시작하기 위해 하이퍼링크를 클릭한 사용자와 같은지 검토합니다. 

### <a id="verify-identity"></a> '매직 넘버'를 사용하여 사용자 ID 확인

봇-웹 사이트 흐름 동안 진행되는 '매직 넘버' 생성(위의 [5단계](#generate-magic-number))을 통해 봇은 웹 사이트 흐름을 시작한 사람이 의도한 사람인지를 확인할 수 있습니다. 예를 들어, 봇이 여러 사용자와 그룹 채팅을 수행하는 경우 이러한 사용자 중 한 명이 하이퍼링크를 클릭하여 웹 사이트 흐름을 시작했을 수 있습니다. '매직 넘버' 유효성 검사 프로세스가 없으면 봇은 사용자가 흐름을 완료했는지 알 수 없습니다. 한 명의 사용자가 인증을 받은 후 다른 사용자의 세션에서 액세스 토큰을 삽입할 수도 있는 것입니다. 

> [!WARNING] 
> 이러한 행동은 그룹 채팅 내에서 단지 위험한 수준이 아닙니다. '매직 넘버' 유효성 검사 프로세스가 없으면 웹 사이트 흐름을 시작하기 위한 하이퍼링크만 있으면 누구든지 사용자의 ID를 스푸핑할 수 있습니다. 

매직 넘버는 강력한 암호화 라이브러리를 사용하여 생성된 난수여야 합니다. C#에서의 생성 프로세스 예제를 보려면 <a href="https://www.nuget.org/packages/BotAuth" target="_blank">BotAuth</a> 라이브러리 내의 <a href="https://github.com/MicrosoftDX/botauth/tree/master/CSharp" target="_blank">이 코드</a>를 참조하세요. BotAuth에서는 Microsoft Bot Framework에서 빌드된 봇이 웹 사이트의 사용자를 인증하고 나중에 인증 프로세스에서 생성된 액세스 토큰을 사용하기 위한 봇-웹 사이트 흐름을 구현할 수 있도록 합니다. BotAuth는 채널 기능에 대해 어떠한 가정도 하지 않으므로 이러한 흐름이 대부분의 채널에서 잘 작동합니다. 

> [!NOTE]
> 채널은 포함된 자체 웹 보기를 빌드하므로 '매직 넘버' 유효성 검사 프로세스에 대한 필요성은 더 이상 지원되지 않습니다.

### <a id="website-signal-to-bot"></a> 웹 사이트는 봇에 어떻게 '신호'할까요?

봇은 사용자가 웹 사이트 흐름을 시작하기 위해 클릭하는 [하이퍼링크를 생성](#generate-hyperlink)할 때 채널의 대화 ID, 채널 ID 및 사용자 ID와 같은 현재 대화의 컨텍스트에 대한 쿼리 문자열 매개 변수를 통해 대상 URL에 데이터를 포함합니다. 이후에 웹 사이트는 이 정보를 사용하여 Bot Builder SDK 또는 REST API를 통해 해당 사용자 또는 대화에 대한 상태 변수를 읽고 쓸 수 있습니다. 웹 사이트가 웹 사이트 흐름이 완료되었음을 봇에 ‘신호로 알리는’ 방법의 예제를 보려면 위의 [6단계](#signal-to-bot)를 참조하세요.

## <a name="sample-code"></a>샘플 코드

이 문서에 설명된 대로 <a href="https://github.com/MicrosoftDX/botauth" target="_blank">BotAuth</a> 라이브러리를 사용하여 OAuth 흐름을 Microsoft Bot Framework에서 .NET 및 Node를 사용하여 빌드한 봇에 바인딩할 수 있습니다.

## <a name="additional-resources"></a>추가 리소스

- [다이얼로그](~/dotnet/bot-builder-dotnet-dialogs.md)
- [다이얼로그를 사용하여 대화 흐름 관리(.NET)](~/dotnet/bot-builder-dotnet-manage-conversation-flow.md)
- [다이얼로그를 사용하여 대화 흐름 관리(Node.js)](~/nodejs/bot-builder-nodejs-manage-conversation-flow.md)
