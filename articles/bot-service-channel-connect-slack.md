---
title: Slack에 봇 연결 | Microsoft Docs
description: Slack에 대한 봇 연결을 구성하는 방법을 알아봅니다.
keywords: 봇 연결, 봇 채널, Slack 봇, Slack 메시징 앱
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 01/09/2019
ms.openlocfilehash: 58416147a057bce8947943521a1226e1d9acbdf1
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693602"
---
# <a name="connect-a-bot-to-slack"></a>Slack에 봇 연결

Slack 메시징 앱을 사용하여 사용자와 통신하도록 봇을 구성할 수 있습니다.

## <a name="create-a-slack-application-for-your-bot"></a>봇에 대한 Slack 애플리케이션 만들기

[Slack](https://slack.com/signin)에 로그인한 다음, [Slack 애플리케이션 만들기](https://api.slack.com/apps) 채널로 이동합니다.

![봇 설정](~/media/channels/slack-NewApp.png)

## <a name="create-an-app-and-assign-a-development-slack-team"></a>앱 만들기 및 개발 Slack 팀 할당

앱 이름을 입력하고 개발 Slack 팀을 선택합니다. 아직 개발 Slack 팀 멤버가 아닌 경우 [만들거나 가입](https://slack.com/)합니다.

![앱 만들기](~/media/channels/slack-CreateApp.png)

**앱 만들기**를 클릭합니다. Slack은 앱을 만들고, 클라이언트 ID 및 클라이언트 비밀을 생성합니다.

## <a name="add-a-new-redirect-url"></a>새 리디렉션 URL 추가

다음으로, 새 리디렉션 URL을 추가합니다.

1. **OAuth 및 권한** 탭을 선택합니다.
2. **새 리디렉션 URL 추가**를 클릭합니다.
3. [https://slack.botframework.com](https://slack.botframework.com ) 을 입력합니다.
4. **추가**를 클릭합니다.
5. **URL 저장**을 클릭합니다.

![리디렉션 URL 추가](~/media/channels/slack-RedirectURL.png)

## <a name="create-a-slack-bot-user"></a>Slack 봇 사용자 만들기

봇 사용자를 추가하면 봇에 대한 사용자 이름을 할당하고 항상 온라인으로 표시될지 여부를 선택할 수 있습니다.

1. **봇 사용자** 탭을 선택합니다.
2. **봇 사용자 추가**를 클릭합니다.

![봇 만들기](~/media/channels/slack-CreateBot.png)

**봇 사용자 추가**를 클릭하여 설정이 유효한지 확인하고, **Always Show My Bot as Online**(내 봇을 항상 온라인으로 표시)을 **켜기**로 설정하고 **변경 내용 저장**을 클릭합니다.

![봇 만들기](~/media/channels/slack-CreateApp-AddBotUser.png)

## <a name="subscribe-to-bot-events"></a>봇 이벤트 구독

6가지 특정 봇 이벤트를 구독하려면 다음 단계를 따릅니다. 봇 이벤트를 구독하면 앱은 지정된 URL의 사용자 활동에 대한 알림을 받게 됩니다.

> [!TIP]
> 봇 핸들은 봇의 이름입니다. 봇의 핸들을 찾으려면 [https://dev.botframework.com/bots](https://dev.botframework.com/bots)를 방문하고 봇을 선택한 후 봇의 이름을 기록합니다.

1. **이벤트 구독** 탭을 선택합니다.
2. **이벤트 사용**을 **켜기**로 설정합니다.
3. **요청 URL**에 `https://slack.botframework.com/api/Events/{YourBotHandle}`을 중괄호 없이 입력합니다. 여기서 `{YourBotHandle}`은 봇 핸들입니다. 이 예제에 사용되는 봇 핸들은 **ContosoBot**입니다.

   ![이벤트 구독: 위](~/media/channels/slack-SubscribeEvents-a.png)

4. **봇 이벤트 구독**에서 **봇 사용자 이벤트 추가**를 클릭합니다.
5. 이벤트 목록에서 이러한 6개 이벤트 유형을 선택합니다.
    * `member_joined_channel`
    * `member_left_channel`
    * `message.channels`
    * `message.groups`
    * `message.im`
    * `message.mpim`

   ![이벤트 구독: 가운데](~/media/channels/slack-SubscribeEvents-b.png)

6. **변경 내용 저장**을 클릭합니다.

   ![이벤트 구독: 아래](~/media/channels/slack-SubscribeEvents-c.png)

## <a name="add-and-configure-interactive-messages-optional"></a>대화형 메시지 추가 및 구성(선택 사항)

봇이 단추와 같은 Slack 관련 기능을 사용하는 경우 다음 단계를 수행합니다.

1. **대화형 구성 요소** 탭을 선택하고 **대화형 구성 요소 사용**을 클릭합니다.
2. **요청 URL**로 `https://slack.botframework.com/api/Actions`를 입력합니다.
3. **변경 내용 저장** 단추를 클릭합니다.

![메시지 사용](~/media/channels/slack-MessageURL.png)

## <a name="gather-credentials"></a>자격 증명 수집

**기본 정보** 탭을 선택하고 **앱 자격 증명** 섹션으로 스크롤합니다.
Slack 봇 구성에 필요한 클라이언트 ID, 클라이언트 암호 및 확인 토큰이 표시됩니다.

![자격 증명 수집](~/media/channels/slack-AppCredentials.png)

## <a name="submit-credentials"></a>자격 증명 제출

별도의 브라우저 창에서 `https://dev.botframework.com/`의 Bot Framework 사이트로 돌아갑니다.

1. **내 봇**을 선택하고 Slack에 연결하려는 봇을 선택합니다.
2. **채널** 섹션에서 Slack 아이콘을 클릭합니다.
3. **Slack 자격 증명 입력** 섹션에서 Slack 웹 사이트의 앱 자격 증명을 해당 필드에 붙여넣습니다.
4. **방문 페이지 URL**은 선택 사항입니다. 생략하거나 변경할 수 있습니다.
5. **저장**을 클릭합니다.

![자격 증명 제출](~/media/channels/slack-SubmitCredentials.png)

개발 Slack 팀에 Slack 앱에 대한 액세스 권한을 부여하려면 지침을 따릅니다.

## <a name="enable-the-bot"></a>봇 사용

Slack 구성 페이지에서 저장 단추 옆에 있는 슬라이더가 **사용**으로 설정되어 있는지 확인합니다.
봇은 Slack의 사용자와 통신하도록 구성됩니다.

## <a name="create-an-add-to-slack-button"></a>Slack에 추가 단추 만들기

Slack은 [이 페이지](https://api.slack.com/docs/slack-button)의 *Slack에 추가 단추* 섹션에서 Slack 사용자가 봇을 찾는 데 도움을 주기 위해 사용할 수 있는 HTML을 제공합니다.
봇에서 이 HTML을 사용하려면 href 값(`https://`로 시작)을 봇의 Slack 채널 설정에 있는 URL로 바꿉니다.
대체 URL을 가져오려면 다음 단계를 수행합니다.

1. [https://dev.botframework.com/bots](https://dev.botframework.com/bots)에서 봇을 클릭합니다.
2. **채널**을 클릭하고 **Slack**이라는 항목을 마우스 오른쪽 단추로 클릭하고 **링크 복사**를 클릭합니다. 이제 이 URL이 클립보드에 포함되었습니다.
3. 클립보드의 이 URL을 Slack 단추에 제공되는 HTML에 붙여넣습니다. 이 URL은 이 봇을 위해 Slack이 제공하는 href 값을 대신합니다.

권한 있는 사용자는 수정된 이 HTML에서 제공하는 **Slack에 추가** 단추를 클릭하여 Slack에서 해당 봇에 연결할 수 있습니다.

## <a name="also-available-as-an-adapter"></a>어댑터로도 사용 가능

이 채널은 [어댑터로도 사용할 수 있습니다](https://botkit.ai/docs/v4/platforms/slack.html). 어댑터와 채널 중 하나를 선택할 때는 [현재 사용 가능한 어댑터](bot-service-channel-additional-channels.md#currently-available-adapters)를 참조하세요.
