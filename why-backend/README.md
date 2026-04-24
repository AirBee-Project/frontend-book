# なぜバックエンドが必要なのか

フロントエンドは静的なサーバーにデプロイされている。そのため、ブラウザで開いたときの挙動は基本的に同じになる。Reactでバックエンドを持たないメモ帳のような機能を作ったとしたら、それらのデータはメモリ上に保存され、リロードすれば消えてしまう。

また、ブラウザ上にも永続化可能なストレージ（ローカルストレージ,Cookie）などがある。これらを使えば永続化することができるが、ブラウザのキャッシュ削除などによって消滅する。また、同じユーザーが異なるデバイスから同じ内容を参照することなどが難しい。

これらの課題を解決するのがバックエンドである。バックエンドはたいていの場合（最近は計算資源や特定のデータをを提供するサービスもあるが...）それ自体が広義の意味のストレージデバイスである。つまり、ブラウザの動作寿命を超えて、データを保存し続けるためのものである。

# Pythonでバックエンドを書いてみよう

## 文字列を送ろう

```py
from wsgiref.simple_server import make_server

def hello_app(environ, start_response):

    # ステータスコードとヘッダーを決める
    start_response('200 OK', [('Content-type', 'text/plain')])

    # レスポンスを返す
    return [b"Hello World"]

print("起動: http://localhost:8080")
make_server('', 8080, hello_app).serve_forever()
```

起動してアクセスしてみると下記のようなログが流れるはず

```sh
起動: http://localhost:8080
127.0.0.1 - - [24/Apr/2026 12:29:17] "GET / HTTP/1.1" 200 11
127.0.0.1 - - [24/Apr/2026 12:29:17] "GET /favicon.ico HTTP/1.1" 200 11
```

## 時刻を送ろう

```py
import datetime
from wsgiref.simple_server import make_server

def hello_app(environ, start_response):

    # ステータスコードとヘッダーを決める
    start_response('200 OK', [('Content-type', 'text/plain')])

    # 時刻を取得して送ってみる
    time = datetime.datetime.now()
    time_str=str(time).encode('utf-8');

    return [time_str]

print("起動: http://localhost:8080")
make_server('', 8080, hello_app).serve_forever()
```

## 画像を返そう

実行ディレクトリに`sample.jpg`という適当な画像を配置して、実行してみよう。

```py
from wsgiref.simple_server import make_server

def image_app(environ, start_response):

    start_response('200 OK', [('Content-type', 'image/jpeg')])

    with open('sample.jpg', 'rb') as f:
        image_data = f.read()

    return [image_data]

print("起動: http://localhost:8080")
make_server('', 8080, image_app).serve_forever()
```

## HTMLを返そう

```py
import datetime
from wsgiref.simple_server import make_server

def hello_app(environ, start_response):

    start_response('200 OK', [('Content-type', 'text/html; charset=utf-8')])

    time = datetime.datetime.now()

    html_content = f"""
    <!DOCTYPE html>
    <html>
        <head>
            <title>現在時刻</title>
        </head>
        <body>
            <h1>バックエンドから時刻を受け取る</h1>
            <p>サーバーの現在時刻は <strong>{time}</strong> です！</p>
        </body>
    </html>
    """

    return [html_content.encode('utf-8')]

print("起動: http://localhost:8080")
make_server('', 8080, hello_app).serve_forever()
```

### CSSを付け足してみよう

```py
import datetime
from wsgiref.simple_server import make_server

def hello_app(environ, start_response):

    start_response('200 OK', [('Content-type', 'text/html; charset=utf-8')])

    time = datetime.datetime.now()

    html_content = f"""
    <!DOCTYPE html>
    <html>
        <head>
            <title>現在時刻</title>
            <style>
                strong {{
                    color: #007bff;
                    font-weight: bold;
                }}
            </style>
        </head>
        <body>
            <h1>バックエンドから時刻を受け取る</h1>
            <p>サーバーの現在時刻は <strong>{time}</strong> です！</p>
        </body>
    </html>
    """

    return [html_content.encode('utf-8')]

print("起動: http://localhost:8080")
make_server('', 8080, hello_app).serve_forever()
```

### JavaScriptを付け足してみよう

```py
import datetime
from wsgiref.simple_server import make_server

def hello_app(environ, start_response):

    start_response('200 OK', [('Content-type', 'text/html; charset=utf-8')])

    time = datetime.datetime.now()

    html_content = f"""
    <!DOCTYPE html>
    <html>
        <head>
            <title>現在時刻</title>
            <style>
                strong {{
                    color: #007bff;
                    font-weight: bold;
                }}
            </style>
        </head>
        <body>
            <h1>バックエンドから時刻を受け取る</h1>
            <p>サーバーの現在時刻は <strong>{time}</strong> です！</p>
            <script>
                alert("時刻を表示するんや");
            </script>
        </body>
    </html>
    """

    return [html_content.encode('utf-8')]

print("起動: http://localhost:8080")
make_server('', 8080, hello_app).serve_forever()
```

