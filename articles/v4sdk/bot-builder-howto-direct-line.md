---
title: 직접 회선 봇 및 클라이언트 만들기 | Microsoft Docs
description: .NET용 Bot Builder SDK V4를 사용하여 직접 회선 봇 및 클라이언트를 만드는 방법을 알아봅니다.
keywords: 직접 회선 봇, 직접 회선 클라이언트, 사용자 지정 채널, 콘솔 기반, 게시
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 4/16/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c13733af0f6b26654952a0aab190f6d8cb06d059
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999980"
---
# <a name="create-a-direct-line-bot-and-client"></a>직접 회선 봇 및 클라이언트 만들기

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Microsoft Bot Framework 직접 회선 봇은 사용자가 디자인한 사용자 지정 클라이언트에서 작동할 수 있는 봇입니다. 직접 회산 봇은 일반 봇과 매우 유사합니다. 단지 제공된 채널을 사용할 필요가 없습니다.

직접 회선 클라이언트는 원하는 대로 작성할 수 있습니다. Android 클라이언트, iOS 클라이언트, 심지어 콘솔 기반 클라이언트 애플리케이션까지도 작성할 수 있습니다.

이 항목에서는 직접 회선 봇을 만들어 배포하는 방법과 콘솔 기반 직접 회선 클라이언트 앱을 만들고 실행하는 방법을 알아봅니다.

## <a name="create-your-bot"></a>봇 만들기

언어마다 봇을 만들 때 다른 경로를 따릅니다. Javascript는 Azure에서 봇을 만든 다음, 코드를 수정하지만, C#은 로컬로 봇을 만든 다음, Azure에 게시합니다. 이 두 가지 방법은 모두 유효하며, 두 언어에서 모두 사용할 수 있습니다. 봇을 Azure에 게시하는 방법에 대한 자세한 내용은 [Azure에서 봇 배포](../bot-builder-howto-deploy-azure.md)를 참조하세요.

# <a name="ctabcscreatebot"></a>[C#](#tab/cscreatebot)

### <a name="create-the-solution-in-visual-studio"></a>Visual Studio에서 솔루션 만들기

Visual Studio 2015 이상에서 직접 회선 봇에 대한 솔루션을 만들려면

1. **Visual C#** > **.NET Core**에서 새 **ASP.NET Core 웹 응용 프로그램**을 만듭니다.

1. 이름으로 **DirectLineBotSample**을 입력하고 **확인**을 클릭합니다.

1. **.NET Core** 및 **ASP.NET Core 2.0**이 선택되어 있는지 확인하고 **비어 있음** 프로젝트 템플릿을 클릭한 후 **확인**을 클릭합니다.

#### <a name="add-dependencies"></a>종속성 추가

1. **솔루션 탐색기**에서 **종속성**을 마우스 오른쪽 단추를 클릭한 후 **NuGet 패키지 관리**를 선택합니다.

1. **찾아보기**를 클릭하고 **시험판 포함** 확인란을 선택합니다.

1. 다음 NuGet 패키지를 검색하고 설치합니다.
    - Microsoft.Bot.Builder
    - Microsoft.Bot.Builder.Core.Extensions
    - Microsoft.Bot.Builder.Integration.AspNet.Core
    - Microsoft.Rest.ClientRuntime
    - Newtonsoft.Json

### <a name="create-the-appsettingsjson-file"></a>appsettings.json 파일 만들기

appsettings.json 파일에는 Microsoft 앱 ID, 앱 암호 및 데이터 연결 문자열이 포함됩니다. 이 봇은 상태 정보를 저장하지 않으므로 데이터 연결 문자열은 비어 있습니다. 또한 Bot Framework Emulator만 사용하는 경우 모두 빈 상태를 유지할 수 있습니다.

**appsettings.json** 파일을 만들려면

1. 마우스 오른쪽 단추로 **DirectLineBotSample** 프로젝트를 클릭하고 **추가** > **새 항목**을 선택합니다.

1. **ASP.NET Core**에서 **ASP.NET 구성 파일**을 클릭합니다.

1. 이름으로 **appsettings.json**을 입력하고 **추가**를 클릭합니다.

appsettings.json 파일의 내용을 다음으로 바꿉니다.

