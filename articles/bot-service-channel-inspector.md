---
title: 채널 검사기를 사용하여 봇 기능 미리 보기 | Microsoft Docs
description: 채널 검사기를 사용하여 다양한 Bot Framework 기능이 여러 다른 채널에서 어떻게 보이고 어떻게 작동하는지 알아봅니다.
keyword: bot, preview features, channel, display, Channel Inspector, Text formatting, markdown, XML
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: def3f811704af2e2a22612ba5755eb5dbd2d8044
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301811"
---
# <a name="preview-bot-features-with-the-channel-inspector"></a>채널 검사기를 사용하여 봇 기능 미리 보기

Bot Framework를 사용하면 회전식 보기 또는 목록 형식에 표시되는 텍스트, 단추, 이미지, 서식 있는 카드와 같은 다양한 기능을 사용하여 봇을 만들 수 있습니다. 그러나 각 채널은 궁극적으로 기능이 메시징 클라이언트에 의해 렌더링되는 방식을 제어합니다.

한 기능이 여러 채널에서 지원되지만, 각 채널은 기능을 약간 다른 방식으로 렌더링할 수 있습니다. 메시지에 채널이 기본적으로 지원하지 않는 기능이 포함된 경우, 채널은 메시지 콘텐츠를 텍스트나 정적 이미지로 다운렌더링하여 클라이언트에서 해당 메시지의 모양에 크게 영향을 줄 수 있습니다. 일부 경우에 채널이 특정 기능을 전혀 지원하지 않을 수도 있습니다. 예를 들어, GroupMe 클라이언트는 입력 표시기를 표시할 수 없습니다.

## <a name="the-channel-inspector"></a>채널 검사기

Bot Framework 기능이 여러 다른 채널에서 어떻게 보이고 어떻게 작동하는지를 미리 보여 주기 위해 [채널 검사기][inspector]가 생성되었습니다. 다양한 채널에서 기능이 렌더링되는 방식을 이해하면 통신하는 채널에서 뛰어난 사용자 환경을 제공하도록 봇을 디자인할 수 있습니다. 또한 채널 검사기는 Bot Framework 기능을 배우고 시각적으로 탐색하는 유용한 방법도 제공합니다.

> [!NOTE]
> 서식 있는 카드는 여러 채널에서 일관되게 표시되도록 하기 위한 봇 정보 교환의 개발 표준입니다. 서식 있는 카드에 대한 자세한 내용은 [.NET][netcard] 또는 [Node.js][nodecard] 설명서를 참조하세요.

## <a name="text-formatting"></a>텍스트 서식 지정

텍스트 서식 지정은 문자 메시지를 시각적으로 개선할 수 있습니다. 봇은 지원하는 채널에 대해 **일반** 텍스트 외에, **markdown** 또는 **xml** 서식을 사용하여 문자 메시지를 전송할 수 있습니다. 다음 표에는 **markdown** 및 **xml**에서 가장 일반적으로 사용되는 텍스트 서식 일부가 나와 있습니다. 각 채널은 여기에 나열된 것보다 더 적거나 더 많은 텍스트 서식을 지원할 수 있습니다. [채널 검사기][inspector]를 확인하여 사용하려는 기능이 대상 채널에서 지원되는지를 알아볼 수 있습니다.

### <a name="markdown-text-format"></a>Markdown 텍스트 형식

이러한 스타일은 `textFormat`이 **markdown**으로 설정되어 있을 때 지원될 수 있습니다.  

