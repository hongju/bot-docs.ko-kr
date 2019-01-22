---
ms.openlocfilehash: 53db401b00e53b964027f08c32ca7a38a4915f60
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360915"
---
```cmd
az group create --name <resource-group-name> --location <geographic-location> --verbose
```

| 옵션 | 설명 |
|:---|:---|
| --name | 리소스 그룹의 고유한 이름입니다. 이름에 공백 또는 밑줄을 넣지 마세요. |
| --location | 리소스 그룹을 만드는 데 사용되는 지리적 위치입니다. 예를 들어 `eastus`, `westus`, `westus2` 등입니다. 위치 목록에는 `az account list-locations`를 사용합니다. |