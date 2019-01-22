---
title: Azure Table Storage를 사용하여 사용자 지정 상태 데이터 관리 | Microsoft Docs
description: .NET용 Bot Framework SDK를 사용하여 Azure Table Storage에서 상태 데이터를 저장 및 검색하는 방법을 알아봅니다.
author: kaiqb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: d5d6dc4e635b41424dfee0e260a769f9ed5f595d
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225788"
---
# <a name="manage-custom-state-data-with-azure-table-storage-for-net"></a>.NET용 Azure Table Storage를 사용하여 사용자 지정 상태 데이터 관리

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

이 문서에서는 봇의 상태 데이터를 저장 및 관리하도록 Azure Table Storage를 구현합니다. 봇에서 사용되는 기본 커넥터 상태 서비스는 프로덕션 환경에서 사용할 수 없습니다. GitHub에서 제공되는 [Azure 확장](https://github.com/Microsoft/BotBuilder-Azure)을 사용하거나 선택한 데이터 저장소 플랫폼을 사용하여 사용자 지정 상태 클라이언트를 구현해야 합니다. 사용자 지정 상태 저장소를 사용하는 몇 가지 이유는 다음과 같습니다.
 - 상태 API 처리량 증가(성능에 대한 제어 강화)
 - 지역 복제에 대한 대기 시간 감소
 - 데이터 저장 위치에 대한 제어
 - 실제 상태 데이터에 대한 액세스
 - 32kb를 초과하는 데이터 저장

## <a name="prerequisites"></a>필수 조건
필요한 사항:
 - [Microsoft Azure 계정](https://azure.microsoft.com/en-us/free/)
 - [Visual Studio 2015 이상](https://www.visualstudio.com/)
 - [Bot Builder Azure NuGet 패키지](https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/)
 - [Autofac Web Api2 NuGet 패키지](https://www.nuget.org/packages/Autofac.WebApi2/)
 - [Bot Framework Emulator](https://emulator.botframework.com/)
 - [Azure Storage 탐색기](http://storageexplorer.com/)
 
## <a name="create-azure-account"></a>Azure 계정 만들기
Azure 계정이 없으면 [여기](https://azure.microsoft.com/en-us/free/)를 클릭하여 체험 계정으로 등록하세요.

## <a name="set-up-the-azure-table-storage-service"></a>Azure Table Storage 서비스 설정
1. Azure Portal에 로그인한 후 **새로 만들기**를 클릭하여 새 Azure Table Storage 서비스를 만듭니다. 
2. Azure 테이블을 구현하는 **저장소 계정**을 검색합니다. 
3. 모든 필드를 다 입력하면 화면 아래쪽에 있는 **만들기** 단추를 클릭하여 새 저장소 서비스를 배포합니다. 새 저장소 서비스가 배포되면 사용 가능한 기능 및 옵션이 표시됩니다.
4. 왼쪽의 **액세스 키** 탭을 선택하고 나중에 사용할 수 있도록 연결 문자열을 복사해둡니다. 봇은 이 연결 문자열을 사용하여 상태 데이터를 저장하기 위한 저장소 서비스를 호출합니다.

## <a name="install-nuget-packages"></a>NuGet 패키지 설치
1. 기존 C# 봇 프로젝트를 열거나 Visual Studio에서 C# 봇 템플릿을 사용하여 새 프로젝트를 만듭니다. 
2. 다음 NuGet 패키지를 설치합니다.
   - Microsoft.Bot.Builder.Azure
   - Autofac.WebApi2

## <a name="add-connection-string"></a>연결 문자열 추가 
Web.config 파일에 다음 항목을 추가합니다. 
```XML
  <connectionStrings>
    <add name="StorageConnectionString"
    connectionString="YourConnectionString"/>
  </connectionStrings>
```
"YourConnectionString"을 이전에 저장한 Azure Table Storage의 연결 문자열로 바꿉니다. Web.config 파일을 저장합니다.

## <a name="modify-your-bot-code"></a>봇 코드 수정
Global.asax.cs 파일에 다음 `using` 문을 추가합니다.
```cs
using Autofac;
using System.Configuration;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;
```
`Application_Start()` 메서드에 `TableBotDataStore` 클래스 인스턴스를 만듭니다. `TableBotDataStore` 클래스가 `IBotDataStore<BotData>` 인터페이스를 구현합니다. `IBotDataStore` 인터페이스를 사용하면 기본 커넥터 상태 서비스 연결을 재정의할 수 있습니다.
 ```cs
 var store = new TableBotDataStore(ConfigurationManager.ConnectionStrings["StorageConnectionString"].ConnectionString);
 ```
아래 표시된 것처럼 서비스를 등록합니다.
 ```cs
 Conversation.UpdateContainer(
            builder =>
            {
                builder.Register(c => store)
                          .Keyed<IBotDataStore<BotData>>(AzureModule.Key_DataStore)
                          .AsSelf()
                          .SingleInstance();

                builder.Register(c => new CachingBotDataStore(store,
                           CachingBotDataStoreConsistencyPolicy
                           .ETagBasedConsistency))
                           .As<IBotDataStore<BotData>>()
                           .AsSelf()
                           .InstancePerLifetimeScope();

                
            });
 ```
Global.asax.cs 파일을 저장합니다.

## <a name="run-your-bot-app"></a>봇 앱 실행
Visual Studio에서 봇을 실행하면 추가한 코드가 Azure에서 사용자 지정 **botdata** 테이블을 만듭니다.

## <a name="connect-your-bot-to-the-emulator"></a>에뮬레이터에 봇 연결
이때 봇은 로컬에서 실행됩니다. 에뮬레이터를 시작한 다음, 에뮬레이터에서 봇에 연결합니다.
1. 주소 표시줄에 http://localhost:port-number/api/messages를 입력합니다. 여기서 port-number는 애플리케이션이 실행 중인 브라우저에 표시되는 포트 번호와 일치합니다. 지금은 <strong>Microsoft App ID</strong> 및 <strong>Microsoft App Password</strong> 필드를 비워 둘 수 있습니다. 나중에 [봇을 등록](~/bot-service-quickstart-registration.md)할 때 이 정보를 가져올 수 있습니다.
2. **Connect**를 클릭합니다. 
3. 에뮬레이터에서 몇몇 메시지를 입력하여 봇을 테스트합니다. 

## <a name="view-data-in-azure-table-storage"></a>Azure Table Storage의 데이터 보기
상태 데이터를 보려면 **Storage 탐색기**를 열고 Azure Portal 자격 증명을 사용하여 Azure에 연결하거나 스토리지 이름과 스토리지 키를 사용하여 테이블에 직접 연결한 다음, 테이블 이름으로 이동합니다.  

## <a name="next-steps"></a>다음 단계
이 문서에서는 봇 데이터를 저장하고 관리하는 Azure Table Storage를 구현했습니다. 다음으로 대화 상자를 사용하여 대화 흐름을 모델링하는 방법을 알아봅니다.

> [!div class="nextstepaction"]
> [대화 흐름 관리](bot-builder-dotnet-manage-conversation-flow.md)


## <a name="additional-resources"></a>추가 리소스

위의 코드에서 사용되는 IOC(Inversion of Control) 컨테이너 및 종속성 주입 패턴에 익숙하지 않은 경우 [Autofac](http://autofac.readthedocs.io/en/latest/) 사이트의 정보를 참조하세요. 

GitHub에서 전체 [Azure Table storage](https://github.com/Microsoft/BotBuilder-Azure/tree/master/CSharp/Samples/AzureTable) 샘플을 다운로드할 수도 있습니다.
