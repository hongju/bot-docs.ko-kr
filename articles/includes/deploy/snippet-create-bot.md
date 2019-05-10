---
ms.openlocfilehash: f95c8a37b1207a26dab0a714b86412a9dba2dcb4
ms.sourcegitcommit: 4ff7a8772124a567f43e2c3e13aded368c4002e3
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/03/2019
ms.locfileid: "65035763"
---
```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name>
```

| 옵션 | 설명 |
|:---|:---|
| --name | Azure에서 봇을 배포하는 데 사용되는 고유 이름입니다. 로컬 봇과 동일한 이름일 수도 있습니다. 이름에 공백 또는 밑줄을 넣지 마세요. |
| --location | 봇 서비스 리소스를 만드는 데 사용되는 지리적 위치입니다. 예를 들어 `eastus`, `westus`, `westus2` 등입니다. |
| --lang | 봇을 만드는 데 사용할 언어는 `Csharp` 또는 `Node`입니다. 기본값은 `Csharp`입니다. |
| --resource-group | 봇을 만들 리소스 그룹의 이름입니다. `az configure --defaults group=<name>`을 사용하여 기본 그룹을 구성할 수 있습니다. |