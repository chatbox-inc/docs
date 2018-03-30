# Cache

- [設定]（＃設定）
     - [ドライバの前提条件]（＃ドライバの前提条件）
- [キャッシュ使用量]（＃cache-usage）
     - [キャッシュインスタンスの取得]（＃取得-a-cache-instance）
     - [キャッシュからアイテムを取得する]（＃キャッシュからアイテムを取得する）
     - [キャッシュ内のアイテムの格納]（＃ストア中のアイテムのキャッシュ）
     - [キャッシュからアイテムを削除する]（キャッシュから項目を削除する＃）
     - [The Cache Helper]（＃the-cache-helper）
- [キャッシュタグ]（＃cache-tags）
     - [タグ付きキャッシュ項目の格納]（＃格納タグ付きキャッシュ項目）
     - [タグ付きキャッシュ項目へのアクセス]（＃アクセスタグ付きキャッシュ項目）
     - [タグ付きキャッシュ項目の削除]（＃remove-tagged-cache-items）
- [カスタムキャッシュドライバの追加]（＃adding-custom-cache-drivers）
     - [ドライバを書く]（＃書き込みドライバ）
     - [ドライバの登録]（＃登録ドライバ）
- [Events]（＃events）

<a name="configuration"></a>
## 構成

Laravelは、様々なキャッシングバックエンドのための表現力豊かな統一APIを提供します。 キャッシュ設定は `config / cache.php`にあります。 このファイルでは、アプリケーション全体でデフォルトで使用するキャッシュドライバを指定できます。 Laravelは、[Memcached]（https://memcached.org）や[Redis]（https://redis.io）のような一般的なキャッシュバックエンドをサポートしています。

キャッシュ構成ファイルには、ファイル内に文書化されている他のさまざまなオプションも含まれているので、これらのオプションを必ず確認してください。 デフォルトでは、Laravelは `file`キャッシュドライバを使用するように設定されています。このドライバは、シリアライズされたキャッシュオブジェクトをファイルシステムに格納します。 大規模なアプリケーションの場合は、MemcachedやRedisなどのより堅牢なドライバを使用することをお勧めします。 同じドライバに対して複数のキャッシュ構成を構成することもできます。

<a name="driver-prerequisites"></a>
### ドライバの前提条件

#### データベース

`database`キャッシュドライバを使用する場合は、キャッシュ項目を格納するテーブルを設定する必要があります。 以下の表の `Schema`宣言の例があります：

    Schema::create('cache', function ($table) {
        $table->string('key')->unique();
        $table->text('value');
        $table->integer('expiration');
    });

> {tip} `php artisan cache：table` Artisanコマンドを使って、適切なスキーマで移行を生成することもできます。

#### Memcached

Memcachedドライバを使用するには、[Memcached PECL package]（https://pecl.php.net/package/memcached）がインストールされている必要があります。 `config / cache.php`設定ファイルにMemcachedサーバをすべてリストすることができます：

    'memcached' => [
        [
            'host' => '127.0.0.1',
            'port' => 11211,
            'weight' => 100
        ],
    ],

