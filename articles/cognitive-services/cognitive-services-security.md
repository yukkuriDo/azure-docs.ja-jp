---
title: Security
titleSuffix: Azure Cognitive Services
description: Cognitive Services を使用するときに考慮する必要のあるセキュリティに関する事柄について説明します。
services: cognitive-services
author: IEvangelist
manager: nitinme
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 03/23/2020
ms.author: dapine
ms.openlocfilehash: c86d806c408c2e8226e632a0b15e1e8729c987f9
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/28/2020
ms.locfileid: "80131529"
---
# <a name="azure-cognitive-services-security"></a>Azure Cognitive Services のセキュリティ

どのようなアプリケーションの開発でも、セキュリティを最優先事項として考慮する必要があります。 人工知能を利用するアプリケーションでは、セキュリティがさらに重要になります。 この記事では、トランスポート層セキュリティの使用、認証、機密データの安全な構成など、Azure Cognitive Services セキュリティのさまざまな側面について説明します。

## <a name="transport-layer-security-tls"></a>トランスポート層セキュリティ (TLS)

HTTP 経由で公開されるすべての Cognitive Services エンドポイントでは、TLS 1.2 が適用されます。 セキュリティ プロトコルが適用されている場合、Cognitive Services エンドポイントを呼び出そうとするコンシューマーは、次のガイドラインに従う必要があります。

* クライアント オペレーティング システム (OS) では、TLS 1.2 がサポートされている必要があります
* HTTP の呼び出しに使用される言語 (およびプラットフォーム) では、要求の一部として TLS 1.2 を指定する必要があります
  * 言語とプラットフォームによっては、TLS の指定は暗黙的または明示的に行われます

.NET ユーザーの場合は、<a href="https://docs.microsoft.com/dotnet/framework/network-programming/tls" target="_blank">トランスポート層セキュリティに関するベスト プラクティス<span class="docon docon-navigate-external x-hidden-focus"></span></a>を検討してください。

## <a name="authentication"></a>認証

認証については、よく誤解されることがいくつかあります。 認証と承認は混同されることがよくあります。 ID も、セキュリティの主要なコンポーネントです。 ID は、<a href="https://en.wikipedia.org/wiki/Principal_(computer_security)" target="_blank">プリンシパル<span class="docon docon-navigate-external x-hidden-focus"></span></a>に関する情報のコレクションです。 ID プロバイダー (IdP) は、認証サービスに対して ID を提供します。 認証とは、ユーザーの ID を検証することです。 承認とは、特定の ID に対してリソースへのアクセス権と特権を指定することです。 Cognitive Services のいくつかのサービス内容には、ロールベースのアクセス制御 (RBAC) が含まれています。 RBAC を使用すると、プリンシパルの手動管理に関する手続きの一部を簡略化できます。 詳しくは、[Azure リソースに対するロールベースのアクセス制御](../role-based-access-control/overview.md)に関する記事をご覧ください。

サブスクリプション キー、アクセス トークン、Azure Active Directory (AAD) での認証について詳しくは、「<a href="https://docs.microsoft.com/azure/cognitive-services/authentication" target="_blank">Azure Cognitive Services に対する要求の認証<span class="docon docon-navigate-external x-hidden-focus"></span></a>」をご覧ください。

## <a name="environment-variables-and-application-configuration"></a>環境変数とアプリケーションの構成

環境変数は名前と値のペアであり、特定の環境内に格納されます。 機密データに対してハードコーディングされた値を使用するより安全な方法は、環境変数を使用することです。 ハードコーディングされた値は安全ではないため、避ける必要があります。

> [!CAUTION]
> 機密データに対してはハードコーディングされた値を**使用しない**でください。使用すると、大きなセキュリティの脆弱性になります。

> [!NOTE]
> 環境変数はプレーンテキストで格納されますが、環境に分離されます。 環境が侵害された場合は、環境に関する変数も同様です。

### <a name="set-environment-variable"></a>環境変数を設定する

環境変数を設定するには、次のいずれかのコマンドを使用します。`ENVIRONMENT_VARIABLE_KEY` は名前付きのキーであり、`value` は環境変数に格納される値です。

# <a name="command-line"></a>[コマンド ライン](#tab/command-line)

永続化された環境変数を作成して割り当て、値を指定します。

```CMD
:: Assigns the env var to the value
setx ENVIRONMENT_VARIABLE_KEY="value"
```

**コマンド プロンプト**の新しいインスタンスで、環境変数を読み取ります。

