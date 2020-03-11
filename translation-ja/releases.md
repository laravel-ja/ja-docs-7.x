# リリースノート

- [バージョニング規約](#versioning-scheme)
- [サポートポリシー](#support-policy)
- [Laravel 7](#laravel-7)

<a name="versioning-scheme"></a>
## バージョニング規約

Laravelとファーストパーティパッケージは、[セマンティックバージョニング](https://semver.org)にしたがっています。メジャーなフレームのリリースは、２月と８月の半年ごとにリリースされます。マイナーとパッチリリースはより細かく毎週リリースされます。マイナーとパッチリリースは、**決して**ブレーキングチェンジを含みません

皆さんのアプリケーションやパッケージからLaravelフレームワークかコンポーネントを参照する場合は、Laravelのメジャーリリースはブレーキングチェンジを含まないわけですから、`^7.0`のようにバージョンを常に指定してください。しかし、新しいメジャーリリースへ１日以内でアップデートできるように、私たちは常に努力しています。

<a name="support-policy"></a>
## サポートポリシー

Laravel6のようなLTSリリースでは、バグフィックスは２年間、セキュリティフィックスは３年間提供します。これらのリリースは長期間に渡るサポートとメンテナンスを提供します。 一般的なリリースでは、バグフィックスは６ヶ月、セキュリティフィックスは１年です。Lumenのようなその他の追加ライブラリでは、最新リリースのみでバグフィックスを受け付けています。また、[Laravelがサポートする](/docs/{{version}}/database#introduction)データベースのサポートについても確認してください。

| バージョン | リリース | バグフィックス期限 | セキュリティフィックス期限 |
| --- | --- | --- | --- |
| 5.5 (LTS) | ２０１７年８月３０日 | ２０１９年８月３０日 | ２０２０年８月３０日 |
| 5.6 | ２０１８年２月７日 | ２０１８年８月７日 | ２０１９年２月７日 |
| 5.7 | ２０１８年９月４日 | ２０１９年３月４日 | ２０１９年９月４日 |
| 5.8 | ２０１９年２月２６日 | ２０１９年８月２６日 | ２０２０年２月２６日 |
| 6 (LTS) | ２０１９年９月３日 | ２０２１年９月３日 | ２０２２年９月３日 |
| 7 | ２０２０年３月３日 | ２０２０年９月３日 | ２０２１年３月３日 |

<a name="laravel-7"></a>
## Laravel 7

Laravel7は、Laravel6.xで行われた向上に加え、以降の変更で構成されています。Laravel Airlockの導入、ルーティングスピードの向上、カスタムEloquentキャスト、Bladeコンポーネントタグ、Fluentな文字列操作、開発者に焦点を当てたHTTPクライアント、ファーストパーティCORSサポート、ルートモデル結合の制約の向上、スタブのカスタマイズ、ならびに多くのバグフィックスとユーザービリティの向上です

### Laravel Airlock

**Laravel Airlockは[Taylor Otwell](https://github.com/taylorotwell)により構築されました。**

Laravel Airlock（エアーロック）はSPA（Single Page Applications）のための、シンプルでトークンベースのAPIを使った羽のように軽い認証システムです。Airlockはアプリケーションのユーザーのアカウントごとに、複数のAPIトークンを生成できます。これらのトークンには、実行可能なアクションを限定するアビリティ・スコープを与えられます。

Airlockの詳細は、[Airlockのドキュメント](/docs/{{version}}/airlock)をご覧ください。

### カスタムEloquentキャスト

**カスタムEloquentキャストは[Taylor Otwell](https://github.com/taylorotwell)により構築されました。**

Laravelには多様な利便性のあるキャストタイプが用意されています。しかし、自分自身でキャストタイプを定義する必要が起きることもまれにあります。これを行うには、`CastsAttributes`インターフェイスを実装したクラスを定義してください。

このインターフェイスを実装するクラスでは、`get`と`set`メソッドを定義します。`get`メソッドはデータベースにある元の値をキャストした値へ変換することに責任を持ちます。一方の`set`メソッドはキャストされている値をデータベースに保存できる元の値に変換します。例として、組み込み済みの`json`キャストタイプをカスタムキャストタイプとして再実装してみましょう。

    <?php

    namespace App\Casts;

    use Illuminate\Contracts\Database\Eloquent\CastsAttributes;

    class Json implements CastsAttributes
    {
        /**
         * 指定された値をキャストする
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  mixed  $value
         * @param  array  $attributes
         * @return array
         */
        public function get($model, $key, $value, $attributes)
        {
            return json_decode($value, true);
        }

        /**
         * 指定された値を保存用に準備
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  array  $value
         * @param  array  $attributes
         * @return string
         */
        public function set($model, $key, $value, $attributes)
        {
            return json_encode($value);
        }
    }

カスタムキャストタイプが定義できたら、クラス名を使いモデル属性へ指定します。

    <?php

    namespace App;

    use App\Casts\Json;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * ネイティブなタイプへキャストする属性
         *
         * @var array
         */
        protected $casts = [
            'options' => Json::class,
        ];
    }

カスタムEloquentキャストの書き方は、[Eloquentドキュメント](/docs/{{version}}/eloquent-mutators#custom-casts)で調べてください。

### Bladeコンポーネントタグと向上

**Bladeコンポーネントタグは[Spatie](https://spatie.be/)、[Marcel Pociot](https://twitter.com/marcelpociot)、[Caleb Porzio](https://twitter.com/calebporzio)、[Dries Vints](https://twitter.com/driesvints)、[Taylor Otwell](https://github.com/taylorotwell)から貢献を受けました。**

> {tip} Bladeコンポーネントはタグベースのレンダリング、属性管理、コンポーネントクラス、インラインビューコンポーネントなどを実現するためにオーバーホールされました。Bladeコンポーネントのオーバーホールは広範囲に及んでいるため、この機能を学ぶために[Bladeコンポーネント全体](/docs/{{version}}/blade#components)を確認してください。

要約すれば、コンポーネントは、受け取るデータを指定する関連付けしたクラスが持てるようになりました。コンポーネントクラス上にあるパブリックのプロパティとメソッドはすべて自動的にコンポーネントビューで利用できます。コンポーネント上で指定された、追加のHTML属性は属性バッグインスタンスである`$attribute`変数で自動的に管理できます。

たとえば、`App\View\Components\Alert`コンポーネントが定義されていると仮定しましょう。

    <?php

    namespace App\View\Components;

    use Illuminate\View\Component;

    class Alert extends Component
    {
        /**
         * alertタイプ
         *
         * @var string
         */
        public $type;

        /**
         * コンポーネントインスタンスの生成
         *
         * @param  string  $type
         * @return void
         */
        public function __construct($type)
        {
            $this->type = $type;
        }

        /**
         * 指定されたalertタイプに対するクラスを取得
         *
         * @return string
         */
        public function classForType()
        {
            return $this->type == 'danger' ? 'alert-danger' : 'alert-warning';
        }

        /**
         * コンポーネントを表すビュー／コンテンツを取得
         *
         * @return \Illuminate\View\View|string
         */
        public function render()
        {
            return view('components.alert');
        }
    }

そして、関連するコンポーネントBladeテンプレートを以下のように定義してあるとしましょう。

    <!-- /resources/views/components/alert.blade.php -->

    <div class="alert {{ $classForType }}" {{ $attributes }}>
        {{ $heading }}

        {{ $slot }}
    </div>

このコンポーネントは他のBladeビューでコンポーネントタグを使用し、レンダーされるでしょう。

    <x-alert type="error" class="mb-4">
        <x-slot name="heading">
            Alertコンテント…
        </x-slot>

        デフォルトスロットの内容…
    </x-alert>

紹介したのはLaravel7で行ったBladeコンポーネントのオーバーホールの小さなサンプルであり、無名コンポーネント、インラインビューコンポーネントや他の数々の機能はデモンストレートしていません。この機能を学ぶには、どうぞ[Bladeコンポーネントの完全なドキュメント](/docs/{{version}}/blade#components)を調べてください。

> {note} 以前の`@component` Bladeコンポーネント記法は残っていますし、削除予定もありません。

### HTTPクライアント

**HTTPクライアントはGuzzleのラッパーであり、[Adam Wathan](https://twitter.com/adamwathan)、[Jason McCreary](https://twitter.com/gonedark)、[Taylor Otwell](https://github.com/taylorotwell)から貢献を受けました。**

他のWebアプリケーションと連携を取る、送信HTTPリクエストを簡単に作成できるよう、Laravelは小さくて読み書きしやすい[Guzzle HTTPクライアント](http://docs.guzzlephp.org/en/stable/)のAPIを提供しています。LaravelのGuzzleラッパーはもっとも繁用されるユースケースと開発者が素晴らしい体験をできることに重点を置いています。例として、クライアントでJSONデータを送る`POST`を簡単に作ってみましょう。

    use Illuminate\Support\Facades\Http;

    $response = Http::withHeaders([
        'X-First' => 'foo',
        'X-Second' => 'bar'
    ])->post('http://test.com/users', [
        'name' => 'Taylor',
    ]);

    return $response['id'];

さらに、このHTTPクライアントは豊富な素晴らしいテスト機能を備えています。

    Http::fake([
        // GitHubエンドポイントに対するJSONレスポンスをスタブ
        'github.com/*' => Http::response(['foo' => 'bar'], 200, ['Headers']),

        // Googleエンドポイントに対する文字列レスポンスをスタブ
        'google.com/*' => Http::response('Hello World', 200, ['Headers']),

        // GitHubエンドポイントに対して一連のレスポンスをスタブ
        'facebook.com/*' => Http::sequence()
                                ->push('Hello World', 200)
                                ->push(['foo' => 'bar'], 200)
                                ->pushStatus(404),
    ]);

HTTPクライアントの機能すべてについて学習するには、[HTTPクライアントドキュメント](/docs/{{version}}/http-client)を調べてください。

### Fluent文字列操作

**Fluent文字列操作は、[Taylor Otwell](https://twitter.com/taylorotwell)から貢献を受けました。**

Laravelにもともとある`Illuminate\Support\Str`クラスには皆さん慣れているでしょう。役に立つ、さまざまな文字列操作機能を提供しています。Laravel7ではよりオブジェクト指向で読み書きしやすい(Fluent)文字列操作ライブラリをこうした機能上に構築しました。Fluentな`Illuminate\Support\Stringable`オブジェクトは、`Str::of`メソッドで生成できます。数多くのメソッドを文字列操作のためにオブジェクトへチェーン可能です。

    return (string) Str::of('  Laravel Framework 6.x ')
                        ->trim()
                        ->replace('6.x', '7.x')
                        ->slug();

Fluentな文字列操作により使用できるメソッドの情報は、[ドキュメント全体](/docs/{{version}}/helpers#fluent-strings)をお読みください。

### ルートモデル結合の向上

**ルートモデル結合の向上は、[Taylor Otwell](https://twitter.com/taylorotwell)から貢献を受けました。**

#### キーのカスタマイズ

ときに`id`以外のカラムを使いEloquentモデルを解決したい場合もあります。そのためLaravel７ではルートパラメータ定義の中でカラムを指定できるようにしました。

    Route::get('api/posts/{post:slug}', function (App\Post $post) {
        return $post;
    });

#### 自動制約

一つの定義中に複数のEloquentモデルを暗黙的に結合し、２つ目のEloquentモデルが最初のEloquentモデルの子である必要がある場合などでは、その２つ目のモデルを取得したいと思うでしょう。例として、特定のユーザーのブログポストをスラグで取得する場合を想像してください。

    use App\Post;
    use App\User;

    Route::get('api/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
        return $post;
    });

カスタムなキーを付けた暗黙の結合をネストしたルートパラメータで使用するとき、親で定義されるリレーションは慣習にしたがい名付けられているだろうとLaravel7は推測し、ネストしたモデルへのクエリを自動的に制約します。この場合、`User`モデルには`Post`モデルを取得するために`posts`（ルートパラメータ名の複数形）という名前のリレーションがあると想定します。

ルートモデル結合の詳細情報は、[ルートのドキュメント](/docs/{{version}}/routing#route-model-binding)をご覧ください。

### 複数メールドライバ

**複数メールドライバのサポートは、[Taylor Otwell](https://twitter.com/taylorotwell)から貢献を受けました。**

Laravel7では１つのアプリケーションで複数の「メーラー」を設定できるようになりました。各メーラーはこのファイルの中でオプションや、独自の「トランスポート」でさえも設定しています。これにより、アプリケーションが特定のメッセージを送るため、異なったメールサービスを利用できるようになっています。たとえばアプリケーションで業務メールはPostmarkを使い、一方でバルクメールはAmazon SESと使い分けることができます。

Laravelは`mail`設定ファイルの`default`メーラーとして設定されているメーラーをデフォルトで使用します。しかし、特定のメール設定を使用してメッセージを遅るために`mailer`メソッドが使用できます。

    Mail::mailer('postmark')
            ->to($request->user())
            ->send(new OrderShipped($order));

### ルートキャッシュスピードの向上

**ルートキャッシュスピードの向上はアップストリームの[Symfony](https://symfony.com)貢献者達と[Dries Vints](https://twitter.com/driesvints)から貢献を受けました。**

Laravel7にはマッチングコンパイラの新しいメソッドや、`route:cache` Artisanコマンドによるルートのキャッシュが含まれています。大きなアプリケーション（たとえば８００以上のルートを持つアプリケーション）においてシンプルな"Hello World"ベンチマークでは、**２倍のスピード向上**という結果が出せます。アプリケーションに特別な変更は必要ありません。

### CORSサポート

**CORSのサポートは[Barry vd. Heuvel](https://twitter.com/barryvdh)から貢献を受けました。**

Laravel7では、Barry vd. Heuvelが書いた人気のLaravel CORSパッケージを統合し、Cross-Origin Resource Sharing (CORS) `OPTIONS`リクエストに対するファーストパーティサポートを提供しています。新しい`cors`設定は[デフォルトLaravelアプリケーションスケルトン](https://github.com/laravel/laravel/blob/develop/config/cors.php)に含まれています。

Laravel7.xにおけるCORSサポートの詳細は、[CORSのドキュメント](/docs/{{version}}/routing#cors)を調べてください。

### クエリ時間キャスト

**クエリ時間キャストは[Matt Barlow](https://github.com/mpbarlow)から貢献を受けました。**

テーブルから元の値でセレクトするときのように、クエリ実行時にキャストを適用する必要が稀に起きます。例として以下のクエリを考えてください。

    use App\Post;
    use App\User;

    $users = User::select([
        'users.*',
        'last_posted_at' => Post::selectRaw('MAX(created_at)')
                ->whereColumn('user_id', 'users.id')
    ])->get();

テーブルから元の値でセレクトするときのように、クエリ実行時にキャストを適用する必要が稀に起きます。例として以下のクエリを考えてください。これを行うにはLaravel7で提供している`withCasts`メソッドを使ってください。

    $users = User::select([
        'users.*',
        'last_posted_at' => Post::selectRaw('MAX(created_at)')
                ->whereColumn('user_id', 'users.id')
    ])->withCasts([
        'last_posted_at' => 'date'
    ])->get();

### MySQL8以上でのデータベースキューの向上

**MySQLデータベースキューの向上は[Mohamed Said](https://github.com/themsaid)から貢献を受けました。**

Laravelの以前のリリースでは、`database`キューは実機環境での使用に対して十分堅牢に考えられていないため、デッドロックを起こしていました。しかしながら、Laravel7ではMySQL8以上で裏打ちされたキューにより、このデータベースを使用したアプリケーションで改善されました。`FOR UPDATE SKIP LOCKED`節とその他のSQLの改善を利用し、高ボリュームの実働アプリケーションでも`database`ドライバーは安全に使えるようになりました。

### Artisan `test`コマンド

**`test`コマンドは、[Nuno Maduro](https://twitter.com/enunomaduro)から貢献を受けました。**

テスト実行では`phpunit`コマンドに加え、`test` Artisanコマンドも使用できます。Artisanテストランナーは美しいコンソールUXと現在実行中のテストに関するより詳しい情報を提供します。さらにテストに失敗した最初の時点で自動的に停止します。

    php artisan test

<p align="center">
<img src="https://res.cloudinary.com/dtfbvvkyp/image/upload/v1582142435/Screen_Shot_2020-02-19_at_2.00.01_PM.png">
</p>

`phpunit`コマンドで使用できる引数はすべてArtisan `test`コマンドにも渡せます。

    php artisan test --group=feature

### Markdownメールテンプレートの向上

**Markdownメールテンプレートの向上は[Taylor Otwell](https://twitter.com/taylorotwell)から貢献を受けました。**

デフォルトMarkdownメールテンプレートはTailwind CSSカラーパレットを基盤としたよりモダンなデザインに一新されました。もちろん、このテンプレートはリソース公開し、アプリケーションのニーズに合わせてカスタマイズできます。

<p align="center">
<img src="https://res.cloudinary.com/dtfbvvkyp/image/upload/v1582142674/Screen_Shot_2020-02-19_at_2.04.11_PM.png">
</p>

Markdownメールのより詳しい情報は、[メールドキュメント](/docs/{{version}}/mail#markdown-mailables)をご覧ください。

### スタブのカスタマイズ

**Stubのカスタマイズは[Taylor Otwell](https://twitter.com/taylorotwell)から貢献を受けました。**

Artisanコンソールの`make`コマンドは、コントローラ、マイグレーション、テストのような数多くのクラスを生成するために使われます。これらのクラスは皆さんの入力を元にして、「スタブ」ファイルへ値を埋め込み生成されます。場合により、Aritsanが生成するファイルを少し変更したい場合もあるでしょう。このためLaravel７は`stub:publish`コマンドで、カスタマイズのためにもっとも一般的なスタブをリソース公開できるようになりました。

    php artisan stub:publish

リソース公開されたスタブはアプリケーションのルート下の`stubs`ディレクトリの中に保存されます。そうしたスタブに加えた変更は、名前に対応するArtisan `make`コマンドを使用して生成するときに反映されます。

### キューの`maxExceptions`設定

**`maxExceptions`プロパティは[Mohamed Said](https://twitter.com/themsaid)から貢献を受けました。**

ジョブを何度も再試行するように指定している場合、指定した回数の例外が発生したことをきっかけにしてその再試行を失敗として取り扱いたい場合も起きると思います。Laravel7では、ジョブクラスに`maxExceptions`プロパティを定義してください。

    <?php

    namespace App\Jobs;

    class ProcessPodcast implements ShouldQueue
    {
        /**
         * 最大試行回数
         *
         * @var int
         */
        public $tries = 25;

        /**
         * 失敗と判定するまで許す最大例外数
         *
         * @var int
         */
        public $maxExceptions = 3;

        /**
         * ジョブの実行
         *
         * @return void
         */
        public function handle()
        {
            Redis::throttle('key')->allow(10)->every(60)->then(function () {
                // ロックが取得でき、ポッドキャストの処理を行う…
            }, function () {
                // ロックが取得できなかった…場合の処理を行う…
                return $this->release(10);
            });
        }
    }

この例の場合、アプリケーションがRedisのロックを取得できない場合は、そのジョブは１０秒でリリースされます。そして、２５回再試行を継続します。しかし発生した例外を３回処理しなかった場合、ジョブは失敗します。
