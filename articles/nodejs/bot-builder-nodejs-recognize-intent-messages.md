---
title: 메시지 콘텐츠에서 의도 인식 | Microsoft Docs
description: 정규식을 사용하거나 메시지 콘텐츠를 확인하여 사용자의 의도를 인식하는 방법에 대해 알아봅니다.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 82541c4ac0848c3e995ab3ad1ed874436072fe63
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998092"
---
# <a name="recognize-user-intent-from-message-content"></a>메시지 콘텐츠에서 사용자 의도 인식

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

봇이 사용자로부터 메시지를 받으면 봇은 **인식기**를 사용하여 메시지를 검사하고 의도를 확인합니다. 의도는 메시지에서 호출할 대화 상자로 매핑을 제공합니다. 이 문서에서는 정규식을 사용하거나 메시지의 콘텐츠를 검사하여 의도를 인식하는 방법을 설명합니다. 예를 들어, 봇은 정규식을 사용하여 메시지에 단어 “help”가 포함되는지 검사하고, 도움말 대화 상자를 호출할 수 있습니다. 또한 봇은 사용자 메시지의 속성을 검사할 수 있습니다. 예를 들어 사용자가 텍스트 대신 이미지를 전송했는지를 확인하고 이미지 처리 대화 상자를 호출합니다. 

> [!NOTE]
> LUIS를 사용하여 의도를 인식하는 데 관한 정보는 [LUIS를 통한 의도 및 엔터티 인식](bot-builder-nodejs-recognize-intent-luis.md)을 참조하세요. 


## <a name="use-the-built-in-regular-expression-recognizer"></a>기본 제공 정규식 인식자 사용
정규식을 사용하여 사용자의 의도를 검색하려면 [RegExpRecognizer][RegExpRecognizer]를 사용합니다. 여러 언어를 지원하는 인식기에 여러 개의 식을 전달할 수 있습니다. 

> [!TIP]
> [지역화 지원](bot-builder-nodejs-localization.md)에서 봇을 지역화하는 데 관한 자세한 내용을 확인하세요.

다음 코드는 `CancelIntent`라는 정규식 인식자를 만들고 봇에 추가합니다. 이 예에서 인식기는 `en_us` 및 `ja_jp` 로캘 모두에 대한 정규식을 제공합니다. 

[!code-js[Add a regular expression recognizer (JavaScript)](../includes/code/node-regex-recognizer.js#addRegexRecognizer)]

인식기가 봇에 추가되면 인식기가 의도를 검색할 때 봇이 호출하도록 하려는 대화 상자에 [triggerAction][triggerAction]을 연결합니다. [일치][matches] 옵션을 사용하여 다음 코드에 표시된 대로 의도 이름을 지정합니다.

[!code-js[Map the CancelIntent recognizer to a cancel dialog (JavaScript)](../includes/code/node-regex-recognizer.js#bindCancelDialogToRegexRecognizer)]

의도 인식기는 글로벌입니다. 즉, 인식기는 사용자로부터 받은 모든 메시지에 대해 실행됩니다. 인식기가 `triggerAction`을 사용하여 대화 상자로 바인딩되는 의도를 검색하는 경우 현재 활성 대화 상자의 중단을 트리거할 수 있습니다. 중단을 허용하고 처리하는 것이 사용자가 실제로 수행하는 일을 설명하는 유연한 디자인입니다.

> [!TIP] 
> 대화 상자를 사용한 `triggerAction`의 작업 방식에 대해 알아보려면 [대화 흐름 관리](bot-builder-nodejs-manage-conversation-flow.md)를 참조하세요. 인식된 의도로 연결할 수 있는 다양한 작업에 자세히 알아보려면 [사용자 작업 처리](bot-builder-nodejs-dialog-actions.md)를 참조하세요.

## <a name="register-a-custom-intent-recognizer"></a>사용자 지정 의도 인식기 등록
또한 사용자 지정 인식기를 구현할 수 있습니다. 이 예제에서는 사용자가 ‘help’ 또는 ‘goodbye’라고 말하길 바라는 간단한 인식기를 추가하지만, 사용자가 이미지를 보낼 때 인식하는 것 같이 더 복잡한 처리를 수행하는 인식기를 손쉽게 추가할 수 있습니다. 


[!code-js[Add a custom recognizer (JavaScript)](../includes/code/node-howto-recognize-intent.js#addCustomRecognizer)]

인식기를 등록한 후 `matches` 절을 사용하여 인식기를 작업과 연결할 수 있습니다.

[!code-js[Bind intents to actions (JavaScript)](../includes/code/node-howto-recognize-intent.js#bindIntentsToActions)]

## <a name="disambiguate-between-multiple-intents"></a>여러 의도를 명확히 구분

봇은 둘 이상의 인식기를 등록할 수 있습니다. 사용자 지정 인식기 예제에는 각 의도에 숫자 점수를 할당하는 것이 포함됩니다. 이는 봇에 둘 이상의 인식기가 있을 수도 있기 때문에 수행되며, Bot Builder SDK는 여러 인식기에서 반환되는 의도들을 명확히 구분하는 기본 제공 논리를 제공합니다. 의도에 할당된 점수는 일반적으로 0.0과 1.0 사이지만, 의도가 항상 Bot Builder SDK의 명확성 논리에 의해 선택되도록 사용자 지정 인식기가 1.1보다 큰 수를 의도에 할당할 수도 있습니다. 

기본적으로 인식기는 병렬로 실행되나, [IIntentRecognizerSetOptions][IntentRecognizerSetOptions]에서 recognizeOrder를 1.0의 점수를 주는 인식기를 발견하는 즉시 프로세스가 종료되도록 설정할 수 있습니다.

Bot Builder SDK에는 [IDisambiguateRouteHandler][IDisambiguateRouteHandler] 구현으로 봇에 사용자 지정 명확성 논리를 제공하는 방법을 설명하는 [샘플][DisambiguationSample]이 포함되어 있습니다.

## <a name="next-steps"></a>다음 단계
정규식을 사용하고 메시지 콘텐츠를 검사하기 위한 논리는 특히 봇의 대화형 흐름이 개방형일 경우 복잡해질 수 있습니다. 봇이 사용자의 텍스트 및 음성 입력을 더 광범위하게 처리하도록 하려면 [LUIS][LUIS]와 같은 의도 인식 서비스를 사용하여 자연어 인식을 봇에 추가할 수 있습니다.

> [!div class="nextstepaction"]
> [LUIS를 사용한 의도 및 엔터티 인식](bot-builder-nodejs-recognize-intent-luis.md)


[LUIS]: https://www.luis.ai/

[triggerAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction

[matches]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions.html#matches

[node-js-bot-how-to]: bot-builder-nodejs-recognize-intent-luis.md

[LUISAzureDocs]: /azure/cognitive-services/LUIS/Home

[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[IntentRecognizerSetOptions]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iintentrecognizersetoptions.html

[LuisRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer

[LUISSample]: https://aka.ms/v3-js-luisSample

[LUISConcepts]: https://docs.botframework.com/en-us/node/builder/guides/understanding-natural-language/

[DisambiguationSample]: https://aka.ms/v3-js-onDisambiguateRoute

[IDisambiguateRouteHandler]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.idisambiguateroutehandler.html

[RegExpRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.regexprecognizer.html

[AlarmBot]: https://aka.ms/v3-js-luisSample
