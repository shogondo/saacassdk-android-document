# SA@CAS SDK for Android Developer's Guide

## 更新履歴

| バージョン | 日付 | 変更内容 |
|-|-|-|
| 1.0 | 2015/4/5 | 初版 |


## SA@CAS SDK for Android について

SA@CAS SDK for Android は、USB接続タイプのSSC通信モジュール経由で超音波通信を行うためのAndroid用のライブラリです。


### 前提条件

このライブラリを使用するための前提条件は以下の通りです。

* Android 4.0 (API Level 14) 以上


### 利用方法

次の二つのjarファイルをプロジェクトに組み込んでください。

* **sscsdk-android-usb-1.0.0.jar**  
  USB接続タイプのSSC通信モジュール経由で超音波通信を行うため低レベルAPIを提供するライブラリです。  
  SA@CAS SDK for Android の超音波通信処理は、このライブラリに依存しています。
* **saacassdk-android-1.0.0.jar**  
  上記ライブラリを抽象化したラッパーライブラリです。


## 実装

SA@CAS SDK for Android を使った超音波通信は、以下の流れで行います。

### 1. 超音波通信環境をセットアップする

超音波通信を行う際は、必ず事前にセットアップ処理を行ってください。

```
public class MainActivity {
    protected void onStart() {
        super.onStart();
        SaacasSSCClient.getInstance().prepare(this);
    }
}
```

なお、このメソッド内でUSBを利用するためのパーミッションリクエストを行います。
そのため、次のようなダイアログが表示されます。

![USBパーミッションリクエスト](require_permission.png)

### 2. 超音波通信コネクションを開く

次に、`SaacasSSCClient#open` を呼び出して、超音波通信コネクションを開きます。
このメソッドを呼び出すと、SSC通信モジュールがポーリングを開始します。
ユーザーがスマートフォンをSSC通信モジュールにかざすと、超音波通信コネクションが確立され、データの送受信が可能になります。

```
SaacasSSCClient.getInstance().open();
```

### 3. データを送信する

超音波通信コネクションが開いている間は、`SaacasSSCClient#send` でスマートフォンにデータを送信できます。
戻り値として、スマートフォンから返されたレスポンスパケットを受け取ることができます。

```
String requestCode = "0000";
String data = "ABCDEFG";
Packet packet = SaacasSSCClient.getInstance().send(requestCode, data);
```

### 4. 超音波通信コネクションを閉じる

必要な通信が完了したら、`SaacasSSCClient#close` で超音波通信コネクションを閉じます。

```
SaacasSSCClient.getInstance().close();
```

なお、`SaacasSSCClient` のメソッドは、呼び出されたスレッドで処理を行うので、UIスレッドをブロックしないように、
2から4はワーカースレッドで実行すべきです。

以下に、`AsyncTask` で超音波通信を行うサンプルを示します。

```
public class Task extends AsyncTask<Object, Integer, Packet> {
    protected Packet doInBackground(Object... params) {
        SaacasSSCClient ssc = SaacasSSCClient.getInstance();
        ssc.open();
        Packet packet = ssc.send("0001", "ABCDEFG");
        ssc.close();
        return packet.
    }
}
```

### 5. 超音波通信環境をクリーンアップする

超音波通信を行う必要がなくなったら（アプリ終了時や画面遷移時など）、`SaacasSSCClient#dispose` を呼び出してください。

```
public class MainActivity {
    protected void onStop() {
        super.onStop();
        SaacasSSCClient.getInstance().dispose();
    }
}
```

## クラスライブラリ

### SaacasSSCClientクラス

超音波通信機能を提供するクラスです。

* `public static SaacasSSCClient getInstance()`  
  SaacasSSCClientのシングルトンインスタンスを取得します。

* `public void prepare(Context context)`  
  超音波通信環境のセットアップ(USBを利用するためのパーミッションリクエストとシリアルポートのオープン)を行います。  
  正常に処理を終了できなかった場合は `SSCException` をスローします。

* `public void open()`  
  超音波通信コネクションをオープンし、リスナー（通信相手のスマートフォン）を探索するためのポーリングを開始します。  
  正常に処理を終了できなかった場合は `SSCException` をスローします。

* `public Packet send(String requestCode, String data)`  
  リスナー（通信相手のスマートフォン）にデータを送信し、リスナーからのレスポンスパケットを返します。  
  正常に処理を終了できなかった場合は `SSCException` をスローします。

* `public void close()`  
  超音波通信コネクションをクローズします。

* `public void dispose()`  
  超音波通信環境のクリーンアップ(シリアルポートの解放)を行います。

* `public void beep()`  
  SSC通信モジュールからビープ音を鳴らします。  
  このメソッドは超音波通信コネクションが確立されていない状態でも動作します。

* `public String getDeviceId()`  
  SSC通信モジュールの端末IDを取得します。  
  このメソッドは超音波通信コネクションが確立されていない状態でも動作します。


### Packetクラス

クライアントとリスナー間で送受信するデータパケットを表します。

* `public String getRequestCode()`  
  処理コードを取得します。

* `public String getResponseCode()`  
  リターンコードを取得します。

