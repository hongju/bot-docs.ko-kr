---
title: 봇 보안 유지 | Microsoft Docs
description: HTTPS 및 Bot Framework 인증을 사용하여 봇 보안을 유지하는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: deb184483bf7e0963f827b20377291ab971c1516
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997341"
---
# <a name="secure-your-bot"></a>봇 보안 유지

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

봇은 Bot Framework Connector 서비스를 통해 여러 다른 통신 채널(Skype, SMS, 메일 및 기타)에 연결될 수 있습니다. 이 문서에서는 HTTPS 및 Bot Framework 인증을 사용하여 봇 보안을 유지하는 방법을 설명합니다.

## <a name="use-https-and-bot-framework-authentication"></a>HTTPS 및 Bot Framework 인증 사용

봇의 끝점에 Bot Framework [Connector](bot-builder-dotnet-concepts.md#connector)에서만 액세스할 수 있도록 하려면 HTTPS만 사용하도록 봇의 끝점을 구성하고, 봇을 [등록](~/bot-service-quickstart-registration.md)하고 해당 appID 및 암호를 가져와 Bot Framework 인증을 사용하도록 설정합니다. 

## <a name="configure-authentication-for-your-bot"></a>봇에 대한 인증 구성

봇의 web.config 파일에 봇의 appID 및 암호를 지정합니다. 

> [!NOTE]
> 봇의 **AppID** 및 **AppPassword**를 찾으려면 [MicrosoftAppID 및 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)를 참조하세요.

```xml
<appSettings>
    <add key="MicrosoftAppId" value="_appIdValue_" />
    <add key="MicrosoftAppPassword" value="_passwordValue_" />
</appSettings>
```

그런 후 .NET용 Bot Builder SDK를 사용하여 봇을 만들 때 `[BotAuthentication]` 특성을 사용하여 인증 자격 증명을 지정합니다. 

web.config 파일에 저장된 인증 자격 증명을 사용하려면 매개 변수 없이 `[BotAuthentication]`을 지정합니다.

[!code-csharp[Use botAuthentication attribute](../includes/code/dotnet-security.cs#attribute1)]

인증 자격 증명에 대해 다른 값을 사용하려면 `[BotAuthentication]` 특성을 지정하고 해당 값을 전달합니다.

[!code-csharp[Use botAuthentication attribute with parameters](../includes/code/dotnet-security.cs#attribute2)]

## <a name="additional-resources"></a>추가 리소스

- [.NET용 봇 작성기 SDK](bot-builder-dotnet-overview.md)
- [.NET용 Bot Builder SDK의 주요 개념](bot-builder-dotnet-concepts.md)
- [Bot Framework에 봇 등록](~/bot-service-quickstart-registration.md)
