# アップグレードガイド

- [6.xから7.0へのアップグレード](#upgrade-7.0)

<a name="high-impact-changes"></a>
## 重要度の高い変更

<div class="content-list" markdown="1">
- [認証スカフォールド](#authentication-scaffolding)
- [データシリアライズ](#date-serialization)
- [Symfony5へのアップグレード](#symfony-5-related-upgrades)
</div>

<a name="medium-impact-changes"></a>
## 重要度が中程度の変更

<div class="content-list" markdown="1">
- [Bladeコンポーネントと"Blade x"](#blade-components-and-blade-x)
- [CORSサポート](#cors-support)
- [Factoryタイプ](#factory-types)
- [Markdownメールテンプレートアップデート](#markdown-mail-template-updates)
- [`Blade::component` メソッド](#the-blade-component-method)
- [`assertSee`アサーション](#assert-see)
- [`different`バリデーションルール](#the-different-rule)
- [一意のルート名](#unique-route-names)
</div>

<a name="upgrade-7.0"></a>
## 6.xから7.0へのアップグレード

#### アップグレード見積もり時間：１５分

> {note} 私達は、互換性を失う可能性がある変更を全部ドキュメントにしようとしています。しかし、変更点のいくつかは、フレームワークの明確ではない部分で行われているため、一部の変更が実際にアプリケーションに影響を与えてしまう可能性があります。

### Symfony5要件

**影響の可能性： 高い**

Laravel7は裏で動作しているSymfonyコンポーネントを5.xへアップグレードしました。新しい最低限のコンパチブルバージョンもこれになりました。

### PHP 7.2.5要件

**影響の可能性： 低い**

新しいPHPの最低動作バージョンは7.2.5になりました。

<a name="updating-dependencies"></a>
### 依存パッケージのアップデート

`composer.json`ファイル中に指定されている以下のパッケージ依存を更新してください。

- `laravel/framework`を`^7.0`へ
- `nunomaduro/collision`を`^4.1`へ
- `phpunit/phpunit`を`^8.5`へ
- `laravel/tinker`を`^2.0`へ
- `facade/ignition`を`^2.0`へ

以下のファーストパーティ製パッケージは、Laravel7に対応する新しいメジャーリリースを出しました。どれか使用しているのであれば、それぞれのアップグレードガイドをLaravel7へのアップグレードの前にお読みください。

- [Browser Kit Testing v6.0](https://github.com/laravel/browser-kit-testing/blob/master/UPGRADE.md)
- [Envoy v2.0](https://github.com/laravel/envoy/blob/master/UPGRADE.md)
- [Horizon v4.0](https://github.com/laravel/horizon/blob/master/UPGRADE.md)
- [Nova v3.0](https://nova.laravel.com/releases)
- [Scout v8.0](https://github.com/laravel/scout/blob/master/UPGRADE.md)
- [Telescope v3.0](https://github.com/laravel/telescope/releases)
- [Tinker v2.0](https://github.com/laravel/tinker/blob/2.x/CHANGELOG.md)
- UI v2.0 (No changes necessary)

最後に、皆さんのアプリケーションで使用する他のサードパーティ製パッケージを調べ、Laravel7に対応するバージョンを使用していることを確認してください。

<a name="symfony-5-related-upgrades"></a>
### Symfony5へのアップグレード

**影響の可能性： 高い**

Laravel7は、Symfonyコンポーネントの5.xシリーズを活用しています。このアップグレードを含め、いくつか細かいアプリケーションの修正が必要となります。

最初に、アプリケーションの`App\Exceptions\Handler`クラスの`report`と`render`、`shouldReport`、`renderForConsole`メソッドは、`Exception`インスタンスに代え`Throwable`インターフェイスのインスタンスを受け取る必要があります。

    use Throwable;

    public function report(Throwable $exception);
    public function shouldReport(Throwable $exception);
    public function render($request, Throwable $exception);
    public function renderForConsole($output, Throwable $exception);

次に、`session`設定ファイルの`secure`オプションをフォールバック値の`null`へ変更してください。

    'secure' => env('SESSION_SECURE_COOKIE', null),

Artisanの裏で動作しているSymfony Consoleはすべてのコマンドが整数値を返すことを期待しています。そのため値を返すすべてのコマンドは確実に整数を返してください。

    public function handle()
    {
        // 以前
        return true;

        // 以後
        return 0;
    }

### 認証

<a name="authentication-scaffolding"></a>
#### スカフォールド

**影響の可能性： 高い**

すべての認証スカフォールドは`laravel/ui`リポジトリへ移動済みです。Laravelの認証スカフォールドを使用する場合は、このパッケージの`^2.0`リリースをインストールしてください。すべての環境でこのパッケージをインストールしてください。アプリケーションの`composer.json`ファイル中で以前にこのパッケージを`require-dev`で依存指定していた時は、`require`に移動する必要があります。

    composer require laravel/ui "^2.0"

#### `TokenRepositoryInterface`

**影響の可能性： 低い**

`Illuminate\Auth\Passwords\TokenRepositoryInterface`インターフェイスへ`recentlyCreatedToken`メソッドを追加しました。このインターフェイスのカスタム実装を書いているときは、このメソッドを追加実装してください。

### Blade

<a name="the-blade-component-method"></a>
#### `component`メソッド

**影響の可能性： 中程度**

`Blade::component`メソッドは`Blade::aliasComponent`へ名前を変えました。このメソッドの呼び出しを対応するように変更してください。

<a name="blade-components-and-blade-x"></a>
#### Bladeコンポーネントと"Blade x"

**影響の可能性： 中程度**

Laravel7はBlade「タグコンポーネント」のファーストパーティサポートを含んでいます。Bladeの組み込みタグコンポーネント機能を無効にしたい場合は、`AppServiceProvider`の`boot`メソッドで`withoutComponentTags`メソッドを呼び出してください。

    use Illuminate\Support\Facades\Blade;

    Blade::withoutComponentTags();

### Eloquent

#### `addHidden`／`addVisible`メソッド

**影響の可能性： 低い**

ドキュメントに乗せなくなっていた`addHidden`と`addVisible`メソッドは削除しました。代わりに`makeHidden`と`makeVisible`メソッドを使用してください。

#### `booting`／`booted`メソッド

**影響の可能性： 低い**

モデルの「起動(boot)」処理中で実行すべきロジックを便利に定義できる箇所を提供するため、`booting`と`booted`メソッドがEloquentへ追加されました。この名前のモデルメソッドをすでに使っている場合は、今回追加した新しいメソッドと衝突するため、リネームが必要になります。

<a name="date-serialization"></a>
#### データシリアライズ

**影響の可能性： 高い**

Laravel7はEloquentモデル上で`toArray`か`toJson`メソッド使用時に、新しい日付シリアライズ形式を使用しています。シリアライズの日付形式にするため、Carbonの`toJson`メソッドを使用するようになりました。これはタイムゾーンと秒の小数部を含んだISO-8601準拠の日付です。さらにこの変更により、クライアントサイドの日付パースライブラリの統合とより良いサポートが実現できました。

以前のシリアライズされた日付形式は`2019-12-02 20:01:00`でした。新しいシリアライズ済み日付形式は`2019-12-02T20:01:00.283041Z`のようになります。ISO-8601日付は常にUTCで表現されることに注意してください。

以前の振る舞いのままが好ましい場合は、モデルの`serializeDate`メソッドをオーバーライドしてください。

    use DateTimeInterface;

    /**
     * 配列／JSONシリアライズのためデータを準備する
     *
     * @param  \DateTimeInterface  $date
     * @return string
     */
    protected function serializeDate(DateTimeInterface $date)
    {
        return $date->format('Y-m-d H:i:s');
    }

> {tip} この変更はモデルとモデルコレクションの、配列とJSONへのシリアライズにだけ影響を及ぼします。この変更はデータベースに日付を保存する方法にはまったく影響しません。

<a name="factory-types"></a>
#### Factoryタイプ

**影響の可能性： 中程度**

Laravel7では「ファクトリタイプ」機能を削除しました。この機能は２０１６年１０月からドキュメントに記載していません。まだこの機能を利用している場合はより柔軟性が高い[ファクトリステート](/docs/{{version}}/database-testing#factory-states)へアップグレードしてください。

#### `getOriginal`メソッド

**影響の可能性： 低い**

`$model->getOriginal()`メソッドはモデルで定義されるキャストとミューテタを反映するようにしました。以前のこのメソッドはキャストせず、元の値をそのまま返していました。元のままのキャストしない値を続けて取得したい場合は、代わりに`getRawOriginal`メソッドを使ってください。

#### ルート結合

**影響の可能性： 低い**

`Illuminate\Contracts\Routing\UrlRoutable`インターフェイスの`resolveRouteBinding`メソッドは、`$field`引数を取るようになりました。このインターフェイスを実装している場合は変更してください。

さらに`Illuminate\Database\Eloquent\Model`クラスの`resolveRouteBinding`メソッドも`$field`引数を取るようになりました。このメソッドをオーバーライドしている場合は、この引数を受け取るように変更する必要があります。

最後に、`Illuminate\Http\Resources\DelegatesToResources`トレイトの`resolveRouteBinding`メソッドでも`$field`引数を取るようになりました。このメソッドをオーバーライドしている場合は、この引数を受け取るように変更する必要があります。

### HTTP

#### PSR-7コンパチビリティ

**影響の可能性： 低い**

PSR-7レスポンスを生成するためのZend Diactorosライブラリが非推奨となりました。このパッケージをPSR-7互換性のために使用している場合は、代わりに`nyholm/psr7` Composerパッケージをインストールしてください。さらに、`symfony/psr-http-message-bridge` Composerパッケージの`^2.0`リリースもインストールしてください。

### メール

#### 設定ファイルの変更

**影響の可能性： 状況による**

複数のメイラーをサポートするために、Laravel7.xではデフォルトの`mail`設定ファイルを変更し、`mailers`の配列を含むようにしました。しかしながら、後方互換性のためにこのファイルのLaravel6.x形式もまだサポートしています。7.xへアップグレードするには変更は**必要**ありませんが、[新しい`mail`設定ファイルを確認し](https://github.com/laravel/laravel/blob/develop/config/mail.php)、変更を反映するほうを皆さん選ばれるでしょう。

<a name="markdown-mail-template-updates"></a>
#### Markdownメールテンプレートアップデート

**影響の可能性： 中程度**

デフォルトMarkdownテンプレートが、よりプロフェッショナルな見かけの良いデザインに刷新されました。さらに、ドキュメントに記載されていない`promotion` Markdownメールコンポーネントが削除されました。

Markdownではインデントが特別な意味を持っているため、MarkdownメールではHTMLがインデントされていないことが期待されています。Laravelのデフォルトメールテンプレートを以前にリソース公開していた場合は、メールテンプレートを再リソース公開するか、自分でアンインデントしてください。

    php artisan vendor:publish --tag=laravel-mail --force

#### Swift Mailer Bindings

**影響の可能性： 低い**

Laravel7.xでは、`swift.mailer`と`swift.transport`のコンテナ結合は提供していません。これらのオブジェクトは、`mailer`の結合を経由してアクセスできます。

    $swiftMailer = app('mailer')->getSwiftMailer();

    $swiftTransport = $swiftMailer->getTransport();

### リソース

#### `Illuminate\Http\Resources\Json\Resource`クラス

**影響の可能性： 低い**

非推奨だった`Illuminate\Http\Resources\Json\Resource`クラスは削除されました。代わりにリソースは`Illuminate\Http\Resources\Json\JsonResource`を拡張してください。

### ルート

#### ルータの`getRoutes`メソッド

**影響の可能性： 低い**

ルータの`getRoutes`メソッドは`Illuminate\Routing\RouteCollection`のインスタンスの代わりに、`Illuminate\Routing\RouteCollectionInterface`のインスタンスを返すようになりました。

<a name="unique-route-names"></a>
#### 一意のルート名

**影響の可能性： 中程度**

公式にはドキュメントに載せていませんが、以前のLaravelリリースでは異なった２つのルートを同じ名前で定義できていました。Laravel7ではこれはできなくなり、いつでもルートに一意の名前を指定する必要があります。重複する名前を持つルートはフレームワークの複数のエリアで、予期せぬ振る舞いを引き起こす可能性があります。

<a name="cors-support"></a>
#### CORSサポート

**影響の可能性： 中程度**

Cross-Origin Resource Sharing (CORS)がデフォルトで統合されました。もしサードパーティ製ライブラリを使用していた場合は、[新しい`cors`設定ファイル](https://github.com/laravel/laravel/blob/master/config/cors.php)を使用することを勧めます。

次に、アプリケーションの依存パッケージへ、Laravelの後ろで動作するCORSライブラリをインストールします。

    composer require fruitcake/laravel-cors

最後に、`App\Http\Kernel`グローバルミドルウェアへ`\Fruitcake\Cors\HandleCors::class`ミドルウェアを追加します。

### セッション

#### `array`セッションドライバ

**影響の可能性： 低い**

`array`セッションドライバのデータは、現在のリクエスト中維持するようになりました。以前は、`array`セッションに保存したデータは、現在のリクエスト中でさえ取得できませんでした。

### テスト

<a name="assert-see"></a>
#### `assertSee`アサーション

**影響の可能性： 中程度**

`TestResponse`クラスの`assertSee`、`assertDontSee`、`assertSeeText`、`assertDontSeeText`、`assertSeeInOrder`、`assertSeeTextInOrder`アサーションは、自動的に値をエスケープするようになりました。これらのアサーションに渡す値を今まで自分でエスケープしていたのであれば、これからはそうしてはいけません。エスケープしない値でアサートする必要がある場合は、メソッドの第２引数に`false`を指定してください。

<a name="test-response"></a>
#### `TestResponse`クラス

**影響の可能性： 低い**

`Illuminate\Foundation\Testing\TestResponse`クラスは`Illuminate\Testing\TestResponse`へリネームされました。もしこのクラスを拡張していた場合は、名前空間を確実に変更してください。

<a name="assert-class"></a>
#### The `Assert` Class

**影響の可能性： 低い**

`Illuminate\Foundation\Testing\Assert`クラスは`Illuminate\Testing\Assert`へリネームされました。このクラスを使用していた場合は、名前空間部分を変更してください。

### バリデーション

<a name="the-different-rule"></a>
#### `different`ルール

**影響の可能性： 中程度**

`different`ルールは、リクエストに指定したパラメータのうちの１つが足りないければ失敗するようになりました。

<a name="miscellaneous"></a>
### その他

`laravel/laravel`の[GitHubリポジトリ](https://github.com/laravel/laravel)で、変更を確認することを推奨します。これらの変更は必須でありませんが、皆さんのアプリケーションではファイルの同期を保つほうが良いでしょう。変更のいくつかは、このアップグレードガイドで取り扱っていますが、設定ファイルやコメントなどの変更は取り扱っていません。変更は簡単に[GitHubの比較ツール](https://github.com/laravel/laravel/compare/6.x...master)で閲覧でき、みなさんにとって重要な変更を選択できます。
