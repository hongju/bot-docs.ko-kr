---
title: 적응형 카드 구성 | Microsoft Docs
description: 적응형 카드를 구성하는 방법을 알아봅니다.
author: vkannan
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: d41c2c24ed38fffe76cd73a6bb8a685d3861ac55
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000430"
---
# <a name="configure-adaptive-cards"></a>적응형 카드 구성
> [!IMPORTANT]
> 대화 디자이너는 아직 모든 고객에게 제공되지 않습니다. 대화 디자이너의 가용성에 대한 자세한 내용은 올해 하반기에 제공됩니다.

<a href="http://adaptivecards.io" target="_blank">적응형 카드</a>는 Microsoft Bot Framework 채널을 비롯한 여러 가지 다른 엔드포인트에서 사용할 수 있는 풍부한 UI 카드를 정의하는 새로운 스키마입니다. 

대화 디자이너는 봇의 적응형 카드를 작성, 미리 보기 및 사용하기 위해 심층적으로 통합된 작성 환경을 제공합니다. 

적응형 카드는 여러 가지 다른 주요 위치에서 정의할 수 있습니다.

- 작업의 동작에 대한 간단한 응답.
- 대화의 피드백 상태.
- 대화의 프롬프트 상태. 프롬프트에는 별도의 카드가 있을 수 있습니다. 하나는 응답용이고 다른 하나는 재 프롬프트용입니다.

적응형 카드를 정의하려면 관련 편집기로 이동합니다. 기존 적응형 카드 템플릿 중 하나를 찾아서 선택하거나 JSON 코드편집기에서 직접 작성합니다. 

카드를 작성하는 대로 카드의 다양한 미리 보기가 작성하는 포털에 렌더링됩니다.

> [!NOTE]
> 적응형 카드의 기능에 대한 개발은 아직 진행 중입니다. 현재는 모든 채널에서 일부 적응형 카드 기능을 지원하지 않습니다. 각 채널이 지원하는 채널을 보려면 채널 상태 섹션을 참조하세요.

## <a name="input-form"></a>입력 양식

적응형 카드는 입력 양식을 포함할 수 있습니다. 대화 디자이너에서 양식은 작업 엔터티와 통합되어 있습니다. 예를 들어 필드에 **myName**의 `id`가 있고 양식 `Submit` 동작이 수행되면 이름이 **myName**인 `taskEntity`가 만들어지고 필드의 값을 포함하게 됩니다. 

아래 코드 조각은 **myName** 엔터티가 코드로 정의되는 방식을 보여줍니다.

``javascript
{
   "type": "Input.Text",
   "id": "myName",
   "placeholder": "Last, First"
}
``

또한 필드에 `@task`의 ID가 있으면 필드의 값은 작업 이름으로 사용됩니다. 이 필드가 트리거(예: 단추 클릭)될 때 명명된 작업이 실행됩니다. 

예를 들어 다음 코드 조각을 참조하세요.

``javascript
{
  'type': 'Action.Submit',
  'title': 'Search',
  'speak': '<s>Search</s>',
  'data': {
    '@task': 'Hotel Search'
  }
}
``

이 단추를 클릭하면 제출 동작이 트리거되고 `context.sticky`가 `Hotel Search`로 설정됩니다. 이로 인해 **Hotel Search** 작업이 실행됩니다. 이 기능을 사용하려면 `@task`가 대화 디자이너에서 정의한 작업 이름과 일치해야 합니다.

## <a name="use-entities-and-language-generation-templates"></a>엔터티 및 언어 생성 템플릿 사용
적응형(adaptive) 카드는 전체 언어 생성 확인을 지원합니다.

* `entityName`은 카드 내 엔터티를 사용합니다.
* `responseTemplateName`은 카드 내 간단한 또는 조건부 응답 템플릿을 사용합니다.

적응형 카드에 대한 자세한 내용은 여기에서 확인할 수 있습니다. TODO: 적응형 카드 스키마 설명서에 대한 링크 삽입 -->

## <a name="sample-adaptive-card-payload"></a>간단한 적응형(adaptive) 카드 페이로드

다음 JSON은 적응형 카드의 페이로드를 보여줍니다.

```json
{
    "$schema": "https://microsoft.github.io/AdaptiveCards/schemas/adaptive-card.json",
    "type": "AdaptiveCard",
    "version": "1.0",
    "body": [
        {
            "speak": "<s>Serious Pie is a Pizza restaurant which is rated 9.3 by customers.</s>",
            "type": "ColumnSet",
            "columns": [
                {
                    "type": "Column",
                    "size": "2",
                    "items": [
                        {
                            "type": "TextBlock",
                            "text": "[Greeting], [TimeOfDayTemplate], You can eat in {location}",
                            "weight": "bolder",
                            "size": "extraLarge"
                        },
                        {
                            "type": "TextBlock",
                            "text": "9.3 · $$ · Pizza",
                            "isSubtle": true
                        },
                        {
                            "type": "TextBlock",
                            "text": "[builtin.feedback.display]",
                            "wrap": true
                        }
                    ]
                },
                {
                    "type": "Column",
                    "size": "1",
                    "items": [
                        {
                            "type": "Image",
                            "url": "http://res.cloudinary.com/sagacity/image/upload/c_crop,h_670,w_635,x_0,y_0/c_scale,w_640/v1397425743/Untitled-4_lviznp.jpg",
                            "size":"auto"
                        }
                    ]
                }
            ]
        }
    ],
    "actions": [
        {
            "type": "Action.Http",
            "method": "POST",
            "title": "More Info",
            "url": "http://foo.com"
        },
        {
            "type": "Action.Http",
            "method": "POST",
            "title": "View on Foursquare",
            "url": "http://foo.com"
        }
    ]
}
```

