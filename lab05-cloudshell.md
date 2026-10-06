# Lab 5: Cloud Shell & テンプレートデプロイ（オプション）

> **受講形式**: ハンズオン（受講者が自分のアカウントで操作）
> 時間に余裕がある場合に実施します。Cloud Shell は Lab 2 の SSH 接続で既に体験しています。

---

## 目的

- Azure Cloud Shell の使い方を理解する
- ARM テンプレート（JSON）によるリソースデプロイを体験する
- Bicep テンプレートによるリソースデプロイを体験する
- Infrastructure as Code の基本概念を理解する

## 所要時間

約 45 分

## 事前準備

このラボでは以下のサンプルテンプレートを使用します:

| ファイル | 形式 | 説明 |
|---------|------|------|
| `templates/storage-account.json` | ARM テンプレート（JSON） | ストレージアカウントをデプロイ |
| `templates/storage-account.bicep` | Bicep テンプレート | ストレージアカウントをデプロイ |

> 両テンプレートとも Standard_LRS のストレージアカウントを作成します（推定コスト $1 未満）。

---

## 手順

### ステップ 1: Cloud Shell の起動と基本操作（10 分）

#### 1-1. Cloud Shell の起動

1. Azure ポータル上部の Cloud Shell アイコン（`>_`）をクリック
2. **Bash** モードで起動

#### 1-2. 基本コマンドの実行

```bash
# サブスクリプション情報の確認
az account show

# 自分のリソースグループ内のリソース一覧
az resource list -g rg-az900-studentXX -o table
```

#### 1-3. Cloud Shell のファイルエディタ

Cloud Shell には組み込みのファイルエディタがあります:

```bash
# エディタを起動
code .
```

> **ポイント**: Cloud Shell はブラウザ上で動作するため、ローカル環境に Azure CLI をインストールする必要がありません。ストレージアカウントにファイルを永続化できます。

---

### ステップ 2: ARM テンプレートのデプロイ（15 分）

#### 2-1. テンプレートファイルの準備

Cloud Shell のエディタで以下の内容のファイル `storage-account.json` を作成します:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storageAccountName": {
            "type": "string",
            "metadata": {
                "description": "ストレージアカウント名"
            }
        },
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
                "description": "リソースのリージョン"
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2023-01-01",
            "name": "[parameters('storageAccountName')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "StorageV2"
        }
    ],
    "outputs": {
        "storageAccountId": {
            "type": "string",
            "value": "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccountName'))]"
        }
    }
}
```

#### 2-2. テンプレートの構造を理解する

| セクション | 説明 |
|-----------|------|
| `$schema` | テンプレートのスキーマ定義（バージョン） |
| `contentVersion` | テンプレートのバージョン |
| `parameters` | デプロイ時に指定する入力パラメータ |
| `resources` | デプロイするリソースの定義 |
| `outputs` | デプロイ後に出力する値 |

#### 2-3. ARM テンプレートのデプロイ

自分のリソースグループにデプロイします:

```bash
az deployment group create \
  --resource-group rg-az900-studentXX \
  --template-file storage-account.json \
  --parameters storageAccountName=stlab05studentXX
```

> **ポイント**: `--parameters` でテンプレートの `parameters` セクションに定義したパラメータに値を渡します。`location` はデフォルト値（リソースグループのリージョン）が使用されるため省略可能です。

#### 2-4. デプロイ結果の確認

Azure ポータルで自分のリソースグループを開き、ストレージアカウントが作成されていることを確認します。

---

### ステップ 3: Bicep テンプレートのデプロイ（10 分）

#### 3-1. Bicep テンプレートの準備

Cloud Shell のエディタで以下の内容のファイル `storage-account.bicep` を作成します:

```bicep
@description('ストレージアカウント名')
param storageAccountName string

@description('リソースのリージョン')
param location string = resourceGroup().location

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}

output storageAccountId string = storageAccount.id
```

#### 3-2. ARM テンプレートとの比較

| 項目 | ARM テンプレート（JSON） | Bicep |
|------|------------------------|-------|
| 構文 | JSON 形式（冗長） | 簡潔な独自構文 |
| 型安全性 | なし | あり（コンパイル時チェック） |
| ファイルサイズ | 大きい | 小さい |
| 変換 | - | コンパイルすると ARM テンプレートに変換される |

> **ポイント**: Bicep は ARM テンプレートの簡潔な代替構文です。内部的には ARM テンプレート（JSON）にコンパイルされて実行されます。

#### 3-3. Bicep テンプレートのデプロイ

```bash
az deployment group create \
  --resource-group rg-az900-studentXX \
  --template-file storage-account.bicep \
  --parameters storageAccountName=stlab05bstudentXX
```

#### 3-4. デプロイ結果の確認

Azure ポータルでリソースグループを開き、2 つ目のストレージアカウントが作成されていることを確認します。

---

### ステップ 4: デプロイ履歴の確認（5 分）

1. Azure ポータルで自分のリソースグループを開く
2. 左メニュー「デプロイ」を選択
3. デプロイ履歴を確認:
   - ARM テンプレートによるデプロイ
   - Bicep テンプレートによるデプロイ
4. 各デプロイの詳細（入力パラメータ、出力、所要時間）を確認

> **ポイント**: テンプレートデプロイは Infrastructure as Code（IaC）の基本です。リソース構成をコードで定義することで、再現可能かつバージョン管理可能な環境構築が実現できます。本コースの環境自体も Terraform（別の IaC ツール）で構築されています。

---

### まとめ

#### 本ラボで学んだこと

| 概念 | 説明 |
|------|------|
| Cloud Shell | ブラウザ上で Azure CLI / PowerShell を実行できる環境 |
| ARM テンプレート | Azure リソースを JSON 形式で定義する IaC ツール |
| Bicep | ARM テンプレートの簡潔な代替構文 |
| テンプレートデプロイ | コードによる再現可能なリソース構築 |
| Infrastructure as Code | インフラ構成をコードで管理する手法 |

#### 質疑応答

不明点があればこのタイミングで講師に確認してください。
