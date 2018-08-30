---
title: Bot Builder SDK for Python を使用してボットを作成する | Microsoft Docs
description: Bot Builder SDK for Python を使用してボットをすばやく作成します。
keywords: Bot Builder SDK, ボットの作成, クイック スタート, Python, 使用の開始
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 02/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ba16adebe6bbb9b79949cd9842e975e35c3f2aa6
ms.sourcegitcommit: d486dd088b87a44fc8142f7a08877ff993861a42
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/26/2018
ms.locfileid: "42928411"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-python"></a>Bot Builder SDK for Python を使用したボットの作成
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Bot Builder SDK for Python は、ボットを開発するための使いやすいフレームワークです。 このクイック スタートでは、ボットを構築し、Bot Framework Emulator でテストします。 SDK v4 はプレビュー段階にあります。詳細については、Python [GitHub リポジトリ](https://github.com/Microsoft/botbuilder-python)を参照してください。 

## <a name="pre-requisite"></a>前提条件
- [Python 3.6.4](https://www.python.org/downloads/) 
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases)

# <a name="create-a-bot"></a>ボットの作成
main.py ファイルに、次の標準モジュールをインポートします。

```python
import http.server
import json
import asyncio
```

そして次の SDK モジュールをインポートします。
```python
from botbuilder.schema import (Activity, ActivityTypes)
from botframework.connector import ConnectorClient
from botframework.connector.auth import (MicrosoftAppCredentials,
                                         JwtTokenValidation, SimpleCredentialProvider)
```
次に、以下のコードを追加し、ConnectorClient を使用してボットを作成します。
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
            loop.run_until_complete(JwtTokenValidation.assert_valid_activity(
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


main.py を保存します。 Windows でサンプルを実行するには、コマンド ライン ウィンドウに次のように入力します。
```
python main.py
```
ローカル ターミナルに 'Started http server on localhost:9000' (localhost:9000 で http サーバーが開始されました) ローカル ターミナルに、'Started http server on localhost:9000' (localhost:9000 で http サーバーが開始されました) というメッセージが表示されます。

### <a name="start-the-emulator-and-connect-your-bot"></a>エミュレーターの起動とボットの接続

次に、エミュレーターを起動し、エミュレーターのボットに接続します。


1. エミュレーターの [ようこそ] タブにある **[新しいボット構成を作成する]** リンクをクリックします。 

2. **ボット名**を入力してから、ボット コードへのディレクトリ パスを入力します。 ボットの構成ファイルはこのパスに保存されます。

3. **[エンドポイント URL]** フィールドに `http://localhost:port-number/api/messages` と入力します。*port-number* は、アプリケーションを実行しているブラウザーに示されているポート番号と同じにします。

4. **[接続]** をクリックしてボットに接続します。 **[Microsoft アプリ ID]** と **[Microsoft アプリ パスワード]** を指定する必要はありません。 現段階では、これらのフィールドは空白のままにしておくことができます。 この情報は、後ほど、ボットを登録するときに取得します。

エミュレーターに「**Hello**」(こんにちは) と入力すると、ボットは「**You said "Hello"**」("こんにちは" と言いましたね) と応答します。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [ボットの基本的な概念](../v4sdk/bot-builder-basics.md)
