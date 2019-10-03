---
title: チャネル リファレンス
description: Bot Framework チャネル リファレンス
keywords: チャネル リファレンス, bot builder チャネル, bot framework チャネル
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 03/01/2019
ms.openlocfilehash: ffb7864eabecd6aa509e2b347f3df48985d00584
ms.sourcegitcommit: e9cd857ee11945ef0b98a1ffb4792494dfaeb126
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/01/2019
ms.locfileid: "71693140"
---
# <a name="categorized-activities-by-channel"></a>チャネル別に分類されたアクティビティ

以下の表は、どのようなイベント (ネットワーク上のアクティビティ) がどのチャネルから発生する可能性があるかを示しています。

各表の凡例を次に示します。

シンボル              | 意味
:------------------:|:------------------------------------------------
:white_check_mark:  |ボットが受信することが予想されるアクティビティ
:x:                 |ボットが受信することの**ない**アクティビティ
:white_large_square:|ボットが受信できるかどうか、現時点では未定

アクティビティは、意味のある方法で別々のカテゴリに分けることができます。 カテゴリごとに、発生する可能性のあるアクティビティの表を以下に示します。

<a name="conversational"></a>会話
--------------

 \                      | Cortana            | Direct Line        | Direct Line (Web チャット) | Email              | Facebook           | GroupMe            | Kik                | Teams              | Slack              | Skype   | Skype Business | Telegram | Twilio  
:---------------------- | :-----:            | :----------------: | :--------------------: |:----:              | :------:           | :-----:            | :-----:            | :---:              | :---:              | :---:   | :------------: | :------: | :----:  
Message                 | :white_check_mark: | :white_check_mark: | :white_check_mark:     | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark:        | :white_check_mark:  | :white_check_mark: 
MessageReaction         | :x:                | :x:                | :x:                    | :x:                | :x:                | :x:                | :x:                | :white_check_mark: | :x:                | :x:      | :x:             | :x:       | :x:      

- すべてのチャネルがメッセージ アクティビティを送信します。
- ダイアログを使用する場合、一般に、Message アクティビティは常にダイアログに渡す必要があります。
- MessageReaction は会話の非常に大きな部分を占めていますが、これは MessageReaction には当てはまらないと考えられます。
- 論理的に 2 種類の MessageReaction があります (Added と Removed)。


> [!TIP]
> "メッセージの反応" は、前のコメントに対する "サムズアップ (賛成、OK、グッドなど)" のようなものです。 これらは順不同で発生する可能性があるため、ボタンと同じように考えることができます。 現在、このアクティビティは Teams チャネルによって送信されます。  


<a name="welcome"></a>ようこそ
-------

 \                         | Cortana            | Direct Line        | Direct Line (Web チャット) | Email   | Facebook             | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Business | Telegram | Twilio  
:----------------------    | :-----:            | :---------:        | :--------------------: |:----:   | :------:             | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
ConversationUpdate         | :white_check_mark: | :white_check_mark: | :white_check_mark:     | :x:     | :white_large_square: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :x:      | :x:             | :white_check_mark:  | :x:     
ContactRelationUpdate      | :x:                | :x:                | :x:                    | :x:     | :x:                  | :x:      | :x:      | :x:      | :x:      | :white_check_mark: | :white_check_mark:        | :x:       | :x:     

- チャネルが ConversationUpdate アクティビティを送信するのが一般的です。
- 論理的に 2 種類の MessageReaction があります (Added と Removed)。
- ConversationUpdate.Added につなぐことで、ボットの "ウェルカム" 動作を簡単に実装できると想定しがちであり、これが機能する場合もあります。
- しかし、これは単純化されたものであり、信頼性のある "ウェルカム" 動作を生成するために、ボット実装で状態を使用することが必要な場合もあります。


<a name="application-extensibility"></a>アプリケーションの機能拡張
-------------------------

 \                      | Cortana            | Direct Line        | Direct Line (Web チャット) | Email   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Business | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Event.*                    | :white_large_square: | :white_check_mark: | :white_check_mark:    | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.CreateConversation   | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.ContinueConversation | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  

