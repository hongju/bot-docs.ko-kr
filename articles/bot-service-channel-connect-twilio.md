---
title: Twilio에 봇 연결 | Microsoft Docs
description: Twilio에 대한 봇 연결을 구성하는 방법을 알아봅니다.
keywords: Twilio, 봇 채널, SMS, 앱, 전화, Twilio 구성, 클라우드 통신, 텍스트
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 7e09126d50cfbebfc0aad0ee7fcb71b4e7551a7d
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301215"
---
# <a name="connect-a-bot-to-twilio"></a>Twilio에 봇 연결

Twilio 클라우드 통신 플랫폼을 사용하여 사람들과 통신하도록 봇을 구성할 수 있습니다.

## <a name="log-in-to-or-create-a-twilio-account-for-sending-and-receiving-sms-messages"></a>SMS 메시지를 보내고 받기 위해 로그인 또는 Twilio 계정 만들기

Twilio 계정이 없는 경우 <a href="https://www.twilio.com/try-twilio" target="_blank">새 계정 만들기</a>

## <a name="create-a-twiml-application"></a>TwiML 응용 프로그램 만들기

<a href="https://www.twilio.com/user/account/messaging/dev-tools/twiml-apps/add" target="_blank">TwiML 응용 프로그램 만들기</a>

![앱 만들기](~/media/channels/twi-StepTwiml.png)

 메시징 아래에서 요청 URL은 **https://sms.botframework.com/api/sms**여야 합니다.

## <a name="select-or-add-a-phone-number"></a>전화 번호 선택 또는 추가

<a href="https://www.twilio.com/user/account/phone-numbers/incoming" target="_blank">전화 번호를 선택 또는 추가</a>합니다. 해당 번호를 클릭하여 만든 TwiML 응용 프로그램에 추가합니다.

![전화 번호 설정](~/media/channels/twi-StepPhone.png)

## <a name="specify-application-to-use-for-messaging"></a>메시징에 사용할 응용 프로그램 지정
**메시징** 섹션에서 **TwiML 앱**을 방금 만든 TwiML 앱의 이름으로 설정합니다.
나중에 사용할 수 있도록 **전화 번호** 값을 복사합니다.

![앱 지정](~/media/channels/twi-StepPhone2.png)

## <a name="gather-credentials"></a>자격 증명 수집

<a href="https://www.twilio.com/user/account/settings" target="_blank">자격 증명을 수집</a>하고 "눈 모양" 아이콘을 클릭하여 인증 토큰을 표시합니다.

![앱 자격 증명 수집](~/media/channels/twi-StepAuth.png)

## <a name="submit-credentials"></a>자격 증명 제출

이전에 복사한 전화 번호, accountSID 및 인증 토큰을 입력하고 **Twilio 자격 증명 제출**을 클릭합니다.

## <a name="enable-the-bot"></a>봇 사용
**Enable this bot on SMS**(SMS에서 이 봇 사용)를 선택합니다. 그런 후 **I'm done configuring SMS**(SMS 구성 완료)를 클릭합니다.

이러한 단계를 완료한 경우, 봇은 Twilio를 사용하여 사용자와 통신하도록 성공적으로 구성됩니다.

