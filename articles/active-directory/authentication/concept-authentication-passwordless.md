---
title: Azure Active Directory へのパスワードレスのサインイン (プレビュー)
description: FIDO2 セキュリティ キーまたは Microsoft Authenticator アプリを使用した Azure Active Directory へのパスワードレスのサインイン オプションについて説明します
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 03/23/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: librown
ms.collection: M365-identity-device-management
ms.openlocfilehash: f4de359f60b556390b8271b728580ef66ae53ffa
ms.sourcegitcommit: 67bddb15f90fb7e845ca739d16ad568cbc368c06
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/28/2020
ms.locfileid: "82201722"
---
# <a name="passwordless-authentication-options-for-azure-active-directory"></a>Azure Active Directory のパスワードレス認証オプション

多要素認証 (MFA) は、組織をセキュリティで保護する優れた方法ですが、多くの場合、ユーザーはパスワードを覚えておく必要があるのに加え、セキュリティ層が増えることに不満を感じます。 パスワードが削除され、ユーザーが持っているものとユーザー自身または知っているものに置き換えられるため、パスワードレスの認証はより便利な方法です。

|   | ユーザーが持っているもの | ユーザー自身またはユーザーが知っているもの |
| --- | --- | --- |
| パスワードレス | Windows 10 デバイス、電話、またはセキュリティ キー | 生体認証または PIN |

認証については、組織ごとにさまざまなニーズがあります。 Microsoft では、Azure Active Directory (Azure AD) と統合された次の 3 つのパスワードレス認証オプションを提供しています。

- Windows Hello for Business
- Microsoft Authenticator アプリ
- FIDO2 セキュリティ キー

![認証: セキュリティと利便性](./media/concept-authentication-passwordless/passwordless-convenience-security.png)

## <a name="windows-hello-for-business"></a>Windows Hello for Business

Windows Hello for Business は、指定した Windows PC を所有しているインフォメーション ワーカーに最適です。 生体認証と PIN はユーザーの PC に直接関連付けられるため、所有者以外のユーザーからのアクセスは防止されます。 公開キー基盤 (PKI) 統合およびシングル サインオン (SSO) の組み込みサポートにより、Windows Hello for Business では、オンプレミスおよびクラウドの会社のリソースにシームレスにアクセスできる便利な方法を提供します。

![Windows Hello for Business を使用したユーザー サインインの例](./media/concept-authentication-passwordless/windows-hellow-sign-in.jpeg)

次の手順では、Azure Active Directory でのサインイン プロセスのしくみについて示します。

![Windows Hello for Business を使用したユーザー サインインに関連する手順の概要を示す図](./media/concept-authentication-passwordless/windows-hello-flow.png)

1. ユーザーは、生体認証または PIN のジェスチャを使用して Windows にサインインします。 このジェスチャによって、Windows Hello for Business の秘密キーのロックが解除され、*クラウド AP プロバイダー*というクラウド認証セキュリティ サポート プロバイダーに送信されます。
1. クラウド AP プロバイダーから Azure AD に nonce が要求されます。
1. Azure AD からは 5 分間有効な nonce が返されます。
1. クラウド AP プロバイダーでは、ユーザーの秘密キーを使用して nonce に署名され、署名済みの nonce が Azure AD に返されます。
1. Azure AD では、ユーザーの安全に登録された公開キーを使用して、署名済みの nonce が nonce の署名に対して検証されます。 署名の検証後、Azure AD では返された署名済み nonce が検証されます。 nonce が検証されると、Azure AD ではデバイスのトランスポート キーに暗号化されたセッション キーを使用してプライマリ更新トークン (PRT) が作成され、それがクラウド AP プロバイダーに返されます。
1. クラウド AP プロバイダーは、セッション キーと共に暗号化された PRT を受け取ります。 クラウド AP プロバイダーでは、デバイスのプライベート トランスポート キーを使用してセッション キーが復号化され、デバイスのトラステッド プラットフォーム モジュール (TPM) を使用してセッション キーが保護されます。
1. クラウド AP プロバイダーから Windows に成功した認証応答が返されます。 その後、ユーザーは再度認証を行う必要なく (SSO)、Windows とクラウドおよびオンプレミスのアプリケーションにアクセスできるようになります。

