---
title: Azure Bot Service의 사용자 인증 | Microsoft Docs
description: Azure Bot Service의 사용자 인증 기능을 알아봅니다.
keywords: Azure Bot Service, 인증, 봇 프레임워크 토큰 서비스
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 05/31/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6eea954f58096d89cd3278058146b93fa04f435f
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693763"
---
# <a name="user-authentication-within-a-conversation"></a>대화 내에서 사용자 인증

사용자 대신 이메일 확인, 캘린더 참조, 항공편 상태 확인 또는 주문하기와 같은 특정 작업을 수행하기 위해, 봇은 Microsoft Graph, GitHub 또는 회사의 REST 서비스와 같은 외부 서비스를 호출해야 합니다.
외부 서비스마다 호출에 보안을 적용하는 방법이 있으며 호출을 보호하는 일반적인 방법은 외부 서비스에서 사용자를 고유하게 식별하는 _사용자 토큰_을 사용하여 요청을 발행하는 것입니다(JWT라고도 함).

외부 서비스에 대한 호출을 보호하기 위해 봇은 서비스에 대한 사용자의 토큰을 얻을 수 있도록 사용자에게 로그인을 요청해야 합니다.
많은 서비스가 OAuth 또는 OAuth2 프로토콜을 통해 토큰 검색을 지원합니다.
Azure Bot Service는 OAuth 프로토콜과 작동하고 토큰 수명 주기를 관리하는 특수 로그인 카드와 서비스를 제공하며 봇은 이러한 기능을 사용하여 사용자 토큰을 확보할 수 있습니다.

- 봇 구성의 일환으로 Azure의 Azure Bot Service 리소스에 OAuth 연결 설정이 등록됩니다. 

    각 연결 설정에는 사용할 외부 서비스 또는 ID 공급자에 대한 정보와 더불어 유효한 OAuth 클라이언트 ID와 비밀, 사용할 OAuth 범위 및 외부 서비스 또는 ID 공급자에게 필요한 기타 연결 메타데이터가 포함됩니다.

- 봇의 코드에서, OAuth 연결 설정은 사용자 로그인을 지원하고 사용자 토큰을 구하는 데 사용됩니다.

이러한 서비스는 로그인 워크플로와 관련이 있습니다.

![인증 개요](./media/bot-builder-concept-authentication.png)

## <a name="about-the-bot-framework-token-service"></a>봇 프레임워크 토큰 서비스 정보

봇 프레임워크 토큰 서비스는 다음 사항을 처리합니다.

- 다양한 외부 서비스에 OAuth 프로토콜을 사용할 수 있도록 합니다.
- 특정 봇, 채널, 대화 및 사용자에 대한 토큰을 안전하게 저장할 수 있도록 합니다.
- 토큰 새로 고침 시도를 포함하여 토큰 수명 주기를 관리합니다.

예를 들어 Microsoft Graph API를 사용하여 사용자의 최근 이메일을 확인할 수 있는 봇에는 Azure Active Directory 사용자 토큰이 필요합니다. 설계 시, 봇 개발자는 (Azure Portal을 통해) Bot Framework Token Service에 Azure Active Directory 애플리케이션을 등록한 후 봇에 대한 OAuth 연결 설정(이름: `GraphConnection`)을 구성합니다. 사용자가 봇과 상호 작용할 때 워크플로는 다음과 같습니다.

1. 사용자가 봇에게 "내 이메일을 확인해주세요"라고 요청합니다.
1. 이 메시지가 포함된 작업이 사용자로부터 봇 프레임워크 채널 서비스로 전송됩니다. 채널 서비스는 작업 내 `userid` 필드가 설정되고 메시지가 봇에 전송되도록 합니다.

    사용자 ID는 Facebook ID나 SMS 전화 번호처럼 채널마다 다릅니다.

