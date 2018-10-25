---
title: Azure에 Skype 실시간 미디어 봇 배포 | Microsoft Docs
description: Visual Studio의 기본 제공 게시 기능을 사용하여 Azure에 Skype 실시간 오디오-비디오 봇을 배포하는 방법을 알아봅니다.
author: MalarGit
ms.author: malarch
manager: ssulzer
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 22cce8ad5bef3c1c6f08a8efc28118e0209dd3af
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999440"
---
# <a name="deploy-a-real-time-media-bot-from-visual-studio-to-azure"></a>Visual Studio에서 Azure로 실시간 미디어 봇 배포

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

실시간 미디어 봇은 "IaaS" Azure Virtual Machine 또는 "클래식" Azure 클라우드 서비스에서 호스트할 수 있습니다. 이 문서에서는 Visual Studio에서 해당 기본 제공 게시 기능을 사용하여 Azure 클라우드 서비스 작업자 역할에 호스트된 봇을 배포하는 방법을 보여 줍니다.

## <a name="prerequisites"></a>필수 조건

Azure에 봇을 배포하려면 Microsoft Azure 구독이 있어야 합니다. 아직 구독이 없는 경우 <a href="https://azure.microsoft.com/en-us/free/" target="_blank">체험 계정</a>으로 등록할 수 있습니다. 또한 이 문서에 설명된 프로세스에는 Visual Studio가 필요합니다. Visual Studio가 아직 없는 경우 <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2017 커뮤니티</a>에서 무료로 다운로드할 수 있습니다.

### <a name="certificate-from-a-valid-certificate-authority"></a>유효한 인증 기관의 인증서
봇은 신뢰할 수 있는 CA(인증 기관)의 유효한 인증서를 사용하여 구성해야 합니다. SN(주체 이름) 또는 인증서에서 SAN(주체 대체 이름)의 마지막 항목은 클라우드 서비스의 이름이어야 합니다. 현재, 와일드카드 인증서는 지원되지 않습니다. CNAME이 클라우드 서비스를 가리키는 데 사용되면 CNAME은 SN 또는 인증서의 마지막 SAN 항목이어야 합니다.

## <a name="configure-application-settings"></a>응용 프로그램 설정 구성
봇이 클라우드에서 제대로 작동하려면 응용 프로그램 설정이 정확한지 확인해야 합니다. 좀 더 구체적으로 말하자면, 작업자 역할의 app.config 파일에서 다음 키 값을 설정합니다.
> <ul><li>MicrosoftAppId</li><li>MicrosoftAppPassword</li></ul>

> [!NOTE]
> 봇의 **AppID** 및 **AppPassword**를 찾으려면 [MicrosoftAppID 및 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)를 참조하세요.

## <a name="create-worker-role-in-the-azure-portal"></a>Azure Portal에서 작업자 역할 만들기
### <a name="step-1-create-cloud-serviceclassic"></a>1단계: 클라우드 서비스 만들기(클래식)
<a href="https://portal.azure.com">Azure Portal</a>에 로그온합니다. 화면 왼쪽에서 **+** 를 클릭하고 **클라우드 서비스(클래식)** 를 선택합니다. 양식에 필요한 정보를 제공하고 **만들기**를 클릭합니다.

![클라우드 서비스 만들기](../media/real-time-media-bot-portal-service-creation.png)

> [!NOTE]
> 봇의 DNS 이름을 봇 등록 URL에 제공해야 합니다.

### <a name="step-2-upload-the-certificate-for-the-bot"></a>2단계: 봇에 대한 인증서 업로드
봇이 만들어지면 봇에 대한 인증서를 업로드합니다.

![인증서 업로드](../media/real-time-media-bot-portal-certificates.png)

## <a name="modify-service-configuration-with-worker-role-details"></a>작업자 역할 정보를 사용하여 서비스 구성 수정
봇의 FQDN(정규화된 도메인 이름)은 Azure RoleEnvironment API를 통해 사용할 수 없습니다. 따라서 봇에 해당 FQDN을 제공해야 합니다. 또한 봇은 HTTPS에 대한 인증서도 알고 있어야 합니다. 이러한 내용은 작업자 역할의 서비스 구성(.cscfg) 파일에서 구성할 수 있습니다.

> [!TIP]
> BotBuilder-RealTimeMediaCalling git 리포지토리에서 샘플을 배포하는 경우
> - 서비스 구성에서 사용될 경우, $DnsName$을 클라우드 서비스 이름 또는 CNAME으로 대체합니다.
>   ```xml
>      <Setting name="ServiceDnsName" value="$DnsName$" />
>   ```
> 
> - 구성의 다음 줄에서 $CertThumbprint$를 봇에 업로드된 인증서의 지문으로 대체합니다.
>   ```xml
>      <Setting name="DefaultCertificate" value="$CertThumbprint$" />
>      <Certificate name="Default" thumbprint="$CertThumbprint$" thumbprintAlgorithm="sha1" />
>   ```

## <a name="publish-the-bot-from-visual-studio"></a>Visual Studio에서 봇 게시
### <a name="step-1-launch-the-microsoft-azure-publishing-wizard-in-visual-studio"></a>1단계: Visual Studio에서 Microsoft Azure 게시 마법사 시작

Visual Studio에서 새 프로젝트를 엽니다. 솔루션 탐색기에서 클라우드 서비스 프로젝트를 마우스 오른쪽 단추로 클릭하고 **게시**를 선택합니다. 이렇게 하면 Microsoft Azure 게시 마법사가 시작됩니다. 자격 증명을 사용하여 해당 구독에 로그인합니다.

![프로젝트를 마우스 오른쪽 단추로 클릭하고 게시를 선택하여 Microsoft Azure 게시 마법사를 시작합니다.](../media/real-time-media-bot-publish-signin.png)

### <a name="step-2-publish-the-bot"></a>2단계: 봇 게시

**다음**을 클릭합니다. 그러면 **설정** 탭이 열립니다. 봇을 배포하기 위한 클라우드 서비스, 환경, 빌드 구성 및 서비스 구성을 지정합니다.

![다음을 클릭하고 설정 탭으로 이동](../media/real-time-media-bot-publish-settings.png)

필요에 따라 **고급 설정**을 선택하고 배포 로그에 대한 저장소 계정을 지정할 수 있습니다(문제를 디버그하는 데 사용할 수 있음).

![고급 설정 탭 클릭](../media/real-time-media-bot-publish-advanced-settings.png)

**요약** 탭에서 구성을 확인하고 **게시**를 클릭하여 Microsoft Azure에 봇을 배포합니다.
