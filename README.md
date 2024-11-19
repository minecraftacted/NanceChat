# Nance Chat
🔰p2p🔰 <= 初心者です
## 概要
Nance Chatは、ピアツーピア（P2P）チャットアプリケーションです。このアプリケーションは、UDP hole punchingを使いますが、NATの状況よってはNAT超えができない場合もあります。（その場合はサーバとしては動きません。）

このNance Chatは一日おきにすべてのチャットとメッセージが消えるようにしています。（プログラム起動時からの時間）

## ピアの種類
通常のピアとブートストラップピアがあります。
ブートストラップピアは基本的に誰かが最初にピアを見つけるためのピアです。
ピアを見つけるためなので、チャットの機能はありません。
そして、ポートを常に開放していなければなりません。

## 機能
- **P2P通信**:中央サーバーを必要とせず、ピア間で直接通信が可能です。
- **STUNサポート**:STUNサーバーを使用して、パブリックIPとNATタイプを自動的に検出します。
- **チャット管理**:ユーザーはチャットルームを作成し、管理できます。
- **メッセージ処理**:リアルタイムでメッセージの送受信をサポートします。

## インストール
Nance Chatを実行するには、Python 3.xがインストールされていることを確認し、必要なライブラリをインストールします。以下のコマンドを使用してライブラリをインストールできます：

```bash
pip install flask requests pyyaml stun
```

## 使用方法
1. **アプリケーションの起動**:メインスクリプトを実行して、サーバーとWebインターフェースを起動します。(この時にブートストラップサーバーのIPが求められるので、事前に知っておいてください)
   ```bash
   python src/nancePeer.py
   ```

2. **Webインターフェースにアクセス**:Webブラウザを開き、`http://localhost:8080`に移動してチャットインターフェースにアクセスします。

3. **新しいチャットの作成**:Webインターフェースを使用して新しいチャットルームを作成し、メッセージを開始します。
## プロトコル
### データサイズ
- 一つの通信は1MBまでです。（base64にエンコードしたときのサイズ）
### 適用されるすべて
ピアやノードとの通信では、UDPで、UTF-8文字列の辞書型データをBase64でエンコードしたバイト配列または文字列が送受信されます。送信されるプロトコルの詳細は以下の通りです：
*注意:大文字と小文字の区別が重要です。*
```json
{
    "m":"Mode(p,r,c,s,R)",
    // 次の部分はモードによって異なります
    "z":"リクエスト識別子（任意）"
}
```

### p(ping)
他のコンピュータとの接続を確認します。
```json
{
    "m":"p"
}
```

### r(request)
取得または登録のためのリクエストです。
```json
{
    "m":"r",
    "t":"Request Type(g,r)",
    "d":"Content Type(c,p,i)",
    "a":{
        // 引数（内容によって異なる）
    }
}
```

#### g(get)
他のピアまたはブートストラップピアから情報を取得します。

##### c(chats)
チャットのリストをリクエストします（チャット名は含まれません）。
```json
{
    "m":"r",
    "t":"g",
    "d":"c",
    "a":{
        "maxChatsLength":"受信可能なチャットの長さ"
    }
}
```

##### m(messages)
チャット内のメッセージをリクエストします（チャット名は含まれません）。
```json
{
    "m":"r",
    "t":"g",
    "d":"m",
    "a":{
        "chatUuid":"チャットUUID"
    }
}
```

##### i(information)
他のピアまたはブートストラップピアからピア情報を取得します。
```json
{
    "m":"g",
    "t":"r",
    "d":"i",
    "a":{
        "maxPeersLength":"受信可能なピアの長さ"
    }
}
```
ネットワーク識別子（n）は、事前にそのチャットのネットワークIDとして知られている必要があります。

#### r(register)
自分の情報をブートストラップピアに登録します。

##### i(information)
ブートストラップピアまたはピアにピア情報を登録します。
これはネットワークに参加するための最初のステップです！
```json
{
    "m":"r",
    "t":"r",
    "d":"i",
    "a":{
        "natConeType":"NAT状態（文字列）（Restricted ConeまたはFull Cone）",
    }
}
```

#### s(send)
登録とは少し異なりますが、軽い情報の登録です。

##### m(message)
```json
{
    "m":"r",
    "t":"s",
    "d":"m",
    "a":{
        "chatUuid":"チャットUUID",
        "name":"チャット作成者の名前",
        "content":"チャット内容"
    }
}
```

### R(response)
リクエストが送信されたときに返されるレスポンスです。
```json
{
    "m":"R",
    "r":"結果（整数、成功の場合は0、エラーの場合はそれ以外）",
    "c":{
        // レスポンスの内容
    }
}
```

#### 結果が0の場合
結果が0の場合は、リクエストが成功したことを示します。

##### レスポンスの内容
- チャットリスト
    ```json
    {
        "m":"R",
        "r":0,
        "c":{
            "chats":[
                {"uuid":"ユニークなチャットUUID", "name":"チャット名", "timestamp":"Unixタイムスタンプ"}
            ]
        }
    }
    ```

- メッセージリスト
    ```json
    {
        "m":"R",
        "r":0,
        "c":{
            "messages":[
                {"uuid":"ユニークなメッセージUUID", "name":"名前", "content":"メッセージ内容", "timestamp":"Unixタイムスタンプ"}
            ]
        }
    }
    ```

- ピア情報の取得
    ```json
    {
        "m":"R",
        "r":0,
        "c":{
            "peers":[
                {"natConeType":"NATタイプ（Restricted ConeまたはFull Cone）", "ip":"ピアのip", "port":"ピアのport"}
            ]
        }
    }
    ```

- ピア情報の登録（ブートストラップピアから返されるレスポンス）
    ```json
    {
        "m":"R",
        "r":0,
        "c":{}
    }
    ```

- ping
    ```json
    {
        "m":"R",
        "r":0,
        "c":{}
    }
    ```

#### 結果が0以外の場合（エラーケース）
エラーが発生した場合は、次のようになります。
```json
{
    "m":"R",
    "r":"エラーコード（整数、1以外）",
    "c":{}
}
```
##### エラーコード
###### 0系列
- 1 :例外が発生しました
- 2 :不明なフォーマット
- 3 :あなたにアクセスできません
###### 10系列（get/send）
- 10 :指定されたアイテムは存在しません
###### 20（send）
- 20 :これ以上挿入できません（メッセージ）
###### 90（local error）
- 90 : レスポンスはもう帰ってきません
### z
リクエストとレスポンスに関係があるかどうかを紐づけられます。
この引数は文字列ならUUIDなどどのようなものでもよく、任意です。(リクエストについている場合はレスポンスにもついている必要があります)
#### 例（リクエスト）
```json
{
    "m":"r",
    "t":"r",
    "d":"i",
    "a":{
        "natConeType":"Full Cone",
    },
    "z":"f79826df-27b1-9ee4-1b2d-c9ccf097137f"
}
```
#### 例（レスポンス）
```json
{
    "m":"R",
    "r":0,
    "c":{},
    "z":"f79826df-27b1-9ee4-1b2d-c9ccf097137f"
}
```