1. 봇이 메시지 작업을 수신하여 사용자의 이메일을 확인하려는 의도를 판단합니다. 봇은 OAuth 연결 설정 `GraphConnection`의 UserId에 대한 토큰이 이미 있는지 묻는 요청을 봇 프레임워크 토큰 서비스에 보냅니다.
1. 사용자가 봇과 상호 작용하는 것은 이번이 처음이라서 이 사용자에 대한 토큰이 봇 프레임워크 토큰 서비스에 아직 없기 때문에 봇에 _NotFound_ 결과가 반환됩니다.
1. 봇은 `GraphConnection`이라는 연결 이름으로 OAuthCard를 생성하고 이 카드를 사용하여 사용자에게 회신하면서 로그인하라고 요청합니다.
1. 작업이 봇 프레임워크 토큰 서비스를 통해 전달되고 봇 프레임워크 토큰 서비스를 호출하여 이 요청에 대해 유효한 OAuth 로그인 URL을 만듭니다. 로그인 URL이 OAuthCard에 추가되고 카드가 사용자에게 반환됩니다.
1. OAuthCard의 로그인 단추를 클릭하여 로그인하라는 메시지가 사용자에게 표시됩니다.
1. 사용자가 로그인 단추를 클릭하면 채널 서비스가 웹 브라우저를 열고 외부 서비스로 호출하여 로그인 페이지를 로드합니다.
1. 외부 서비스에 대한 페이지에 사용자가 로그인합니다. 완료되면, 외부 서비스가 봇 프레임워크 토큰 서비스와 OAuth 프로토콜 교환을 완료하고 외부 서비스가 봇 프레임워크 토큰 서비스에 사용자 토큰을 보냅니다. 봇 프레임워크 토큰 서비스는 이 토큰을 안전하게 저장하고 봇에 이 토큰과 함께 작업을 보냅니다.
1. 봇이 토큰과 함께 작업을 수신하면 이것을 사용하여 Graph API에 대한 호출을 수행할 수 있습니다.

## <a name="securing-the-sign-in-url"></a>로그인 URL 보안 설정

봇 프레임워크가 사용자 로그인을 지원할 때 중요하게 고려할 사항은 로그인 URL을 보호하는 방법입니다. 사용자에게 로그인 URL이 제공될 때, 이 URL은 해당 봇의 사용자 ID 및 특정 대화 ID와 연결됩니다. 이 URL은 공유하면 안 됩니다. 그러면 특정 봇 대화에 대해 잘못된 로그인을 발생시킬 수 있기 때문입니다. 로그인 URL 공유와 관련된 보안 공격을 완화하기 위해서는, 로그인 URL을 클릭하는 사람과 머신이 대화 창을 소유(_own_)하는 사람인지 확인해야 합니다.

Cortana, Teams, Direct Line 및 WebChat과 같은 일부 채널은 사용자 모르게 이런 작업을 수행할 수 있습니다. 예를 들어 WebChat은 세션 쿠키를 사용하여 로그인 흐름이 WebChat 대화와 동일한 브라우저에서 수행되는지 확인합니다. 하지만 다른 채널의 경우 사용자에게 6자리의 매직 코드(_magic code_)가 표시되는 경우가 많습니다. 이것은 기본 제공되는 다단계 인증과 유사합니다. 로그인한 사람이 6자로 코드를 입력하여 채팅 환경에 액세스할 수 있다는 것을 증명하는 최종 인증을 사용자가 완료하지 않으면 봇 프레임워크 토큰 서비스가 봇에 토큰을 공개하지 않기 때문입니다.

## <a name="azure-activity-directory-application-registration"></a>Azure Activity Directory 애플리케이션 등록

Azure Bot Service로 등록된 모든 봇은 Azure AD(Activity Directory) 애플리케이션 ID를 사용합니다. 사용자 로그인에 이 애플리케이션 ID와 암호를 다시 사용하지 **않는** 것이 중요합니다. Azure Bot Service의 Azure AD 애플리케이션 ID는 봇과 봇 프레임워크 채널 서비스 간의 통신을 지원하는 서비스를 보호하는 데 사용되어야 합니다 사용자가 Azure AD에 로그인되도록 하려면, 적절한 권한과 범위로 별도의 Azure AD 애플리케이션 등록을 생성해야 합니다.

## <a name="configure-an-oauth-connection-setting"></a>OAuth 연결 설정 구성

OAuth 연결 설정을 등록하고 사용하는 방법에 대한 자세한 내용은 [봇에 인증 추가](bot-builder-authentication.md)를 참조하세요.
