---
title: 作業時間外に VM を開始/停止するソリューション
description: この VM 管理ソリューションでは、スケジュールに従った Azure 仮想マシンの起動と停止、および Azure Monitor ログからのプロアクティブな監視が実行されます。
services: automation
ms.subservice: process-automation
ms.date: 04/28/2020
ms.topic: conceptual
ms.openlocfilehash: f7e30fd0d53af7ee61d919b56e9ffcd1f1b6bd36
ms.sourcegitcommit: 34a6fa5fc66b1cfdfbf8178ef5cdb151c97c721c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/28/2020
ms.locfileid: "82207600"
---
# <a name="startstop-vms-during-off-hours-solution-in-azure-automation"></a>Azure Automation の作業時間外に VM を開始/停止するソリューション

**作業時間外に VM を開始/停止する**ソリューションは、Azure 仮想マシンを開始または停止します。 ユーザー定義のスケジュールでマシンを開始または停止し、Azure Monitor ログを介して分析情報を取得し、[アクション グループ](../azure-monitor/platform/action-groups.md)を使用してオプションのメールを送信することができます。 このソリューションは、ほとんどのシナリオで Azure Resource Manager とクラシック VM の両方をサポートしています。 

このソリューションは、[Start-AzureRmVM](https://docs.microsoft.com/powershell/module/azurerm.compute/start-azurermvm?view=azurermps-6.13.0) コマンドレットを使用して VM を開始します。 VM を停止するためには、[Stop-AzureRmVM](https://docs.microsoft.com/powershell/module/AzureRM.Compute/Stop-AzureRmVM?view=azurermps-6.13.0) を使用します。

> [!NOTE]
> **作業時間外に VM を開始/停止する**ソリューションは、利用可能な最新バージョンの Azure モジュールをサポートするように更新されています。 AzureRM から Az モジュールに移行したため、このソリューションの更新版 (Marketplace から入手可能) では、AzureRM モジュールはサポートされません。

このソリューションは、VM のコストを最適化する必要があるユーザー向けに、分散型の低コストな自動化オプションを提供します。 このソリューションでは次のことが可能です。

- [VM の起動および停止をスケジュールする](automation-solution-vm-management-config.md#schedule)。
- [Azure タグを使用](automation-solution-vm-management-config.md#tags)して昇順での VM の起動および停止をスケジュールする (クラシック VM ではサポートされません)。
- [CPU 使用率の低さ](automation-solution-vm-management-config.md#cpuutil)に基づいて VM を自動停止する。

現在のソリューションでの制限を次に示します。

- 任意のリージョンの VM が管理されますが、Azure Automation アカウントと同じサブスクリプションでのみ使用できます。
- Azure と Azure Government の Log Analytics ワークスペース、Azure Automation アカウント、および Alerts がサポートされているリージョンで利用できます。 現在、Azure Government の各リージョンでは電子メール機能はサポートされていません。

## <a name="solution-prerequisites"></a>ソリューションの前提条件

このソリューションの Runbook は、[Azure 実行アカウント](automation-create-runas-account.md)で動作します。 認証方法としては、実行アカウントの使用をお勧めします。有効期限が切れたり頻繁に変わったりするパスワードではなく、証明書を使った認証が使用されるためです。

**作業時間外に VM を開始/停止する**ソリューションには、別の Automation アカウントを使用することをお勧めします。 多くの場合、Azure モジュールのバージョンがアップグレードされ、そのパラメーターが変更される可能性があります。 このソリューションは同じペースでアップグレードされないため、使用するコマンドレットの新しいバージョンでは動作しない可能性があります。 モジュールの更新は、実稼働用の Automation アカウントにインポートする前に、テスト用の Automation アカウントでテストすることをお勧めします。

## <a name="solution-permissions"></a>ソリューションのアクセス許可

**作業時間外に VM を開始/停止する**ソリューションをデプロイするには、特定のアクセス許可が必要です。 事前に作成された Automation アカウントと Log Analytics ワークスペースをソリューションに使用する場合と、ソリューションのデプロイ時に新しいアカウントとワークスペースを作成する場合とでは、必要なアクセス許可が異なります。 

サブスクリプションの共同作成者であり、かつ Azure Active Directory テナントの全体管理者である場合は、アクセス許可を構成する必要はありません。 これらの権限がない場合、またはカスタム ロールを構成する必要がない場合は、以下で説明するアクセス許可を持っていることを確認してください。

### <a name="permissions-for-pre-existing-automation-account-and-log-analytics-workspace"></a>既存の Automation アカウントと Log Analytics ワークスペースのアクセス許可

既存の Automation アカウントと Log Analytics ワークスペースに**作業時間外に VM を開始/停止する**ソリューションをデプロイするには、そのソリューションをデプロイするユーザーはリソース グループ スコープに次のアクセス許可が必要です。 ロールの詳細については、「[Azure リソースのカスタム ロール](../role-based-access-control/custom-roles.md)」をご覧ください。

| 権限 | Scope|
| --- | --- |
| Microsoft.Automation/automationAccounts/read | リソース グループ |
| Microsoft.Automation/automationAccounts/variables/write | リソース グループ |
| Microsoft.Automation/automationAccounts/schedules/write | リソース グループ |
| Microsoft.Automation/automationAccounts/runbooks/write | リソース グループ |
| Microsoft.Automation/automationAccounts/connections/write | リソース グループ |
| Microsoft.Automation/automationAccounts/certificates/write | リソース グループ |
| Microsoft.Automation/automationAccounts/modules/write | リソース グループ |
| Microsoft.Automation/automationAccounts/modules/read | リソース グループ |
| Microsoft.automation/automationAccounts/jobSchedules/write | リソース グループ |
| Microsoft.Automation/automationAccounts/jobs/write | リソース グループ |
| Microsoft.Automation/automationAccounts/jobs/read | リソース グループ |
| Microsoft.OperationsManagement/solutions/write | リソース グループ |
| Microsoft.OperationalInsights/workspaces/* | リソース グループ |
| Microsoft.Insights/diagnosticSettings/write | リソース グループ |
| Microsoft.Insights/ActionGroups/Write | リソース グループ |
| Microsoft.Insights/ActionGroups/read | リソース グループ |
| Microsoft.Resources/subscriptions/resourceGroups/read | リソース グループ |
| Microsoft.Resources/deployments/* | リソース グループ |

### <a name="permissions-for-new-automation-account-and-new-log-analytics-workspace"></a>新しい Automation アカウントと新しい Log Analytics ワークスペースのアクセス許可

**作業時間外に VM を開始/停止する**ソリューションは、新しい Automation アカウントと Log Analytics ワークスペースにデプロイできます。 この場合、ソリューションをデプロイするユーザーには、前のセクションで定義されたアクセス許可と、このセクションで定義されているアクセス許可が必要です。 

ソリューションをデプロイするユーザーには、次のロールが必要です。

- サブスクリプションの共同管理者。 クラシック VM を管理する場合は、クラシック実行アカウントを作成するためにこのロールが必要です。 [クラシック実行アカウント](automation-create-standalone-account.md#create-a-classic-run-as-account)は、既定では作成されなくなりました。
- [Azure Active Directory](../active-directory/users-groups-roles/directory-assign-admin-roles.md) アプリケーション開発者ロールのメンバーシップ。 実行アカウントの構成の詳細については、「[実行アカウントを構成するためのアクセス許可](manage-runas-account.md#permissions)」を参照してください。
- サブスクリプションまたは次のアクセス許可の共同作成者。

| 権限 |Scope|
| --- | --- |
| Microsoft.Authorization/Operations/read | サブスクリプション|
| Microsoft.Authorization/permissions/read |サブスクリプション|
| Microsoft.Authorization/roleAssignments/read | サブスクリプション |
| Microsoft.Authorization/roleAssignments/write | サブスクリプション |
| Microsoft.Authorization/roleAssignments/delete | サブスクリプション |
| Microsoft.Automation/automationAccounts/connections/read | リソース グループ |
| Microsoft.Automation/automationAccounts/certificates/read | リソース グループ |
| Microsoft.Automation/automationAccounts/write | リソース グループ |
| Microsoft.OperationalInsights/workspaces/write | リソース グループ |

## <a name="solution-components"></a>ソリューションのコンポーネント

**作業時間外に VM を開始/停止するソリューション**には、構成済みの Runbook、スケジュール、および Azure Monitor ログとの統合が含まれています。 これらの要素を使用して、ビジネス ニーズに合わせて VM のスタートアップとシャットダウンを調整できます。

### <a name="runbooks"></a>Runbooks

ソリューションによって Automation アカウントにデプロイされる Runbook の一覧を次の表に示します。 Runbook のコードは変更しないでください。 新しい機能が必要なときは、独自の Runbook を記述してください。

> [!IMPORTANT]
> 名前の末尾に **child** が付いている Runbook を直接実行しないでください。

すべての親 Runbook には `WhatIf` パラメーターがあります。 このパラメーターを True に設定すると、パラメーターなしで実行されたときの Runbook の実行内容を詳細に指定し、正しい VM が対象になっていることを検証することができます。 `WhatIf` パラメーターが False に設定されている場合、Runbook では定義されているアクションのみが実行されます。

|Runbook | パラメーター | 説明|
| --- | --- | ---|
|AutoStop_CreateAlert_Child | VMObject <br> AlertAction <br> WebHookURI | 親 Runbook から呼び出されます。 この Runbook は、Auto-Stop シナリオでリソースごとにアラートを作成します。|
|AutoStop_CreateAlert_Parent | VMList<br> WhatIf: True または False  | 対象となるサブスクリプションまたはリソース グループ内の VM 上で、Azure アラート ルールを作成または更新します。 <br> `VMList` は VM のコンマ区切りリストです。 たとえば、「 `vm1, vm2, vm3` 」のように入力します。<br> `WhatIf` を使用すると、Runbook ロジックを実行せずに検証できます。|
|AutoStop_Disable | なし | Auto-Stop アラートと既定のスケジュールを無効にします。|
|AutoStop_VM_Child | WebHookData | 親 Runbook から呼び出されます。 アラート ルールによってこの Runbook が呼び出され、クラシック VM が停止されます。|
|AutoStop_VM_Child_ARM | WebHookData |親 Runbook から呼び出されます。 アラート ルールによってこの Runbook が呼び出され、VM が停止されます。  |
|ScheduledStartStop_Base_Classic | CloudServiceName<br> アクション:開始または停止<br> VMList  | Cloud Services によるクラシック VM グループでのアクションの開始または停止を実行します。 |
|ScheduledStartStop_Child | VMName <br> アクション:開始または停止 <br> ResourceGroupName | 親 Runbook から呼び出されます。 停止スケジュールの起動または停止アクションを実行します。|
|ScheduledStartStop_Child_Classic | VMName<br> アクション:開始または停止<br> ResourceGroupName | 親 Runbook から呼び出されます。 クラシック VM に対して、スケジュールに従って開始または停止アクションを実行します。 |
|ScheduledStartStop_Parent | アクション:開始または停止 <br>VMList <br> WhatIf: True または False | サブスクリプション内のすべての VM を開始または停止します。 これらの対象のリソース グループに対してのみ実行するように変数 `External_Start_ResourceGroupNames` および `External_Stop_ResourceGroupNames` を編集します。 `External_ExcludeVMNames` 変数を更新することで、特定の VM を除外することもできます。|
|SequencedStartStop_Parent | アクション:開始または停止 <br> WhatIf: True または False<br>VMList| 開始または停止アクティビティのシーケンスを指定する各 VM に、**sequencestart** および **sequencestop** という名前のタグを作成します。 これらのタグ名では、大文字と小文字が区別されます。 タグの値は、起動または停止する順序に対応する正の整数 (1、2、3) にする必要があります。 <br>**注**:VM は、`External_Start_ResourceGroupNames`、`External_Stop_ResourceGroupNames`、および `External_ExcludeVMNames` 変数で定義されたリソース グループ内にある必要があります。 アクションを有効にするための適切なタグが必要です。|

### <a name="variables"></a>変数

次の表は、Automation アカウント内に作成される変数の一覧です。 `External` のプレフィックスが付いた変数のみを変更します。 `Internal` プレフィックスが付いた変数を変更すると、望ましくない影響が生じます。

> [!NOTE]
> VM 名とリソース グループの制限は、主に可変サイズの結果です。 「[Azure Automation での変数アセット](https://docs.microsoft.com/azure/automation/shared-resources/variables)」を参照してください。

|変数 | 説明|
|---------|------------|
|External_AutoStop_Condition | アラートをトリガーする前の条件を構成するのに必要な条件演算子。 使用できる値は、`GreaterThan`、`GreaterThanOrEqual`、`LessThan`、および `LessThanOrEqual` です。|
|External_AutoStop_Description | CPU の割合がしきい値を超えた場合に VM を停止するアラート。|
|External_AutoStop_Frequency | ルールの評価頻度。 このパラメーターは、timespan 形式で入力を受け入れます。 使用可能な値は、5 分 ～ 6 時間です。 |
|External_AutoStop_MetricName | Azure アラート ルールが構成されるパフォーマンス メトリックの名前。|
|External_AutoStop_Severity | メトリック アラートの重大度。0 から 4 の範囲が可能です。 |
|External_AutoStop_Threshold | 変数 `External_AutoStop_MetricName` に指定された Azure 警告ルールのしきい値。 パーセント値の範囲は 1 から 100 です。|
|External_AutoStop_TimeAggregationOperator | 条件を評価するために選択した時間枠のサイズに適用される時間の集計演算子。 使用できる値は、`Average`、`Minimum`、`Maximum`、`Total`、および `Last` です。|
|External_AutoStop_TimeWindow | アラートをトリガーするために選択されたメトリックを Azure で分析する時間枠のサイズ。 このパラメーターは、timespan 形式で入力を受け入れます。 使用可能な値は、5 分 ～ 6 時間です。|
|External_EnableClassicVMs| クラシック VM がソリューションの対象であるかどうかを指定する値。 既定値は True です。 Azure クラウド ソリューション プロバイダー (CSP) サブスクリプションの場合は、この変数を False に設定します。 クラシック VM には[クラシック実行アカウント](automation-create-standalone-account.md#create-a-classic-run-as-account)が必要です。|
|External_ExcludeVMNames | 除外する VM 名のコンマ区切りリスト。上限は 140 VM です。 一覧に 140 個を超える VM を追加すると、除外するように設定した VM が誤って開始または停止される可能性があります。|
|External_Start_ResourceGroupNames | 開始アクションの対象となる 1 つまたは複数のリソース グループのコンマ区切りリスト。|
|External_Stop_ResourceGroupNames | 停止アクションの対象となる 1 つまたは複数のリソース グループのコンマ区切りリスト。|
|External_WaitTimeForVMRetrySeconds |VM 上で **SequencedStartStop_Parent** Runbook に対して実行されるアクションの待機時間 (秒)。 この変数を使用すると、Runbook で次のアクションに進む前に、指定された秒数の間、子操作を待機できます。 最大待機時間は 10800 (3 時間) です。 既定値は 2100 秒です。|
|Internal_AutomationAccountName | Automation アカウントの名前を指定します。|
|Internal_AutoSnooze_ARM_WebhookURI | VM に対する AutoStop シナリオで呼び出される Webhook URI。|
|Internal_AutoSnooze_WebhookUri | クラシック VM に対する AutoStop シナリオで呼び出される Webhook URI。|
|Internal_AzureSubscriptionId | Azure サブスクリプション ID。|
|Internal_ResourceGroupName | Automation アカウントのリソース グループ名。|

>[!NOTE]
>変数 `External_WaitTimeForVMRetryInSeconds` の場合、既定値は 600 から 2100 に更新されています。 

すべてのシナリオで、変数 `External_Start_ResourceGroupNames`、`External_Stop_ResourceGroupNames`、および `External_ExcludeVMNames` は、VM のターゲット設定に必要です。ただし、**AutoStop_CreateAlert_Parent**、**SequencedStartStop_Parent**、および **ScheduledStartStop_Parent** Runbook のコンマ区切りの VM リストは除きます。 つまり、開始および停止アクションを発生させるには、対象のリソース グループに VM が属している必要があります。 このロジックは Azure Policy に似ています。サブスクリプションまたはリソース グループを対象にすることができ、アクションは新しく作成された VM に継承されます。 この方法により、VM ごとに個別のスケジュールを保持したり、スケールで起動および停止を管理したりする必要がなくなります。

### <a name="schedules"></a>スケジュール

次の表は、Automation アカウント内に作成される既定のスケジュールの一覧です。 それらを変更したり、独自のカスタム スケジュールを作成したりできます。 既定では、すべてのスケジュールは **Scheduled_StartVM** と **Scheduled_StopVM** スケジュールを除いて無効になっています。

すべてのスケジュールを有効にしないでください。スケジュール操作が重複する可能性があります。 実行する必要がある最適化を特定し、それに応じて変更することをお勧めします。 詳細については、概要セクションのシナリオ例をご覧ください。

|スケジュール名 | 頻度 | 説明|
|--- | --- | ---|
|Schedule_AutoStop_CreateAlert_Parent | 8 時間ごと | **AutoStop_CreateAlert_Parent** Runbook を 8 時間ごとに実行し、`External_Start_ResourceGroupNames`、`External_Stop_ResourceGroupNames`、および `External_ExcludeVMNames` 変数の VM ベースの値を停止します。 または、`VMList` パラメーターを使用して VM のコンマ区切りリストを指定できます。|
|Scheduled_StopVM | ユーザー定義、毎日 | `Stop` パラメーターを持つ **ScheduledStopStart_Parent** Runbook を毎日指定された時刻に実行します。 変数資産によって定義されたルールを満たす VM すべてを自動的に停止します。 関連するスケジュール (**Scheduled-StartVM**) を有効にしてください。|
|Scheduled_StartVM | ユーザー定義、毎日 | `Start` パラメーター値を持つ **ScheduledStopStart_Parent** Runbook を毎日指定された時刻に実行します。 変数資産によって定義されたルールを満たす VM すべてを自動的に開始します。 関連するスケジュール (**Scheduled-StopVM**) を有効にしてください。|
|Sequenced-StopVM | 午前 1 時 00 分 (UTC)、毎週金曜日 | `Stop` パラメーター値を持つ **Sequenced_StopStop_Parent** Runook を毎週金曜日の指定された時刻に実行します。 適切な変数で定義された **SequenceStop** のタグを持つ VM すべてを順番 (昇順) に停止します。 タグ値と資産である変数の詳細については、「[Runbook](#runbooks)」を参照してください。 関連するスケジュール (**Sequenced-StartVM**) を有効にしてください。|
|Sequenced-StartVM | 午後 1 時 00 分 (UTC)、毎週月曜日 | `Start` のパラメーター値を持つ **SequencedStopStart_Parent** Runbook を毎週月曜日の指定された時刻に実行します。 適切な変数で定義された **SequenceStart** のタグを持つ VM すべてを順番 (降順)に 起動します。 タグ値と変数資産の詳細については、「[Runbook](#runbooks)」を参照してください。 関連するスケジュール (**Sequenced-StopVM**) を有効にしてください。

## <a name="use-of-the-solution-with-classic-vms"></a>クラシック VM でのソリューションの使用

クラシック VM に**作業時間外に VM を開始/停止するソリューション**を使用している場合、Automation によってクラウド サービスごとにすべての VM が順番に処理されます。 VM は、異なる複数のクラウド サービスでまだ並列に処理されています。 

クラシック VM でこのソリューションを使用するには、既定では作成されないクラシック実行アカウントが必要です。 クラシック実行アカウントの作成手順については、「[クラシック実行アカウントの作成](automation-create-standalone-account.md#create-a-classic-run-as-account)」を参照してください。

クラウド サービスあたり 20 個を超える VM がある場合は、次のような推奨事項があります。

* 親 Runbook の **ScheduledStartStop_Parent** で複数のスケジュールを作成し、スケジュールごとに 20 個の VM を指定します。 
* スケジュール プロパティで `VMList` パラメーターを使用して、コンマ区切りリストとして VM 名を指定します。 

そうしないと、このソリューションの Automation ジョブが 3 時間を超えて実行された場合、そのジョブは[フェア シェア](automation-runbook-execution.md#fair-share)制限に従って一時的にアンロードまたは停止されます。

Azure CSP サブスクリプションでは、Azure Resource Manager モデルのみがサポートされます。 Azure Resource Manager 以外のサービスは、プログラムでは使用できません。 **作業時間外に VM を開始/停止する**ソリューションの実行中は、クラシック リソースを管理するコマンドレットがあるため、エラーが発生することがあります。 CSP について詳しくは、[CSP サブスクリプションで利用可能なサービス](https://docs.microsoft.com/azure/cloud-solution-provider/overview/azure-csp-available-services)に関するページをご覧ください。 CSP サブスクリプションを使用する場合、デプロイ後に [External_EnableClassicVMs](#variables) 変数を False に設定する必要があります。

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="enable-the-solution"></a>ソリューションを有効にする

ソリューションの使用を開始するには、[VM の開始または停止ソリューションを有効にする](automation-solution-vm-management-enable.md)方法に関するページの手順に従います。

## <a name="view-the-solution"></a>ソリューションの表示

ソリューションを有効にした後にアクセスするには、次のいずれかのメカニズムを使用します。

* Automation アカウントから、 **[関連リソース]** の **[VM の開始/停止]** を選択します。 [VM の起動/停止] ページで、ページの右側の **[VM ソリューションの開始/停止]** の下にある **[ソリューションの管理]** を選択します。

* Automation アカウントにリンクされている Log Analytics ワークスペースに移動します。 ワークスペースを選択した後、左側のペインから **[ソリューション]** を選択します。 [ソリューション] ページで、一覧から **[Start-Stop-VM[ワークスペース]]** ソリューションを選択します。  

ソリューションを選択すると、**Start-Stop-VM[ワークスペース]** ソリューション ページが表示されます。 ここでは、**StartStopVM** タイルの情報など、重要な詳細を確認できます。 Log Analytics ワークスペースと同様、このタイルには、そのソリューションに関して開始された Runbook ジョブの数と、正常に終了した Runbook ジョブの数、およびそのグラフが表示されます。

![Automation Update Management ソリューション ページ](media/automation-solution-vm-management/azure-portal-vmupdate-solution-01.png)

ジョブ レコードの詳細な分析を実行するには、ドーナツ タイルをクリックします。 ソリューション ダッシュボードに、ジョブ履歴と定義済みのログ検索クエリが表示されます。 Log Analytics Advanced ポータルに切り替えて、検索クエリに基づいて検索を実行します。

## <a name="update-the-solution"></a>ソリューションを更新する

このソリューションの以前のバージョンをデプロイしている場合は、更新済みリリースをデプロイする前に、まず以前のバージョンをご自身のアカウントから削除します。 次の手順に従って[ソリューションを削除](#remove-the-solution)してから、[ソリューションをデプロイ](automation-solution-vm-management-enable.md)してください。

## <a name="remove-the-solution"></a>ソリューションを削除する

ソリューションを使用する必要がなくなった場合、ソリューションを Automation アカウントから削除できます。 ソリューションを削除すると、Runbook のみが削除されます。 ソリューションの追加時に作成されたソリューションまたは変数は削除されません。 これらの資産を他の Runbook で使用していない場合は、手動で削除します。

ソリューションを削除するには:

1. Automation アカウントで、 **[関連リソース]** の下にある **[リンクされたワークスペース]** を選択します。

2. **[ワークスペースに移動]** を選択します。

3. **[全般]** の下にある **[ソリューション]** をクリックします。 

4. [ソリューション] ページで、ソリューション **[Start-Stop-VM[ワークスペース]]** を選択します。 

5. **VMManagementSolution[ワークスペース]** ページで、メニューから **[削除]** を選択します。<br><br> ![VM 管理ソリューションの削除](media/automation-solution-vm-management/vm-management-solution-delete.png)

6. **[ソリューションの削除]** ウィンドウで、ソリューションを削除するかどうかを確定します。

7. 情報が検証され、ソリューションが作成されている間、メニューから **[通知]** を選択してその進行状況を追跡することができます。 ソリューションの削除プロセスが開始すると、[ソリューション] ページに戻ります。

Automation アカウントと Log Analytics ワークスペースは、このプロセスの一部として削除されません。 Log Analytics ワークスペースを保持しない場合は、Azure portal から手動で削除する必要があります。

1. **[Log Analytics ワークスペース]** を探して選択します。

2. [Log Analytics ワークスペース] ページで、ワークスペースを選択します。

3. ワークスペースの設定ページのメニューから **[削除]** を選択します。

4. Azure Automation アカウントの[ソリューション コンポーネント](#solution-components)を保持しない場合は、各コンポーネントを手動で削除することができます。

## <a name="next-steps"></a>次のステップ

お使いの Azure VM に対して**作業時間外に VM を開始/停止する**ソリューションを[有効にします](automation-solution-vm-management-enable.md)。
