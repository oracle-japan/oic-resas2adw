# Oracle Integration Cloud チュートリアル: RESAS-API のデータを Oracle ADB に保存する

2022年11月

このチュートリアルは、 Oracle Integration Cloud を使用して [RESAS-API が提供する市区町村データ](https://opendata.resas-portal.go.jp/docs/api/v1/cities.html) を、Oracle ADB 上の `CITIES` という名前の表に保存してみます。

## 前提

このチュートリアルに沿って作業を進めるためには、次の設定が完了している必要があります。

* Oracle Integration Cloud と Oracle Autononmous Database (ADB) のインスタンス作成
* 市区町村データを格納する Oracle ADB の表の作成
* RESAS-API の API キーの取得

### インスタンスの作成

このチュートリアルは、Oracle Integration Cloud と Oracle ADB のインスタンスの作成が完了し、コンソールにログインていることを前提としています。
まだインスタンスを作成していない場合は、次のページを参照してインスタンスを作成してください。

* [Oracle Integration Cloud インスタンスの作成](https://oracle-japan.github.io/ocitutorials/integration/integration-for-commons-1-instance)
* Oracle ADB インスタンスの作成

### 表の作成

このチュートリアルでは、`CITIES` 表を使用します。
`CITIES` 表には、次の4つの列が定義されています。

| 列名 | データ型 | Null値の指定 | 備考 |
|:----|:----|:---:|:----|
| `PREF_CODE` | `NUMBER` | 不可 | |
| `CITY_CODE` | `VARCHAR2(10)` | 不可 | 主キーに設定 |
| `CITY_NAME` | `VARCHAR2(30)` | 不可 | |
| `BIG_CITY_FLAG` | `VARCHAR2(1)` | 不可 | |

### RESAS-API の API キーの取得

RESAS-APIを利用するには API キーが必要です。
API キーは、[利用登録](https://opendata.resas-portal.go.jp/form.html)することで取得できます。

> ***Note*** このチュートリアルでは、あらかじめ取得した API キーを使用するので、利用登録は不要です。

### このチュートリアルの表記方法

このチュートリアルで使用している表記方法は次のとおりです。

| 表記方法 | 説明 |
|:----|:----|
| **「太字」** | ボタン、各種フィールドのラベルなどの GUI 要素 |
| <プレースホルダ> | 使用する環境などによって置き換える部分を表すプレースホルダー |
| `固定幅フォント` | 実行するコマンド、URL、サンプルコード、入力するテキスト |

## 接続の作成

Oracle Integration Cloud では、使用する API やデータベースなどへのアクセス情報を "接続" として定義します。
Oracle Integration Cloud は、多数のアダプタを提供しており、アクセスに必要な情報を入力するだけです。

このチュートリアルでは、作成済みの次の２つの接続を使用します。

* RESAS-API 呼び出すための REST アダプタを使用した接続: RESAS-API
* Oracle ADB アダプタを使用した接続: ATP_CONN

## 統合の作成

Oracle Integration Cloud の "統合" は、システム間の連携の流れを定義したものです。
このチュートリアルで作成する統合は、次の処理を実行します。

1. RESAS-API の市区町村 API を呼び出し、データを取得
1. 取得したデータを Oracle ADB のテーブル `CITIES` に保存

### 統合の新規作成

1.  Oralce Integration Cloud にログインします。
    **「接続」** ページを開いている場合は、ナビゲーション・ペインで **「統合」** をクリックします。

    **「ようこそ」** または **「ホーム」** ページを開いている場合は、ナビゲーション・ペインで **「統合」** をクリックして **「統合」** のメニューを開き、 **「統合」** を選択します。

1.  **「統合」** ページが表示されたら、ページの右上にある **「作成」** ボタンをクリックします。

    ![Oracle Integration Cloud](images/ss04-01.png)

1.  **「統合スタイル」** ボックスが表示されます。
    **「スケジュールされたオーケストレーション」** をクリックしてから、ボックスの右下にある **「選択」** ボタンをクリックします。

    ![Oracle Integration Cloud](images/ss04-02.png)

1.  **「新規統合の作成」** ボックスが表示されます。
    次の情報を入力する必要があります。

    | 入力項目 | 入力する値 |
    |:----|:----|
    | **「統合にどのような名前をつけますか。」** | `Load Cities <ユーザーのイニシャル>` |
    | **「識別子」** | `LOAD_CITIES_<ユーザーのイニシャル>` (Load Cities と名前をつけると自動的に設定される) |
    | **「バージョン」** | `01.00.0000` （初期状態で入力されている値をそのまま使用） |
    | **「Documentation URL」** | 入力しない |
    | **「キーワード」** | 入力しない |
    | **「パッケージ」** | 入力しない |
    | **「説明」** | 実行される処理についての説明（任意） |

    ![Oracle Integration Cloud](images/ss04-03.png)

    入力したら右下の **「作成」** ボタンをクリックします。

1.  統合が作成されると、キャンバス・ビューで表示されます。
    **「レイアウト」** を初期状態の **「垂直」** から **「水平」** に変更します。

    ![Oracle Integration Cloud](images/ss04-04.png)

1.  **「スケジュール」** アイコンを選択すると表示される **「スケジュール・パラメータの編集」** アイコンをクリックします。

    ![Oracle Integration Cloud](images/ss04-05.png)

1.  **「スケジュール・パラメータ」** ページが表示されます。
    ページの右下にある **「追加」** アイコンをクリックします。

    ![Oracle Integration Cloud](images/ss04-06.png)

1.  **「パラメータ名」** に `prefCode`、 **「値」** に `1` と入力します。

    ![Oracle Integration Cloud](images/ss04-07.png)

    入力し終わったらページの右上にある **「閉じる」** ボタンをクリックします。

### RESAS-API 呼び出しの設定

1.  キャンバス・ビューの右端に表示されている ![呼び出し](images/btn_invokes.png) アイコンをクリックします。

    ![Oracle Integration Cloud](images/ss05-01.png)

    ロールが呼び出しに設定されている接続のリストが表示されます。
    **「REST」** をクリックして、 **「RESAS-API」** を見つけます。

1.  **「RESAS-API」** を、キャンバス・ビューの **「スケジュール」** アイコンから **「停止」** アイコン（背景が緑色のアイコン）に向けられた矢印の上にドラッグし、表示された **「＋」** マークの上でドロップします。

    ![Oracle Integration Cloud](images/ss05-02.png)

1.  **「REST エンドポイントの構成」** ボックスが表示されます。
    **「基本情報」** では、次のように入力します。

    | 入力項目 | 入力する値 |
    |:----|:----|
    | **「エンドポイントにどのような名前をつけますか。」** | `GetCities` |
    | **「エンドポイントの相対リソースURIは何ですか。」** | `/cities` |
    | **「エンドポイントでどのアクションを実行しますか。」** | **「GET」** を選択 |
    | **「このエンドポイントのパラメータを追加して確認」** | チェックする |
    | **「このエンドポイントを構成してレスポンスを受信」** | チェックする |

    ![Oracle Integration Cloud](images/ss05-03.png)

    入力したら、**「REST エンドポイントの構成」** ボックスの右上の方にある **「次」** ボタンをクリックします。

1.  **「REST エンドポイントの構成」** ボックスの **「リクエスト・パラメータ」** では、RESAS-API を呼び出す際に指定する問合せパラメータを指定します。
    **「問合せパラメータの指定」** セクションにあるテーブルの **「＋」** アイコンをクリックし、次のように入力します。

    | 入力項目 | 入力する値 |
    |:----|:----|
    | **「名前」** | `prefCode` |
    | **「データ型」** | **「integer」** を選択 |

    ![Oracle Integration Cloud](images/ss05-04.png)

    入力し終わったら、**「次」** ボタンをクリックします。

1.  **「REST エンドポイントの構成」** ボックスの **「レスポンス」** では、 REST-API から返ってくるレスポンスの形式を指定します。
    **「レスポンス・ペイロード」** で **「JSONサンプル」** を選択してから、 **「<<< inline >>>」** リンクをクリックします。

    ![Oracle Integration Cloud](images/ss05-05.png)

1.  **「REST エンドポイントの構成」** ボックスの **「Enter Sample JSON」** に、RESAS-API の市区町村一覧が返すレスポンスのサンプルを入力します。
    [RESAS-API の市区町村一覧のドキュメント](https://opendata.resas-portal.go.jp/docs/api/v1/cities.html) の **「sample」** セクションの JSON のコードブロックか、次のコードブロックをコピーして、**「Enter Sample JSON」** のテキストエリアに、ペーストします。

    ```json
    {
        "message": null,
        "result": [{
            "prefCode": 1,
            "cityCode": "01100",
            "cityName": "札幌市",
            "bigCityFlag": "2"
        }, {
            "prefCode": 1,
            "cityCode": "01101",
            "cityName": "札幌市中央区",
            "bigCityFlag": "1"
        }, {
            "prefCode": 1,
            "cityCode": "01102",
            "cityName": "札幌市北区",
            "bigCityFlag": "1"
        }, {
            "prefCode": 1,
            "cityCode": "01103",
            "cityName": "札幌市東区",
            "bigCityFlag": "1"
        }]
    }
    ```

    ![Oracle Integration Cloud](images/ss05-07.png)

    **「OK」** ボタンをクリックします。

1.  **「REST エンドポイントの構成」** ボックスの右上にある **「次」** ボタンをクリックします。

    ![Oracle Integration Cloud](images/ss05-08.png)

1.  **「REST エンドポイントの構成」** ボックスの **「サマリー」** が表示されます。
    右上に表示される **「完了」** ボタンをクリックします。

    ![Oracle Integration Cloud](images/ss05-09.png)

1.  キャンバス・ビューは次の図のような表示になっています。

    ![Oracle Integration Cloud](images/ss05-10.png)

    - **「マップ先 GetCities」** は、REST API のリクエストにパラメータを指定する必要がある場合に使用するマッピングです。
      このチュートリアルでは、スケジュール・パラメータ `prefCode` で指定した値を RESAS-API を呼び出す際に指定する問い合わせパラメータにマップする設定を行います。
    - **「GetCities」** は、これまでに設定した REST API の呼び出しの設定です。

1.  キャンバス・ビューで **「マップ先 GetCities」** を選択し、表示された **「編集」** ボタンをクリックします。

    ![Oracle Integration Cloud](images/ss05-11.png)

1.  **「マップ先 GetCities」** ページが表示されます。

    ページの左側に表示される **「Sources」** ツリーに **「prefCode」** が表示されていることを確認します。
    この **「prefCode」** は、先ほど設定したスケジュール・パラメータです。

    次に、ページの右側に表示されている **「Targets」** ツリーに **「execute」** → **「QueryParameters」** → **「Pre fCode」** が表示されていることを確認します。
    この **「Pref Code」** は、RESAS-API の市区町村一覧 API を呼び出す際に指定する問い合わせパラメータです。

1.  **「Sources」** ツリーの **「prefCode」** を、 **「Targets」** ツリーの **「Pref Code」** の上にドラッグ＆ドロップします。

    ![Oracle Integration Cloud](images/ss05-12.png)

    **「Sources」** ツリーの **「$prefCode」** を、 **「Targets」** ツリーの **「prefCode」** が直線で結ばれます。

1.  ページの右上にある **「検証」** ボタンをクリックします。
    **「マッピングは有効で、使用する準備ができています。」** というメッセージが表示されたら、 **「閉じる」** ボタンをクリックします。

1.  これで、RESAS-API の市区町村一覧を取得するための設定が終わりました。
    ここまでの設定を保存するために、ページの右上に表示される **「保存」** ボタンをクリックします。

### Oracle ADB へのデータの書き込み

1.  キャンバス・ビューの右端に表示されている ![呼び出し](images/btn_invokes.png) （呼び出し）アイコンをクリックします。
    **「Oracle ATP」** の下をクリックすると **「ATP_CONN」** が表示されます。

    ![Oracle Integration Cloud](images/ss06-01.png)

1.  **「ATP_CONN」** を、キャンバス・ビューの **「GetCities」** アイコンから **「停止」** アイコン（背景が緑色のアイコン）に向けられた矢印の上にドラッグし、表示された **「＋」** マークの上でドロップします。

    ![Oracle Integration Cloud](images/ss06-02.png)

1.  **「Oracleアダプタ・エンドポイント構成ウィザード」** が表示されます。
    次のように **「基本情報」** を入力します。

    |入力項目|入力する値|
    |:----|:----|
    | **「エンドポイントにどのような名前をつけますか。」** | `MergeCities` |
    | **「どの操作を実行しますか。」** | **「Perform an Operation on a Table」** を選択 |
    | **「どの操作を表で実行しますか。」** | **「Insert or Update (Merge)」** を選択 |

    ![Oracle Integration Cloud](images/ss06-03.png)

    入力し終わったら、 **「次」** ボタンをクリックします。

1.  **「Oracleアダプタ・エンドポイント構成ウィザード」** の **「表での操作」** が表示されます。
    **「スキーマ」** では、今回割り当てられたデータベース・ユーザー名を選択し、 **「表タイプ」** では **「TABLE」** が選択されていることを確認したら、 **「表名」** フィールドの右隣にある **「検索」** ボタンをクリックします。

    **「使用可能」** リストに **「CITIES」** が表示されています。
    **「CITIES」** を選択した状態で、 **「>」** ボタンをクリックします。
    **「CITIES」** が、 **「選択済み」** リストに移動したことを確認したら、 **「表のインポート」** ボタンをクリックします。

    ![Oracle Integration Cloud](images/ss06-04.png)

1.  **「親データベース表の選択」** が表示されます。
    今回はデータを保存するのテーブルは `CITIES` だけなので、初期状態のままでウィザードの右上にある **「次」** ボタンをクリックします。

    ![Oracle Integration Cloud](images/ss06-05.png)

1.  **「Oracleアダプタ・エンドポイント構成ウィザード」** の **「サマリー」** が表示されます。
    ウィザードの右上にある **「完了」** ボタンをクリックします。

    ![Oracle Integration Cloud](images/ss06-06.png)

1.  キャンバス・ビューは次の図のような表示になっています。

    ![Oracle Integration Cloud](images/ss06-07.png)

    - **「マップ先 MergeCities」** は、 Oracle ADB に保存するデータのマッピングの設定を行います。
      次のセクションで設定します。
    - **「MergeCities」** は、これまでに設定した Oracle ADB の呼び出しの設定です。

1.  これで、RESAS-API の市区町村一覧のデータを Oracle ADB に保存するための設定が終わりました。
    ここまでの設定を保存するために、ページの右上に表示される **「保存」** ボタンをクリックします。

### データ・マッピングの設定

ここでは、RESAS-API により取得した市区町村一覧データと、Oracle ADB に保存するデータのマッピングの設定を行います。

1.  キャンバス・ビューで、 **「マップ先 MergeCities」** を選択します。
    表示される **「編集」** アイコンをクリックします。

    ![Oracle Integration Cloud](images/ss07-01.png)

1.  データのマッピングの設定を行うためのマッパーが表示されます。
    マッパーの画面左側に表示されている **「Sources」** ペインの **「GetCities Response (REST)」** → **「Execute Response」** → **「Response Wrapper」** → **「Result」** を、画面右側に表示されている **「Target」** ペインの **「MergeCities Request (Oracle ATP)」** → **「Cities」** にドラッグ＆ドロップします。

    ![Oracle Integration Cloud](images/ss07-03.png)

1.  **「Sources」** ペインの **「Pref Code」** を、 **「Target」** ペインの **「prefCode」** にドラッグ＆ドロップします。

    ![Oracle Integration Cloud](images/ss07-04.png)

1.  **「Sources」** ペインの **「City Code」** を、 **「Target」** ペインの **「cityCode」** にドラッグ＆ドロップします。

1.  **「Sources」** ペインの **「City Name」** を、 **「Target」** ペインの **「cityName」** にドラッグ＆ドロップします。

1.  **「Sources」** ペインの **「Big City Flag」** を、 **「Target」** ペインの **「bigCityFlag」** にドラッグ＆ドロップします。

    ![Oracle Integration Cloud](images/ss07-07.png)

1.  画面左上にある **「検証」** ボタンをクリックして、マッピングが有効かどうかを確認します。
    画面の上部に **「マッピングは有効で使用する準備ができています。」** というメッセージが表示されたら、 **「閉じる」** ボタンをクリックします。

### トラッキングの設定

Oracle Integration Cloud では、作成した統合が実行状況をトラッキングするための設定を行う必要があります。
トラッキングの設定は次の手順で行います。

1.  統合キャンバスの右上にある ![アクション](images/btn_actions.png)（アクション）アイコンをクリックします。
    表示されたメニューから **「トラッキング」** を選択します。

    ![Oracle Integration Cloud](images/ss08-01.png)

1.  **「トラッキング用のビジネス識別子」** ボックスが表示されます。
    ボックス左側のツリーから **「schedule」** の下の **「startTime」** を、ボックスの右側に表示されているテーブルの **「トラッキング・フィールド」** 列の1行目にドラッグ＆ドロップします。

    ![Oracle Integration Cloud](images/ss08-02.png)

1.  ボックス左側のツリーから **「$prefCode」** を、ボックスの右側に表示されているテーブルの **「トラッキング・フィールド」** 列の2行目にドラッグ＆ドロップします。

    ![Oracle Integration Cloud](images/ss08-04.png)

    ボックスの右下にある **「閉じる」** ボタンをクリックします。

1.  ページの右上にある **「保存」** ボタンをクリックしてから **「閉じる」** ボタンをクリックします。
    **「統合」** の一覧が表示されるページに戻ります。

    ![Oracle Integration Cloud](images/ss08-06.png)

## 統合の実行

作成した統合を実行してみましょう。

1.  Oracle Integration Cloud で作成した統合を実行するには、まずアクティブ化します。
    実行する統合にマウスポインタを合わせ、表示された **「アクティブ化」** アイコンをクリックします。

    ![Oracle Integration Cloud](images/ss09-01.png)

1.  **「統合のアクティブ化」** ボックスが表示されます。
    ボックスの右下に表示されている **「アクティブ化」** ボタンをクリックします。

    ![Oracle Integration Cloud](images/ss09-02.png)

    統合のステータスが **「アクティブ」** に変更されていることを確認します。

1.  実行する統合にマウスポインタを合わせ、表示された **「実行」** アイコンをクリックします。

    ![Oracle Integration Cloud](images/ss09-04.png)

1.  ポップアップの下に表示されている **「すぐに送信」** リンクをクリックします。

    ![Oracle Integration Cloud](images/ss09-05.png)

1.  **「すぐに送信」** ボックスが表示されたら、**「確認」** ボタンをクリックします。

1.  **「スケジュール・パラメータ」** ページが表示されます。
    今回はデフォルト値をそのまま使用することにして、ページの右上にある **「送信」** ボタンをクリックします。

    ![Oracle Integration Cloud](images/ss09-07.png)

    > ***Note:*** デフォルト値とは異なる値を設定する場合は **「新しい値」** に入力します。

1.  実行結果を確認してみましょう。
    実行結果は、トラッキングの機能を使用して確認できます。
    ナビゲーション・ペインの左上にある **「<」** アイコンをクリックします。

    ![Oracle Integration Cloud](images/ss09-09.png)

    ナビゲーション・ペインに **「Oracle Integration」** のメニューが表示されたら、 **「モニタリング」** をクリックします。

    ![Oracle Integration Cloud](images/ss09-10.png)

    **「統合」** を選択します。

    ![Oracle Integration Cloud](images/ss09-11.png)

    **「トラッキング」** を選択します。

    ![Oracle Integration Cloud](images/ss09-12.png)

1.  **「インスタンスのトラッキング」** ページが表示されます。

    ![Oracle Integration Cloud](images/ss09-13.png)

    ステータスが **「成功」** と表示されていることを確認します。
    インスタンスにマウス・ポインタを合わせると右側に表示される **「詳細の表示」** アイコンをクリックします。

1.  アクティビティ・ストリームが表示され、処理の流れが確認できます。

    ![Oracle Integration Cloud](images/ss09-15.png)

以上でこのチュートリアルは終了です。
