# Chat+社内文書検索

## セットアップガイド

> **重要:** このサンプルをデプロイするには、**Azure Open AI サービスが有効になっているサブスクリプションが必要です**。Azure Open AI サービスへのアクセス申請は[こちら](https://aka.ms/oaiapply)から行ってください。

### 事前準備

#### ローカル開発環境
このデモをデプロイするためには、ローカルに以下の開発環境が必要です。
> **重要** このサンプルは Windows もしくは Linux 環境で動作します。ただし、WSL2 の環境では正常に動作しません。

| ツール名 | 確認コマンド | 推奨バージョン | 
| --- | --- | --- |
| [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) | `az --version` | 2.50.0 以降 |
| [Python 3+](https://www.python.org/downloads/) | `python --version` | 3.9 以降 |
| pip ( Pythonと一緒にインストール ) | `pip --version` | 23.1 以降 |
| [Node.js](https://nodejs.org/en/download/) | `node --version` | 16 以降 |
| [Git](https://git-scm.com/downloads) | `git --version` | 2.33 以降 |


>注意: 実行するユーザの AAD アカウントは、`Microsoft.Authorization/roleAssignments/write` 権限を持っている必要があります。この権限は [ユーザーアクセス管理者](https://learn.microsoft.com/azure/role-based-access-control/built-in-roles#user-access-administrator) もしくは [所有者](https://learn.microsoft.com/azure/role-based-access-control/built-in-roles#owner)が保持しています。  
`az role assignment list --assignee <your-Azure-email-address> --subscription <subscription-id> --output table`

### インストール

#### 環境変数の設定
1. `az ad user show --id your_account@your_tenant -o tsv --query id` を実行して、操作をするユーザの AAD アカウントのオブジェクトID を取得します。
1. 取得したオブジェクトID を環境変数`AZURE_PRINCIPAL_ID`にセットします。
    - Windows 環境で実行している場合は、`$Env:AZURE_PRINCIPAL_ID="Your Object ID"`を実行します。
    - Linux 環境で実行している場合は、`export AZURE_PRINCIPAL_ID="Your Object ID"`を実行します。

#### フロントエンドのビルド
```
cd jp-azureopenai-samples/5.internal-document-search/src/frontend
npm install
npm run build
```

#### Azureのリソースを作成
```
AZURE_LOCATION=eastus
RG=rg-temp
WEBAPP=app-temp
DEP_NAME=tempdeployment
AZURE_ENV_NAME=supertemp
cd ../../infra
az deployment sub create --name $DEP_NAME --location $LOC --template-file main.bicep --parameters principalId=$AZURE_PRINCIPAL_ID environmentName=$AZURE_ENV_NAME location=$AZURE_LOCATION
```

#### data配下のPDFを利用して Search Index を作成
```
cd jp-azureopenai-samples/5.internal-document-search/scripts
prepdocs.ps1 or prepdocs.sh
```

#### Pythonコードのデプロイ
```
cd jp-azureopenai-samples/5.internal-document-search/src/backend
zip -r app.zip .
az webapp deployment source config-zip --resource-group $RG --name $WEBAPP --src ./app.zip
```

コマンドの実行が終了すると、アプリケーションにアクセスする為の URL が表示されます。この URL をブラウザで開き、サンプルアプリケーションの利用を開始してください
> 注意: アプリケーションのデプロイ完了には数分かかることがあります。"Python Developer" のウェルカムスクリーンが表示される場合は、数分待ってアクセスし直してください。

#### アプリケーションのローカル実行
アプリケーションをローカルで実行する場合には、以下のコマンドを実行してください。`azd up`で既に Azure 上にリソースがデプロイされていることを前提にしています。

1. `azd login` を実行する。
2. `src` フォルダに移動する。
3. `./start.ps1` もしくは `./start.sh` を実行します。

##### VS Codeでのデバッグ実行
1. `src\backend`フォルダに異動する
2. `code .`でVS Codeを開く
3. Run>Start Debugging または F5

#### FrontendのJavaScriptのデバッグ
1. src/frontend/vite.config.tsのbuildに`minify: false`を追加
2. ブラウザのDeveloper tools > Sourceでブレイクポイントを設定して実行

### GPT-4モデルの利用
2023年6月現在、GPT-4 モデルは申請することで利用可能な状態です。このサンプルは GPT-4 モデルのデプロイに対応していますが、GPT-4 モデルを利用する場合には、[こちら](https://learn.microsoft.com/ja-jp/azure/cognitive-services/openai/how-to/create-resource?pivots=web-portal#deploy-a-model)を参考に、GPT-4 モデルをデプロイしてください。また、GPT-4 モデルの利用申請は[こちらのフォーム](https://aka.ms/oai/get-gpt4)から可能です。

GPT-4 モデルのデプロイ後、以下の操作を実行してください。

1. このサンプルをデプロイした際に、プロジェクトのディレクトリに `./${環境名}/.env` ファイルが作成されています。このファイルを任意のエディタで開きます。
1. 以下の行を探して、デプロイした GPT-4 モデルのデプロイ名を指定してください。
> AZURE_OPENAI_GPT_4_DEPLOYMENT="" # GPT-4モデルのデプロイ名
AZURE_OPENAI_GPT_4_32K_DEPLOYMENT="" # GPT-4-32Kモデルのデプロイ名

1. `azd up` を実行します。

GPT-4 モデルは、チャット機能、文書検索機能のオプションで利用することができます。

### Easy Authの設定（オプション）
必要に応じて、Azure AD に対応した Easy Auth を設定します。Easy Auth を設定した場合、UI の右上にログインユーザのアカウント名が表示され、チャットの履歴ログにもアカウント名が記録されます。
Easy Auth の設定は、[こちら](https://learn.microsoft.com/ja-jp/azure/app-service/scenario-secure-app-authentication-app-service)を参考にしてください。
