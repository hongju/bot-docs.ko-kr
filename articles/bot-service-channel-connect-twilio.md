---
title: Twilio에 봇 연결 | Microsoft Docs
description: Twilio에 대한 봇 연결을 구성하는 방법을 알아봅니다.
keywords: Twilio, 봇 채널, SMS, 앱, 전화, Twilio 구성, 클라우드 통신, 텍스트
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/9/2018
ms.openlocfilehash: 4de93d821c6b652021a9f695536350610776f5b4
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405899"
---
# <a name="connect-a-bot-to-twilio"></a>Twilio에 봇 연결

Twilio 클라우드 통신 플랫폼을 사용하여 사람들과 통신하도록 봇을 구성할 수 있습니다.

## <a name="log-in-to-or-create-a-twilio-account-for-sending-and-receiving-sms-messages"></a>SMS 메시지를 보내고 받기 위해 로그인 또는 Twilio 계정 만들기

Twilio 계정이 없는 경우 <a href="https://www.twilio.com/try-twilio" target="_blank">새 계정을 만듭니다</a>.

## <a name="create-a-twiml-application"></a>TwiML 애플리케이션 만들기

지침을 따라 <a href="https://support.twilio.com/hc/articles/223180928-How-Do-I-Create-a-TwiML-App-" target="_blank">TwiML 애플리케이션을 만듭니다</a>.

![앱 만들기](~/media/channels/twi-StepTwiml.png)

**속성** 아래에 **FRIENDLY NAME**을 입력합니다. 이 자습서에서는 "내 TwiML 앱"을 예로 사용합니다. 음성 아래의 **REQUEST URL**을 비워 둘 수 있습니다. **메시징** 아래에서 **요청 URL**은 `https://sms.botframework.com/api/sms`여야 합니다.

## <a name="select-or-add-a-phone-number"></a>전화 번호 선택 또는 추가

콘솔 사이트를 통해 인증된 호출자 ID를 추가하려면 <a href = "https://support.twilio.com/hc/articles/223180048-Adding-a-Verified-Phone-Number-or-Caller-ID-with-Twilio" target="_blank">여기</a>의 지침을 따르세요. 완료되면 **번호 관리**의 **활성 번호**에서 인증된 번호를 확인할 수 있습니다.

![전화 번호 설정](~/media/channels/twi-StepPhone.png)

## <a name="specify-application-to-use-for-voice-and-messaging"></a>음성 및 메시징에 사용할 애플리케이션 지정

번호를 클릭하고 **구성**으로 이동합니다. 음성 및 메시징 아래에서 **CONFIGURE WITH**를 TwiML 앱으로 설정하고 **TWIML APP**을 내 TwiML 앱으로 설정합니다. 완료되면 **저장**을 클릭합니다.

![애플리케이션 지정](~/media/channels/twi-StepPhone2.png)

**번호 관리**로 돌아가면 음성 및 메시징의 구성이 TwiML 앱으로 변경됩니다.

![지정된 번호](~/media/channels/twi-StepPhone3.png)


## <a name="gather-credentials"></a>자격 증명 수집

[콘솔 홈 페이지](https://www.twilio.com/console/)로 돌아가면 아래 표시된 것처럼 프로젝트 대시보드에 계정 SID 및 인증 토큰이 표시됩니다.

![앱 자격 증명 수집](~/media/channels/twi-StepAuth.png)

## <a name="submit-credentials"></a>자격 증명 제출

별도의 창에서 https://dev.botframework.com/ 의 Bot Framework 사이트로 돌아갑니다. 

- **내 봇**을 선택하고 Twilio에 연결하려는 봇을 선택합니다. 그러면 Azure Portal로 이동됩니다.
- **봇 관리** 아래에서 **채널**을 선택합니다. Twilio(SMS) 아이콘을 클릭합니다.
- 앞서 기록한 전화 번호, 계정 SID 및 인증 토큰을 입력합니다. 완료되면 **저장**을 클릭합니다.

![자격 증명 제출](~/media/channels/twi-StepSubmit.png)

이러한 단계를 완료한 경우, 봇은 Twilio를 사용하여 사용자와 통신하도록 성공적으로 구성됩니다.

## <a name="also-available-as-an-adapter"></a>어댑터로도 사용 가능

이 채널은 [어댑터로도 사용할 수 있습니다](https://botkit.ai/docs/v4/platforms/twilio-sms.html). 어댑터와 채널 중 하나를 선택할 때는 [현재 사용 가능한 어댑터](bot-service-channel-additional-channels.md#currently-available-adapters)를 참조하세요.