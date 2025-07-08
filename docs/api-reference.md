# APIリファレンス

Pigeonium APIサーバーは、Pigeoniumネットワークと対話するためのRESTful APIを提供します。

**ベースURL**: `http://<your-server-ip>:14540`

**インタラクティブドキュメント**:
APIサーバーが起動している場合、以下のURLからSwagger UIやRedocといったインタラクティブなAPIドキュメントにアクセスできます。

  * **Swagger UI**: `http://<your-server-ip>:14540/docs`
  * **Redoc**: `http://<your-server-ip>:14540/redoc`


## ネットワーク

### `GET /`

ネットワークの基本情報を取得します。 クライアントが最初に接続する際に使用します。

  * **レスポンス (200 OK)**
    ```json
    {
      "networkName": "Pigeonium",
      "networkId": 0,
      "contractDeployCost": 100,
      "contractExecutionCost": 10,
      "inputDataCost": 10,
      "decimals": 6,
      "adminPublicKey": "...",
      "baseCurrency": {
        "currencyId": "00000000000000000000000000000000",
        "name": "Pigeon",
        "symbol": "Pigeon",
        "issuer": "...",
        "supply": 1000000000000
      }
    }
    ```


## 通貨

### `GET /currency/{currencyId}`

指定されたIDの通貨情報を取得します。

  * **パスパラメータ**:
      * `currencyId` (string): 通貨ID (16進数文字列)
  * **レスポンス (200 OK)**
    ```json
    {
      "currencyId": "...",
      "name": "MyToken",
      "symbol": "MTK",
      "issuer": "...",
      "supply": 500000000
    }
    ```


## 残高

### `GET /balances/{address}`

指定されたアドレスが保有するすべての通貨の残高を取得します。

  * **パスパラメータ**:
      * `address` (string): ウォレットアドレス (16進数文字列)
  * **レスポンス (200 OK)**
    ```json
    {
      "00000000000000000000000000000000": 99988700,
      "abcdef1234567890abcdef1234567890": 10000
    }
    ```

### `GET /balance/{address}/{currencyId}`

指定されたアドレスの、指定された単一通貨の残高を取得します。

  * **パスパラメータ**:
      * `address` (string): ウォレットアドレス (16進数文字列)
      * `currencyId` (string): 通貨ID (16進数文字列)
  * **レスポンス (200 OK)**
    ```json
    {
      "currencyId": "00000000000000000000000000000000",
      "amount": 99988700
    }
    ```


## トランザクション

### `POST /transaction`

新しいトランザクションをネットワークにブロードキャストします。

  * **リクエストボディ**:
    ```json
    {
      "source": "...",
      "dest": "...",
      "currencyId": "...",
      "amount": 10000,
      "feeAmount": 0,
      "inputData": "...",
      "publicKey": "...",
      "signature": "..."
    }
    ```
  * **レスポンス (200 OK)**: 実行後のトランザクションオブジェクト

### `GET /transaction/{indexId}`

指定されたインデックスIDのトランザクションを取得します。

  * **パスパラメータ**:
      * `indexId` (integer): トランザクションのインデックスID
  * **レスポンス (200 OK)**: トランザクションオブジェクト

### `GET /transactions`

条件に一致するトランザクションのリストを取得します。ページネーションやフィルタリングが可能です。

  * **クエリパラメータ (一部)**:
      * `address` (string): 指定したアドレスが `source` または `dest` に含まれるトランザクションを検索
      * `limit` (integer, default: 20): 取得する件数
      * `offset` (integer, default: 0): スキップする件数
  * **レスポンス (200 OK)**: トランザクションオブジェクトの配列


## スマートコントラクト

### `POST /contract`

新しいスマートコントラクトをデプロイします。

  * **リクエストボディ**:
    ```json
    {
      "sender": "...",
      "script": "...",
      "publicKey": "...",
      "signature": "...",
      "deployTransaction": {
        "source": "...",
        "dest": "00000000000000000000000000000000",
        "currencyId": "...",
        "amount": 12300,
        "feeAmount": 0,
        "inputData": "...",
        "publicKey": "...",
        "signature": "..."
      }
    }
    ```
  * **レスポンス (200 OK)**: デプロイトランザクションのオブジェクト

### `GET /script/{address}`

指定されたコントラクトアドレスのソースコードを取得します。

  * **パスパラメータ**:
      * `address` (string): コントラクトアドレス (16進数文字列)
  * **レスポンス (200 OK)**
    ```json
    {
      "script": "if transaction.inputData: setVariable(inputData, b'Hello Pigeonium!')"
    }
    ```
