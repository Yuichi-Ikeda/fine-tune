# Microsoft Foundry Fine-tuning サンプル

このリポジトリは、Microsoft Foundry を使用したモデルのファインチューニングのサンプルプロジェクトです。

## 概要

Microsoft Foundry（旧 Azure AI Studio）を使用して、カスタムデータセットでモデルをファインチューニングする方法を示します。このサンプルでは、JSONL 形式のトレーニングデータを使用してモデルをカスタマイズします。

## プロジェクト構造

```
fine-tune/
├── data_char.jsonl          # メインのトレーニングデータ（文字データ）
├── images/                  # 画像ファイル（ドキュメント用）
└── README.md                # このファイル
```

## 前提条件

- Azure サブスクリプション
- Microsoft Foundry プロジェクト
- Python 3.8 以上
- Azure CLI または Azure SDK for Python

## セットアップ

### 1. Microsoft Foundry の準備

1. [Azure Portal](https://portal.azure.com) にサインイン
2. Microsoft Foundry リソースを作成
3. プロジェクトを作成し、エンドポイントを設定

### 2. Python 環境のセットアップ

```bash
# 仮想環境の作成
python -m venv venv

# 仮想環境の有効化
# Windows
venv\Scripts\activate
# macOS/Linux
source venv/bin/activate

# 必要なパッケージのインストール
pip install azure-ai-ml azure-identity
```

## データ形式

トレーニングデータは JSONL（JSON Lines）形式で提供されています。各行は以下のような JSON オブジェクトです：

```jsonl
{"messages": [{"role": "system", "content": "あなたは役立つアシスタントです。"}, {"role": "user", "content": "質問内容"}, {"role": "assistant", "content": "回答内容"}]}
```

### データファイルの説明

- **data_char.jsonl**: 文字ベースのトレーニングデータ
- **backup/data_ring.jsonl**: リングデータのバックアップ
- **backup/data_text.jsonl**: テキストデータのバックアップ

## ファインチューニングの実行

### Microsoft Foundry UI を使用する場合

1. Microsoft Foundry ポータルにアクセス
2. 「Fine-tuning」セクションに移動
3. ベースモデルを選択（GPT-3.5-turbo、GPT-4 など）
4. トレーニングデータ（`data_char.jsonl`）をアップロード
5. ハイパーパラメータを設定
6. ファインチューニングジョブを開始

### Azure SDK を使用する場合

```python
from azure.ai.ml import MLClient
from azure.identity import DefaultAzureCredential
from azure.ai.ml.entities import FineTuningJob

# クライアントの初期化
credential = DefaultAzureCredential()
ml_client = MLClient(
    credential=credential,
    subscription_id="<your-subscription-id>",
    resource_group_name="<your-resource-group>",
    workspace_name="<your-workspace-name>"
)

# ファインチューニングジョブの作成
fine_tuning_job = FineTuningJob(
    training_data="./data_char.jsonl",
    model="gpt-35-turbo",  # または他のサポートされているモデル
    hyperparameters={
        "n_epochs": 3,
        "batch_size": 1,
        "learning_rate_multiplier": 0.1
    }
)

# ジョブの送信
job = ml_client.fine_tuning.begin_create(fine_tuning_job)
print(f"Fine-tuning job created: {job.name}")
```

## ハイパーパラメータの推奨設定

- **n_epochs**: 3-10（データセットのサイズに応じて調整）
- **batch_size**: 1-4
- **learning_rate_multiplier**: 0.05-0.2
- **validation_split**: 0.1-0.2（検証データの割合）

## モデルのデプロイ

ファインチューニングが完了したら：

1. Microsoft Foundry でファインチューニングされたモデルを選択
2. 「Deploy」をクリック
3. エンドポイント名とコンピューティングリソースを設定
4. デプロイを完了

## 使用方法

デプロイされたモデルを使用する：

```python
import openai
from azure.identity import DefaultAzureCredential, get_bearer_token_provider

token_provider = get_bearer_token_provider(
    DefaultAzureCredential(), 
    "https://cognitiveservices.azure.com/.default"
)

client = openai.AzureOpenAI(
    azure_endpoint="<your-endpoint>",
    azure_ad_token_provider=token_provider,
    api_version="2024-08-01-preview"
)

response = client.chat.completions.create(
    model="<your-fine-tuned-model-deployment-name>",
    messages=[
        {"role": "system", "content": "あなたは役立つアシスタントです。"},
        {"role": "user", "content": "こんにちは！"}
    ]
)

print(response.choices[0].message.content)
```

## トラブルシューティング

### データ検証エラー

- JSONL ファイルの各行が有効な JSON であることを確認
- `messages` フィールドが正しい形式であることを確認
- `role` フィールドが "system"、"user"、"assistant" のいずれかであることを確認

### クォータエラー

- Azure サブスクリプションにファインチューニングのクォータがあることを確認
- 必要に応じてクォータの増加をリクエスト

## リソース

- [Microsoft Foundry ドキュメント](https://learn.microsoft.com/azure/ai-studio/)
- [ファインチューニングガイド](https://learn.microsoft.com/azure/ai-services/openai/how-to/fine-tuning)
- [Azure OpenAI Service](https://learn.microsoft.com/azure/ai-services/openai/)