- Event アクティビティは、Direct Line ("_Web チャットとも呼ばれます_") の機能拡張メカニズムです。
- クライアントとサーバーの両方を所有するアプリケーションでは、この Event アクティビティを使用して、サービスを介して独自のイベントをトンネリングできます。


<a name="microsoft-teams"></a>Microsoft Teams
------------------

 \                      | Cortana            | Direct Line        | Direct Line (Web チャット) | Email   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Business | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Invoke.TeamsVerification   | :x:      | :x:          | :x: | :x:   | :x:       | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
Invoke.ComposeResponse     | :x:      | :x:          | :x: | :x:   | :x:       | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     

- Microsoft Teams では、他の型指定された多数のアクティビティに加え、Teams 固有の Invoke アクティビティがいくつか定義されています。
- Invoke アクティビティはアプリケーションに固有のものであり、クライアントが定義するものではありません。
- アクティビティの特定のサブタイプを呼び出すという一般的概念はありません。
- 現在、ボット上で要求/応答動作をトリガーするアクティビティは Invoke だけです。

これは非常に重要です。ダイアログを使用して OAuth プロンプトを動作させる場合、Invoke.TeamsVerification アクティビティをダイアログに転送する必要があります。


<a name="message-update"></a>メッセージの更新
--------------

 \                      | Cortana            | Direct Line        | Direct Line (Web チャット) | Email   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Business | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
MessageUpdate | :x:      | :x:          | :x:    | :x: | :x:      | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
MessageDelete | :x:      | :x:          | :x:    | :x: | :x:      | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     

- メッセージの更新は、Teams で現在サポートされています。


<a name="oauth"></a>OAuth
-------

 \                      | Cortana            | Direct Line        | Direct Line (Web チャット) | Email   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Business | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Event.TokenResponse| :white_large_square:  | :white_check_mark:   | :white_check_mark:    | :x:    | :white_large_square: | :white_large_square: | :white_large_square: | :x:    | :white_large_square: | :white_large_square: | :white_large_square:       | :white_large_square: | :white_large_square: 

これは非常に重要です。ダイアログを使用して OAuth プロンプトを動作させる場合、Event.TokenResponse アクティビティをダイアログに転送する必要があります。


<a name="uncategorized"></a>未分類 
-------------

 \                      | Cortana  | Direct Line        | Direct Line (Web チャット) | Email | Facebook | GroupMe | Kik     | Teams | Slack | Skype | Skype Business | Telegram | Twilio  
:---------------------- | :-----:  | :---------:        | :--------------------: |:----: | :------: | :-----: | :-----: | :---: | :---: | :---: | :------------: | :------: | :----:  
EndOfConversation       | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
InstallationUpdate      | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Typing (入力)                  | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Handoff                 | :x:      | :x:                | :x:                    | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     


<a name="out-of-use-includes-payment-specific-invoke"></a>使用されていないアクティビティ (支払い固有の呼び出しを含む)
---------------------------------------------
- DeleteUserData 
- Invoke.PaymentRequest  
- Invoke.Address
- ping

---

## <a name="summary-of-activities-supported-per-channel"></a>各チャネルでサポートされているアクティビティの概要

<a name="cortana"></a>Cortana
-------
- Message
- ConversationUpdate
- _Event.TokenResponse_
- _EndOfConversation (ウィンドウが閉じたとき?)_

<a name="direct-line"></a>Direct Line
--------
- Message
- ConversationUpdate
- Event.TokenResponse
- Event.*
- _Event.CreateConversation_
- _Event.ContinueConversation_

<a name="email"></a>Email
-----
- Message

<a name="facebook"></a>Facebook
--------
- Message
- _Event.TokenResponse_

<a name="groupme"></a>GroupMe
-------
- Message
- ConversationUpdate
- _Event.TokenResponse_

<a name="kik"></a>Kik
---
- Message
- ConversationUpdate
- _Event.TokenResponse_

<a name="teams"></a>Teams
-----
- Message
- ConversationUpdate
- MessageReaction
- MessageUpdate
- MessageDelete
- Invoke.TeamsVerification
- Invoke.ComposeResponse


<a name="slack"></a>Slack
-----
- Message
- ConversationUpdate
- _Event.TokenResponse_


