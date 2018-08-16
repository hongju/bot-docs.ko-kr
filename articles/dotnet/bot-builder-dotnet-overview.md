---
title: .NET용 Bot Builder SDK | Microsoft Docs
description: 봇을 빌드하기 위한 강력하고 편리한 프레임워크인 .NET용 Bot Builder SDK를 시작합니다.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: bd89017714a46c541468ccc9ed9990dde14719b7
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301842"
---
# <a name="bot-builder-sdk-for-net"></a>.NET용 Bot Builder SDK
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-overview.md)
> - [Node.js](../nodejs/bot-builder-nodejs-overview.md)
> - [REST](../rest-api/bot-framework-rest-overview.md)

.NET용 Bot Builder SDK는 자유 형식 상호 작용 및 사용자가 가능한 값에서 선택하는 자세한 단계별 대화를 둘 다 처리할 수 있는 봇을 생성하기 위한 강력한 프레임워크입니다. 이 SDK는 사용하기 쉬우며 C#을 사용하여 .NET 개발자가 봇을 작성할 수 있는 친숙한 방법을 제공합니다.

SDK를 사용하여 다음 SDK 기능을 활용하는 봇을 빌드할 수 있습니다. 

- 격리되고 구성 가능한 다이얼로그를 사용하는 강력한 다이얼로그 시스템
- 예/아니요, 문자열, 숫자 및 열거형과 같은 간단한 항목에 대한 기본 제공 프롬프트
- <a href="http://luis.ai" target="_blank">LUIS</a>와 같은 강력한 AI 프레임워크를 활용하는 기본 제공 다이얼로그
- 필요에 따라 도움말, 탐색, 설명 및 확인을 제공하여 대화를 통해 사용자를 안내하는 봇을 (C# 클래스에서) 자동으로 생성하기 위한 FormFlow

> [!IMPORTANT]
> 2017년 7월 31일에 Bot Framework 보안 프로토콜에서는 획기적인 변경이 구현되었습니다. 이러한 변경 내용이 봇에 부정적인 영향을 미치지 않도록 하려면 응용 프로그램이 Bot Builder SDK v3.5 이상을 사용하는지 확인해야 합니다. 2017년 1월 5일(Bot Builder SDK v3.5 릴리스 날짜) 이전에 획득한 SDK를 사용하여 봇을 빌드한 경우 Bot Builder SDK v3.5 이상으로 업그레이드해야 합니다.

## <a name="get-the-sdk"></a>SDK 가져오기

SDK는 NuGet 패키지로, 그리고 <a href="https://github.com/Microsoft/BotBuilder" target="_blank">GitHub</a>에서 오픈 소스로 사용할 수 있습니다.

> [!IMPORTANT]
> .NET용 Bot Builder SDK에는 .NET Framework 4.6 이상이 필요합니다. 하위 버전의 .NET Framework를 대상으로 하는 기존 프로젝트에 SDK를 추가하는 경우 먼저 .NET Framework 4.6을 대상으로 지정하도록 프로젝트를 업데이트해야 합니다.

Visual Studio 프로젝트 내에서 SDK를 설치하려면 다음 단계를 수행합니다.

1. **솔루션 탐색기**에서 프로젝트 이름을 마우스 오른쪽 단추로 클릭하고 **NuGet 패키지 관리...** 를 선택합니다.
2. **찾아보기** 탭에서 검색 상자에 “Microsoft.Bot.Builder”를 입력합니다.
3. 결과 목록에서 **Microsoft.Bot.Builder**를 클릭하고, **설치**를 클릭한 다음, 변경 내용을 적용합니다.

## <a name="get-code-samples"></a>코드 샘플 가져오기

이 SDK에는 .NET용 Bot Builder SDK의 기능을 사용하는 [소스 코드 샘플](bot-builder-dotnet-samples.md)이 포함되어 있습니다.

## <a name="next-steps"></a>다음 단계

이 섹션 전체에서 다음으로 시작하는 문서를 검토하여 .NET용 Bot Builder SDK를 사용하여 봇을 빌드하는 방법을 자세히 알아봅니다.

- [빠른 시작](bot-builder-dotnet-quickstart.md): 이 단계별 자습서의 지침에 따라 간단한 봇을 빠르게 빌드하고 테스트합니다.
- [주요 개념](bot-builder-dotnet-concepts.md): .NET용 Bot Builder SDK의 핵심 개념에 대해 알아봅니다.

문제가 발생하거나 .NET용 Bot Builder SDK에 대한 제안이 있는 경우 사용 가능한 리소스의 목록은 [지원](../bot-service-resources-links-help.md)을 참조하세요. 