| Style | Markdown | 예 |
| ---- | ---- | ---- |
| 굵게 | \*\*text\*\* | **text** |
| 기울임꼴 | \*text\* | *text* |
| 머리글(1-5) | # H1 | # H1 |
| 취소선 | \~\~text\~\~ | ~~text~~ |
| 가로줄 | ---- | ---- |
| 순서가 지정되지 않은 목록 | \* text |  <ul><li>text</li></ul> |
| 순서가 지정된 목록 | 1. text | 1. text |
| 미리 서식이 지정된 텍스트 | \`text\` | `text`  |
| 블록 따옴표 | \> text | <blockquote>text</blockquote> |
| 하이퍼링크 | \[bing](http://www.bing.com) | [bing](http://www.bing.com) |
| 이미지 링크| !\[duck](http://aka.ms/Fo983c) | ![duck](http://aka.ms/Fo983c) |

> [!NOTE]
> **Markdown**의 HTML 태그는 Microsoft Bot Framework 웹 채팅 채널에서 지원되지 않습니다. **Markdown**에서 HTML 태그를 사용해야 하는 경우 지원하는 [직접 회선](~/bot-service-channel-connect-directline.md) 채널에서 렌더링할 수 있습니다. 또는 `textFormat`을 **xml**로 설정하여 아래의 HTML 태그를 사용하고 봇을 Skype 채널에 연결할 수 있습니다.

### <a name="xml-text-format"></a>XML 텍스트 형식

이러한 스타일은 `textFormat`이 **xml**로 설정되어 있을 때 지원될 수 있습니다.

|      Style      |                       XML                       |                   예                   |
|-----------------|-------------------------------------------------|---------------------------------------------|
|      굵게       |                 \<b\>text\</b\>                 |            <strong>text</strong>            |
|     기울임꼴      |                 \<i\>text\</i\>                 |                <em>text</em>                |
|    밑줄    |                 \<u\>text\</u\>                 |                 <u>text</u>                 |
|  취소선  |                 \<s\>text\</s\>                 |                 <s>text</s>                 |
|    하이퍼링크    |   \<a href="http://www.bing.com"\>bing\</a\>    |   <a href="http://www.bing.com">bing</a>    |
|    단락    |                 \<p\>text\</p\>                 |                 <p>text</p>                 |
|   줄 바꿈    |                     \<br/\>                     |             line 1 <br/>line 2              |
| 가로줄 |                     \<hr/\>                     |                    <hr/>                    |
|  머리글(1-4)   |                \<h1\>text\</h1\>                |             <h1>제목 1</h1>              |
|      코드       |           \<code\>code block\</code\>           |           <code>code block</code>           |
|      이미지      | \<img src="<http://aka.ms/Fo983c>" alt="Duck"\> | <img src="http://aka.ms/Fo983c" alt="Duck"> |

> [!NOTE]
> `textFormat`**xml**Skype 채널에서만 지원됩니다.

## <a name="preview-features-across-various-channels"></a>다양 한 채널에서 기능 미리 보기

채널이 특정 기능을 렌더링하는 방법을 보려면 [채널 검사기][inspector]로 이동한 후 **채널** 목록에서 채널을 선택하고  **기능** 목록에서 기능을 선택합니다. 예를 들어, Skype가 Hero 카드를 렌더링하는 방법을 보려면 **채널**을 *Skype*로 설정하고, **기능**을 *HeroCard*로 설정합니다.

![Skype 채널과 Hero 카드를 표시하는 채널 검사기](~/media/bot-service-channel-inspector.png)

채널 검사기에는 지정된 채널에 의해 렌더링될 선택된 기능의 미리 보기가 표시됩니다. **참고** 섹션은 메시지 제한 사항 및/또는 표시 변경에 대한 중요 정보를 전달합니다. 예를 들어, 일부 유형의 서식 있는 카드는 하나의 이미지만 지원하고, 일부 기능은 특정 채널에서 다운렌더링될 수 있습니다.

> [!NOTE]
> 채널 검사기는 현재 다음과 같은 채널을 지원합니다.
> * Cortana
> * Email
> * Facebook
> * GroupMe
> * Kik
> * Skype
> * 비즈니스용 Skype
> * Slack
> * sms
> * Microsoft 팀
> * Telegram
> * WeChat
> * 웹 채팅
>
> 추가 채널은 나중에 추가할 수 있습니다.

## <a name="features-that-can-be-previewed"></a>미리 볼 수 있는 기능

현재, 채널 검사기를 사용하면 다음과 같은 기능을 미리 볼 수 있습니다.

|기능 | 설명|
| --- | ----|
| Adaptive Cards | 텍스트, 음성, 이미지, 단추 및 입력 필드의 조합을 포함할 수 있는 카드입니다. |
| Buttons| 사용자가 클릭할 수 있는 단추입니다. 메시지가 속한 대화 캔버스에 단추가 나타납니다. |
| Carousel| 스크롤 가능한 압축된 카드 가로 목록입니다. 세로 레이아웃의 경우는 목록을 사용합니다.|
| ChannelData| 카드, 텍스트 및 첨부 파일 이외의 채널 관련 기능에 액세스하기 위한 메타데이터를 전달하는 방법입니다.|
| Codesample| 컴퓨터 코드를 표시하도록 디자인된 미리 서식이 지정된 여러 줄의 텍스트 블록입니다.|
| DirectMessage| 그룹 대화의 단일 멤버에게 보낸 메시지입니다.
| 문서| 미디어가 아닌 첨부 파일을 보내고 받습니다. |
| Emoji| 지원되는 이모지를 표시합니다.
| GroupChat| 봇이 그룹 대화에 참여하기 위해 메시지를 보냅니다. |
| HeroCard| 일반적으로 설명이 포함된 텍스트 콘텐츠와 함께 하나의 큰 이미지, 탭 작업 및 단추(선택 사항)를 포함하는 서식이 지정된 첨부 파일입니다. |
| 이미지| 이미지 첨부 파일을 표시합니다. |
| List Layout| 카드의 세로 목록입니다. 스크롤 가능한 가로 레이아웃의 경우는 Carousel을 사용합니다.|
| 위치| 사용자의 실제 위치를 봇과 공유합니다. |
| Markdown| Markdown으로 서식이 지정된 텍스트를 렌더링합니다.|
| 멤버| 그룹 채팅 중 봇과의 대화에서 멤버 목록을 공유합니다. |
| Receipt Card| 사용자에게 수신 확인을 표시합니다. |
| Signin Card| 사용자에게 인증 자격 증명을 입력하도록 요구합니다.|
| Suggested Actions | 사용자가 입력하기 위해 탭할 수 있는 단추로 표시되는 작업입니다. |
| Thumbnail Card| 설명이 포함된 텍스트 콘텐츠와 함께 하나의 작은 이미지(미리 보기), 탭 작업 및 단추(선택 사항)를 포함하는 서식이 지정된 첨부 파일입니다. |
| Typing| 입력 표시기를 표시합니다. 이 기능은 봇은 여전히 작동하지만 백그라운드에서 몇 가지 작업을 수행하고 있음을 사용자에게 알리는 데 도움이 됩니다.|
| 비디오| 비디오 첨부 파일 및 재생 컨트롤을 표시합니다.|

## <a name="additional-resources"></a>추가 리소스

* [채널 검사기][inspector]
* [Node.js][nodecard] 및 [.NET][netcard]의 서식 있는 카드
* [Node.js][nodemedia] 및 [.NET][netmedia]의 미디어 첨부 파일
* [Node.js][nodebutton] 및 [.NET][netbutton]의 제안된 작업

[inspector]: https://docs.botframework.com/en-us/channel-inspector/channels/Skype/

[syntax]: https://daringfireball.net/projects/markdown/syntax

[netcard]: ~/dotnet/bot-builder-dotnet-add-rich-card-attachments.md
[nodecard]: ~/nodejs/bot-builder-nodejs-send-rich-cards.md

[netmedia]: ~/dotnet/bot-builder-dotnet-add-media-attachments.md
[nodemedia]: ~/nodejs/bot-builder-nodejs-send-receive-attachments.md

[netbutton]: ~/dotnet/bot-builder-dotnet-add-suggested-actions.md
[nodebutton]: ~/nodejs/bot-builder-nodejs-send-suggested-actions.md
