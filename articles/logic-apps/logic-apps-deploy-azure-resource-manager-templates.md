---
title: ロジック アプリ テンプレートのデプロイ
description: Azure Logic Apps 用に作成された Azure Resource Manager テンプレートをデプロイする方法について説明します
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 08/01/2019
ms.openlocfilehash: 432e22879ce0eba89f04a1084e2d4a93a487dd45
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/28/2020
ms.locfileid: "82086438"
---
# <a name="deploy-azure-resource-manager-templates-for-azure-logic-apps"></a>Azure Logic Apps 用の Azure Resource Manager テンプレートをデプロイする

Logic Apps 用の Azure Resource Manager テンプレートを作成したら、次の方法でテンプレートをデプロイできます。

* [Azure Portal](#portal)
* [Visual Studio](#visual-studio)
* [Azure PowerShell](#powershell)
* [Azure CLI](#cli)
* [Azure リソース マネージャー REST API](../azure-resource-manager/templates/deploy-rest.md)
* [Azure DevOps](#azure-pipelines)

<a name="portal"></a>

## <a name="deploy-through-azure-portal"></a>Azure portal を使用してデプロイする

Logic Apps テンプレートを Azure に自動的にデプロイするには、次の **[Azure に配置する]** ボタンを選択します。これにより、Azure portal にサインインし、Logic Apps に関する情報の入力を求められます。 その後、Logic Apps テンプレートまたはパラメーターに必要な変更を加えることができます。

[![Azure へのデプロイ](./media/logic-apps-deploy-azure-resource-manager-templates/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-logic-app-create%2Fazuredeploy.json)

たとえば、Azure portal にサインインした後に次の情報の入力を求められます。

* Azure サブスクリプション名
* 使用するリソース グループ
* Logic Apps の場所
* Logic Apps の名前
* テスト URI
* 規定の使用条件への同意

詳細については、以下のトピックを参照してください。

* [概要:Azure Resource Manager テンプレートを使用した Logic Apps のデプロイの自動化](logic-apps-azure-resource-manager-templates-overview.md)
* [Azure Resource Manager テンプレートと Azure portal を使用してリソースをデプロイする](../azure-resource-manager/templates/deploy-portal.md)

<a name="visual-studio"></a>

## <a name="deploy-with-visual-studio"></a>Visual Studio でのデプロイ

Visual Studio を使用して作成した Azure Resource Group プロジェクトから Logic Apps テンプレートをデプロイするには、[これらの手順に従って手動で Logic Apps を Azure にデプロイします](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md#deploy-logic-app-to-azure)。

<a name="powershell"></a>

## <a name="deploy-with-azure-powershell"></a>Azure PowerShell でのデプロイ

特定の "*Azure リソース グループ*" にデプロイするには、次のコマンドを使用します。

```powershell
New-AzResourceGroupDeployment -ResourceGroupName <Azure-resource-group-name> -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-logic-app-create/azuredeploy.json
```

詳細については、以下のトピックを参照してください。

* [Resource Manager テンプレートと Azure PowerShell を使用したリソースのデプロイ](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy)
* [`New-AzResourceGroupDeployment`](https://docs.microsoft.com/powershell/module/azurerm.resources/new-azurermresourcegroupdeployment)

<a name="cli"></a>

## <a name="deploy-with-azure-cli"></a>Azure CLI でのデプロイ

特定の "*Azure リソース グループ*" にデプロイするには、次のコマンドを使用します。

```azurecli
az group deployment create -g <Azure-resource-group-name> --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-logic-app-create/azuredeploy.json
```

詳細については、以下のトピックを参照してください。

* [Resource Manager テンプレートと Azure CLI を使用したリソースのデプロイ](../azure-resource-manager/templates/deploy-cli.md)
* [`az group deployment create`](https://docs.microsoft.com/cli/azure/group/deployment?view=azure-cli-latest#az-group-deployment-create)

<a name="azure-pipelines"></a>

## <a name="deploy-with-azure-devops"></a>Azure DevOps を使用してデプロイする

Logic Apps テンプレートをデプロイし、環境を管理するために、チームは一般的に [Azure DevOps](https://docs.microsoft.com/azure/devops/user-guide/what-is-azure-devops-services) の [Azure Pipelines](https://docs.microsoft.com/azure/devops/pipelines/get-started/what-is-azure-pipelines) のようなツールを使用します。 Azure Pipelines は、あらゆるビルドまたはリリース パイプラインに追加できる [[Azure リソース グループの配置] タスク](https://github.com/Microsoft/azure-pipelines-tasks/tree/master/Tasks/AzureResourceGroupDeploymentV2) を提供します。 リリース パイプラインをデプロイおよび生成するための承認には、Azure Active Directory (AD) [サービス プリンシパル](../active-directory/develop/app-objects-and-service-principals.md)も必要です。 詳細は、[Azure Pipelines でのサービス プリンシパルの使用](https://docs.microsoft.com/azure/devops/pipelines/library/connect-to-azure)に関するページを参照してください。

Azure Pipelines と Azure Resource Manager テンプレートの継続的インテグレーションおよび継続的デプロイ (CI/CD) の詳細については、以下のトピックとサンプルを参照してください。

* [Azure Pipelines を使用した Resource Manager テンプレートの統合](../azure-resource-manager/templates/add-template-to-azure-pipelines.md)
* [チュートリアル:Azure Pipelines を使用した Azure Resource Manager テンプレートの継続的インテグレーション](../azure-resource-manager/templates/deployment-tutorial-pipeline.md)
* [サンプル:Azure Logic Apps から Azure Service Bus キューに接続し、Azure DevOps に Azure Pipelines を使用してデプロイする](https://docs.microsoft.com/samples/azure-samples/azure-logic-apps-deployment-samples/connect-to-azure-service-bus-queues-from-azure-logic-apps-and-deploy-with-azure-devops-pipelines/)
* [サンプル:Azure Logic Apps から Azure Storage アカウントに接続し、Azure DevOps に Azure Pipelines を使用してデプロイする](https://docs.microsoft.com/samples/azure-samples/azure-logic-apps-deployment-samples/connect-to-azure-storage-accounts-from-azure-logic-apps-and-deploy-with-azure-devops-pipelines/)
* [サンプル:Azure Logic Apps の関数アプリ アクションを設定し、Azure DevOps に Azure Pipelines を使用してデプロイする](https://docs.microsoft.com/samples/azure-samples/azure-logic-apps-deployment-samples/set-up-an-azure-function-app-action-for-azure-logic-apps-and-deploy-with-azure-devops-pipelines/)
* [サンプル:Azure Logic Apps から統合アカウントに接続し、Azure DevOps に Azure Pipelines を使用してデプロイする](https://docs.microsoft.com/samples/azure-samples/azure-logic-apps-deployment-samples/connect-to-an-integration-account-from-azure-logic-apps-and-deploy-by-using-azure-devops-pipelines/)
* [サンプル:Azure Logic Apps を使用した Azure Pipelines の調整](https://docs.microsoft.com/samples/azure-samples/azure-logic-apps-pipeline-orchestration/azure-devops-orchestration-with-logic-apps/)

Azure Pipelines を使用するための一般的な大まかな手順は次のとおりです。

1. Azure Pipelines で、空のパイプラインを作成します。

1. Logic Apps テンプレートやテンプレート パラメーター ファイルなど、パイプラインに必要なリソースを選択します。これらは手動で、またはビルド プロセスの一環として生成します。

1. エージェント ジョブの場合は、 **[Azure リソース グループの配置]** タスクを見つけて追加します。

   ![[Azure リソース グループの配置] タスクの追加](./media/logic-apps-deploy-azure-resource-manager-templates/add-azure-resource-group-deployment-task.png)

1. [サービス プリンシパル](https://docs.microsoft.com/azure/devops/pipelines/library/connect-to-azure)を使用して構成します。

1. Logic Apps テンプレートとテンプレート パラメーター ファイルへの参照を追加します。

1. 必要に応じて、その他の環境、自動化されたテスト、承認者のために、リリース プロセスの手順の構築を続行します。

<a name="authorize-oauth-connections"></a>

## <a name="authorize-oauth-connections"></a>OAuth 接続を作成する

デプロイ後、Logic Apps は有効なパラメーターを使用してエンド ツー エンドで動作します。 ただし、引き続き OAuth 接続を承認し、[資格情報を認証する](../active-directory/develop/authentication-scenarios.md)ための有効なアクセス トークンを生成する必要があります。 OAuth 接続を認証する方法は次のとおりです。

* 自動デプロイの場合は、各 OAuth 接続に同意するスクリプトを使用できます。 例として、[LogicAppConnectionAuth](https://github.com/logicappsio/LogicAppConnectionAuth) プロジェクトの GitHub のスクリプトがあります。

* OAuth 接続を手動で承認するには、Azure portal または Visual Studio で、Logic App Designer で Logic Apps を開きます。 デザイナーでは、必要な接続を承認します。

接続を承認するのではなく Azure Active Directory (Azure AD) [サービス プリンシパル](../active-directory/develop/app-objects-and-service-principals.md)を使用する場合は、[Logic Apps テンプレートでサービス プリンシパル パラメーターを指定する方法](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md#authenticate-connections)のページを参照してください。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [Logic Apps の監視](../logic-apps/monitor-logic-apps.md)
