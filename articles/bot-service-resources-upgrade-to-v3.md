---
title: 봇을 Bot Framework API v3로 업그레이드 | Microsoft Docs
description: 봇을 Bot Framework API v3로 업그레이드하는 방법을 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: dbd7d437b7a1cdffd446337192e77a8c3d93e4a0
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405777"
---
# <a name="upgrade-your-bot-to-bot-framework-api-v3"></a>봇을 Bot Framework API v3로 업그레이드

Build 2016 Microsoft은 Microsoft Bot Framework와 Bot Connector API의 초기 버전, Bot Builder 및 Bot Connector SDK를 발표했습니다. 이때부터 사용자의 의견을 수집하면서 REST API 및 SDK 개선을 위해 노력해왔습니다.

2016년 7월에는 Bot Framework API v3가 릴리스되었으며 Bot Framework API v1이 지원이 중단되었습니다. API v1을 사용하는 봇은 2016년 12월에는 Skype에서, 2017년 2월 23일에는 나머지 모든 채널에서 작동이 중단되었습니다. API v1을 사용하여 봇을 만들었으며 다시 작동하고 싶은 경우 이 문서의 지침에 따라 API v3로 업그레이드해야 합니다. 업그레이드 프로세스를 엔드투엔드로 이해하려면 시작하기 전에 이 문서 전체를 읽어보세요. 

## <a name="step-1-get-your-app-id-and-password-from-the-bot-framework-portal"></a>1단계: Bot Framework 포털에서 앱 ID 및 암호 가져오기

