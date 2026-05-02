# Todoアプリを作ってみよう
これまでの例は「バックエンドからデータを取得する」だけだった。次は「フロントエンドからデータを送り、バックエンドに保存する（POST）」という双方向のやり取りを実装してみよう。

## ファイルの構成
```
backend-practice-1/
├── frontend/
│   ├── index.html
│   ├── src/
│   │   ├── App.tsx
│   │   └── ...
│   └── ...
└── backend/
    └── backend.py
```

## バックエンド（Todo API）
```py
import json
from wsgiref.simple_server import make_server

todos = [
    {"id": 1, "task": "バックエンドの基礎を理解する", "done": False},
    {"id": 2, "task": "フロントエンドと繋いでみる", "done": False}
]
next_id = 3

def todo_api(environ, start_response):
    global next_id
    method = environ['REQUEST_METHOD']
    
    # 全リクエスト共通で返すヘッダー（CORS対応含む）
    headers = [
        ('Content-type', 'application/json'),
        ('Access-Control-Allow-Origin', '*'),
        ('Access-Control-Allow-Headers', 'Content-Type'),
        ('Access-Control-Allow-Methods', 'GET, POST, OPTIONS')
    ]

    # 1. ブラウザの事前確認
    if method == 'OPTIONS':
        start_response('200 OK', headers)
        return [b'']
    
    # 2. データの取得
    elif method == 'GET':
        start_response('200 OK', headers)
        return [json.dumps(todos).encode('utf-8')]
    
    # 3. データの追加
    elif method == 'POST':
        # 送られてきたデータを読み込む
        size = int(environ.get('CONTENT_LENGTH', 0))
        data = json.loads(environ['wsgi.input'].read(size))
        
        # リストに追加
        new_todo = {"id": next_id, "task": data.get("task", ""), "done": False}
        todos.append(new_todo)
        next_id += 1
        
        start_response('201 Created', headers)
        return [json.dumps(new_todo).encode('utf-8')]
        
    # 想定外のメソッド
    start_response('405 Method Not Allowed', headers)
    return [b'{}']

print("起動: http://localhost:8080")
make_server('', 8080, todo_api).serve_forever()
```

## フロントエンド（React）

```tsx
import { useCallback, useEffect,useState } from "react";

// Todoの型定義
type Todo = {
  id: number;
  task: string;
  done: boolean;
};

function App() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [newTask, setNewTask] = useState("");

  // 1. サーバーからデータをもらう（GET）
  const fetchTodos = useCallback(async () => {
    const response = await fetch("http://localhost:8080");
    const data: Todo[] = await response.json();
    setTodos(data);
  }, []);

  // アプリを開いた時に1回だけ実行
  useEffect(() => {
    fetchTodos();
  }, [fetchTodos]);

  // 2. サーバーへデータを送る（POST）
  const handleAddTodo = async () => {
    if (!newTask) return; // 空なら無視

    await fetch("http://localhost:8080", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ task: newTask }), // JSONにして送る
    });

    setNewTask(""); // 入力欄を空にする
    fetchTodos();   // 最新のリストをサーバーから取り直す
  };

  return (
    <div>
      <h2>自分のTodoリスト</h2>
      
      {/* 入力エリア */}
      <div>
        <input 
          type="text" 
          value={newTask} 
          onChange={(e) => setNewTask(e.target.value)}
          placeholder="新しいタスクを入力"
        />
        <button onClick={handleAddTodo} type="submit">追加</button>
      </div>
      
      {/* リスト表示エリア */}
      <ul>
        {todos.map((todo) => (
          <li key={todo.id}>
            <input type="checkbox" checked={todo.done} readOnly />
            {todo.task}
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```

# HTTPメソッド

先ほどのTodoアプリのPythonコードを見ると、`environ['REQUEST_METHOD']` という部分で **「GET」** や **「POST」** という単語を判定していました。これが**HTTPメソッド**です。

## 1. GET（取得）
* **用途:** データベースやサーバーから情報を読み取るときに使います。
* **例:** Todoリストの一覧を表示する。


注:ブラウザでURLを入力してアクセスしたときは、必ずこの「GET」メソッドが使われます。何度実行してもサーバーの状態は変わりません（安全な操作）。

## 2. POST（作成）
* **用途:** フロントエンドからデータを送り、サーバーに新しい情報を登録させるときに使います。
* **例:** 新しいTodoを入力して「追加」ボタンを押す。
* **特徴:** リクエストの「ボディ（中身）」にデータを詰めて送ります。

## 3. PUT / PATCH（更新）
* **用途:** すでにある情報を書き換えるときに使います。

## 4. DELETE（削除）
* **用途:** データを削除するときに使います。
* **例:** Todoの横にあるゴミ箱ボタンを押す。

### なぜ分ける必要があるのか

「全部GETやPOSTで送ればいいのでは？」と思うかもしれません。実際、技術的にはPOSTだけで全ての操作を行うことも可能です。しかし、メソッドを分けることで「この通信が何をしようとしているのか」が、人間にも、ブラウザにも、中継するネットワーク機器にも一目でわかるようになります。例えば、「GETは安全だから結果をキャッシュしておこう」「DELETEはデータが消える危険な操作だから、勝手に再実行しないようにしよう」といった制御ができるようになるため、Web全体の効率と安全性が保たれているのです。

# データは本当に永続化できているのか?
上記のTodoアプリでタスクをいくつか追加した後、Pythonのバックエンドサーバーを一度再起動 してみてください。すると、追加したはずのTodoが消え、初期状態の2つだけに戻ってしまいます。なぜなら、今回のコードでは `todos = [...]` というPythonのメモリ上にデータを保存しているだけだからです。