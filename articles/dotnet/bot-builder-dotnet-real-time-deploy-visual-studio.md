---
redirect_url: https://aka.ms/realTimeMediaCalling-repo
ms.openlocfilehash: 1df0192632cdb9b35259b8ce1ec5c8b3be46c750
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032945"
---
<a name="--"></a><!--
---
title:Skype リアルタイム メディア ボットを Azure にデプロイする | Microsoft Docs description:Visual Studio の組み込みの発行機能を使用して、Skype リアルタイム音声/動画ボットを Azure にデプロイする方法について説明します。
作成者:MalarGit ms.author: malarch manager: ssulzer ms.topic: article ms.service: bot-service ms.subservice: sdk ms.date:12/13/17 monikerRange: 'azure-bot-service-3.0'
---

# <a name="deploy-a-real-time-media-bot-from-visual-studio-to-azure"></a>Visual Studio から Azure へのリアルタイム メディア ボットのデプロイ

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

リアルタイム メディア ボットは、"IaaS" Azure 仮想マシンまたは "クラシック" Azure クラウド サービスのいずれかでホストできます。 この記事では、Visual Studio の組み込みの発行機能を使用して、Azure クラウド サービス worker ロールでホストされているボットを Visual Studio からデプロイする方法について説明します。

## <a name="prerequisites"></a>前提条件

ボットを Azure にデプロイするには、Microsoft Azure サブスクリプションが必要です。 サブスクリプションがない場合は、<a href="https://azure.microsoft.com/en-us/free/" target="_blank">無料アカウント</a>に登録できます。 また、この記事で説明するプロセスには Visual Studio が必要です。 Visual Studio がない場合は、<a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2017 Community</a> を無料でダウンロードできます。

### <a name="certificate-from-a-valid-certificate-authority"></a>有効な証明機関の証明書
信頼できる証明機関 (CA) の有効な証明書でボットを構成する必要があります。 証明書のサブジェクト名 (SN) またはサブジェクト代替名 (SAN) の最後のエントリが、クラウド サービスの名前である必要があります。 ワイルドカード証明書は現在サポートされていません。 CNAME を使用してクラウド サービスを参照する場合、CNAME は証明書の SN または最後の SAN エントリである必要があります。

## <a name="configure-application-settings"></a>アプリケーション設定の構成
ボットがクラウドで正常に機能するように、アプリケーション設定が正しいことを確認する必要があります。 具体的には、worker ロールの app.config ファイルに次のキー値を設定します。
> <ul><li>MicrosoftAppId</li><li>MicrosoftAppPassword</li></ul>

> [!NOTE]
> ボットの **AppID** と **AppPassword** を確認する方法については、「[MicrosoftAppID and MicrosoftAppPassword (MicrosoftAppID と MicrosoftAppPassword)](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)」をご覧ください。

## <a name="create-worker-role-in-the-azure-portal"></a>Azure portal での worker ロールの作成
### <a name="step-1-create-cloud-serviceclassic"></a>手順 1:クラウド サービス (クラシック) を作成する
<a href="https://portal.azure.com">Azure portal</a> にログオンします。 画面の左側の **+** をクリックし、**[Cloud Services (クラシック)]** を選択します。 必要な情報をフォームに入力し、**[作成]** をクリックします。

![クラウド サービスを作成する](../media/real-time-media-bot-portal-service-creation.png)

> [!NOTE]
> ボット登録の URL には、ボットの DNS 名を指定する必要があります。

### <a name="step-2-upload-the-certificate-for-the-bot"></a>手順 2:ボットの証明書をアップロードする
ボットが作成されたら、ボットの証明書をアップロードします。

![証明書のアップロード](../media/real-time-media-bot-portal-certificates.png)

## <a name="modify-service-configuration-with-worker-role-details"></a>worker ロールの詳細によるサービス構成の変更
ボットの完全修飾ドメイン名 (FQDN) は、Azure RoleEnvironment API では提供されません。 そのため、ボットは FQDN で提供する必要があります。 また、ボットは HTTPS の証明書について認識しておく必要があります。 これらは、worker ロールのサービス構成 (.cscfg) ファイルで構成できます。

> [!TIP]
> BotBuilder-RealTimeMediaCalling Git リポジトリからサンプルをデプロイする場合
> - $DnsName$ をクラウド サービス名または CNAME (サービス構成で使用されている場合) に置き換えます。
>   ```xml
>      <Setting name="ServiceDnsName" value="$DnsName$" />
>   ```
> 
> - 構成の次の行で、$CertThumbprint$ をボットにアップロードした証明書の拇印に置き換えます。
>   ```xml
>      <Setting name="DefaultCertificate" value="$CertThumbprint$" />
>      <Certificate name="Default" thumbprint="$CertThumbprint$" thumbprintAlgorithm="sha1" />
>   ```

## <a name="publish-the-bot-from-visual-studio"></a>Visual Studio からボットを発行する
### <a name="step-1-launch-the-microsoft-azure-publishing-wizard-in-visual-studio"></a>手順 1:Visual Studio で Microsoft Azure 発行ウィザードを起動する

Visual Studio でプロジェクトを開きます。 ソリューション エクスプローラーで、クラウド サービス プロジェクトを右クリックし、**[発行]** を選択します。 これにより、Microsoft Azure 発行ウィザードが起動します。 資格情報を使用して適切なサブスクリプションにサインインします。

![プロジェクトを右クリックし、[発行] を選択して Microsoft Azure 発行ウィザードを起動する](../media/real-time-media-bot-publish-signin.png)

### <a name="step-2-publish-the-bot"></a>手順 2:ボットを発行する

**[次へ]** をクリックします。 **[設定]** タブが開きます。ボットをデプロイするためのクラウド サービス、環境、ビルド構成、サービス構成を指定します。

![[次へ] をクリックし、[設定] タブに移動する](../media/real-time-media-bot-publish-settings.png)

必要に応じて **[詳細設定]** を選択し、(問題のデバッグに使用できる) デプロイ ログ用のストレージ アカウントを指定することもできます。

![[詳細設定] タブをクリックする](../media/real-time-media-bot-publish-advanced-settings.png)

**[概要]** タブで構成を確認し、**[発行]** をクリックしてボットを Microsoft Azure にデプロイします。
-->