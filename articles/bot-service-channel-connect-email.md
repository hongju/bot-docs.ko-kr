---
title: Office 365 메일에 봇 연결 | Microsoft Docs
description: Office 365를 사용하여 메일을 주고받도록 봇을 구성하는 방법을 알아봅니다.
keywords: Office 365, 봇 채널, 메일, 메일 자격 증명, azure portal, 사용자 지정 메일
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/10/2018
ms.openlocfilehash: 93270dd6211d8aef1ff44fb8e272855df2058b8a
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997210"
---
# <a name="connect-a-bot-to-office-365-email"></a>Office 365 메일에 봇 연결

봇은 기타 [채널](~/bot-service-manage-channels.md) 외에 Office 365 메일을 통해 사용자와 통신할 수 있습니다. 메일 계정에 액세스하도록 구성된 봇은 새 메일이 도착할 때 메시지를 받습니다. 이후 봇은 비즈니스 논리에 지정된 대로 응답할 수 있습니다. 예를 들어, 봇은 다음 메시지와 함께 메일을 받았음을 확인하는 메일 회신을 보낼 수 있습니다. “안녕하세요! 주문해 주셔서 감사합니다! 주문 처리가 즉시 시작될 예정입니다.”

> [!WARNING]
> 원치 않거나 요청하지 않은 벌크 메일을 보내는 봇을 포함하여 “스팸 봇”을 만드는 것은 Bot Framework [준수 사항](https://www.botframework.com/Content/Microsoft-Bot-Framework-Preview-Online-Services-Agreement.htm)을 위반하는 것입니다.

## <a name="configure-email-credentials"></a>메일 자격 증명 구성

메일 채널 구성에 Office 365 자격 증명을 입력하여 봇을 메일 채널에 연결할 수 있습니다.
AAD를 대체하는 공급업체를 사용하는 페더레이션 인증은 지원되지 않습니다.

> [!NOTE]
> 고유한 개인 메일 계정을 봇에 사용하면 안 됩니다. 해당 메일에 전송되는 모든 메시지가 봇에 전달되기 때문입니다. 이로 인해 봇이 부적절하게 보낸 사람에게 응답을 보낼 수 있습니다. 이런 이유로 봇은 전용 O365 메일 계정만 사용해야 합니다.

메일 채널을 추가하려면 [Azure Portal](https://portal.azure.com/)에서 봇을 열고, **채널** 블레이드를 클릭한 다음, **메일**을 클릭합니다. 유효한 메일 자격 증명을 입력하고 **저장**을 클릭합니다.

![메일 자격 증명 입력](~/media/bot-service-channel-connect-email/bot-service-channel-connect-email-credentials.png)

메일 채널은 현재 Office 365에서만 작동합니다. 다른 메일 서비스는 현재 지원되지 않습니다.

## <a name="customize-emails"></a>메일 사용자 지정

메일 채널은 `channelData` 속성을 사용하여 더 향상된 사용자 지정 메일을 만들기 위해 사용자 지정 속성을 보내는 기능을 지원합니다.

[!INCLUDE [Email channelData table](~/includes/snippet-channelData-email.md)]

다음 예제 메시지는 이러한 `channelData` 속성을 포함하는 JSON 파일을 보여 줍니다.

```json
{
    "type": "message",
    "locale": "en-Us",
    "channelID": "email",
    "from": { "id": "mybot@mydomain.com", "name": "My bot"},
    "recipient": { "id": "joe@otherdomain.com", "name": "Joe Doe"},
    "conversation": { "id": "123123123123", "topic": "awesome chat" },
    "channelData":
    {
        "htmlBody": "<html><body style = /"font-family: Calibri; font-size: 11pt;/" >This is more than awesome.</body></html>",
        "subject": "Super awesome message subject",
        "importance": "high",
        "ccRecipients": "Yasemin@adatum.com;Temel@adventure-works.com"
    }
}
```

::: moniker range="azure-bot-service-3.0"
`channelData` 사용에 대한 자세한 내용은 [Node.js](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-ChannelData) 샘플 또는 [.NET](~/dotnet/bot-builder-dotnet-channeldata.md) 문서를 참조하세요.
::: moniker-end

::: moniker range="azure-bot-service-4.0"
`channelData` 사용 방법에 대한 자세한 내용은 [채널 관련 기능을 구현하는 방법](~/v4sdk/bot-builder-channeldata.md)을 참조하세요.
::: moniker-end

## <a name="additional-resources"></a>추가 리소스

<!-- Put whole list in monikers, even though it's just the second item that needs to be different. -->
::: moniker range="azure-bot-service-3.0"
* [채널](~/bot-service-manage-channels.md)에 봇 연결
* .NET용 Bot Builder SDK를 사용하여 [채널 관련 기능 구현](dotnet/bot-builder-dotnet-channeldata.md)
* [채널 검사기](bot-service-channel-inspector.md)를 사용하여 채널이 봇 응용 프로그램의 특정 기능을 렌더링하는 방법 확인
::: moniker-end
::: moniker range="azure-bot-service-4.0"
* [채널](~/bot-service-manage-channels.md)에 봇 연결
* .NET용 Bot Builder SDK를 사용하여 [채널 관련 기능 구현](~/v4sdk/bot-builder-channeldata.md)
* [채널 검사기](bot-service-channel-inspector.md)를 사용하여 채널이 봇 응용 프로그램의 특정 기능을 렌더링하는 방법 확인
::: moniker-end
