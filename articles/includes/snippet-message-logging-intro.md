---
ms.openlocfilehash: 12ead266dea859c84450e08ae12ed98d4952d698
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54226018"
---
Bot Framework SDK의 **미들웨어** 기능을 사용하면 봇이 사용자와 봇 간에 교환되는 모든 메시지를 가로채게 할 수 있습니다. 가로채는 각각의 메시지에 대해 메시지를 사용자가 지정한 데이터 저장소에 저장하거나, 대화 로그를 만들거나, 어떤 식으로 메시지를 검사하고, 코드에서 지정하는 작업을 수행하도록 선택할 수 있습니다. 

> [!NOTE]
> Bot Framework는 자동으로 대화 세부 정보를 저장하지 않습니다. 이렇게 하면 봇과 사용자가 외부와 공유하지 않으려는 비공개 정보를 수집할 가능성이 있기 때문입니다. 봇이 대화 세부 정보를 저장할 경우 해당 사용자에게 이 사실을 알리고 데이터를 어떻게 사용할 것인지 설명해야 합니다.