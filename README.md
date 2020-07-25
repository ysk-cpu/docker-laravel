# docker-laravel

API完成品

## 動作確認

1. Talend API Testerを開く
2. METHOD欄にGET、SCHEME欄にhttp://localhost:80/api/posts/を入力してSend
3. 200 OKが出れば成功。右下で返ってきたデータが見れるが、空になっているはず。

このAPIが正常に動作するかどうかの確認方法

## ここまで作る方法

### 環境設定編

参考：[最強のLaravel開発環境をDockerを使って構築する【新編集版】](https://qiita.com/ucan-lab/items/5fc1281cd8076c8ac9f4)

ターミナル作業ディレクトリを作り、入る

```
(terminal)
mkdir tmp
cd tmp
```

dockerでlaravelを動かすテンプレートがインターネット上にあるのでそれを利用する。

```
(terminal)
git clone https://github.com/ucan-lab/docker-laravel.git
cd infrastructure
```

新しいlaravelプロジェクトを作成し、必要なパッケージをインストールする。

（makeはコマンドの省略。元々どんなコマンドが省略されているかは/infrastructure/Makefileを参照。）

（makeはMakefileがあるinfrastructureディレクトリでしか使えないので、makeを使うときはcdで移動しておく。）

```
(terminal)
make create-project
make install-recommend-packages
```

### API作成編

参考：[作成してみよう！LaravelでAPIを扱う方法【初心者向け】](https://techacademy.jp/magazine/18786)

Postという名前のマイグレーションファイル（/backend/database/migrations/(日付)_create_posts_table.php）、モデルファイル（/backend/app/Post.php）、コントローラファイル（/backend/app/Http/Contrllers/PostController.php）を1コマンドで作成。

（説明）

（マイグレーションとはデータベースの構造を定義すること）

（モデルとは扱うデータの構造。例えば投稿(Post)には題名(title)と内容(content)が含まれているなど。）

（コントローラとはデータに対する操作を定義するもの。index()はデータ一覧を取得する、store()は新規データを追加するなど。）

```
(terminal)
cd ..       #1つ上のディレクトリへ移動
cd backend  #backendディレクトリに入る
php artisan make:model -m -c -r Post
```

マイグレーションファイルでAPIでやりとりしたいデータを定義する。

```
(/backend/database/migrations/(日付)_create_posts_table.php)
17行目と18行目の間に以下を挿入
$table->string('title');
$table->text('content');
```

マイグレーション実行

```
(terminal)
cd ../infrastructure  #再びinfrastructureディレクトリへ戻る
make migrate
```

モデルに$fillableプロパティを定義

```
(/backend/app/Post.php)
9行目と10行目の間に以下を挿入
protected $fillable = ['title', 'content'];
```

コントローラ実装

```
(/backend/app/Http/Contrllers/PostController.php)
以下のようにする
<?php

namespace App\Http\Controllers;

use App\Post;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        // 投稿を作成日時の新しい順でペジネーションした結果を返す
        return Post::latest()->paginate();
    }

    /**
     * Store a newly created resource in storage.
     *
     * @param \Illuminate\Http\Request $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        // リクエストパラメータから取得した値をもとにPostを作成する
        return Post::create($request->all());
    }

    /**
     * Display the specified resource.
     *
     * @param \App\Post $post
     * @return \Illuminate\Http\Response
     */
    public function show(Post $post)
    {
        // Eloquentオブジェクトを返す
        return $post;
    }

    /**
     * Update the specified resource in storage.
     *
     * @param \Illuminate\Http\Request $request
     * @param \App\Post $post
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, Post $post)
    {
        // リクエストパラメータから取得した値をもとにPostを更新する
        $post->update($request->all());
        return $post;
    }

    /**
     * Remove the specified resource from storage.
     *
     * @param \App\Post $post
     * @return \Illuminate\Http\Response
     */
    public function destroy(Post $post)
    {
        // Eloquentオブジェクトを削除するexit
        $deleted = $post->delete();
        return compact('deleted');
    }
}

```

APIルートを追加する

```
(/backend/routes/api.php)
末尾（20行目）に以下を追加
Route::apiResource('posts', 'PostController');
```

ルート確認

```
(terminal)
cd ../backend  #backendディレクトリに戻る
php artisan route:list --path=posts
```

完成