```json
{
    "Logging": {
        "IncludeScopes": false,
        "Debug": {
            "LogLevel": {
                "Default": "Warning"
            }
        },
        "Console": {
            "LogLevel": {
                "Default": "Warning"
            }
        }
    },

    "MicrosoftAppId": "",
    "MicrosoftAppPassword": "",
    "DataConnectionString": ""
}
```

### <a name="edit-startupcs"></a>Startup.cs 편집

Startup.cs 파일의 내용을 다음으로 바꿉니다.

**참고**: **DirectBot**은 다음 단계에서 정의되므로 아래 표시되는 빨간색 밑줄은 무시해도 됩니다.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace DirectLineBotSample
{
    public class Startup
    {
        public IConfiguration Configuration { get; }

        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();
            Configuration = builder.Build();
        }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton(_ => Configuration);
            services.AddBot<DirectBot>(options =>
            {
                options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
            });
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseDefaultFiles();
            app.UseStaticFiles();
            app.UseBotFramework();
        }
    }
}
```

### <a name="create-the-directbot-class"></a>DirectBot 클래스 만들기

DirectBot 클래스는 이 봇의 논리 대부분을 포함합니다.

1. 마우스 오른쪽 단추로 **DirectLineBotSample** > **추가** > **새 항목**을 클릭합니다.

1. **ASP.NET Core**에서 **클래스**를 클릭합니다.

1. 이름으로 **DirectBot.cs**를 입력하고 **추가**를 클릭합니다.

DirectBot.cs 파일의 내용을 다음으로 바꿉니다.

```csharp
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

namespace DirectLineBotSample
{
    public class DirectBot : IBot
    {
        public Task OnTurn(ITurnContext context)
        {
            // Respond to the various activity types.
            switch (context.Activity.Type)
            {
                case ActivityTypes.Message:
                    // Respond to the incoming text message.
                    RespondToMessage(context);
                    break;

                case ActivityTypes.ConversationUpdate:
                    break;

                case ActivityTypes.ContactRelationUpdate:
                    break;

                case ActivityTypes.Typing:
                    break;

                case ActivityTypes.DeleteUserData:
                    break;
            }

            return Task.CompletedTask;
        }

