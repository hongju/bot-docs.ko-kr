---
ms.openlocfilehash: 14d9632ad578014a36b5f13e6dee883e2a6e1722
ms.sourcegitcommit: a47183f5d1c2b2454c4a06c0f292d7c075612cdd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/19/2019
ms.locfileid: "67252648"
---
1. [**애플리케이션 등록 포털**](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade)로 이동합니다.
1. **앱 추가**를 클릭하여 애플리케이션을 등록하고, **애플리케이션 ID**를 만들고, **새 암호 생성**을 수행합니다. 애플리케이션 및 암호가 이미 있지만 암호가 기억 나지 않는 경우 애플리케이션 비밀 섹션에서 새 암호를 생성해야 합니다.
1. `az bot create` 명령에 사용할 수 있도록 방금 생성한 애플리케이션 ID와 새 암호를 모두 저장합니다.  

```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name> --appid "<application-id>" --password "<application-password>" --verbose
```

| 옵션 | 설명 |
|:---|:---|
| --name | Azure에서 봇을 배포하는 데 사용되는 고유 이름입니다. 로컬 봇과 동일한 이름일 수도 있습니다. 이름에 공백 또는 밑줄을 넣지 마세요. |
| --location | 봇 서비스 리소스를 만드는 데 사용되는 지리적 위치입니다. 예를 들어 `eastus`, `westus`, `westus2` 등입니다. |
| --lang | 봇을 만드는 데 사용할 언어는 `Csharp` 또는 `Node`입니다. 기본값은 `Csharp`입니다. |
| --resource-group | 봇을 만들 리소스 그룹의 이름입니다. `az configure --defaults group=<name>`을 사용하여 기본 그룹을 구성할 수 있습니다. |
| --appid | 봇에서 사용할 Microsoft 계정 ID(MSA ID)입니다. |
| --password | 봇의 Microsoft 계정(MSA) 암호입니다. |
