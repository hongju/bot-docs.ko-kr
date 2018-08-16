---
title: 활동 개요 | Microsoft Docs
description: .NET용 Bot Builder SDK 내에서 사용할 수 있는 다양한 활동 유형에 대해 알아봅니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 90e53ed5001ce1c91646644bf815bb51b6a843c1
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39303818"
---
# <a name="activities-overview"></a>활동 개요

[!INCLUDE [Activity concept overview](../includes/snippet-dotnet-concept-activity.md)]

## <a name="activity-types-in-the-bot-builder-sdk-for-net"></a>.NET용 Bot Builder SDK의 활동 유형

.NET용 Bot Builder SDK에서 다음 활동 유형이 지원됩니다.

| Activity.Type | 인터페이스 | 설명 |
|------|------|------|
| [message](#message) | IMessageActivity | 봇과 사용자 간의 통신을 나타냅니다. |
| [conversationUpdate](#conversationupdate) | IConversationUpdateActivity | 봇이 대화에 추가되었거나, 다른 구성원이 대화에서 추가 또는 제거되었거나, 대화 메타데이터가 변경되었음을 나타냅니다. |
| [contactRelationUpdate](#contactrelationupdate) | IContactRelationUpdateActivity | 봇이 사용자의 연락처 목록에서 추가 또는 제거되었음을 나타냅니다. |
| [typing](#typing) | ITypingActivity | 대화의 상대편인 사용자 또는 봇이 응답을 컴파일하는 중임을 나타냅니다. | 
| [ping](#ping) | 해당 없음 | 봇의 엔드포인트에 액세스할 수 있는지 여부를 확인하려는 시도를 나타냅니다. | 
| [deleteUserData](#deleteuserdata) | 해당 없음 | 사용자가 봇이 저장했을 수도 있는 모든 사용자 데이터를 봇에게 삭제하도록 요청했음을 나타냅니다. |
| [endOfConversation](#endofconversation) | IEndOfConversationActivity | 대화의 종료를 나타냅니다. |
| [event](#event) | IEventActivity | 사용자에게 표시되지 않는 봇에게 전송한 통신을 나타냅니다. |
| [invoke](#invoke) | IInvokeActivity | 특정 작업을 수행하도록 요청하는 봇에 전송된 통신을 나타냅니다. 이 활동 유형은 Microsoft Bot Framework에 의해 내부 용도로 예약됐습니다. |
| [messageReaction](#messagereaction) | IMessageReactionActivity | 사용자가 기존 활동에 대응했음을 나타냅니다. 예를 들어 사용자가 메시지에 "좋아요" 단추를 클릭합니다. |

## <a name="message"></a>Message

봇은 사용자에게 정보를 전달하기 위해 **message** 활동을 전송하고 사용자에게서 **message** 활동을 수신합니다. 일부 메시지는 단순히 일반 텍스트로 구성되는 반면 다른 메시지에는 [음성 텍스트](bot-builder-dotnet-text-to-speech.md), [제안된 동작](bot-builder-dotnet-add-suggested-actions.md), [미디어 첨부 파일](bot-builder-dotnet-add-media-attachments.md), [리치(rich) 카드](bot-builder-dotnet-add-rich-card-attachments.md) 및 [채널 관련 데이터](bot-builder-dotnet-channeldata.md)와 같은 다양한 콘텐츠가 포함될 수 있습니다. 일반적으로 사용되는 메시지 속성에 대한 자세한 내용은 [메시지 만들기](bot-builder-dotnet-create-messages.md)를 참조하세요.

## <a name="conversationupdate"></a>conversationUpdate

봇이 대화에 추가되거나, 다른 멤버가 대화에 추가 또는 제거되거나, 대화 메타데이터가 변경될 때마다 봇은 **conversationUpdate** 활동을 수신합니다. 

대화에 멤버를 추가한 경우 활동의 `MembersAdded` 속성에는 새 멤버를 식별하기 위한 `ChannelAccount` 개체의 배열이 포함됩니다. 

봇을 대화에 추가했는지 여부(예: 새 멤버 중 하나)를 확인하려면 활동에 대한 `Recipient.Id` 값(예: 봇의 ID)이 `MembersAdded` 배열에서 계정에 대한 `Id` 속성과 일치하는지 여부를 평가합니다.

대화에서 멤버를 제거한 경우 `MembersRemoved` 속성에는 제거된 멤버를 식별하기 위한 `ChannelAccount` 개체의 배열이 포함됩니다. 

> [!TIP]
> 봇이 사용자가 대화에 조인했음을 나타내는 **conversationUpdate** 활동을 받는 경우 해당 사용자에게 환영 메시지를 보내 응답하도록 선택할 수 있습니다. 

## <a name="contactrelationupdate"></a>contactRelationUpdate

봇은 사용자의 연락처 목록에 추가 또는 제거될 때마다 **contactRelationUpdate** 활동을 수신합니다. 활동의 `Action` 속성의 값(추가 | 제거)은 봇이 사용자의 연락처 목록에서 추가 또는 제거되었는지 여부를 나타냅니다.

## <a name="typing"></a>typing

봇은 사용자가 응답을 입력 중임을 나타내는 **typing** 활동을 수신합니다. 봇이 요청을 수행하거나 응답을 컴파일하기 위해 작동 중임을 사용자에게 표시하는 **typing** 활동을 전송할 수 있습니다. 

## <a name="ping"></a>ping

봇은 해당 엔드포인트에 액세스할 수 있는지 여부를 결정하는 **ping** 활동을 수신합니다. 봇은 HTTP 상태 코드 200(정상), 403(금지됨) 또는 401(권한 없음)을 사용하여 응답해야 합니다.

## <a name="deleteuserdata"></a>deleteUserData

봇이 이전에 사용자를 위해 보관했던 모든 데이터의 삭제를 사용자가 요청할 경우 봇은 **deleteUserData** 활동을 수신합니다. 봇이 이 형식의 활동을 수신하는 경우 요청한 사용자를 위해 이전에 저장한 PII(개인 식별 정보)를 모두 삭제해야 합니다.

## <a name="endofconversation"></a>endOfConversation 

봇은 사용자가 대화를 종료했음을 나타내기 위해 **endOfConversation** 활동을 수신합니다. 봇은 사용자에게 대화가 끝나감을 나타내기 위해 **endOfConversation** 활동을 전송할 수 있습니다. 

## <a name="event"></a>event

봇은 사용자에게 정보를 표시하지 않으면서 봇에 정보를 전달하려는 외부 프로세스 또는 서비스로부터 **event** 활동을 받을 수 있습니다. 일반적으로 **event** 활동의 발송자는 봇이 어떤 방식으로든 수신을 승인할 것으로 기대하지 않습니다.

## <a name="invoke"></a>호출

봇은 특정 작업을 수행하기 위한 요청을 나타내는 **invoke** 활동을 받을 수 있습니다. **invoke** 활동의 발송자는 일반적으로 봇이 HTTP 응답을 통해 수신을 승인할 것으로 기대합니다. 이 활동 유형은 Microsoft Bot Framework에 의해 내부 용도로 예약됐습니다.

## <a name="messagereaction"></a>messageReaction

사용자가 기존 작업에 반응하는 경우 일부 채널은 봇에게 **messageReaction** 활동을 보냅니다. 예를 들어 사용자가 메시지에 "좋아요" 단추를 클릭합니다. **ReplyToId** 속성은 사용자가 반응한 활동을 나타냅니다.

**messageReaction** 활동은 채널이 정의한 많은 **messageReactionTypes**와 부합할 수 있습니다. 예를 들어 채널이 보낼 수 있는 반응 유형에는 "좋아요" 또는 "PlusOne"이 있습니다. 

## <a name="additional-resources"></a>추가 리소스

- [활동 보내기 및 받기](bot-builder-dotnet-connector.md)
- [메시지 만들기](bot-builder-dotnet-create-messages.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">활동 클래스</a>
