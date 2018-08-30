---
title: Bot Builder SDK for Java を使用してボットを作成する | Microsoft Docs
description: Bot Builder SDK for Java を使用してボットをすばやく作成します。
keywords: Bot Builder SDK、ボットの作成、クイック スタート、java、使用の開始
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/02/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3b618bfb7cd1a462390aee4d564778c8ec0a7247
ms.sourcegitcommit: d486dd088b87a44fc8142f7a08877ff993861a42
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/26/2018
ms.locfileid: "42928431"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-java"></a>Bot Builder SDK for Java を使用したボットの作成
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Bot Builder SDK for Java では、Java 開発者がボットを作成するための手慣れた方法を提供しています。 SDK v4 はプレビュー段階にあります。詳細については、Java [GitHub リポジトリ](https://github.com/Microsoft/botbuilder-java)を参照してください。

> [!NOTE]
> コード サンプルおよびドキュメントでは現在、Java バージョン 1.8 を対象にしています。

## <a name="getting-started"></a>Getting Started (概要)

v4 SDK は、一連の[ライブラリ](https://github.com/Microsoft/botbuilder-java/tree/master/libraries)で構成されています。 ローカルに作成するには、「[Building the SDK](https://github.com/Microsoft/botbuilder-java/wiki/building-the-sdk)」(SDK を作成する) を参照してください。

- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases) をインストールします

### <a name="create-echobot"></a>EchoBot を作成する

App.java ファイルに、以下を追加します。

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

Maven を使用している場合は、このリポジトリのサンプル フォルダーから pom.xml ファイルをコピーできます。 実行可能ファイルの実行を開始すると、Bot Framework Emulator が起動されます。

### <a name="start-the-emulator-and-connect-your-bot"></a>エミュレーターの起動とボットの接続

この時点では、ボットはローカルで実行されています。
次に、エミュレーターを起動し、エミュレーターのボットに接続します。

1. エミュレーターの [ようこそ] タブにある **[新しいボット構成を作成する]** リンクをクリックします。 

2. **ボット名**を入力してから、ボット コードへのディレクトリ パスを入力します。 ボットの構成ファイルはこのパスに保存されます。

3. **[エンドポイント URL]** フィールドに `http://localhost:port-number/api/messages` と入力します。*port-number* は、アプリケーションを実行しているブラウザーに示されているポート番号と同じにします。

4. **[接続]** をクリックしてボットに接続します。 **[Microsoft アプリ ID]** と **[Microsoft アプリ パスワード]** を指定する必要はありません。 現段階では、これらのフィールドは空白のままにしておくことができます。 この情報は、後ほど、ボットを登録するときに取得します。

### <a name="interact-with-your-bot"></a>ボットでのやり取り
ボットに「Hi」と送信すると、ボットはそのメッセージを再現して返します。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [ボットの基本的な概念](../v4sdk/bot-builder-basics.md)
