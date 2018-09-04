---
title: 앱에 봇 포함 | Microsoft Docs
description: 다른 앱 내에 포함되는 봇을 디자인하는 방법을 알아봅니다.
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 08/15/2018
ms.openlocfilehash: 68d2d4f7a19aecfcb2c630e5e9e55ca5b8a21d89
ms.sourcegitcommit: e8c513d3af5f0c514cadcbcd0a737a7393405afa
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/20/2018
ms.locfileid: "42756532"
---
# <a name="embed-a-bot-in-an-app"></a>앱에 봇 포함

봇은 앱 외부에서 가장 일반적으로 존재하지만 앱에 통합될 수도 있습니다. 예를 들어, 앱 내에 [기술 봇](~/bot-service-design-pattern-knowledge-base.md)을 포함하여 복잡한 앱 구조 내에서 찾기 어려울 수 있는 정보를 빠르게 찾도록 도와줄 수 있습니다. 지원 센터 앱 내에 봇을 포함하여 들어오는 사용자 요청에 대한 첫 번째 응답자 역할을 하도록 할 수 있습니다. 봇은 간단한 문제는 독립적으로 해결하고 좀 더 복잡한 문제는 상담원에게 [전달](~/bot-service-design-pattern-handoff-human.md)할 수 있습니다. 

## <a name="integrating-bot-with-app"></a>앱에 봇 통합

앱에 봇을 통합하는 방법은 앱 유형에 따라 달라집니다. 

### <a name="native-mobile-app"></a>네이티브 모바일 앱

네이티브 코드로 작성된 앱은 REST 또는 websocket을 통해 [직접 회선 API][directLineAPI]을 사용하여 Bot Framework와 통신할 수 있습니다.

### <a name="web-based-mobile-app"></a>웹 기반 모바일 앱

<a href="https://cordova.apache.org/" target="_blank">Cordova</a>와 같이 웹 언어 및 프레임워크를 사용하여 빌드된 모바일 앱은 [웹 사이트 내에 포함된 봇](~/bot-service-design-pattern-embed-web-site.md)이 사용하는 동일한 구성 요소(네이티브 앱의 셸 내에 캡슐화된)를 사용하여 Bot Framework와 통신할 수 있습니다.

### <a name="iot-app"></a>IoT 앱

IoT 앱은 [직접 회선 API][directLineAPI]를 사용하여 Bot Framework와 통신할 수 있습니다. 일부 시나리오에서는 <a href="https://www.microsoft.com/cognitive-services/" target="_blank">Microsoft Cognitive Services</a>를 사용하여 이미지 인식 및 음성과 같은 기능을 사용하도록 설정할 수도 있습니다.

### <a name="other-types-of-apps-and-games"></a>다른 유형의 앱 및 게임

다른 유형의 앱 및 게임은 [직접 회선 API][directLineAPI]를 사용하여 Bot Framework와 통신할 수 있습니다. 

## <a name="creating-a-cross-platform-mobile-app-that-runs-a-bot"></a>봇을 실행하는 플랫폼 간 모바일 앱 만들기

봇을 실행하는 모바일 앱을 만드는 이 예제에서는 플랫폼 간 모바일 응용 프로그램을 빌드하기 위한 인기 있는 도구인 <a href="https://www.xamarin.com/" target="_blank">Xamarin</a>을 사용합니다. 

먼저 간단한 웹 보기 구성 요소를 만들고 <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">웹 채팅 컨트롤</a>을 호스트하는 데 사용합니다. 그런 다음, Azure Portal을 사용하여 웹 채팅 채널을 추가합니다. 

다음으로, 등록된 웹 채팅 URL을 Xamarin 앱의 웹 보기 컨트롤에 대한 원본으로 지정합니다.

```cs
public class WebPage : ContentPage
{
    public WebPage()
    {
        var browser = new WebView();
        browser.Source = "https://webchat.botframework.com/embed/<YOUR SECRET KEY HERE>";
        this.Content = browser;
    }
}
```

이 프로세스에서는 웹 채팅 컨트롤을 사용하여 포함된 웹 보기를 렌더링하는 플랫폼 간 모바일 응용 프로그램을 만들 수 있습니다.

![백채널](~/media/bot-service-design-pattern-embed-app/xamarin-apps.png)

## <a name="sample-code"></a>샘플 코드

봇을 실행하는 플랫폼 간 모바일 앱을 만드는 방법을 보여 주는 전체 샘플(이 문서에 설명됨)에 대해서는 GitHub의 <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/capability-BotInApps" target="_blank">앱 내 봇 샘플</a>을 참조하세요.

## <a name="additional-resources"></a>추가 리소스

- [직접 회선 API][directLineAPI]
- <a href="https://www.microsoft.com/cognitive-services/" target="_blank">Microsoft Cognitive Services</a>

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle
