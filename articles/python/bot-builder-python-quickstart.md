---
title: Python용 Bot Builder SDK를 사용하여 봇 만들기 | Microsoft Docs
description: Python용 Bot Builder SDK를 사용하여 봇을 빠르게 만듭니다.
keywords: Bot Builder SDK, 봇 만들기, 빠른 시작, Python, 시작
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 08/30/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4536bef820bc1e6e21ba2905fb643fe5608b3788
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999020"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-python"></a>Python용 Bot Builder SDK를 사용하여 봇 만들기

>[!NOTE] 
> Python SDK는 **미리 보기**로 제공되고 있습니다. 자세한 내용은 Python [GitHub 리포지토리](https://github.com/Microsoft/botbuilder-python)를 참조하세요. 

이 빠른 시작에서는 봇을 빌드한 다음, Bot Framework Emulator를 사용하여 테스트하는 과정을 안내합니다. 

## <a name="pre-requisite"></a>필수 구성 요소
- [Python 3.6.4](https://www.python.org/downloads/) 
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases)

## <a name="create-a-bot"></a>봇 만들기
main.py 파일에서 다음과 같은 표준 모듈을 가져옵니다.

```python
import http.server
import json
import asyncio
```

다음 SDK 모듈도 가져옵니다.
```python
from botbuilder.schema import (Activity, ActivityTypes)
from botframework.connector import ConnectorClient
from botframework.connector.auth import (MicrosoftAppCredentials,
                                         JwtTokenValidation, SimpleCredentialProvider)
```
다음 코드를 추가하여 ConnectorClient를 사용해서 봇을 만듭니다.
```python
APP_ID = ''
APP_PASSWORD = ''


class BotRequestHandler(http.server.BaseHTTPRequestHandler):

    @staticmethod
    def __create_reply_activity(request_activity, text):
        return Activity(
            type=ActivityTypes.message,
            channel_id=request_activity.channel_id,
            conversation=request_activity.conversation,
            recipient=request_activity.from_property,
            from_property=request_activity.recipient,
            text=text,
            service_url=request_activity.service_url)

    def __handle_conversation_update_activity(self, activity):
        self.send_response(202)
        self.end_headers()
        if activity.members_added[0].id != activity.recipient.id:
            credentials = MicrosoftAppCredentials(APP_ID, APP_PASSWORD)
            reply = BotRequestHandler.__create_reply_activity(activity, 'Hello and welcome to the echo bot!')
            connector = ConnectorClient(credentials, base_url=reply.service_url)
            connector.conversations.send_to_conversation(reply.conversation.id, reply)

    def __handle_message_activity(self, activity):
        self.send_response(200)
        self.end_headers()
        credentials = MicrosoftAppCredentials(APP_ID, APP_PASSWORD)
        connector = ConnectorClient(credentials, base_url=activity.service_url)
        reply = BotRequestHandler.__create_reply_activity(activity, 'You said: %s' % activity.text)
        connector.conversations.send_to_conversation(reply.conversation.id, reply)

    def __handle_authentication(self, activity):
        credential_provider = SimpleCredentialProvider(APP_ID, APP_PASSWORD)
        loop = asyncio.new_event_loop()
        try:
            loop.run_until_complete(JwtTokenValidation.authenticate_request(
                activity, self.headers.get("Authorization"), credential_provider))
            return True
        except Exception as ex:
            self.send_response(401, ex)
            self.end_headers()
            return False
        finally:
            loop.close()

    def __unhandled_activity(self):
        self.send_response(404)
        self.end_headers()

    def do_POST(self):
        body = self.rfile.read(int(self.headers['Content-Length']))
        data = json.loads(str(body, 'utf-8'))
        activity = Activity.deserialize(data)

        if not self.__handle_authentication(activity):
            return

        if activity.type == ActivityTypes.conversation_update.value:
            self.__handle_conversation_update_activity(activity)
        elif activity.type == ActivityTypes.message.value:
            self.__handle_message_activity(activity)
        else:
            self.__unhandled_activity()


try:
    SERVER = http.server.HTTPServer(('localhost', 9000), BotRequestHandler)
    print('Started http server on localhost:9000')
    SERVER.serve_forever()
except KeyboardInterrupt:
    print('^C received, shutting down server')
    SERVER.socket.close()
```


main.py를 저장합니다. Windows에서 샘플을 실행하려면 명령줄 창에 다음을 입력합니다.
```
python main.py
```
로컬 터미널에 ‘Started http server on localhost:9000’ 메시지가 표시됩니다.

## <a name="start-the-emulator-and-connect-your-bot"></a>에뮬레이터 시작 및 봇 연결

에뮬레이터를 시작한 다음, 에뮬레이터에서 봇에 연결합니다.

1. 에뮬레이터 "시작" 탭에서 **봇 열기** 링크를 클릭합니다. 
2. 프로젝트를 만든 디렉터리에 있는 .bot 파일을 선택합니다.

## <a name="interact-with-your-bot"></a>봇과의 상호 작용

봇에 메시지를 보내면 봇이 메시지를 통해 응답합니다.
![에뮬레이터 실행](../media/emulator-v4/emulator-running.png)


## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [봇 개념](../v4sdk/bot-builder-basics.md)
