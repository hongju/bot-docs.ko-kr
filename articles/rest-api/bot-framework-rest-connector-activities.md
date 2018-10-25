---
title: 작업 개요 | Microsoft Docs
description: Bot Connector Service 내에서 사용할 수 있는 다양한 작업 형식에 대해 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: b246e9e07243e4064f92e72ee3909541f642714e
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999930"
---
# <a name="activities-overview"></a>작업 개요

Bot Connector Service는 [작업][Activity] 개체를 전달함으로써 봇과 채널(사용자) 사이에 정보를 교환합니다. 작업의 가장 일반적인 형식은 **메시지**이지만 봇 또는 채널에 다양한 형식의 정보를 전달하는 데 사용할 수 있는 다른 작업 형식이 있습니다. 

## <a name="activity-types-in-the-bot-connector-service"></a>Bot Connector Service의 작업 형식

다음 작업 형식이 Bot Connector Service에서 지원됩니다.

| 활동 유형 | 설명 |
|------|------|------|
| Message | 봇과 사용자 간의 통신을 나타냅니다. |
| conversationUpdate | 봇이 대화에 추가되었거나, 다른 구성원이 대화에서 추가 또는 제거되었거나, 대화 메타데이터가 변경되었음을 나타냅니다. |
| contactRelationUpdate | 봇이 사용자의 연락처 목록에서 추가 또는 제거되었음을 나타냅니다. |
| typing | 대화의 상대편인 사용자 또는 봇이 응답을 컴파일하는 중임을 나타냅니다. | 
| deleteUserData | 사용자가 봇이 저장했을 수도 있는 모든 사용자 데이터를 봇에게 삭제하도록 요청했음을 봇에게 나타냅니다. |
| endOfConversation | 대화의 종료를 나타냅니다. |

## <a name="message"></a>Message

봇은 사용자에게 정보를 전달하기 위해 **message** 활동을 전송하고 사용자에게서 **message** 활동을 수신합니다. 일부 메시지는 일반 텍스트일 수 있으며, 다른 메시지는 [미디어 첨부 파일](bot-framework-rest-connector-add-media-attachments.md), [단추 및 카드](bot-framework-rest-connector-add-rich-cards.md) 또는 [채널 관련 데이터](bot-framework-rest-connector-channeldata.md)와 같은 다양한 콘텐츠를 포함할 수도 있습니다. 일반적으로 사용되는 메시지 속성에 대한 자세한 내용은 [메시지 만들기](bot-framework-rest-connector-create-messages.md)를 참조하세요. 메시지를 보내고 받는 방법에 대한 정보는 [메시지 보내기 및 받기](bot-framework-rest-connector-send-and-receive-messages.md)를 참조하세요. 

## <a name="conversationupdate"></a>conversationUpdate

봇이 대화에 추가되거나, 다른 구성원이 대화에서 추가 또는 제거되거나, 대화 메타데이터가 변경될 때마다 봇이 **conversationUpdate** 작업을 수신합니다. 

구성원이 대화에 추가되면 작업의 `addedMembers` 속성은 새 구성원을 식별합니다. 구성원이 대화에서 제거되면 `removedMembers` 속성은 제거된 구성원을 식별합니다. 

> [!TIP]
> 봇이 사용자가 대화에 참여했음을 나타내는 **conversationUpdate** 작업을 수신하는 경우 해당 사용자에게 환영 메시지를 보내 응답하도록 선택할 수 있습니다. 

## <a name="contactrelationupdate"></a>contactRelationUpdate

봇은 사용자의 연락처 목록에 추가 또는 제거될 때마다 **contactRelationUpdate** 활동을 수신합니다. 활동의 `action` 속성의 값(추가 | 제거)은 봇이 사용자의 연락처 목록에서 추가 또는 제거되었는지 여부를 나타냅니다.

## <a name="typing"></a>typing

봇은 사용자가 응답을 입력 중임을 나타내는 **typing** 활동을 수신합니다. 봇이 요청을 수행하거나 응답을 컴파일하기 위해 작동 중임을 사용자에게 표시하는 **typing** 활동을 전송할 수 있습니다. 

## <a name="deleteuserdata"></a>deleteUserData

봇이 이전에 사용자를 위해 보관했던 모든 데이터의 삭제를 사용자가 요청할 경우 봇은 **deleteUserData** 활동을 수신합니다. 봇이 이 형식의 활동을 수신하는 경우 요청한 사용자를 위해 이전에 저장한 PII(개인 식별 정보)를 모두 삭제해야 합니다.

## <a name="endofconversation"></a>endOfConversation 

봇은 사용자가 대화를 종료했음을 나타내기 위해 **endOfConversation** 활동을 수신합니다. 봇은 사용자에게 대화가 끝나감을 나타내기 위해 **endOfConversation** 활동을 전송할 수 있습니다. 

## <a name="additional-resources"></a>추가 리소스

- [메시지 만들기](bot-framework-rest-connector-create-messages.md)
- [메시지 보내기 및 받기](bot-framework-rest-connector-send-and-receive-messages.md)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object