        /// <summary>
        /// Responds to the incoming message by either sending a hero card, an image, 
        /// or echoing the user's message.
        /// </summary>
        /// <param name="context">The context of this conversation.</param>
        private void RespondToMessage(ITurnContext context)
        {
            switch (context.Activity.Text.Trim().ToLower())
            {
                case "hi":
                case "hello":
                case "help":
                    // Send the user an instruction message.
                    context.SendActivity("Welcome to the Bot to showcase the DirectLine API. " +
                    "Send \"Show me a hero card\" or \"Send me a BotFramework image\" to see how the " +
                    "DirectLine client supports custom channel data. Any other message will be echoed.");
                    break;

                case "show me a hero card":
                    // Create the hero card.
                    HeroCard heroCard = new HeroCard()
                    {
                        Title = "Sample Hero Card",
                        Text = "Displayed in the DirectLine client"
                    };

                    // Attach the hero card to a new activity.
                    context.SendActivity(MessageFactory.Attachment(heroCard.ToAttachment()));
                    break;

                case "send me a botframework image":
                    // Create the image attachment.
                    Attachment imageAttachment = new Attachment()
                    {
                        ContentType = "image/png",
                        ContentUrl = "https://docs.microsoft.com/en-us/bot-framework/media/how-it-works/architecture-resize.png",
                    };

                    // Attach the image attachment to a new activity.
                    context.SendActivity(MessageFactory.Attachment(imageAttachment));
                    break;

                default:
                    // No command was encountered. Echo the user's message.
                    context.SendActivity($"You said \"{context.Activity.Text}\"");
                    break;
            }
        }
    }
}
```

**참고:** 기존 Program.cs 파일을 변경할 필요는 없습니다.

항목이 올바른지 확인하려면 **F6** 키를 눌러 프로젝트를 빌드합니다. 경고나 오류는 표시되지 않아야 합니다.

### <a name="publish-the-bot-from-visual-studio"></a>Visual Studio에서 봇 게시

1. Visual Studio의 솔루션 탐색기에서 **DirectLineBotSample** 프로젝트를 마우스 오른쪽 단추로 클릭하고 **시작 프로젝트로 설정**을 클릭합니다.

    ![Visual studio 게시 페이지](media/bot-builder-howto-direct-line/visual-studio-publish-page.png)

1. 마우스 오른쪽 단추로 **DirectLineBotSample**을 클릭하고 **게시**를 클릭합니다. Visual Studio **게시** 페이지가 나타납니다.

1. 이 봇에 대한 게시 프로필에 이미 있는 경우 해당 프로필을 선택하고 **게시** 단추를 클릭한 후 다음 섹션으로 이동합니다.

1. 새 게시 프로필을 만들려면 **Microsoft Azure App Service** 아이콘을 선택합니다.

1. **새로 만들기**를 선택합니다.

1. **게시** 단추를 클릭합니다. **App Service 만들기** 대화 상자가 나타납니다.

    ![App Service 만들기 대화 상자](media/bot-builder-howto-direct-line/create-app-service-dialog.png)

    - 나중에 찾을 수 있는 이름을 앱 이름으로 지정합니다. 예를 들어, 앱 이름 맨 앞에 이메일 이름을 추가할 수 있습니다.

    - 올바른 구독을 사용하는지 확인합니다.

    - 리소스 그룹에 대해 올바른 리소스 그룹을 사용하고 있는지 확인합니다. 리소스 그룹은 봇의 **개요** 블레이드에서 찾을 수 있습니다. **참고:** 잘못된 리소스 그룹은 수정하기가 어렵습니다.

    - 올바른 App Service 계획을 사용하고 있는지 확인합니다.
    
1. **만들기** 단추를 클릭합니다. Visual Studio에서 봇 배포가 시작됩니다.

봇을 게시하면 브라우저에 봇의 URL 끝점이 표시됩니다.

### <a name="create-bot-channels-registration-bot-on-microsoft-azure"></a>Microsoft Azure에서 봇 채널 등록 봇 만들기

직접 회선 봇은 어떤 플랫폼에도 호스트할 수 있습니다. 이 예제에서 봇의 호스트는 Microsoft Azure입니다. 

Microsoft Azure에서 봇을 만들려면

1. Microsoft Azure Portal에서 **리소스 만들기**를 클릭한 후 "봇 채널 등록"을 검색합니다.

1. **만들기**를 클릭합니다. 봇 채널 등록 블레이드가 표시됩니다.

    ![봇 채널 등록 블레이드, 봇 이름용 필드, 구독, 리소스 그룹, 위치, 가격 책정 계층, 메시징 끝점, 기타 필드](media/bot-builder-howto-direct-line/bot-service-registration-blade.png)

1. 봇 채널 등록 블레이드에서 **봇 이름**, **구독**, **리소스 그룹**, **위치** 및 **가격 책정 계층**을 입력합니다.

1. **메시징 끝점**은 비워 둡니다. 이 값은 나중에 입력됩니다.

1. **Microsoft 앱 ID 및 암호**를 클릭하고 **자동 앱 ID 및 암호 만들기**를 클릭합니다.

1. **대시보드에 고정** 확인란을 선택합니다.

1. **만들기** 단추를 클릭합니다.

배포에 몇 분 가량 소요되며 완료되면 대시보드에 표시됩니다.

### <a name="update-the-appsettingsjson-file"></a>appsettings.json 파일 업데이트

1. 대시보드에서 봇을 클릭하거나 **모든 리소스**를 클릭하고 봇 채널 등록의 이름을 검색하여 새 리소스로 이동합니다.

1. **개요**를 클릭합니다.

1. **DirectLineClientSample** 프로젝트의 **Program.cs**에 있는 **botId** 문자열에 리소스 그룹의 이름을 복사합니다.

1. **설정**을 클릭하고 **Microsoft 앱 ID** 필드 근처에 있는 **관리**를 클릭합니다.

1. 애플리케이션 ID를 복사한 후 **appsettings.json** 파일의 **"MicrosoftAppId"** 필드에 붙여넣습니다.

1. **새 암호 생성** 단추를 클릭합니다.

1. 새 암호를 복사한 후 **appsettings.json** 파일의 **"MicrosoftAppPassword"** 필드에 붙여넣습니다.

### <a name="set-the-messaging-endpoint"></a>메시징 끝점 설정

봇에 끝점을 추가하려면

1. 봇을 게시한 후 팝업되는 브라우저에서 URL 주소를 복사합니다.

    ![메시징 끝점을 강조 표시하는 봇 채널 등록의 설정 블레이드](media/bot-builder-howto-direct-line/bot-channels-registration-settings.png)

1. 봇의 **설정** 블레이드가 표시됩니다.

1. 주소를 **메시징 끝점**에 붙여넣습니다.

1. "https://"로 시작하고 "/api/messages"로 끝나도록 주소를 편집합니다. 예를 들어 브라우저에서 복사한 주소가 "http://v-royhar-dlbot-directlinebotsample20180329044602.azurewebsites.net"이면 "https://v-royhar-dlbot-directlinebotsample20180329044602.azurewebsites.net/api/messages"(으)로 편집합니다.

1. 설정 블레이드의 **저장** 단추를 클릭합니다.

**참고:** 게시가 실패하는 경우 먼저 Azure에서 봇을 중지해야 봇에 대한 변경 내용을 게시할 수 있습니다.

1. App Service 이름("https://"와 ".azurewebsites.net" 사이에 있음)을 찾습니다.

1. Azure Portal의 "모든 리소스"에서 해당 리소스를 검색합니다.

1. App Service 리소스를 클릭합니다. App Serivce 블레이드가 표시됩니다.

1. App Service 블레이드의 **중지** 단추를 클릭합니다.

1. Visual Studio에서 봇을 다시 게시합니다.

1. App Service 블레이드의 **시작** 단추를 클릭합니다.

### <a name="test-your-bot-in-webchat"></a>웹 채팅에서 봇 테스트

봇이 작동하는지 확인하려면 웹 채팅에서 봇을 확인합니다.

1. 봇의 봇 채널 등록 블레이드에서 **설정**을 클릭한 후 **웹 채팅에서 테스트**를 클릭합니다.

1. "Hi"를 입력합니다. 봇이 환영 메시지를 사용하여 응답합니다.

1. "show me a hero card"를 입력합니다. 봇이 Hero 카드를 표시합니다.

1. "send me a botframework image"를 입력합니다. 봇이 Bot Framework 설명서의 이미지를 표시합니다.

1. 다른 내용을 입력하면 큰따옴표 안에 "You said"와 사용자가 입력한 메시지를 사용해서 응답합니다.

### <a name="troubleshooting"></a>문제 해결

봇이 작동하지 않는 경우 Microsoft 앱 ID 및 암호를 확인하거나 다시 입력합니다. 이전를 복사했더라도, 봇 채널 등록의 설정 블레이드에 있는 Microsoft 앱 ID를 appsettings.json 파일의 **"MicrosoftAppId"** 필드에 있는 값과 비교해서 확인합니다.

암호를 확인하거나 새 암호를 만든 후 사용합니다. 

1. 봇 채널 등록 블레이드의 **Microsoft 앱 ID** 필드 옆에 있는 **관리**를 클릭합니다.

1. 애플리케이션 등록 포털에 로그인합니다.

1. 암호의 처음 세 문자가 **appsettings.json** 파일의 **"MicrosoftAppPassword"** 필드와 일치하는지 확인합니다. 

1. 두 값이 일치하지 않는 경우 새 암호를 생성하고 **appsettings.json** 파일의 **"MicrosoftAppPassword"** 필드에 해당 값을 저장합니다.

# <a name="javascripttabjscreatebot"></a>[JavaScript](#tab/jscreatebot)

### <a name="create-web-app-bot-on-microsoft-azure"></a>Microsoft Azure에서서 웹앱 봇 만들기

Microsoft Azure에서 봇을 만들려면 

1. Microsoft Azure Portal에서 **리소스 만들기**를 클릭한 후 "웹앱 봇"을 검색합니다.

1. **만들기**를 클릭합니다. 웹앱 봇 블레이드가 표시됩니다.

![웹앱 봇 등록](media/bot-builder-howto-direct-line/web-app-bot-registration.png)

1. 웹앱 봇 블레이드에서 **봇 이름**, **구독**, **리소스 그룹**, **위치**, **가격 책정 계층**, **앱 이름**을 입력합니다.

1. 사용하려는 봇 템플릿을 선택합니다. **SDK 버전: SDK v4** 및 **SDK 언어: Node.js** 

1. 기본 v4 미리 보기 템플릿을 선택합니다. 

1. **App Service 계획/위치**, **Azure Storage** 및 **Application Insights 위치**를 선택합니다.

1. 대시보드에 고정 확인란을 선택합니다.

1. 만들기 단추를 클릭합니다.

1. 봇이 배포될 때까지 기다립니다. 대시보드에 고정 확인란을 선택했으므로 봇이 대시보드에 표시됩니다.

### <a name="edit-your-direct-line-bot"></a>직접 회선 봇 편집

이제 일부 기능을 에코 봇에 추가하겠습니다.

1. 웹앱 봇 블레이드에서 빌드를 클릭합니다. 

1. zip 파일 다운로드를 클릭합니다. 

1. 로컬 디렉터리에 .zip 파일의 압축을 풉니다.

1. 압축을 푼 폴더로 이동하고 선호하는 IDE에서 원본 파일을 엽니다.

1. app.js 파일에서 아래 코드를 복사하여 붙여넣습니다.

```javascript
// Copyright (c) Microsoft Corporation. All rights reserved.
// Licensed under the MIT License.

