---
title: 실시간 미디어 봇의 요구 사항 및 고려 사항 | Microsoft Docs
description: .NET용 Bot Framework SDK를 사용하여 Skype용 실시간 미디어 봇을 만드는 작업에 관련된 중요한 요구 사항과 고려 사항을 살펴봅니다.
author: ssulzer
ms.author: ssulzer
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: d8cd326a3027fe5fcb440d9b205ba7d32a8b1640
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224938"
---
# <a name="requirements-and-considerations-for-real-time-media-bots"></a>실시간 미디어 봇의 요구 사항 및 고려 사항

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

메시징 및 IVR 호출 봇 개발에 적용되는 모든 지침이 실시간 미디어 봇 빌드에 똑같이 적용되는 것은 아닙니다. 이 문서에서는 실시간 미디어 봇 개발을 위한 일부 중요한 요구 사항 및 고려 사항을 설명합니다. 

> [!NOTE]
> 봇용 실시간 미디어 플랫폼은 미리 보기 기술이므로 이 문서의 지침은 변경될 수 있습니다.

## <a name="real-time-media-bot-development-requires-cnet-and-windows-server"></a>실시간 미디어 봇 개발에 C#/.NET 및 Windows Server가 필요함

- 실시간 미디어 봇이 오디오 및 비디오 미디어에 액세스하려면 `Microsoft.Skype.Bots.Media`.NET 라이브러리 (<a href="https://www.nuget.org/" target="_blank">NuGet</a>을 통해 사용 가능)가 필요하고, 봇은 Windows Server 머신(또는 Azure의 Windows Server 게스트 OS)에 배포되어야 합니다. 따라서 봇은 C# 및 표준 .NET Framework를 사용하여 개발하고 Azure에 배포해야 합니다. 실시간 미디어에 액세스하는 데는 C++ 및 Node.js API를 사용할 수 없습니다.

- 실시간 미디어 봇은 최신 버전의 `Microsoft.Skype.Bots.Media` .NET 라이브러리에서 실행되고 있어야 합니다. 봇은 가장 최근 제공된 버전의 <a href="https://www.nuget.org/" target="_blank">NuGet</a> 패키지 또는 3개월 이내에 제공된 버전을 사용해야 합니다. 이전 버전의 라이브러리는 사용되지 않을 예정이며 몇 달 후에는 작동하지 않습니다.

## <a name="real-time-media-calls-stay-on-the-machine-where-they-were-created"></a>실시간 미디어 호출은 만들어진 머신에서 유지됩니다.

- 실시간 미디어 봇은 상태 저장 봇입니다. 실시간 미디어 호출은 들어오는 호출을 수락한 VM(가상 머신) 인스턴스에 고정됩니다. Skype 호출자의 음성 및 비디오 미디어는 해당 VM 인스턴스로 흐르고 봇이 다시 호출자에게 보내는 미디어도 해당 VM에서 시작되어야 합니다.

- VM이 중지될 때 진행 중인 실시간 미디어 호출이 있으면 해당 호출은 갑자기 종료됩니다. 봇이 보류 중인 VM 종료를 알 수 있는 경우에는 “정상적으로” 호출을 종료하려고 시도할 수 있습니다.

## <a name="real-time-media-bots-must-be-directly-accessible-on-the-internet"></a>실시간 미디어 봇은 인터넷에서 직접 액세스할 수 있어야 합니다.

- 실시간 미디어 봇을 호스트하는 각 VM 인스턴스는 인터넷에서 직접 액세스할 수 있어야 합니다. Azure에서 호스트되는 실시간 미디어 봇의 경우 각 VM 인스턴스에는 ILPIP(인스턴스 수준 공용 IP 주소)가 있어야 합니다. ILPIP 가져오기 및 구성에 대한 자세한 내용은 <a href="/azure/virtual-network/virtual-networks-instance-level-public-ip" target="_blank">인스턴스 수준 공용 IP(클래식) 개요</a>를 참조하세요. 기본적으로 Azure 구독은 5개의 ILPIP 주소를 가져올 수 있습니다. 구독 할당량을 늘리려면 Azure 지원 팀에 문의하세요.

- 공용 IP 주소 요구 사항 때문에 실시간 미디어 봇은 “IaaS” Azure Virtual Machine 또는 “클래식” Azure 클라우드 서비스에서 호스트되어야 합니다. VM Scale Sets를 포함하는 Bot Service 또는 Service Fabric과 같은 기타 런타임 환경은 지원되지 않고 ILPIP를 지원하지 않습니다.

- 실시간 미디어 봇은 [Bot Framework Emulator](../bot-service-debug-emulator.md)에서도 지원되지 않습니다.

## <a name="scalability-and-performance-considerations"></a>확장성 및 성능 고려 사항

- 실시간 미디어 봇의 경우 메시징 봇보다 많은 계산 및 네트워크(대역폭) 용량이 필요하며 운영 비용이 훨씬 더 높을 수 있습니다. 실시간 미디어 봇 개발자는 봇의 확장성을 신중하게 측정하고, 봇이 관리할 수 있는 것보다 많은 동시 호출을 허용하지 않는지 확인해야 합니다. 비디오 사용 봇은 CPU 코어당 하나 또는 두 개의 동시 실시간 미디어 호출만 지속할 수 있습니다.

- 봇용 실시간 미디어 플랫폼의 현재 미리 보기 릴리스에는 봇 개발자가 알고 있어야 하는 특정 확장성 제한 사항이 있습니다(이러한 제한 사항은 향후 릴리스에서 개선될 수 있음). 
  1. 단일 VM 인스턴스는 한 번에 10개가 넘는 비디오 소켓을 만들 수 있습니다.
  2. 현재 실시간 미디어 플랫폼은 H.264 비디오 인코딩/디코딩을 오프로드하는 데 VM에서 사용 가능한 GPU(그래픽 처리 장치)를 활용하지 않습니다. 대신에 CPU의 소프트웨어에서 비디오 인코딩 및 디코딩을 수행합니다. GPU를 사용할 수 있는 경우, 봇은 고유한 그래픽 렌더링에 해당 GPU를 활용할 수 있습니다(예: 봇이 3D 그래픽 엔진을 사용하는 경우).

- 실시간 미디어 봇을 호스트하는 VM 인스턴스에는 2개 이상의 CPU 코어가 있어야 합니다. Azure의 경우 Dv2 시리즈 가상 머신이 권장됩니다. Azure VM 형식에 대한 자세한 내용은 <a href="/azure/virtual-machines/windows/sizes-general" target="_blank">Azure 문서</a>를 참조하세요. 
