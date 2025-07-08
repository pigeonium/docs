# チュートリアル

このチュートリアルでは、`pigeonium_client.py`を使用して、Pigeoniumネットワークの基本的な操作を体験します。


## 1\. 環境設定

```
pip install pigeonium requests && \
git clone https://github.com/pigeonium/python-client.git --recursive && \
cd python-client
```


## 2\. クライアントの初期化

Pythonスクリプトを作成し、`PigeoniumClient`をインポートして初期化します。クライアントはAPIサーバーに接続し、ネットワーク設定を自動で取得します。

```python
import pigeonium
from pigeonium_client import PigeoniumClient

# APIサーバーのURLを指定してクライアントを初期化
# ローカルで起動している場合は "http://127.0.0.1:14540"
API_URL = "http://localhost:14540" 
client = PigeoniumClient(API_URL)

# 基軸通貨のIDを取得
base_currency_id = pigeonium.Config.BaseCurrency.currencyId
```


## 3\. ウォレットの作成と復元

新しいウォレットを生成したり、既存の秘密鍵からウォレットを復元したりできます。

```python
# 新しいウォレットを生成
wallet1 = client.generate_wallet()
print(f"新しいウォレットのアドレス: {wallet1.address.hex()}")
print(f"秘密鍵 (安全な場所に保管): {wallet1.privateKey.hex()}")

# 秘密鍵からウォレットを復元
# wallet1_private_key = "..." # ここに保存した秘密鍵を貼り付け
# wallet1 = client.wallet_from_private_key(wallet1_private_key)
# print(f"復元したウォレットのアドレス: {wallet1.address.hex()}")
```


## 4\. 残高の確認

特定のアドレスが保有する通貨の残高を確認します。

```python
# 確認したいウォレットのアドレス
target_address = wallet1.address

print(f"\n--- {target_address.hex()} の残高 ---")
balances = client.get_balances(target_address)

if not balances:
    print("このアドレスには残高がありません。")
else:
    for currency_id, amount in balances.items():
        currency_info = client.get_currency(currency_id)
        # pigeonium.Utils.convertAmount を使って、最小単位から通常の単位に変換
        readable_amount = pigeonium.Utils.convertAmount(amount)
        print(f"- {readable_amount} {currency_info.symbol} ({currency_info.name})")
```

*注意: 新しく作成したウォレットの残高は0です。*


## 5\. トランザクションの送信

あるウォレットから別のウォレットへ通貨を送信します。

```python
# 送信先ウォレットを準備
wallet2 = client.generate_wallet()

# wallet1からwallet2へ基軸通貨を 0.01 Pigeon 送信
# 金額は最小単位で指定 (1 Pigeon = 1,000,000)
amount_to_send = 10000 

print("\n--- トランザクションの送信 ---")
try:
    tx_receipt = client.send_transaction(
        source_wallet=wallet1,
        dest_address=wallet2.address,
        currency_id=base_currency_id,
        amount=amount_to_send
    )
    print("トランザクション成功！")
    print(f"Index ID: {tx_receipt.indexId}")
    print(f"Webエクスプローラーで確認: {API_URL.replace('14540', '8080')}/transaction.html?id={tx_receipt.indexId}")
except Exception as e:
    print(f"トランザクション失敗: {e}")

```

*注意: 送信元ウォレットに残高がない場合、この処理は失敗します。*


## 6\. スマートコントラクトのデプロイと実行

簡単なスマートコントラクトをデプロイし、それを実行してみましょう。

### 6.1. デプロイ

このコントラクトは、`inputData`で受け取った値をキーとして、`'Hello Pigeonium!'`という文字列を自身のストレージに保存します。

```python
# デプロイするスマートコントラクトのスクリプト
contract_script = """
# トランザクションに入力データがあれば実行
if transaction.inputData:
  # setVariable(キー, 値) でコントラクト内にデータを保存
  setVariable(transaction.inputData, b'Hello Pigeonium!')
"""

print("\n--- スマートコントラクトのデプロイ ---")
try:
    # デプロイ費用は wallet1 が支払う
    deploy_receipt = client.deploy_contract(
        sender_wallet=wallet1,
        script=contract_script
    )
    contract_address = deploy_receipt.inputData
    print("コントラクトのデプロイ成功！")
    print(f"コントラクトアドレス: {contract_address.hex()}")
except Exception as e:
    print(f"デプロイ失敗: {e}")
```

*注意: デプロイには基軸通貨Pigeonの手数料がかかります。*

### 6.2. 実行（対話）

デプロイしたコントラクトのアドレス宛にトランザクションを送信して、コントラクトのロジックを起動します。

```python
# コントラクトに渡すデータ（キーとして使用）
data_for_contract = b"my_first_key"

print("\n--- スマートコントラクトの実行 ---")
try:
    # コントラクトアドレス宛にトランザクションを送信
    exec_receipt = client.send_transaction(
        source_wallet=wallet1,
        dest_address=contract_address, # 宛先はコントラクトのアドレス
        currency_id=base_currency_id,
        amount=0, # 通貨を送信しない場合でも0を指定
        input_data=data_for_contract
    )
    print("コントラクトの実行に成功しました！")
    print(f"Index ID: {exec_receipt.indexId}")
    print("Webエクスプローラーでコントラクトの状態変数を確認してみてください。")
except Exception as e:
    print(f"コントラクトの実行に失敗: {e}")
```