```CMD
:: Prints the env var value
echo %ENVIRONMENT_VARIABLE_KEY%
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

永続化された環境変数を作成して割り当て、値を指定します。

```powershell
# Assigns the env var to the value
[System.Environment]::SetEnvironmentVariable('ENVIRONMENT_VARIABLE_KEY', 'value', 'User')
```

**Windows PowerShell** の新しいインスタンスで、環境変数を読み取ります。

```powershell
# Prints the env var value
[System.Environment]::GetEnvironmentVariable('ENVIRONMENT_VARIABLE_KEY')
```

# <a name="bash"></a>[Bash](#tab/bash)

永続化された環境変数を作成して割り当て、値を指定します。

```Bash
# Assigns the env var to the value
echo export ENVIRONMENT_VARIABLE_KEY="value" >> /etc/environment && source /etc/environment
```

**Bash** の新しいインスタンスで、環境変数を読み取ります。

```Bash
# Prints the env var value
echo "${ENVIRONMENT_VARIABLE_KEY}"

# Or use printenv:
# printenv ENVIRONMENT_VARIABLE_KEY
```

---

> [!TIP]
> 環境変数を設定した後、統合開発環境 (IDE) を再起動して、新しく追加した環境変数を使用できることを確認します。

### <a name="get-environment-variable"></a>環境変数を取得する

環境変数を取得するには、メモリに読み込む必要があります。 使用している言語に応じて、次のコード スニペットを参考にしてください。 これらのコード スニペットでは、`ENVIRONMENT_VARIABLE_KEY` で指定した環境変数を取得し、`value` という名前の変数に代入する方法が示されています。

# <a name="c"></a>[C#](#tab/csharp)

詳しくは、「<a href="https://docs.microsoft.com/dotnet/api/system.environment.getenvironmentvariable" target="_blank">`Environment.GetEnvironmentVariable` <span class="docon docon-navigate-external x-hidden-focus"></span></a>」をご覧ください。

```csharp
using static System.Environment;

class Program
{
    static void Main()
    {
        // Get the named env var, and assign it to the value variable
        var value =
            GetEnvironmentVariable(
                "ENVIRONMENT_VARIABLE_KEY");
    }
}
```

# <a name="c"></a>[C++](#tab/cpp)

詳しくは、「<a href="https://docs.microsoft.com/cpp/c-runtime-library/reference/getenv-wgetenv" target="_blank">`getenv` <span class="docon docon-navigate-external x-hidden-focus"></span></a>」をご覧ください。

```cpp
#include <stdlib.h>

int main()
{
    // Get the named env var, and assign it to the value variable
    auto value =
        getenv("ENVIRONMENT_VARIABLE_KEY");
}
```

# <a name="java"></a>[Java](#tab/java)

詳しくは、「<a href="https://docs.oracle.com/javase/7/docs/api/java/lang/System.html#getenv(java.lang.String)" target="_blank">`System.getenv` <span class="docon docon-navigate-external x-hidden-focus"></span></a>」をご覧ください。

```java
import java.lang.*;

public class Program {
   public static void main(String[] args) throws Exception {
    // Get the named env var, and assign it to the value variable
    String value =
        System.getenv(
            "ENVIRONMENT_VARIABLE_KEY")
   }
}
```

# <a name="nodejs"></a>[Node.js](#tab/node-js)

詳しくは、「<a href="https://nodejs.org/api/process.html#process_process_env" target="_blank">`process.env` <span class="docon docon-navigate-external x-hidden-focus"></span></a>」をご覧ください。

```javascript
// Get the named env var, and assign it to the value variable
const value =
    process.env.ENVIRONMENT_VARIABLE_KEY;
```

# <a name="python"></a>[Python](#tab/python)

詳しくは、「<a href="https://docs.python.org/2/library/os.html#os.environ" target="_blank">`os.environ` <span class="docon docon-navigate-external x-hidden-focus"></span></a>」をご覧ください。

```python
import os

# Get the named env var, and assign it to the value variable
value = os.environ['ENVIRONMENT_VARIABLE_KEY']
```

# <a name="objective-c"></a>[Objective-C](#tab/objective-c)

詳しくは、「<a href="https://developer.apple.com/documentation/foundation/nsprocessinfo/1417911-environment?language=objc" target="_blank">`environment` <span class="docon docon-navigate-external x-hidden-focus"></span></a>」をご覧ください。

```objectivec
// Get the named env var, and assign it to the value variable
NSString* value =
    [[[NSProcessInfo processInfo]environment]objectForKey:@"ENVIRONMENT_VARIABLE_KEY"];
```

---

## <a name="next-steps"></a>次のステップ

* さまざまな [Cognitive Services](welcome.md) を調べます
* [Cognitive Services の仮想ネットワーク](cognitive-services-virtual-networks.md)について詳しく学習します
