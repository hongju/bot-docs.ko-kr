---
title: Cortana에 봇 연결 | Microsoft Docs
description: Cortana 인터페이스를 통해 액세스하기 위해 봇을 구성하는 방법을 알아봅니다.
keywords: cortana, 봇 채널 cortana 구성
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/30/2018
ms.openlocfilehash: 9e3f2f19c480a9d2fe6df0baea74d449bb584b4f
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999092"
---
# <a name="connect-a-bot-to-cortana"></a>Cortana에 봇 연결

Cortana는 텍스트 대화뿐 아니라 음성 메시지도 주고 받을 수 있는 음성 지원 채널입니다. Cortana에 연결하도록 고안된 봇은 텍스트 뿐만 아니라 음성용으로도 디자인되어야 합니다. Cortana *스킬*은 Cortana 클라이언트를 사용하여 호출할 수 있는 봇입니다. 봇을 게시하면 사용 가능한 스킬 목록에 봇이 추가됩니다.

Cortana 채널을 추가하려면 [Azure Portal](https://portal.azure.com/)에서 봇을 열고, **채널** 블레이드를 클릭한 다음, **Cortana**를 클릭합니다.

![Cortana 채널 추가](~/media/channels/cortana-addchannel.png)

## <a name="configure-cortana"></a>Cortana 구성

Cortana 채널에 봇을 연결할 때 봇에 대한 일부 기본 정보가 등록 양식에 미리 채워집니다. 이 정보를 주의 깊게 검토합니다. 이 양식은 다음 필드로 구성됩니다.

| 필드 | 설명 |
|------|------|
| **스킬 아이콘** | 스킬이 호출될 때 Cortana 캔버스에 표시되는 아이콘입니다. 스킬을 검색 가능한 위치(예: Microsoft 스토어)에서도 사용됩니다. (최대 32KB, PNG만 해당)|
| **표시 이름** | Cortana 스킬의 이름은 시각적 UI의 맨 위에 표시됩니다. (30자 제한) |
| **호출 이름** | 스킬을 호출할 때 사용자가 말하는 이름입니다. 3단어 이하이며 발음하기 쉬워야 합니다. 이 이름의 선택 방법에 대한 자세한 내용은 [호출 이름 지침][invocation]을 참조하세요.|

![기본 설정](~/media/channels/cortana-defaultsettings.png)

## <a name="general-bot-information"></a>일반 봇 정보

**연결된 서비스 섹션을 통해 사용자 ID 관리** 아래에서 사용하도록 설정하기 위한 옵션을 누릅니다. 양식을 채웁니다.

별표(*)로 표시된 모든 필드는 필수입니다. 봇을 Azure에 게시한 후에 Cortana에 연결할 수 있습니다.

![사용자 ID 관리, 1부](~/media/channels/cortana-manageidentity-1.png)
![사용자 ID 관리, 2부](~/media/channels/cortana-manageidentity-2.png)

### <a name="when-should-cortana-prompt-for-a-user-to-sign-in"></a>Cortana에서 사용자에게 로그인을 요청해야 하는 경우

사용자의 스킬 호출 시 Cortana에서 사용자를 로그인하도록 하려면 **호출 시 로그인**을 선택합니다.

사용자가 Bot Service 로그인 카드를 사용하여 로그인하도록 하려면 **필요 시 로그인**을 선택합니다. 일반적으로 인증이 필요한 기능을 사용하는 경우에만 사용자를 로그인하려면 이 옵션을 사용합니다. 사용자 스킬이 SignIn Card를 첨부 파일로 포함하는 메시지를 보낼 경우 Cortana는 SignIn Card를 무시하고 계정 연결 설정을 사용하여 권한 부여 흐름을 수행합니다.

### <a name="account-name"></a>계정 이름

사용자가 스킬에 로그인할 경우 표시하려는 스킬의 이름입니다.

### <a name="client-id-for-third-party-services"></a>타사 서비스에 대한 클라이언트 ID

봇의 응용 프로그램 ID입니다. 봇을 등록할 때 이 ID를 수신했을 것입니다.

### <a name="space-separated-list-of-scopes"></a>공백으로 구분된 범위 목록

서비스에 필요한 범위를 지정합니다(서비스 설명서 참조).

### <a name="authorization-url"></a>권한 부여 URL

`https://login.microsoftonline.com/common/oauth2/v2.0/authorize`로 설정합니다.

### <a name="token-options"></a>토큰 옵션

**POST**를 선택합니다.

### <a name="grant-type"></a>권한 부여 유형

코드 권한 부여 흐름을 사용하려면 **권한 부여 코드**를 선택하고, 암시적 흐름을 사용하려면 **암시적**을 선택합니다.

### <a name="token-url"></a>토큰 URL

**인증 코드** 권한 부여 유형에서 `https://login.microsoftonline.com/common/oauth2/v2.0/token`으로 설정합니다.

### <a name="client-secretpassword-for-third-party-services"></a>타사 서비스에 대한 클라이언트 비밀/암호

봇의 암호입니다. 봇을 등록할 때 이 암호를 수신했을 것입니다.

### <a name="client-authentication-scheme"></a>클라이언트 인증 체계계

**HTTP 기본**을 선택합니다.

### <a name="internet-access-required-to-authenticate-users"></a>사용자를 인증하는 데 필요한 인터넷 액세스

선택 취소된 상태로 둡니다.

### <a name="request-user-profile-data-optional"></a>사용자 프로필 데이터 요청(선택 사항)

Cortana에서는 사용자를 위해 봇을 사용자 지정하는 데 사용할 수 있는 여러 다른 유형의 사용자 프로필 정보에 액세스할 수 있습니다. 예를 들어, 스킬이 사용자의 이름 및 위치에 대해 액세스 권한이 있는 경우 스킬은 "Hello Kamran, I hope you are having a pleasant day in Bellevue, Washington"과 같은 사용자 지정 응답을 받을 수 있습니다.

**사용자 프로필 요청 추가**를 클릭한 다음, 드롭다운 목록에서 원하는 사용자 프로필 정보를 선택합니다. 봇의 코드에서 이 정보에 액세스하는 데 사용할 친숙한 이름을 추가합니다.

### <a name="deploy-on-cortana"></a>Cortana에 배포

Cortana 스킬에 대한 등록 양식을 다 채운 후에는 **Cortana에 배포**를 클릭하여 연결을 완료합니다. 이렇게 하면 봇의 채널 블레이드로 다시 이동되며, 이제 Cortana에 연결되어 있음을 알 수 있습니다.

이때 봇은 사용자 계정에 Cortana 스킬로 배포됩니다.

## <a name="next-steps"></a>다음 단계

* [Cortana 스킬 키트](https://aka.ms/CortanaSkillsDocs)
* [디버깅 사용](bot-service-debug-cortana-skill.md)
* [Cortana 스킬 게시][publish]

[invocation]: https://docs.microsoft.com/en-us/cortana/skills/cortana-invocation-guidelines
[publish]: https://docs.microsoft.com/en-us/cortana/skills/publish-skill
[connected]: https://aka.ms/CortanaSkillsBotConnectedAccount
[CortanaEntity]: https://aka.ms/lgvcto