<a name="skype"></a>Skype
-----
- Message
- ContactRelationUpdate
- _Event.TokenResponse_


<a name="skype-business"></a>Skype Business
--------------
- Message
- ContactRelationUpdate 
- _Event.TokenResponse_


<a name="telegram"></a>Telegram
--------
- Message
- ConversationUpdate
- _Event.TokenResponse_


<a name="twilio"></a>Twilio
------
- Message

## <a name="summary-table-all-activities-to-all-channels"></a>すべてのチャネルに対するすべてのアクティビティの概要表

 \                         | Cortana              | Direct Line          | Direct Line (Web チャット) | Email                | Facebook             | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Business | Telegram | Twilio  
:----------------------    | :-----:              | :---------:          | :--------------------: |:----:                | :------:             | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Message                    | :white_check_mark:   | :white_check_mark:   | :white_check_mark:     | :white_check_mark:   | :white_check_mark:   | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark:        | :white_check_mark:  | :white_check_mark: 
MessageReaction            | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:      | :x:      | :x:             | :x:       | :x:      
ConversationUpdate         | :white_check_mark:   | :white_check_mark:   | :white_check_mark:     | :x:                  | :white_large_square: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :x:      | :x:             | :white_check_mark:  | :x:     
ContactRelationUpdate      | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :x:      | :x:      | :white_check_mark: | :white_check_mark:        | :x:       | :x:     
Event.*                    | :white_large_square: | :white_check_mark:   | :white_check_mark:     | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.CreateConversation   | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.ContinueConversation | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Invoke.TeamsVerification   | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
Invoke.ComposeResponse     | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
MessageUpdate              | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
MessageDelete              | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
Event.TokenResponse        | :white_large_square: | :white_check_mark:   | :white_check_mark:     | :x:                  | :white_large_square: | :white_large_square: | :white_large_square: | :x:    | :white_large_square: | :white_large_square: | :white_large_square:       | :white_large_square: | :white_large_square: 
EndOfConversation          | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
InstallationUpdate         | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Typing (入力)                     | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Handoff                    | :x:                  | :x:                  | :x:                    | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     

## <a name="web-chat"></a>Web チャット 
Web チャットは以下を送信します。
- "message": "text" や "attachments" が含まれます。
- "event": "name" と "value" が (JSON/文字列として) 含まれます。
- "typing": ユーザーがオプション ("sendTypingIndicator") を設定した場合、Web チャットは "contactRelationUpdate" を送信しなくなります。 また、Web チャットでは "messageReaction" はサポートされていません。Microsoft はお客様からこの機能のサポートを明示的に要求されたことはありません。

既定では、Web チャットは以下をレンダリングします。
- "message": アクティビティのオプションに応じて、カルーセルまたはスタックとしてレンダリングします。
- "typing": 5 秒間レンダリングして非表示にするか、次のアクティビティが発生するまでレンダリングします。
- "conversationUpdate": 非表示になります。
- "event": 非表示になります。
- その他: 警告ボックスを表示します (運用環境で表示されたことはありません)。このレンダリング パイプラインを変更して、カスタム レンダリングを追加、削除、または置換できます。

Web チャットを使用して任意の種類のアクティビティとペイロードを送信できますが、この機能は文書化されておらず、推奨もされていません。 代わりに "event" アクティビティを使用してください。

## <a name="action-support-by-channel"></a>チャネル別のアクションのサポート

次の表に、カード アクションと推奨されるアクションに対するチャネル別のサポートを示します。

 \                      | Cortana  | Direct Line | Direct Line (Web チャット) | Email | Facebook | GroupMe |   Kik   | Line  | Teams | Slack | Skype | Skype Business | Telegram | Twilio  
:---------------------- | :-----:  | :---------: | :--------------------: |:----: | :------: | :-----: | :-----: | :---: | :---: | :---: | :---: | :------------: | :------: | :----:  
Suggested Actions (推奨されるアクション)       |    0     |     100     |          100           |   0   |    10    |    0    |   20    |  13   |   0   |  100  |  10   |       0        |    100   |   0     
カード アクション            |   100    |     100     |          100           |   0   |     3    |    0    |   20    |  99   |   3   |  100  |   3   |       0        |     0    |   0     
