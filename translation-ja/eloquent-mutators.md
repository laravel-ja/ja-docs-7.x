# Eloquent：ミューテタ

- [イントロダクション](#introduction)
- [アクセサとミューテタ](#accessors-and-mutators)
    - [アクセサの定義](#defining-an-accessor)
    - [ミューテタの定義](#defining-a-mutator)
- [日付ミューテタ](#date-mutators)
- [属性キャスト](#attribute-casting)
    - [Custom Casts](#custom-casts)
    - [配列とJSONのキャスト](#array-and-json-casting)
    - [日付のキャスト](#date-casting)
    - [Query Time Casting](#query-time-casting)

<a name="introduction"></a>
## イントロダクション

アクセサとミューテタはモデルの取得や値を設定するときに、Eloquent属性のフォーマットを可能にします。たとえば[Laravelの暗号化](/docs/{{version}}/encryption)を使いデータベース保存時に値を暗号化し、Eloquentモデルでアクセスする時には自動的にその属性を復元するように設定できます。

カスタムのアクセサやミューテタに加え、Eloquentは日付フールドを自動的に[Carbon](https://github.com/briannesbitt/Carbon)インスタンスにキャストしますし、[テキストフィールドをJSONにキャスト](#attribute-casting)することもできます。

<a name="accessors-and-mutators"></a>
## アクセサとミューテタ

<a name="defining-an-accessor"></a>
### アクセサの定義

アクセサを定義するには、アクセスしたいカラム名が「studlyケース（Upper Camel Case）」で`Foo`の場合、`getFooAttribute`メソッドをモデルに作成します。以下の例では、`first_name`属性のアクセサを定義しています。`first_name`属性の値にアクセスが起きると、Eloquentは自動的にこのアクセサを呼び出します。

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * ユーザーのファーストネームを取得
         *
         * @param  string  $value
         * @return string
         */
        public function getFirstNameAttribute($value)
        {
            return ucfirst($value);
        }
    }

ご覧の通り、アクセサにはそのカラムのオリジナルの値が渡されますので、それを加工し値を返します。アクセサの値にアクセスするには、モデルインスタンスの`first_name`属性へアクセスしてください。

    $user = App\User::find(1);

    $firstName = $user->first_name;

既存の属性を元に算出した、新しい値をアクセサを使用し返すことも可能です。

    /**
     * ユーザーのフルネーム取得
     *
     * @return string
     */
    public function getFullNameAttribute()
    {
        return "{$this->first_name} {$this->last_name}";
    }

> {tip} これらの計算済みの値をモデルのarray／JSON表現に追加したい場合は、プロパティに[追加する必要があります](/docs/{{version}}/eloquent-serialization#appending-values-to-json)。

<a name="defining-a-mutator"></a>
### ミューテタの定義

ミューテタを定義するにはアクセスしたいカラム名が`Foo`の場合、モデルに「ローワーキャメルケース」で`setFooAttribute`メソッドを作成します。今回も`first_name`属性を取り上げ、ミューテタを定義しましょう。このミューテタはモデルの`first_name`属性へ値を設定する時に自動的に呼びだされます。

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * ユーザーのファーストネームを設定
         *
         * @param  string  $value
         * @return void
         */
        public function setFirstNameAttribute($value)
        {
            $this->attributes['first_name'] = strtolower($value);
        }
    }

ミューテタは属性に設定しようとしている値を受け取りますのでこれを加工し、Eloquentモデルの`$attributes`内部プロパティへ加工済みの値を設定します。では`Sally`を`first_name`属性へ設定してみましょう。

    $user = App\User::find(1);

    $user->first_name = 'Sally';

上記の場合、`setFirstNameAttribute`メソッドが呼び出され、`Sally`の値が渡されます。このミューテタはそれから名前に`strtolower`を適用し、その値を`$attributes`内部配列へ設定します。

<a name="date-mutators"></a>
## 日付ミューテタ

デフォルトでEloquentは`created_at`と`updated_at`カラムを[Carbon](https://github.com/briannesbitt/Carbon)インスタンスへ変換します。CarbonはPHPネイティブの`DateTime`クラスを拡張しており、便利なメソッドを色々と提供しています。モデルの`$dates`プロパティをセットすることにより、データ属性を追加できます。

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 日付を変形する属性
         *
         * @var array
         */
        protected $dates = [
            'seen_at',
        ];
    }

> {tip} モデルの`$timestamps`プロパティを`false`へセットすることにより、デフォルトの`created_at`と`updated_at`タイムスタンプを無効にできます。

日付だと推定されるカラムで、値はUnixタイムスタンプ、日付文字列(`Y-m-d`)、日付時間文字列、`DateTime`や`Carbon`インスタンスを値としてセットできます。日付の値は自動的に正しく変換され、データベースへ保存されます。

    $user = App\User::find(1);

    $user->deleted_at = now();

    $user->save();

前記の通り`$dates`プロパティにリストした属性を取得する場合、自動的に[Carbon](https://github.com/briannesbitt/Carbon)インスタンスへキャストされますので、その属性でCarbonのメソッドがどれでも使用できます。

    $user = App\User::find(1);

    return $user->deleted_at->getTimestamp();

#### Dateフォーマット

デフォルトのタイムスタンプフォーマットは`'Y-m-d H:i:s'`です。タイムスタンプフォーマットをカスタマイズする必要があるなら、モデルの`$dateFormat`プロパティを設定してください。このプロパティは日付属性がデータベースにどのように保存されるかと同時に、モデルが配列やJSONにシリアライズされる時のフォーマットを決定します。

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * モデルの日付カラムの保存形式
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

<a name="attribute-casting"></a>
## 属性キャスト

モデルの`$casts`プロパティは属性を一般的なデータタイプへキャストする便利な手法を提供します。`$casts`プロパティは配列で、キーにはキャストする属性名を指定し、値にはそのカラムに対してキャストしたいタイプを指定します。サポートしているキャストタイプは`integer`、`real`、`float`、`double`、`decimal:<桁数>`、`string`、`boolean`、`object`、`array`、`collection`、`date`、`datetime`、`timestamp`です。`decimal`へキャストする場合は、桁数を`decimal:2`のように定義してください。

属性キャストのデモンストレーションとして、データベースには整数の`0`と`1`で保存されている`is_admin`属性を論理値にキャストしてみましょう。

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be cast.
         *
         * @var array
         */
        protected $casts = [
            'is_admin' => 'boolean',
        ];
    }

これでデータベースには整数で保存されていても`is_admin`属性にアクセスすれば、いつでも論理値にキャストされます。

    $user = App\User::find(1);

    if ($user->is_admin) {
        //
    }

<a name="custom-casts"></a>
### Custom Casts

Laravel has a variety of built-in, helpful cast types; however, you may occasionally need to define your own cast types. You may accomplish this by defining a class that implements the `CastsAttributes` interface.

Classes that implement this interface must define a `get` and `set` methods. The `get` method is responsible for transforming a raw value from the database into a cast value, while the `set` method should transform a cast value into a raw value that can be stored in the database. As an example, we will re-implement the built-in `json` cast type as a custom cast type:

    <?php

    namespace App\Casts;

    use Illuminate\Contracts\Database\Eloquent\CastsAttributes;

    class Json implements CastsAttributes
    {
        /**
         * Cast the given value.
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
         * Prepare the given value for storage.
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

Once you have defined a custom cast type, you may attach it to a model attribute using its class name:

    <?php

    namespace App;

    use App\Casts\Json;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be cast.
         *
         * @var array
         */
        protected $casts = [
            'options' => Json::class,
        ];
    }

#### Value Object Casting

You are not limited to casting values to primitive types. You may also cast values to objects. Defining custom casts that cast values to objects is very similar to casting to primitive types; however, the `set` method should return an array of key / value pairs that will be used to set raw, storable values on the model.

As an example, we will define a custom cast class that casts multiple model values into a single `Address` value object. We will assume the `Address` value has two public properties: `lineOne` and `lineTwo`:

    <?php

    namespace App\Casts;

    use App\Address;
    use Illuminate\Contracts\Database\Eloquent\CastsAttributes;

    class Address implements CastsAttributes
    {
        /**
         * Cast the given value.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  mixed  $value
         * @param  array  $attributes
         * @return \App\Address
         */
        public function get($model, $key, $value, $attributes)
        {
            return new Address(
                $attributes['address_line_one'],
                $attributes['address_line_two']
            );
        }

        /**
         * Prepare the given value for storage.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  \App\Address  $value
         * @param  array  $attributes
         * @return array
         */
        public function set($model, $key, $value, $attributes)
        {
            return [
                'address_line_one' => $value->lineOne,
                'address_line_two' => $value->lineTwo,
            ];
        }
    }

When casting to value objects, any changes made to the value object will automatically be synced back to the model before the model is saved:

    $user = App\User::find(1);

    $user->address->lineOne = 'Updated Address Value';

    $user->save();

#### Inbound Casting

Occasionally, you may need to write a custom cast that only transforms values that are being set on the model and does not perform any operations when attributes are being retrieved from the model. A classic example of an inbound only cast is a "hashing" cast. Inbound only custom casts should implement the `CastsInboundAttributes` interface, which only requires a `set` method to be defined.

    <?php

    namespace App\Casts;

    use Illuminate\Contracts\Database\Eloquent\CastsInboundAttributes;

    class Hash implements CastsInboundAttributes
    {
        /**
         * The hashing algorithm.
         *
         * @var string
         */
        protected $algorithm;

        /**
         * Create a new cast class instance.
         *
         * @param  string|null  $algorithm
         * @return void
         */
        public function __construct($algorithm = null)
        {
            $this->algorithm = $algorithm;
        }

        /**
         * Prepare the given value for storage.
         *
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @param  string  $key
         * @param  array  $value
         * @param  array  $attributes
         * @return string
         */
        public function set($model, $key, $value, $attributes)
        {
            return is_null($this->algorithm)
                        ? bcrypt($value);
                        : hash($this->algorithm, $value);
        }
    }

#### Cast Parameters

When attaching a custom cast to a model, cast parameters may be specified by separating them from the class name using a `:` character and comma-delimiting multiple parameters. The parameters will be passed to the constructor of the cast class:

    /**
     * The attributes that should be cast.
     *
     * @var array
     */
    protected $casts = [
        'secret' => Hash::class.':sha256',
    ];

<a name="array-and-json-casting"></a>
### 配列とJSONのキャスト

`array`キャストタイプは、シリアライズされたJSON形式で保存されているカラムを取り扱う場合とくに便利です。たとえば、データベースにシリアライズ済みのJSONを持つ`JSON`か`TEXT`フィールドがある場合です。その属性に`array`キャストを追加すれば、Eloquentモデルにアクセスされた時点で自動的に非シリアライズ化され、PHPの配列へとキャストされます。

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be cast.
         *
         * @var array
         */
        protected $casts = [
            'options' => 'array',
        ];
    }

キャストを定義後、`options`属性にアクセスすると自動的に非シリアライズされPHP配列になります。`options`属性へ値をセットすると配列は保存のために自動的にJSONへシリアライズされます。

    $user = App\User::find(1);

    $options = $user->options;

    $options['key'] = 'value';

    $user->options = $options;

    $user->save();

<a name="date-casting"></a>
### 日付のキャスト

`date`や`datetime`キャストタイプを使用する場合、日付のフォーマットを指定できます。このフォーマットは、[モデルを配列やJSONへシリアライズする](/docs/{{version}}/eloquent-serialization)場合に使用します。

    /**
     * The attributes that should be cast.
     *
     * @var array
     */
    protected $casts = [
        'created_at' => 'datetime:Y-m-d',
    ];

<a name="query-time-casting"></a>
### Query Time Casting

Sometimes you may need to apply casts while executing a query, such as when selecting a raw value from a table. For example, consider the following query:

    use App\Post;
    use App\User;

    $users = User::select([
        'users.*',
        'last_posted_at' => Post::selectRaw('MAX(created_at)')
                ->whereColumn('user_id', 'users.id')
    ])->get();

The `last_posted_at` attribute on the results of this query will be a raw string. It would be convenient if we could apply a `date` cast to this attribute when executing the query. To accomplish this, we may use the `withCasts` method:

    $users = User::select([
        'users.*',
        'last_posted_at' => Post::selectRaw('MAX(created_at)')
                ->whereColumn('user_id', 'users.id')
    ])->withCasts([
        'last_posted_at' => 'date'
    ])->get();
