---
title: GroupMe에 봇 연결 | Microsoft Docs
description: GroupMe에 대한 봇 연결을 구성하는 방법을 알아봅니다.
keywords: 봇 채널, GroupMe, GroupMe 만들기, 자격 증명
author: RobStand
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: a2004293ff10cfbc7132f58b7c0c834a2012cfd1
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563567"
---
# <a name="connect-a-bot-to-groupme"></a>GroupMe에 봇 연결

GroupMe 그룹 메시징 앱을 사용하여 사용자와 통신하도록 봇을 구성할 수 있습니다.

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

## <a name="sign-up-for-a-groupme-account"></a>GroupMe 계정에 등록

GroupMe 계정이 없는 경우 [새 계정을 등록](https://web.groupme.com/signup)합니다.

## <a name="create-a-groupme-application"></a>GroupMe 애플리케이션 만들기

봇에 대한 [GroupMe 애플리케이션을 만듭니다](https://dev.groupme.com/applications/new).

콜백 URL `https://groupme.botframework.com/Home/Login`을 사용합니다.

![앱 만들기](~/media/channels/GM-StepApp.png)

## <a name="gather-credentials"></a>자격 증명 수집

1. **리디렉션 URL** 필드에서 **client_id=** 다음의 값을 복사합니다.
2. **액세스 토큰** 값을 복사합니다.

![클라이언트 ID 및 액세스 토큰 복사](~/media/channels/GM-StepClientId.png)


## <a name="submit-credentials"></a>자격 증명 제출

1. dev.botframework.com에서 방금 복사한 **client_id** 값을 **클라이언트 ID** 필드에 붙여넣습니다.
2. **액세스 토큰** 값을 **액세스 토큰** 필드에 붙여넣습니다.
2. **저장**을 클릭합니다.

![자격 증명 입력](~/media/channels/GM-StepClientIDToken.png)