Windows Hello for Business の[計画ガイド](https://docs.microsoft.com/windows/security/identity-protection/hello-for-business/hello-planning-guide)を使用すると、Windows Hello for Business の展開の種類と、検討する必要がある選択肢を決定することができます。

## <a name="microsoft-authenticator-app"></a>Microsoft Authenticator アプリ

従業員の電話をパスワードレスの認証方法として使用できるようにします。 パスワードに加え、便利な多要素認証オプションとして Microsoft Authenticator アプリを既に使用されているかもしれません。 Authenticator アプリをパスワードレスのオプションとして使用することもできます。

![Microsoft Authenticator アプリを使用して Microsoft Edge にサインインする](./media/concept-authentication-passwordless/concept-web-sign-in-microsoft-authenticator-app.png)

Authenticator アプリは、あらゆる iOS や Android フォンを、強力なパスワードレスの資格情報に変えます。 ユーザーは、自分の電話で通知を受け取り、画面に表示される番号と電話の番号を照合してから、生体認証 (指紋または顔) あるいは PIN を使用して確認できます。 インストールの詳細については、「[Microsoft Authenticator アプリのダウンロードとインストール](https://docs.microsoft.com/azure/active-directory/user-help/user-help-auth-app-download-install)」を参照してください。

Authenticator アプリを使用したパスワードレス認証では、Windows Hello for Business と同じ基本的なパターンに従います。 これは、Azure AD によって使用されている Microsoft Authenticator アプリのバージョンを見つけられるように、ユーザーを識別する必要があるため、少し複雑になります。

![Microsoft Authenticator アプリでのユーザー サインインに関連する手順の概要を示す図](./media/concept-authentication-passwordless/authenticator-app-flow.png)

1. ユーザーは自分のユーザー名を入力します。
1. Azure AD では、ユーザーが強力な資格情報を持っていることが検出され、強力な資格情報フローが開始されます。
1. iOS デバイス上では Apple Push Notification Service (APNS) を介して、Android デバイス上では Firebase Cloud Messaging (FCM) を介して、通知がアプリに送信されます。
1. ユーザーはプッシュ通知を受け取り、アプリを開きます。
1. アプリでは Azure AD が呼び出され、プレゼンス証明のチャレンジと nonce を受け取ります。
1. ユーザーは、自分の生体認証または PIN を入力して秘密キーのロックを解除することでチャレンジを完了します。
1. nonce は秘密キーで署名され、Azure AD に返送されます。
1. Azure AD では公開キーと秘密キーの検証が実行され、トークンが返されます。

## <a name="fido2-security-keys"></a>FIDO2 セキュリティ キー

FIDO2 セキュリティ キーは、フォーム ファクターとして提供される可能性のある、フィッシングできない標準ベースのパスワードレス認証方法です。 Fast Identity Online (FIDO) は、パスワードレス認証のオープン標準です。 FIDO は、ユーザーおよび組織が標準を利用し、外部のセキュリティ キーまたはデバイスに組み込まれているプラットフォーム キーを使用して、ユーザー名やパスワードレスでリソースにサインインできるようにします。

パブリック プレビューでは、従業員はセキュリティ キーを使用して、Azure AD またはハイブリッド Azure AD に参加済みの Windows 10 デバイスにサインインし、クラウドおよびオンプレミス リソースへのシングル サインオンを行うことができます。 ユーザーは、サポートされているブラウザーにサインインすることもできます。 FIDO2 セキュリティ キーは、セキュリティに非常に敏感であるか、2 番目のファクターとしての電話の使用を望まない、あるいは使用できないシナリオまたは従業員が存在する企業向けの優れたオプションです。

![セキュリティ キーを使用して Microsoft Edge にサインインする](./media/concept-authentication-passwordless/concept-web-sign-in-security-key.png)

次のプロセスは、ユーザーが FIDO2 セキュリティ キーを使用してサインインするときに使用されます。

![FIDO2 セキュリティ キーを使用したユーザー サインインに関連する手順の概要を示す図](./media/concept-authentication-passwordless/fido2-security-key-flow.png)

1. ユーザーは、FIDO2 セキュリティ キーをコンピューターにプラグインします。
2. Windows で FIDO2 セキュリティ キーが検出されます。
3. Windows から認証要求が送信されます。
4. Azure AD から nonce が返送されます。
5. ユーザーはジェスチャを完了して、FIDO2 セキュリティ キーのセキュリティで保護されたエンクレーブに保存されている秘密キーのロックを解除します。
6. FIDO2 セキュリティ キーでは、秘密キーを使用して nonce に署名されます。
7. 署名された nonce を含むプライマリ更新トークン (PRT) トークン要求が Azure AD に送信されます。
8. Azure AD で、FIDO2 公開キーを使用して署名済みの nonce が検証されます。
9. Azure AD から PRT が返され、オンプレミスのリソースにアクセスできるようになります。

FIDO Alliance によって FIDO2 認定されたキーが多数存在する場合、Microsoft では、最大限のセキュリティと最適なエクスペリエンスを保証するために、ベンダーによって実装される FIDO2 Client-to-Authenticator Protocol (CTAP) 仕様の一部の省略可能な拡張機能が必要になります。

セキュリティ キーでは、Microsoft との互換性を維持するために、FIDO2 CTAP プロトコルの以下の機能と拡張機能を実装する**必要があります**。

| # | 機能 / 拡張機能 (信頼) | この機能または拡張機能が必要な理由 |
| --- | --- | --- |
| 1 | Resident Key | この機能により、セキュリティ キーが可搬となり、セキュリティキー内に資格情報が保存されます。 |
| 2 | Client pin | この機能では、2 番目の認証要素で資格情報を保護できます。これは、ユーザー インターフェイスがないセキュリティ キーに適用されます。 |
| 3 | hmac-secret | この拡張機能では、オフライン時または機内モード時に確実にデバイスにサインインできます。 |
| 4 | RP あたりの複数のアカウント | この機能を利用すれば、Microsoft アカウントや Azure Active Directory などの複数のサービスで同じセキュリティ キーを使用できるようになります。 |

以下のプロバイダーでは、パスワードレスのエクスペリエンスと互換性があると認識されている、さまざまなフォーム ファクターの FIDO2 セキュリティ キーが提供されます。 Microsoft では、お客様に対し、これらのキーのセキュリティ プロパティを評価するために、FIDO Alliance だけでなく、ベンダーにも連絡するようお勧めしています。

| プロバイダー | Contact |
| --- | --- |
| Yubico | [https://www.yubico.com/support/contact/](https://www.yubico.com/support/contact/) |
| Feitian | [https://www.ftsafe.com/about/Contact_Us](https://www.ftsafe.com/about/Contact_Us) |
| HID | [https://www.hidglobal.com/contact-us](https://www.hidglobal.com/contact-us) |
| Ensurity | [https://www.ensurity.com/contact](https://www.ensurity.com/contact) |
| eWBM | [https://www.ewbm.com/support](https://www.ewbm.com/support) |
| AuthenTrend | [https://authentrend.com/about-us/#pg-35-3](https://authentrend.com/about-us/#pg-35-3) |
| Gemalto (Thales Group) | [https://safenet.gemalto.com/multi-factor-authentication/authenticators/passwordless-authentication/](https://safenet.gemalto.com/multi-factor-authentication/authenticators/passwordless-authentication/) |
| OneSpan Inc. | [https://www.onespan.com/products/fido](https://www.onespan.com/products/fido) |
| IDmelon Technologies Inc. | [https://www.idmelon.com/#idmelon](https://www.idmelon.com/#idmelon) | 

> [!NOTE]
> NFC ベースのセキュリティ キーを購入して使用する予定の場合は、セキュリティ キーとしてサポートされている NFC リーダーが必要になります。 NFC リーダーは、Azure の要件でも制限事項でもありません。 使用している NFC ベースのセキュリティ キーでサポートされている NFC リーダーの一覧については、ベンダーにご確認ください。

ベンダーとして、サポートされているデバイスのリストにデバイスを載せたい場合は、[Fido2Request@Microsoft.com](mailto:Fido2Request@Microsoft.com) にご連絡ください。

## <a name="what-scenarios-work-with-the-preview"></a>プレビューで動作するシナリオ

- 管理者は、テナントに対してパスワードレスの認証方法を有効にすることができます
- 管理者は、すべてのユーザーをターゲットにすることも、方法ごとにテナント内のユーザー/グループを選ぶこともできます
- エンド ユーザーは、アカウント ポータルでこれらのパスワードレスの認証方法を登録して管理することができます
- エンド ユーザーは、これらのパスワードレスの認証方法でサインインすることができます
   - Microsoft Authenticator アプリ:すべてのブラウザー間で、Windows 10 の Out Of Box (OOBE) セットアップ時に、また、オペレーティング システム上の統合されたモバイル アプリでなど、Azure AD 認証が使用されるシナリオで動作します。
   - セキュリティ キー:Windows 10 のロック画面、および Microsoft Edge (従来のものと新しい Edge の両方) などのサポートされているブラウザーの Web で動作します。

## <a name="choose-a-passwordless-method"></a>パスワードレスの方法を選択する

これら 3 つのパスワードレス オプションのいずれを選択するかは、会社のセキュリティ、プラットフォーム、アプリの要件によって変わります。

Microsoft のパスワードレス テクノロジを選択する際には、考慮する必要がある要素がいくつかあります。

||**Windows Hello for Business**|**Microsoft Authenticator アプリでのパスワードレスのサインイン**|**FIDO2 セキュリティ キー**|
|:-|:-|:-|:-|
|**前提条件**| Windows 10 バージョン 1809 以降<br>Azure Active Directory| Microsoft Authenticator アプリ<br>電話 (iOS デバイスおよび Android 6.0 以降を実行している Android デバイス)|Windows 10 バージョン 1809 以降<br>Azure Active Directory|
|**モード**|プラットフォーム|ソフトウェア|ハードウェア|
|**システムとデバイス**|トラステッド プラットフォーム モジュール (TPM) が組み込まれた PC<br>PIN と生体認証の認識 |電話での PIN と生体認証の認識|Microsoft 互換の FIDO2 セキュリティ デバイス|
|**ユーザー エクスペリエンス**|Windows デバイスで PIN または生体認証の認識 (顔、虹彩、または指紋) を使用してサインインします。<br>Windows Hello の認証はデバイスに関連付けられています。ユーザーが会社のリソースにアクセスするには、デバイスと、PIN や生体認証要素などのサインイン コンポーネントの両方が必要です。|指紋スキャン、顔または虹彩の認識、または PIN を使用して携帯電話を使用してサインインします。<br>ユーザーは自分の PC または携帯電話から職場または個人用のアカウントにサインインします。|FIDO2 セキュリティ デバイス (生体認証、PIN、および NFC) を使用してサインインします<br>ユーザーは組織の管理に基づいてデバイスにアクセスし、PIN、USB セキュリティ キーや NFC 対応のスマート カードなどのデバイスを使用した生体認証、キー、またはウェアラブル デバイスに基づいて認証できます。|
|**有効なシナリオ**| Windows デバイスでのパスワードレスのエクスペリエンス。<br>デバイスやアプリケーションへのシングル サインオン機能を使用する専用の作業用 PC に適用できます。|携帯電話を使用するパスワードレスの任意の場所のソリューション。<br>任意のデバイスから Web 上の仕事用または個人用アプリケーションにアクセスするために適しています。|生体認証、PIN、および NFC を使用するワーカー向けのパスワードレスのエクスペリエンス。<br>共有の PC や携帯電話が実行可能な選択肢ではない場合 (ヘルプ デスク担当者、公共のキオスク、病院のチームなど) に適用できます|

次の表を使用して、お客様の要件とユーザーをサポートする方法を選択します。

|ペルソナ|シナリオ|環境|パスワードレスのテクノロジ|
|:-|:-|:-|:-|
|**管理者**|管理タスク用デバイスへのセキュリティで保護されたアクセス|割り当てられた Windows 10 デバイス|Windows Hello for Business、FIDO2 セキュリティ キー、またはその両方|
|**管理者**|Windows 以外のデバイスでの管理タスク| モバイルまたは Windows 以外のデバイス|Microsoft Authenticator アプリでのパスワードレスのサインイン|
|**インフォメーション ワーカー**|業務効率化作業|割り当てられた Windows 10 デバイス|Windows Hello for Business、FIDO2 セキュリティ キー、またはその両方|
|**インフォメーション ワーカー**|業務効率化作業| モバイルまたは Windows 以外のデバイス|Microsoft Authenticator アプリでのパスワードレスのサインイン|
|**現場のワーカー**|工場、プラント、小売店、またはデータ入力でのキオスク|共有 Windows 10 デバイス|FIDO2 セキュリティ キー|

## <a name="next-steps"></a>次のステップ

[組織内で FIDO2 セキュリティ キーのパスワードレスオプションを有効にする](howto-authentication-passwordless-security-key.md)

[組織内で電話ベースのパスワードレスオプションを有効にする](howto-authentication-passwordless-phone.md)

### <a name="external-links"></a>外部リンク

[FIDO Alliance](https://fidoalliance.org/)

[FIDO2 CTAP の仕様](https://fidoalliance.org/specs/fido-v2.0-id-20180227/fido-client-to-authenticator-protocol-v2.0-id-20180227.html)