* `public String getData()`  
  データを取得します。

### Configurationクラス

超音波通信の各種設定を表すクラスです。

* `public int getBaudRate()` / `public void setBaudRate(int value)`  
  ボーレートを取得、設定します。デフォルト値は38400です。  
  超音波通信の基本仕様が変わらない限り、この値を変更する必要はありません。

* `public DataBit getDataBit()` / `public void setDataBit(DataBit value)`  
  データビットを取得、設定します。デフォルト値は `DataBit.Eight` です。  
  超音波通信の基本仕様が変わらない限り、この値を変更する必要はありません。

* `public StopBit getStopBit()` / `public void setStopBit(StopBit value)`  
  ストップビットを取得、設定します。デフォルト値は `StopBit.One` です。  
  超音波通信の基本仕様が変わらない限り、この値を変更する必要はありません。

* `public Parity getParity()` / `public void setParity(Parity value)`  
  パリティを取得、設定します。デフォルト値は `Parity.None` です。  
  超音波通信の基本仕様が変わらない限り、この値を変更する必要はありません。

* `public int getReadTimeout()` / `public void setReadTimeout(int value)`  
  シリアルポートからのデータ読み取りのタイムアウト(ミリ秒)を取得、設定します。デフォルト値は2000です。

* `public int getWriteTimeout()` / `public void setWriteTimeout(int value)`  
  シリアルポートへのデータ書き込みのタイムアウト(ミリ秒)を取得、設定します。デフォルト値は2000です。

* `public byte getRetryCount()` / `public void setRetryCount(byte value)`  
  `SaacasSSCClient#send` での通信失敗時のリトライ回数を取得、設定します。デフォルト値は3です。

* `public byte getPollingCount()` / `public void setPollingCount(byte value)`  
  `SaacasSSCClient#open` 時のポーリング回数(10回単位)を取得、設定します。デフォルト値は1です。

* `public byte getPollingInterval()` / `public void setPollingInterval(byte value)`  
  `SaacasSSCClient#open` 時のポーリング間隔(ミリ秒)を取得、設定します。デフォルト値は60です。

* `public byte getWaitInterval()` / `public void setWaitInterval(byte value)`  
  `SaacasSSCClient#send` のリトライ時のインターバル(ミリ秒)を取得、設定します。デフォルト値は60です。

* `public boolean getEnableFEC()` / `public void setEnableFEC(boolean value)`  
  前方誤り訂正の有効フラグを取得、設定します。デフォルト値はfalseです。

* `public byte getSendLevel()` / `public void setSendLevel(byte value)`  
  送信出力レベルを取得、設定します。デフォルト値は127です。

* `public byte getReceiveLevel()` / `public void setReceiveLevel(byte value)`  
  送信閾値を取得、設定します。デフォルト値は127です。

* `public short getPrivateCode()` / `public void setPrivateCode(short value)`  
  プライベートコードを取得、設定します。デフォルト値は0です。

* `public short getPassCode()` / `public void setPassCode(short value)`  
  暗証番号を取得、設定します。デフォルト値は-1です。

* `public boolean getUsePassCode()` / `public void setUsePassCode(boolean value)`  
  暗証番号を利用するかどうかを取得、設定します。デフォルト値はfalseです。


### RequestCodeクラス

処理コードを定義した定数管理クラスです。

*処理コード未定*

### ResponseCodeクラス

リターンコードを定義した定数管理クラスです。

*リターンコード未定*

### SSCExceptionクラス

超音波通信関連で発生したエラーを表す例外クラスです。

* `public Packet getResponsePacket()`  
  レスポンスパケットを取得します。  
  エラーに関する情報はレスポンスパケットのリターンコードを参照してください。


## サンプルアプリについて

次の２つのアプリをそれぞれクーポン受信端末、スマートフォンにインストールして、SDKの動作確認ができます。

* **saacassdk-android-sample-client-1.0.0.apk**  
  クライアント側サンプルアプリです。クーポン受信端末にインストールしてください。

* **saacassdk-android-sample-listener-1.0.0.apk**  
  リスナー側サンプルアプリです。スマートフォンにインストールしてください。

インストールが完了したら、次の手順で超音波通信の確認をしてください。

1. **【クーポン受信端末】** アプリを起動してください。
  USBのアクセス許可を求めるダイアログが表示されるので、OKをタップしてください。  
  ![USBパーミッションリクエスト](require_permission.png)

2. **【スマートフォン】** アプリを起動してください。クーポンの一覧が表示されるので、使いたいクーポンを選んでください。  
  ![クーポン一覧](coupon_list.png)

3. **【クーポン受信端末】** 「クーポンを受け取る」ボタンをタップしてください。
  下図のようなダイアログが表示され、クーポン受信待ち状態になります。  
  ![タップ前](before_button_click.png)  
  ![タップ後](after_button_click.png)

4. **【スマートフォン】** スマートフォンをSSC通信モジュールにかざしてください。

5. **【クーポン受信端末】** 正常に受信できると、受け取ったクーポンが画面上に表示されます。  
  ![受信したクーポン](coupon_received.png)  