[Bot Framework 포털](https://dev.botframework.com/)에 로그인하고 **내 봇**을 클릭한 후 봇을 선택하여 해당 대시보드를 엽니다. 다음으로, **봇 관리** 아래에서 페이지 왼쪽에 있는 **설정** 링크를 클릭합니다. 

[설정] 페이지의 **구성** 섹션 내에서 **Microsoft 앱 ID** 필드의 내용을 검사하고 다음 단계를 계속 진행합니다.

<!-- TODO: Remove this 
### Case 1: App ID field is already populated

If the **App ID** field is already populated, complete these steps:
-->

1. **Microsoft 앱 ID 및 암호 관리**를 클릭합니다.  
![구성](./media/upgrade/manage-app-id.png)

2. **새 암호 생성**을 클릭합니다.  
![새 암호 생성](./media/upgrade/generate-new-password.png)

3. MSA 앱 ID와 함께 새 암호를 복사하고 저장합니다. 나중에 이러한 값이 필요합니다.  
![새 암호](./media/upgrade/new-password-generated.png)

**Microsoft 앱 ID 및 암호**를 검색하는 또 다른 방법은 다음 [지침](https://blog.botframework.com/2018/07/03/find-your-azure-bots-appid-and-appsecret/)에 따라 수행할 수 있습니다.

<!-- TODO: These steps are no longer valid. AppID will always be generated, confirmed with Support Engineers
### Case 2: App ID field is empty

If the **App ID** field is empty, complete these steps:

1. Click **Create Microsoft App ID and password**.  
   ![Create App ID and password](~/media/upgrade/generate-appid-and-password.png)
   > [!IMPORTANT]
   > Do not select the **Version 3.0** radio button yet. You will do this later, after you have [updated your bot code](#update-code).</div>

2. Click **Generate a password to continue**.  
   ![Generate app password](~/media/upgrade/generate-a-password-to-continue.png)

3. Copy and save the new password along with the MSA App Id; you will need these values in the future.  
   ![New password](~/media/upgrade/new-password-generated.png)

4. Click **Finish and go back to Bot Framework**.  
   ![Finish and go back to Portal](~/media/upgrade/finish-and-go-back-to-bot-framework.png)

5. Back on the bot settings page in the Bot Framework Portal, scroll to the bottom of the page and click **Save changes**.  
   ![Save changes](~/media/upgrade/save-changes.png)
-->

## <a id="update-code"></a> 2단계: 봇 코드를 버전 4.0으로 업데이트

V1 봇은 더 이상 호환되지 않습니다. 봇을 업데이트하려면 V3에서 새 봇을 만들어야 합니다. 이전 코드를 유지하려면 코드를 수동으로 마이그레이션해야 합니다.

가장 쉬운 방법은 새 [SDK](https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0)를 사용하여 봇을 다시 만들어 배포하는 것입니다. 

이전 코드를 유지하려면 아래 단계를 따릅니다.

1. 새 봇 애플리케이션을 만듭니다.
2. 이전 코드를 새 봇 애플리케이션에 복사합니다.
3. Nuget 패키지 관리자를 통해 SDK를 최신 버전으로 업그레이드합니다.
4. 발생하는 오류를 모두 수정하고, 새 [SDK](https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0)를 참조합니다.
5. 다음 [지침](https://docs.microsoft.com/azure/bot-service/bot-builder-howto-deploy-azure?view=azure-bot-service-4.0)에 따라 Azure에 봇을 배포합니다.

<!-- TODO: Remove outdated code 
To update your bot code to version 3.0, complete these steps:

1. Update to the latest version of the [Bot Framework SDK](https://github.com/Microsoft/BotBuilder) for your bot's language.
2. Update your code to apply the necessary changes, according the guidance below.
3. Use the [Bot Framework Emulator](~/bot-service-debug-emulator.md) to test your bot locally and then in the cloud.

The following sections describe the key differences between API v1 and API v3. After you have updated your code to API v3, you can finish the upgrade process by [updating your bot settings](#step-3) in the Bot Framework Portal.
-->

### <a name="botbuilder-and-connector-are-now-one-sdk"></a>BotBuilder 및 커넥터는 이제 하나의 SDK임

여러 NuGet 패키지(또는 NPM 모듈)를 사용하여 작성기 및 커넥터에 대해 별도 SDK를 설치할 필요 없이, 두 라이브러리를 단일 Bot Framework SDK로 가져올 수 있습니다.

- .NET용 Bot Framework SDK: `Microsoft.Bot.Builder` NuGet 패키지
- Node.js용 Bot Framework SDK: `botbuilder` NPM 모듈

독립 실행형 `Microsoft.Bot.Connector` SDK는 이제 더 이상 사용되지 않고 유지 관리되지도 않습니다.

### <a name="message-is-now-activity"></a>메시지가 이제 활동임

`Message` 개체는 API v3에서 `Activity` 개체로 바뀌었습니다. 가장 일반적인 형식의 활동은 **메시지**이지만 다양한 형식의 정보를 봇이나 채널에 전달하는 데 사용할 수 있는 다른 활동 형식이 있습니다. 메시지에 대한 자세한 내용은 [메시지 만들기](~/dotnet/bot-builder-dotnet-create-messages.md) 및 [활동 보내기 및 받기](~/dotnet/bot-builder-dotnet-connector.md)를 참조하세요.

### <a name="activity-types--events"></a>활동 형식 및 이벤트

API v3에서 일부 이벤트의 이름이 바뀌었거나 리팩터링되었습니다. 또한 특정 활동 형식을 기억할 필요가 없도록 커넥터에 새 `ActivityTypes` 열거형이 추가되었습니다.

- `conversationUpdate` 활동 형식은 Bot/User Added/Removed To/From Conversation을 단일 메서드로 바꿉니다.
- 새 `typing` 활동 형식은 봇이 응답을 컴파일하고 있다는 사실을 나타내고 사용자가 응답을 입력하고 있음을 알 수 있도록 합니다.
- 새 `contactRelationUpdate` 활동 형식은 봇이 사용자의 연락처 목록에 추가 또는 제거되었는지를 알 수 있도록 합니다.

봇이 `conversationUpdate` 활동을 수신하면 `MembersRemoved` 속성 및 `MembersAdded` 속성은 대화에 추가되었거나 제거된 사용자를 나타냅니다. 봇이 `contactRelationUpdate` 활동을 수신하면 `Action` 속성은 사용자가 연락처 목록에 봇을 추가했는지 또는 제거했는지를 나타냅니다. 활동 형식에 대한 자세한 내용은 [활동 개요](~/dotnet/bot-builder-dotnet-activities.md)를 참조하세요.

### <a name="addressing-messages"></a>메시지 주소 지정

메시지 내에서 보낸 사람, 받는 사람 및 채널 정보가 지정되는 위치가 API v3에서 약간 변경되었습니다.

|API v1 필드 | API v3 필드|
|--------|--------|
| `From` 개체 | `From` 개체 |
| `To` 개체 | `Recipient` 개체 |
| `ChannelConversationID` 속성 | `Conversation` 개체|
| `ChannelId` 속성 | `ChannelId` 속성 |

메시지 주소 지정에 대한 자세한 내용은 [활동 보내기 및 받기](~/dotnet/bot-builder-dotnet-connector.md)를 참조하세요.

### <a name="sending-replies"></a>회신 보내기

Bot Framework API v3에서 사용자에게 보내는 모든 회신은 봇으로 들어오는 메시지용 HTTP POST를 사용하여 인라인으로 진행되지 않고 별도로 시작된 HTTP 요청에 대해 비동기적으로 전송됩니다. 커넥터를 통해 사용자에게 인라인으로 메시지가 반환되지 않으므로 봇의 POST 메서드에 대한 반환 형식은 `HttpResponseMessage`입니다. 즉, 봇은 여러분이 사용자에게 보내려는 문자열을 동기적으로 "반환"하지 않고, 들어오는 POST에 대한 응답으로 회신할 필요 없이 코드의 어디에서든지 회신 메시지를 전송합니다. 다음 두 메서드 모두 대화로 메시지를 보냅니다.

- `SendToConversation`
- `ReplyToConversation`

`SendToConversation` 메서드는 지정된 메시지를 대화의 끝에 추가하지만, `ReplyToConversation` 메서드(지원하는 대화용)는 지정된 메시지를 대화의 이전 메시지에 직접 회신으로 추가합니다. 이러한 메서드에 대한 자세한 내용은 [활동 보내기 및 받기](~/dotnet/bot-builder-dotnet-connector.md)를 참조하세요.

### <a name="starting-conversations"></a>대화 시작

Bot Framework API v3에서는 새 메서드 `CreateDirectConversation`을 사용하여 단일 사용자와 프라이빗 대화를 시작하거나, 새 메서드 `CreateConversation`을 사용하여 여러 사용자와 그룹 대화를 시작할 수 있습니다. 대화를 시작 하는 방법에 대한 자세한 내용은 [활동 보내기 및 받기](~/dotnet/bot-builder-dotnet-connector.md#start-a-conversation)를 참조하세요.

### <a name="attachments-and-options"></a>첨부 파일 및 옵션

Bot Framework API v3에서는 첨부 파일 및 카드를 보다 강력하게 구현할 수 있습니다. `Options` 형식은 API v3에서 더 이상 지원되지 않으며, 대신 카드로 대체되었습니다. .NET을 사용하여 메시지에 첨부 파일을 추가하는 방법에 대한 자세한 내용은 [메시지에 미디어 첨부 파일 추가](~/dotnet/bot-builder-dotnet-add-media-attachments.md) 및 [메시지에 서식 있는 카드 첨부 파일 추가](~/dotnet/bot-builder-dotnet-add-rich-card-attachments.md)를 참조하세요.

### <a name="bot-data-storage-bot-state"></a>봇 데이터 저장소(봇 상태)

Bot Framework API v1에서 봇 상태 데이터를 관리하기 위한 API는 메시징 API에 포함되었습니다. Bot Framework API v3에서는 이러한 API는 별개입니다. 이제, Bot Status Service를 사용하여 상태 데이터를 가져오고(대신, `Message` 개체 내에 포함될 것으로 가정함) 및 상태 데이터를 저장해야 합니다(대신 `Message` 개체의 일부로 전달). Bot Status Service를 사용하여 봇 상태 데이터를 관리하는 방법은 [상태 데이터 관리](~/dotnet/bot-builder-dotnet-state.md)를 참조하세요.

> [!IMPORTANT]
> Bot Framework 상태 서비스 API는 프로덕션 환경에는 권장되지 않으며 이후 릴리스에서 사용되지 않을 수 있습니다. 테스트 용도로 메모리 내 저장소를 사용하거나 프로덕션 봇용으로 **Azure 확장**을 사용하도록 봇 코드를 업데이트하는 것이 좋습니다. 자세한 내용은 [.NET](~/dotnet/bot-builder-dotnet-state.md) 또는 [Node](~/nodejs/bot-builder-nodejs-state.md) 구현에 대한 **상태 데이터 관리** 항목을 참조하세요.

### <a name="webconfig-changes"></a>Web.config 변경 내용

Bot Framework API v1에서는 이러한 키와 함께 인증 속성을 **Web.Config**에 저장했습니다.

- `AppID`
- `AppSecret`

Bot Framework API v3에서는 이러한 키와 함께 인증 속성을 **Web.Config**에 저장합니다.

- `MicrosoftAppID`
- `MicrosoftAppPassword`

## <a id="step-3"></a> 3단계: Azure에 업데이트 봇을 배포합니다.

봇 코드를 API v3로 업그레이드한 후에는 다음 [지침](https://docs.microsoft.com/azure/bot-service/bot-builder-howto-deploy-azure?view=azure-bot-service-4.0)에 따라 Azure에 봇을 배포합니다. V1이 더 이상 지원되지 않으므로 모든 봇은 Azure 서비스에 배포되면 자동으로 V3 API를 사용합니다.

<!-- TODO: Documentation set for removal 
1. Sign in to the [Bot Framework Portal](https://dev.botframework.com/).

2. Click **My bots** and select your bot to open its dashboard. 

3. Click the **SETTINGS** link that is located near the top-right corner of the page. 

4. Under **Version 3.0** within the **Configuration** section, paste your bot's endpoint into the **Messaging endpoint** field.  
![Version 3 configuration](~/media/upgrade/paste-new-v3-enpoint-url.png)

5. Select the **Version 3.0** radio button.  
![Select version 3.0](~/media/upgrade/switch-to-v3-endpoint.png)

6. Scroll to the bottom of the page and click **Save changes**.  
![Save changes](~/media/upgrade/save-changes.png)
-->