const { BotFrameworkAdapter, MessageFactory, CardFactory } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});

// Responds to the incoming message by either sending a hero card, an image, 
// or echoing the user's message.

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        if (context.activity.type === 'message') {
            const text = (context.activity.text || '').trim().toLowerCase()

            switch(text){
                case 'hi':
                case "hello":
                case "help":
                    // Send the user an instruction message.
                    return context.sendActivity("Welcome to the Bot to showcase the DirectLine API. " +
                    "Send \"Show me a hero card\" or \"Send me a BotFramework image\" to see how the " +
                    "DirectLine client supports custom channel data. Any other message will be echoed.");
                    break;

                case 'show me a hero card':
                    // Create the hero card.
                    const message = MessageFactory.attachment(
                        CardFactory.heroCard(   
                        'Sample Hero Card', //cards title
                        'Displayed in the DirectLine client' //cards text
                        )
                    );
                    return context.sendActivity(message);
                    break;
                    
                case 'send me a botframework image':
                    // Create the image attachment.
                    const imageOrVideoMessage = MessageFactory.contentUrl('https://docs.microsoft.com/en-us/azure/bot-service/media/how-it-works/architecture-resize.png', 'image/png')
                    return context.sendActivity(imageOrVideoMessage);
                    break;
                
                default:
                    // No command was encountered. Echo the user's message.
                    return context.sendActivity(`You said ${context.activity.text}`);
                    break;
                    
            }
        }
    });
});