# 疑問生まれる

上述のPythonの例のように、バックエンドのコードのみでWEBページを表示することは可能である。しかも、CSSをつけたり、JavaScirptをつけたりすることもできる。理論的には1つの完全なページを作ることもできるだろう。

ではなぜフロントエンドが必要なのだろうか?全てをサーバー側でいい感じに書いてすべてをフロントエンドに送ればいいのではないか?

なぜ、バックエンドとフロントエンドは分かれているのか?

# フロントエンドとバックエンドを分けてみよう

## バックエンド

```py
import datetime
import json
from wsgiref.simple_server import make_server

def time_api(environ, start_response):
    headers = [
        ('Content-type', 'application/json; charset=utf-8'),
        ('Access-Control-Allow-Origin', '*')
    ]
    start_response('200 OK', headers)

    data = {
        "time": str(datetime.datetime.now()),
        "message": "Hello from Python API!"
    }

    return [json.dumps(data).encode('utf-8')]

print("APIサーバー起動: http://localhost:8080")
make_server('', 8080, time_api).serve_forever()
```

## フロントエンド

```sh
bun create vite@latest
```

からプロジェクトを作成してください。React&TypeScriptになるようにプロジェクトを設定してください。

```tsx
import { useState, useEffect } from "react";

type ApiResponse = {
  time: string;
  message: string;
};

function App() {
  const [serverData, setServerData] = useState<ApiResponse | null>(null);
  const [loading, setLoading] = useState<boolean>(true);

  const fetchTime = async () => {
    setLoading(true);
    try {
      const response = await fetch("http://localhost:8080");
      const data: ApiResponse = await response.json();
      setServerData(data);
    } catch (error) {
      console.error("通信エラー:", error);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchTime();
  }, []);

  return (
    <div style={{ padding: "20px", textAlign: "center" }}>
      <div
        style={{
          border: "1px solid #ccc",
          padding: "20px",
          borderRadius: "10px",
        }}
      >
        {loading ? (
          <p>読み込み中...</p>
        ) : (
          <>
            <p>
              サーバーからのメッセージ: <strong>{serverData?.message}</strong>
            </p>
            <p>
              取得した時刻:{" "}
              <span style={{ color: "#007bff", fontSize: "1.2rem" }}>
                {serverData?.time}
              </span>
            </p>
          </>
        )}
      </div>
      <button onClick={fetchTime} style={{ marginTop: "20px" }}>
        手動で更新
      </button>
    </div>
  );
}

export default App;
```

# 疑問の答え

## 拡張性に優れるため

例えば、モバイルアプリを新規に作りたい場合や、別ページで同じAPIを実行したい場合にはAPIごとに切り出しておいたほうが、拡張が楽である。

## ユーザー体験をよくするため

すべてをバックエンドで処理して完成されたHTMLを返す方式だと、サーバーでのデータ取得処理がすべて終わるまで、ユーザーの画面は「真っ白な状態」でフリーズしてしまう。つまり「読み込み中...」という文字すら表示させることができない。

しかし、フロントエンドを分離する現代の手法では、まずフロントエンドが「枠組みだけの画面」をユーザーに一瞬で表示する。そして、裏側でJavaScriptが fetch を使ってデータを取得しにいき、待機時間にはクルクル回るアニメーションや「読み込み中」といった表示を出すことができる。

また、トップページで「最新のお知らせAPI」と「ユーザープロフィールAPI」の複数のAPIを同時に叩く場合。サーバーからHTMLを送る方式だと、処理が遅いAPIに足を引っ張られ、すべての準備が整うまで画面が出ない。フロントエンドから個別のAPIを叩く形式にすれば、取得できたデータから順番に画面にと表示していくことができるため、ユーザーに「待たされている感」を与えない快適な体験を提供できる。

# バックエンドが学びにくい理由はなぜか

近年、バックエンドとフロントエンドの表層的な統合が進み、初心者がいきなりNext.jsなどの環境を触ることで古き良きfetchでAPIをたたいて、フロントエンドでレンダリングするという状態が見えずらくなってしまっている。そればかりか、「バックエンド入門」時に特定の言語やフレームワークをもとに学んでしまい、「バックエンド」が指し示す言葉の本質がわかりにくくなってしまっている。こういうのが増えているので、フロントエンドに書いてはいけない情報を書いてしまうアホが現れる。