host`オプションをUNIXソケットパスに設定することもできます。これを行うと、 `port`オプションは` 0`に設定されます：

    'memcached' => [
        [
            'host' => '/var/run/memcached/memcached.sock',
            'port' => 0,
            'weight' => 100
        ],
    ],

#### レディス

LaravelでRedisキャッシュを使う前に、Composer経由で `predis / predis`パッケージ（〜1.0）をインストールするか、PECL経由でPhpRedis PHP拡張機能をインストールする必要があります。

Redisの設定の詳細については、[Laravel documentation page]（/ docs / {{version}} / redis＃configuration）を参照してください。

<a name="cache-usage"></a>
## キャッシュの使用法

<a name="obtaining-a-cache-instance"></a>
### キャッシュインスタンスの取得

`Illuminate \ Contracts \ Cache \ Factory`と` Illuminate \ Contracts \ Cache \ Repository` [契約]（/ docs / {{version}} /コントラクト）はLaravelのキャッシュサービスへのアクセスを提供します。 `Factory`契約はあなたのアプリケーション用に定義されたすべてのキャッシュドライバへのアクセスを提供します。 `Repository`契約は通常、あなたのアプリケーション用のデフォルトキャッシュドライバの実装で、` cache`設定ファイルで指定されています。

しかし、このドキュメント全体で使用する `Cache`ファサードを使用することもできます。 `Cache`ファサードは、Laravelのキャッシュ規約の基礎となる実装への便利で簡潔なアクセスを提供します：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * Show a list of all users of the application.
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');

            //
        }
    }

#### 複数のキャッシュストアへのアクセス

`Cache`ファサードを使って、` store`メソッドを使って様々なキャッシュストアにアクセスすることができます。 `store`メソッドに渡されるキーは、` cache`コンフィギュレーションファイルの `stores`コンフィギュレーション配列にリストアップされているストアの1つに対応する必要があります：

    $value = Cache::store('file')->get('foo');

    Cache::store('redis')->put('bar', 'baz', 10);

<a name="retrieving-items-from-the-cache"></a>
### キャッシュからアイテムを取得する

`Cache`ファサードの` get`メソッドは、キャッシュからアイテムを取り出すために使用されます。 アイテムがキャッシュに存在しない場合、 `null`が返されます。 もしあなたが望むなら、項目が存在しない場合に返されるデフォルト値を指定する第2引数を `get`メソッドに渡すことができます：

    $value = Cache::get('key');

    $value = Cache::get('key', 'default');

あなたはデフォルト値として `Closure`を渡すことさえできます。 指定された項目がキャッシュに存在しない場合、 `Closure`の結果が返されます。 クロージャを渡すと、データベースや他の外部サービスからのデフォルト値の取得を延期できます。

    $value = Cache::get('key', function () {
        return DB::table(...)->get();
    });

#### アイテム存在の確認

`has`メソッドは、アイテムがキャッシュ内に存在するかどうかを判断するために使用されます。 このメソッドは、値が `null`または` false`の場合は `false`を返します。

    if (Cache::has('key')) {
        //
    }

#### インクリメント/デクリメント値

`increment`と` decrement`メソッドは、キャッシュ内の整数項目の値を調整するために使用されます。 これらのメソッドはどちらも、アイテムの値をインクリメントまたはデクリメントする量を示すオプションの第2引数を受け取ります。

    Cache::increment('key');
    Cache::increment('key', $amount);
    Cache::decrement('key');
    Cache::decrement('key', $amount);

#### 検索とストア

場合によっては、キャッシュからアイテムを取得することもできますが、要求されたアイテムが存在しない場合はデフォルト値を保存することもできます。 たとえば、すべてのユーザーをキャッシュから取得したい場合や、存在しない場合はデータベースから取得してキャッシュに追加することができます。 これは `Cache :: remember`メソッドを使って行うことができます：

    $value = Cache::remember('users', $minutes, function () {
        return DB::table('users')->get();
    });

アイテムがキャッシュに存在しない場合、 `remember`メソッドに渡された` Closure`が実行され、その結果がキャッシュに置かれます。

`rememberForever`メソッドを使って、キャッシュからアイテムを取り出したり、永遠にそれを保存することができます：

    $value = Cache::rememberForever('users', function() {
        return DB::table('users')->get();
    });

#### 検索と削除

キャッシュからアイテムを取得してからアイテムを削除する必要がある場合は、 `pull`メソッドを使用することができます。 `get`メソッドと同様に、アイテムがキャッシュに存在しない場合は` null`が返されます：

    $value = Cache::pull('key');

<a name="storing-items-in-the-cache"></a>
### キャッシュ内にアイテムを格納する

`Cache`ファサードで` put`メソッドを使ってアイテムをキャッシュに保存することができます。アイテムをキャッシュに配置するときには、値をキャッシュする時間を分単位で指定する必要があります。

    Cache::put('key', 'value', $minutes);

分数を整数で渡す代わりに、分数を整数として渡す代りに、キャッシングされた項目の有効期限を表する `DateTime`を渡すこともできます：

    $expiresAt = now()->addMinutes(10);

    Cache::put('key', 'value', $expiresAt);

#### 存在しない場合はストア

`add`メソッドは、アイテムがキャッシュストアにまだ存在しない場合にのみ、そのアイテムをキャッシュに追加します。 アイテムが実際にキャッシュに追加された場合、このメソッドは `true`を返します。 それ以外の場合、メソッドは `false`を返します。

    Cache::add('key', 'value', $minutes);

#### アイテムを永遠に保管する

永久にキャッシュに項目を格納するために `forever`メソッドを使用することができます。 これらの項目は期限切れにならないので、 `forget`メソッドを使ってキャッシュから手動で削除する必要があります：

    Cache::forever('key', 'value');

> {tip} Memcachedドライバを使用している場合、キャッシュがサイズ制限に達すると「永遠に」保存されているアイテムが削除されることがあります。

<a name="removing-items-from-the-cache"></a>
### キャッシュからアイテムを削除する

あなたは `forget`メソッドを使ってキャッシュから項目を削除することができます：

    Cache::forget('key');

`flush`メソッドを使ってキャッシュ全体をクリアすることができます：

    Cache::flush();

> {note}キャッシュをフラッシュすると、キャッシュのプレフィックスは無視され、すべてのエントリがキャッシュから削除されます。 他のアプリケーションで共有されているキャッシュをクリアするときは、これを注意深く検討してください。

<a name="the-cache-helper"></a>
### キャッシュヘルパー

`Cache`ファサードや[cache contract]（/ docs / {{version}} / contracts）を使うことに加えて、グローバルな` cache`関数を使ってキャッシュを介してデータを取り出して保存することもできます。 `cache`関数が単一の文字列引数で呼び出されると、与えられたキーの値を返します：

    $value = cache('key');

関数にキーと値のペアと有効期限の配列を指定すると、指定した期間のキャッシュに値が格納されます。

    cache(['key' => 'value'], $minutes);

    cache(['key' => 'value'], now()->addSeconds(10));

> {tip}グローバルな `cache`関数の呼び出しをテストするときは、あなたが[ファサードをテストする]（/ docs / {{version}} / mocking＃mocking- ファサード）。

<a name="cache-tags"></a>
## キャッシュタグ

> {note}キャッシュタグは、 `file`または` database`キャッシュドライバを使用しているときはサポートされていません。 さらに、「永遠に」保存されたキャッシュで複数のタグを使用する場合、古いキャッシュを自動的に消去する `memcached`などのドライバでパフォーマンスが最適になります。

<a name="storing-tagged-cache-items"></a>
### タグ付きキャッシュ項目の格納

キャッシュタグを使用すると、キャッシュ内の関連項目にタグを付けてから、指定されたタグが割り当てられたすべてのキャッシュされた値をフラッシュできます。 タグ付きキャッシュにアクセスするには、順序付けられたタグ名の配列を渡します。 例えば、タグ付きキャッシュとキャッシュ内の `put`値にアクセスしましょう：

    Cache::tags(['people', 'artists'])->put('John', $john, $minutes);

    Cache::tags(['people', 'authors'])->put('Anne', $anne, $minutes);

<a name="accessing-tagged-cache-items"></a>
### タグ付きキャッシュ項目へのアクセス

タグ付きキャッシュ項目を取得するには、タグの同じ順序付きリストを `tags`メソッドに渡し、次に取得するキーで` get`メソッドを呼び出します。

    $john = Cache::tags(['people', 'artists'])->get('John');

    $anne = Cache::tags(['people', 'authors'])->get('Anne');

<a name="removing-tagged-cache-items"></a>
### タグ付きキャッシュ項目の削除

タグまたはタグのリストが割り当てられているすべてのアイテムをフラッシュすることができます。 例えば、この文は、 `people`、` authors`、またはその両方でタグ付けされたすべてのキャッシュを削除します。 したがって、AnneとJohnの両方がキャッシュから削除されます。

    Cache::tags(['people', 'authors'])->flush();

対照的に、この文は `authors`でタグ付けされたキャッシュのみを削除するので、` Anne`は削除されますが、 `John`では削除されません。

    Cache::tags('authors')->flush();

<a name="adding-custom-cache-drivers"></a>
## カスタムキャッシュドライバの追加

<a name="writing-the-driver"></a>
### ドライバの作成

カスタムキャッシュドライバを作成するには、最初に `Illuminate¥Contracts¥Cache¥Store` [contract]（/ docs / {{version}} /コントラクト）を実装する必要があります。 ですから、MongoDBのキャッシュ実装は次のようになります：

    <?php

    namespace App\Extensions;

    use Illuminate\Contracts\Cache\Store;

    class MongoStore implements Store
    {
        public function get($key) {}
        public function many(array $keys);
        public function put($key, $value, $minutes) {}
        public function putMany(array $values, $minutes);
        public function increment($key, $value = 1) {}
        public function decrement($key, $value = 1) {}
        public function forever($key, $value) {}
        public function forget($key) {}
        public function flush() {}
        public function getPrefix() {}
    }

これらのメソッドのそれぞれをMongoDB接続を使って実装するだけです。 これらの各メソッドを実装する方法の例については、フレームワークのソースコードで `Illuminate \ Cache \ MemcachedStore`を見てください。 実装が完了したら、カスタムドライバ登録を完了できます。

    Cache::extend('mongo', function ($app) {
        return Cache::repository(new MongoStore);
    });

> {tip}カスタムキャッシュドライバのコードをどこに置くのか不思議なら、 `app`ディレクトリ内に` Extensions`名前空間を作ることができます。 ただし、Laravelには厳格なアプリケーション構造がなく、好みに応じてアプリケーションを自由に整理することができます。

<a name="registering-the-driver"></a>
### ドライバの登録

カスタムキャッシュドライバをLaravelに登録するには、 `Cache`ファサードで` extend`メソッドを使います。 `Cache :: extend`の呼び出しは、新しいLaravelアプリケーションに付属しているデフォルトの` App \ Providers \ AppServiceProvider`の `boot`メソッドで行うことができます。あるいは、独自のサービスプロバイダを作成して拡張機能を格納することもできます プロバイダを `config / app.php`プロバイダ配列に登録することを忘れないでください：

    <?php

    namespace App\Providers;

    use App\Extensions\MongoStore;
    use Illuminate\Support\Facades\Cache;
    use Illuminate\Support\ServiceProvider;

    class CacheServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Cache::extend('mongo', function ($app) {
                return Cache::repository(new MongoStore);
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

`extend`メソッドに渡される最初の引数は、ドライバの名前です。 これは `config / cache.php`設定ファイルの` driver`オプションに対応します。 2番目の引数はClosureで、Illuminate \ Cache \ Repositoryインスタンスを返す必要があります。 Closureは `$ app`インスタンスを渡します。これは[サービスコンテナ]（/ docs / {{version}} / container）のインスタンスです。

拡張機能が登録されたら、 `config / cache.php`設定ファイルの` driver`オプションをあなたの拡張機能の名前に更新してください。

<a name="events"></a>
## イベント

すべてのキャッシュ操作でコードを実行するには、キャッシュによって起動された[events]（/ docs / {{version}} / events）を待ち受けることができます。 通常、これらのイベントリスナを `EventServiceProvider`内に配置する必要があります：

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Cache\Events\CacheHit' => [
            'App\Listeners\LogCacheHit',
        ],

        'Illuminate\Cache\Events\CacheMissed' => [
            'App\Listeners\LogCacheMissed',
        ],

        'Illuminate\Cache\Events\KeyForgotten' => [
            'App\Listeners\LogKeyForgotten',
        ],

        'Illuminate\Cache\Events\KeyWritten' => [
            'App\Listeners\LogKeyWritten',
        ],
    ];