```

준비가 완료되면 원본을 다시 Azure에 게시할 수 있습니다. 봇을 [Azure](../bot-service-build-download-source-code.md)에 다시 게시하는 방법을 알아보려면 다음 단계를 수행합니다.

---


## <a name="create-the-console-client-app"></a>콘솔 클라이언트 앱 만들기

# <a name="ctabcsclientapp"></a>[C#](#tab/csclientapp)

콘솔 클라이언트 애플리케이션은 두 개의 스레드로 작동합니다. 기본 스레드는 사용자 입력을 수락하고 봇에 메시지를 보냅니다. 보조 스레드는 초당 1번씩 봇을 폴링하여 봇에서 메시지를 가져온 다음, 수신된 메시지를 표시합니다.

콘솔 프로젝트를 만들려면

1. 솔루션 탐색기에서 **솔루션 'DirectLineBotSample'** 을 마우스 오른쪽 단추로 클릭합니다.

1. **추가** > **새 프로젝트**를 선택합니다.

1. **Visual C#** > **Windows 클래식 데스크톱*에서 **콘솔 앱(.NET Framework)** 을 선택합니다.
 
    **참고:** **콘솔 앱(.NET Core)** 은 선택하지 마세요.

1. 이름으로 **DirectLineClientSample**을 입력합니다.

1. **확인**을 클릭합니다.

### <a name="add-the-nuget-packages-to-the-console-app"></a>콘솔 앱에 NuGet 패키지 추가

1. **참조**를 마우스 오른쪽 단추로 클릭합니다.

1. **NuGet 패키지 관리**를 클릭합니다.

1. **찾아보기**를 클릭합니다. **시험판 포함** 확인란을 선택했는지 확인합니다.

1. 다음 NuGet 패키지를 검색하고 설치합니다.
    - Microsoft.Bot.Connector.DirectLine(v3.0.2)
    - Newtonsoft.Json

### <a name="edit-the-programcs-file"></a>Program.cs 파일 편집

DirectLineClientSample **Program.cs** 파일의 내용을 다음으로 바꿉니다.

```csharp
using System;
using System.Diagnostics;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Bot.Connector.DirectLine;
using Newtonsoft.Json;

