---
title: Java용 Bot Builder SDK를 사용하여 봇 만들기 | Microsoft Docs
description: Java용 Bot Builder SDK를 사용하여 봇을 빠르게 만듭니다.
keywords: Bot Builder SDK, 봇 만들기, 빠른 시작, Java, 시작
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/02/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ca4714d3b3988fd08021f55a4905d9426996b7eb
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301907"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-java"></a>Java용 Bot Builder SDK를 사용하여 봇 만들기
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Java용 Bot Builder SDK는 Java 개발자가 봇을 작성할 수 있는 친숙한 방법을 제공합니다. SDK v4는 미리 보기로 제공됩니다. 자세한 내용은 Java [GitHub 리포지토리](https://github.com/Microsoft/botbuilder-java)를 참조하세요.

> [!NOTE]
> 코드 샘플 및 설명서는 현재 Java 버전 1.8을 대상으로 합니다.

## <a name="getting-started"></a>시작하기

v4 SDK는 일련의 [라이브러리](https://github.com/Microsoft/botbuilder-java/tree/master/libraries)로 구성됩니다. 로컬로 빌드하려면 [SDK 빌드](https://github.com/Microsoft/botbuilder-java/wiki/building-the-sdk)를 참조하세요.

- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases) 설치

### <a name="create-echobot"></a>EchoBot 만들기

App.java 파일에서서 다음을 추가합니다.

```Java
package com.microsoft.bot.connector.sample;

import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.microsoft.aad.adal4j.AuthenticationException;
import com.microsoft.bot.connector.customizations.CredentialProvider;
import com.microsoft.bot.connector.customizations.CredentialProviderImpl;
import com.microsoft.bot.connector.customizations.JwtTokenValidation;
import com.microsoft.bot.connector.customizations.MicrosoftAppCredentials;
import com.microsoft.bot.connector.implementation.ConnectorClientImpl;
import com.microsoft.bot.schema.models.Activity;
import com.microsoft.bot.schema.models.ActivityTypes;
import com.microsoft.bot.schema.models.ResourceResponse;
import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import com.sun.net.httpserver.HttpServer;

import java.io.IOException;
import java.io.InputStream;
import java.net.InetSocketAddress;
import java.net.URLDecoder;
import java.util.logging.Level;
import java.util.logging.Logger;

public class App {
    private static final Logger LOGGER = Logger.getLogger( App.class.getName() );
    private static String appId = "";       // <-- app id -->
    private static String appPassword = ""; // <-- app password -->

    public static void main( String[] args ) throws IOException {
        CredentialProvider credentialProvider = new CredentialProviderImpl(appId, appPassword);
        HttpServer server = HttpServer.create(new InetSocketAddress(3978), 0);
        server.createContext("/api/messages", new MessageHandle(credentialProvider));
        server.setExecutor(null);
        server.start();
    }

    static class MessageHandle implements HttpHandler {
        private ObjectMapper objectMapper;
        private CredentialProvider credentialProvider;
        private MicrosoftAppCredentials credentials;

        MessageHandle(CredentialProvider credentialProvider) {
            this.objectMapper = new ObjectMapper()
                    .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
                    .findAndRegisterModules();
            this.credentialProvider = credentialProvider;
            this.credentials = new MicrosoftAppCredentials(appId, appPassword);
        }

        public void handle(HttpExchange httpExchange) throws IOException {
            if (httpExchange.getRequestMethod().equalsIgnoreCase("POST")) {
                Activity activity = getActivity(httpExchange);
                String authHeader = httpExchange.getRequestHeaders().getFirst("Authorization");
                try {
                    JwtTokenValidation.assertValidActivity(activity, authHeader, credentialProvider);

                    // send ack to user activity
                    httpExchange.sendResponseHeaders(202, 0);
                    httpExchange.getResponseBody().close();

                    if (activity.type().equals(ActivityTypes.MESSAGE)) {
                        // reply activity with the same text
                        ConnectorClientImpl connector = new ConnectorClientImpl(activity.serviceUrl(), this.credentials);
                        ResourceResponse response = connector.conversations().sendToConversation(activity.conversation().id(),
                                new Activity()
                                        .withType(ActivityTypes.MESSAGE)
                                        .withText("Echo: " + activity.text())
                                        .withRecipient(activity.from())
                                        .withFrom(activity.recipient())
                        );
                    }
                } catch (AuthenticationException ex) {
                    httpExchange.sendResponseHeaders(401, 0);
                    httpExchange.getResponseBody().close();
                    LOGGER.log(Level.WARNING, "Auth failed!", ex);
                } catch (Exception ex) {
                    LOGGER.log(Level.WARNING, "Execution failed", ex);
                }
            }
        }

        private String getRequestBody(HttpExchange httpExchange) throws IOException {
            StringBuilder buffer = new StringBuilder();
            InputStream stream = httpExchange.getRequestBody();
            int rByte;
            while ((rByte = stream.read()) != -1) {
                buffer.append((char)rByte);
            }
            stream.close();
            if (buffer.length() > 0) {
                return URLDecoder.decode(buffer.toString(), "UTF-8");
            }
            return "";
        }

        private Activity getActivity(HttpExchange httpExchange) {
            try {
                String body = getRequestBody(httpExchange);
                LOGGER.log(Level.INFO, body);
                return objectMapper.readValue(body, Activity.class);
            } catch (Exception ex) {
                LOGGER.log(Level.WARNING, "Failed to get activity", ex);
                return null;
            }

        }
    }
}
```

Maven을 사용하는 경우 이 리포지토리의 샘플 폴더에서 pom.xml 파일을 복사할 수 있습니다. 실행 파일 실행을 시작한 후에는 Bot Framework Emulator를 시작합니다.

### <a name="start-the-emulator-and-connect-your-bot"></a>에뮬레이터 시작 및 봇 연결

이때 봇은 로컬에서 실행됩니다.
에뮬레이터를 시작한 다음, 에뮬레이터에서 봇에 연결합니다.

1. 에뮬레이터 “시작” 탭에서 **새 봇 구성 만들기** 링크를 클릭합니다. 

2. **봇 이름**을 입력하고 봇 코드에 대한 디렉터리 경로를 입력합니다. 봇 구성 파일은 이 경로에 저장됩니다.

3. **끝점 URL** 필드에 `http://localhost:port-number/api/messages`를 입력합니다. 여기서 *port-number*는 응용 프로그램이 실행되는 브라우저에 표시된 포트 번호와 일치합니다.

4. **연결**을 클릭하여 봇에 연결합니다. **Microsoft 앱 ID** 및 **Microsoft 앱 암호**를 지정하지 않아도 됩니다. 지금은 이러한 필드를 비워 둘 수 있습니다. 나중에 봇을 등록할 때 이 정보를 가져올 수 있습니다.

### <a name="interact-with-your-bot"></a>봇과의 상호 작용
봇에 "Hi"를 보내면 봇은 해당 메시지를 다시 에코합니다.

## <a name="next-steps"></a>다음 단계

이제 봇과 봇의 작동 방식을 설명하는 개념으로 이동합니다.

> [!div class="nextstepaction"]
> [기본 봇 개념](../v4sdk/bot-builder-basics.md)
