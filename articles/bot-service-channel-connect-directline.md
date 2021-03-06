---
title: 직접 회선에 봇 연결 | Microsoft Docs
description: 봇을 직접 회선에 연결하도록 구성하는 방법을 알아봅니다.
keywords: 직접 회선, 봇 채널, 사용자 지정 클라이언트, 채널에 연결, 구성
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/11/2018
ms.openlocfilehash: 9383e15590569458e795e9a0603df21f63609001
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997740"
---
# <a name="connect-a-bot-to-direct-line"></a>직접 회선에 봇 연결

직접 회선 채널을 사용하여 클라이언트 애플리케이션이 봇과 통신하도록 할 수 있습니다. 

## <a name="add-the-direct-line-channel"></a>직접 회선 채널 추가

직접 회선 채널을 추가하려면 [Azure Portal](https://portal.azure.com/)에서 봇을 열고, **채널** 블레이드를 클릭한 다음, **직접 회선**을 클릭합니다.

![직접 회선 채널 추가](~/media/bot-service-channel-connect-directline/directline-addchannel.png)

## <a name="add-new-site"></a>새 사이트 추가

다음으로, 봇에 연결할 클라이언트 애플리케이션을 나타내는 새 사이트를 추가합니다. **새 사이트 추가**를 클릭하고, 사이트 이름을 입력하고, **완료**를 클릭합니다.

![직접 회선 사이트 추가](~/media/bot-service-channel-connect-directline/directline-addsite.png)

## <a name="manage-secret-keys"></a>비밀 키 관리

사이트가 만들어지면 Bot Framework는 클라이언트 애플리케이션이 봇과 통신하기 위해 만드는 Direct Line API 요청을 [인증](~/rest-api/bot-framework-rest-direct-line-3-0-authentication.md)하는 데 사용할 수 있는 비밀 키를 생성합니다. 일반 텍스트로 키를 보려면 해당 키의 **표시**를 클릭합니다.

![직접 회선 키 표시](~/media/bot-service-channel-connect-directline/directline-showkey.png)

표시되는 키를 복사하여 안전하게 저장합니다. 이 키를 사용하여 클라이언트가 봇과 통신하기 위해 만드는 Direct Line API 요청을 [인증](~/rest-api/bot-framework-rest-direct-line-3-0-authentication.md)합니다.
또는 Direct Line API를 사용하여 클라이언트가 단일 대화의 범위 내에서 후속 요청을 인증하는 데 사용할 수 있는 [토큰의 키를 교환](~/rest-api/bot-framework-rest-direct-line-3-0-authentication.md#generate-token)합니다.

![직접 회선 키 복사](~/media/bot-service-channel-connect-directline/directline-copykey.png)

## <a name="configure-settings"></a>설정 구성

마지막으로, 사이트 설정을 구성합니다.

- 클라이언트 애플리케이션이 봇과 통신하는 데 사용할 직접 회선 프로토콜 버전을 선택합니다.

> [!TIP]
> 클라이언트 애플리케이션과 봇 간에 새 연결을 만드는 경우 Direct Line API 3.0을 사용합니다.

여기까지 마쳤으면 **완료**를 클릭하여 사이트 구성을 저장합니다. [새 사이트 추가](#add-new-site)부터 시작하여 봇과 연결할 클라이언트 응용 프로그램마다 이 프로세스를 반복할 수 있습니다.