namespace DirectLineClientSample
{
    class Program
    {
        // ************
        // Replace the following values with your Direct Line secret and the name of your bot resource ID.
        //*************
        private static string directLineSecret = "*** Replace with Direct Line secret ***";
        private static string botId = "*** Replace with the resource name of your bot ***";

        // This gives a name to the bot user.
        private static string fromUser = "DirectLineClientSampleUser";

        static void Main(string[] args)
        {
            StartBotConversation().Wait();
        }


        /// <summary>
        /// Drives the user's conversation with the bot.
        /// </summary>
        /// <returns></returns>
        private static async Task StartBotConversation()
        {
            // Create a new Direct Line client.
            DirectLineClient client = new DirectLineClient(directLineSecret);

            // Start the conversation.
            var conversation = await client.Conversations.StartConversationAsync();

            // Start the bot message reader in a separate thread.
            new System.Threading.Thread(async () => await ReadBotMessagesAsync(client, conversation.ConversationId)).Start();

            // Prompt the user to start talking to the bot.
            Console.Write("Type your message (or \"exit\" to end): ");

            // Loop until the user chooses to exit this loop.
            while (true)
            {
                // Accept the input from the user.
                string input = Console.ReadLine().Trim();

                // Check to see if the user wants to exit.
                if (input.ToLower() == "exit")
                {
                    // Exit the app if the user requests it.
                    break;
                }
                else
                {
                    if (input.Length > 0)
                    {
                        // Create a message activity with the text the user entered.
                        Activity userMessage = new Activity
                        {
                            From = new ChannelAccount(fromUser),
                            Text = input,
                            Type = ActivityTypes.Message
                        };

                        // Send the message activity to the bot.
                        await client.Conversations.PostActivityAsync(conversation.ConversationId, userMessage);
                    }
                }
            }
        }


        /// <summary>
        /// Polls the bot continuously and retrieves messages sent by the bot to the client.
        /// </summary>
        /// <param name="client">The Direct Line client.</param>
        /// <param name="conversationId">The conversation ID.</param>
        /// <returns></returns>
        private static async Task ReadBotMessagesAsync(DirectLineClient client, string conversationId)
        {
            string watermark = null;

            // Poll the bot for replies once per second.
            while (true)
            {
                // Retrieve the activity set from the bot.
                var activitySet = await client.Conversations.GetActivitiesAsync(conversationId, watermark);
                watermark = activitySet?.Watermark;

                // Extract the activies sent from our bot.
                var activities = from x in activitySet.Activities
                                 where x.From.Id == botId
                                 select x;

                // Analyze each activity in the activity set.
                foreach (Activity activity in activities)
                {
                    // Display the text of the activity.
                    Console.WriteLine(activity.Text);

                    // Are there any attachments?
                    if (activity.Attachments != null)
                    {
                        // Extract each attachment from the activity.
                        foreach (Attachment attachment in activity.Attachments)
                        {
                            switch (attachment.ContentType)
                            {
                                // Display a hero card.
                                case "application/vnd.microsoft.card.hero":
                                    RenderHeroCard(attachment);
                                    break;

                                // Display the image in a browser.
                                case "image/png":
                                    Console.WriteLine($"Opening the requested image '{attachment.ContentUrl}'");
                                    Process.Start(attachment.ContentUrl);
                                    break;
                            }
                        }
                    }

                    // Redisplay the user prompt.
                    Console.Write("\nType your message (\"exit\" to end): ");
                }

                // Wait for one second before polling the bot again.
                await Task.Delay(TimeSpan.FromSeconds(1)).ConfigureAwait(false);
            }
        }


        /// <summary>
        /// Displays the hero card on the console.
        /// </summary>
        /// <param name="attachment">The attachment that contains the hero card.</param>
        private static void RenderHeroCard(Attachment attachment)
        {
            const int Width = 70;
            // Function to center a string between asterisks.
            Func<string, string> contentLine = (content) => string.Format($"{{0, -{Width}}}", string.Format("{0," + ((Width + content.Length) / 2).ToString() + "}", content));

            // Extract the hero card data.
            var heroCard = JsonConvert.DeserializeObject<HeroCard>(attachment.Content.ToString());

            // Display the hero card.
            if (heroCard != null)
            {
                Console.WriteLine("/{0}", new string('*', Width + 1));
                Console.WriteLine("*{0}*", contentLine(heroCard.Title));
                Console.WriteLine("*{0}*", new string(' ', Width));
                Console.WriteLine("*{0}*", contentLine(heroCard.Text));
                Console.WriteLine("{0}/", new string('*', Width + 1));
            }
        }
    }
}
```

항목이 올바른지 확인하려면 **F6** 키를 눌러 프로젝트를 빌드합니다. 경고나 오류는 표시되지 않아야 합니다.

# <a name="javascripttabjsclientapp"></a>[JavaScript](#tab/jsclientapp)

### <a name="create-a-direct-line-client"></a>직접 회선 클라이언트 만들기 

웹앱 봇을 배포했으므로 직접 회선 클라이언트를 만들 수 있습니다.

```
    md DirectLineClient
    cd DirectLineClient
    npm init
```

다음으로, npm에서 이러한 패키지 설치

```
    npm install --save open
    npm install --save request
    npm install --save request-promise
    npm install --save swagger-client
```

**app.js** 파일을 만듭니다. 아래 코드를 복사한 후 이 파일에 붙여넣습니다.

다음 단계에서는 directLineSecret가 제공됩니다.
```javascript
var Swagger = require('swagger-client');
var open = require('open');
var rp = require('request-promise');

// config items
var pollInterval = 1000;
// Change the Direct Line Secret to your own 
var directLineSecret = 'your secret here';
var directLineClientName = 'DirectLineClient';
var directLineSpecUrl = 'https://docs.botframework.com/en-us/restapi/directline3/swagger.json';

var directLineClient = rp(directLineSpecUrl)
    .then(function (spec) {
        // client
        return new Swagger({
            spec: JSON.parse(spec.trim()),
            usePromise: true
        });
    })
    .then(function (client) {
        // add authorization header to client
        client.clientAuthorizations.add('AuthorizationBotConnector', new Swagger.ApiKeyAuthorization('Authorization', 'Bearer ' + directLineSecret, 'header'));
        return client;
    })
    .catch(function (err) {
        console.error('Error initializing DirectLine client', err);
    });

// once the client is ready, create a new conversation
directLineClient.then(function (client) {
    client.Conversations.Conversations_StartConversation()                          // create conversation
        .then(function (response) {
            return response.obj.conversationId;
        })                            // obtain id
        .then(function (conversationId) {
            sendMessagesFromConsole(client, conversationId);                        // start watching console input for sending new messages to bot
            pollMessages(client, conversationId);                                   // start polling messages from bot
        })
        .catch(function (err) {
            console.error('Error starting conversation', err);
        });
});

// Read from console (stdin) and send input to conversation using DirectLine client
function sendMessagesFromConsole(client, conversationId) {
    var stdin = process.openStdin();
    process.stdout.write('Command> ');
    stdin.addListener('data', function (e) {
        var input = e.toString().trim();
        if (input) {
            // exit
            if (input.toLowerCase() === 'exit') {
                return process.exit();
            }

            // send message
            client.Conversations.Conversations_PostActivity(
                {
                    conversationId: conversationId,
                    activity: {
                        textFormat: 'plain',
                        text: input,
                        type: 'message',
                        from: {
                            id: directLineClientName,
                            name: directLineClientName
                        }
                    }
                }).catch(function (err) {
                    console.error('Error sending message:', err);
                });

            process.stdout.write('Command> ');
        }
    });
}

// Poll Messages from conversation using DirectLine client
function pollMessages(client, conversationId) {
    console.log('Starting polling message for conversationId: ' + conversationId);
    var watermark = null;
    setInterval(function () {
        client.Conversations.Conversations_GetActivities({ conversationId: conversationId, watermark: watermark })
            .then(function (response) {
                watermark = response.obj.watermark;                                 // use watermark so subsequent requests skip old messages
                return response.obj.activities;
            })
            .then(printMessages);
    }, pollInterval);
}

// Helpers methods
function printMessages(activities) {
    if (activities && activities.length) {
        // ignore own messages
        activities = activities.filter(function (m) { return m.from.id !== directLineClientName });

        if (activities.length) {
            process.stdout.clearLine();
            process.stdout.cursorTo(0);

            // print other messages
            activities.forEach(printMessage);

            process.stdout.write('Command> ');
        }
    }
}

function printMessage(activity) {
    if (activity.text) {
        console.log(activity.text);
    }

    if (activity.attachments) {
        activity.attachments.forEach(function (attachment) {
            switch (attachment.contentType) {
                case "application/vnd.microsoft.card.hero":
                    renderHeroCard(attachment);
                    break;

                case "image/png":
                    console.log('Opening the requested image ' + attachment.contentUrl);
                    open(attachment.contentUrl);
                    break;
            }
        });
    }
}

function renderHeroCard(attachment) {
    var width = 70;
    var contentLine = function (content) {
        return ' '.repeat((width - content.length) / 2) +
            content +
            ' '.repeat((width - content.length) / 2);
    }

    console.log('/' + '*'.repeat(width + 1));
    console.log('*' + contentLine(attachment.content.title) + '*');
    console.log('*' + ' '.repeat(width) + '*');
    console.log('*' + contentLine(attachment.content.text) + '*');
    console.log('*'.repeat(width + 1) + '/');
}
```

---

## <a name="configure-the-direct-line-channel"></a>직접 회선 채널 구성

직접 회선 채널을 추가하면 이 봇이 직접 회선 봇이 됩니다.

직접 회선 채널을 구성하려면

![직접 회선 채널 단추가 강조 표시된 채널에 연결 블레이드](media/bot-builder-howto-direct-line/direct-line-channel-button.png)

1. **봇 채널 등록** 블레이드에서 **채널**을 클릭합니다. **채널에 연결** 블레이드가 표시됩니다.

    ![두 비밀 키가 표시되는 직접 회선 구성 블레이드](media/bot-builder-howto-direct-line/configure-direct-line.png)

1. **채널에 연결** 블레이드에서 **직접 회선 채널 구성** 단추를 클릭합니다. 

1. **3.0**을 선택하지 않은 경우 선택합니다.

1. 하나 이상의 비밀 키에 대해 **표시**를 클릭합니다.

1. 비밀 키 중 하나를 복사하고 클라이언트 앱의 **directLineSecret** 문자열에 붙여넣습니다.

1. 완료를 클릭합니다.

## <a name="run-the-client-app"></a>클라이언트 앱을 실행합니다.

# <a name="ctabcsrunclient"></a>[C#](#tab/csrunclient)

이제 봇은 직접 회선 콘솔 클라이언트 애플리케이션과 통신할 수 있습니다. 콘솔 앱을 실행하려면 다음을 수행합니다.

1. Visual Studio에서 마우스 오른쪽 단추로 **DirectLineClientSample** 프로젝트를 클릭하고 **시작 프로젝트로 설정**을 선택합니다.

1. **DirectLineClientSample** 프로젝트에서 **Program.cs** 파일을 검사합니다.

    - 하나의 직접 회선 비밀 코드가 **directLineSecret** 문자열에 있는지 확인합니다.

1. **directLineSecret**가 올바르지 않으면 Azure Portal로 이동한 후 직접 회선 봇을 클릭하고 **채널**, **직접 회선 채널 구성**(또는 **편집**)을 차례로 클릭하고 키를 표시한 후 해당 키를 **directLineSecret** 문자열에 복사합니다.

    - 리소스 그룹이 **botId** 문자열에 있는지 확인합니다.

1. 이 문자열에 없으면 **개요** 블레이드에서 **리소스 그룹**을 복사한 후 **botId** 문자열에 붙여넣습니다.

1. F5 키를 누르거나 디버깅을 시작합니다.

콘솔 클라이언트 앱이 시작됩니다. 앱을 테스트하려면 

1. "Hi"를 입력합니다. 봇이 'You said "Hi"'를 표시합니다.

1. "show me a hero card"를 입력합니다. 봇이 Hero 카드를 표시합니다.

1. "send me a botframework image"를 입력합니다. 봇이 브라우저를 시작하고 Bot Framework 설명서의 이미지를 표시합니다.

1. 다른 내용을 입력하면 큰따옴표 안에 "You said"와 사용자가 입력한 메시지를 사용해서 응답합니다.

# <a name="javascripttabjsrunclient"></a>[JavaScript](#tab/jsrunclient)

샘플을 실행하려면 DirectLineClient 앱을 실행해야 합니다.

1. CMD 콘솔을 열고 DirectLineClient 디렉터리로 명령

1. `node app.js` 실행

클라이언트 콘솔에서 사용자 지정 메시지 show me a hero card 또는 send me a botframework image를 테스트하면 다음 결과가 표시됩니다.

![결과](media/bot-builder-howto-direct-line/outcome.png)

---


### <a name="next-steps"></a>다음 단계
