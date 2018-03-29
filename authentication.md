＃認証

- [はじめに]（＃序論）
     - [データベースの検討事項]（＃introduction-database-considerations）
- [認証クイックスタート]（＃authentication-quickstart）
     - [ルーティング]（＃included-routing）
     - [Views]（＃included-views）
     - [認証中]（＃included-authenticating）
     - [認証されたユーザの取得]（＃認証されたユーザの取得）
     - [ルートの保護]（＃保護ルート）
     - [ログインスロットル]（＃login-throttling）
- [手動認証ユーザ]（＃認証ユーザ）
     - [ユーザーの記憶]（＃remembering-users）
     - [その他の認証方法]（＃other-authentication-methods）
- [HTTP基本認証]（＃http-basic-authentication）
     - [ステートレスHTTP基本認証]（＃stateless-http-basic-authentication）
- [社会認証]（https://github.com/laravel/socialite）
- [カスタムガードの追加]（＃追加 - カスタムガード）
- [カスタムユーザプロバイダの追加]（＃adding-custom-user-providers）
     - [利用者提供者契約]（＃利用者提供者契約）
     - [認証可能契約]（＃認証可能契約）
- [Events]（＃events）

<a name="introduction"></a>
## 前書き

> {tip} **早く開始したいですか？** freshmeat_linux / Linux]アプリケーションで `php artisan make：auth`と` php artisan migrate`を実行するだけです。次に、あなたのブラウザを `http：// your-app.dev / register`またはあなたのアプリケーションに割り当てられている他のURLにナビゲートしてください。これら2つのコマンドは、認証システム全体を足場で管理します。

Laravelは認証の実装を非常に簡単にします。実際には、ほぼすべてがあなたのために構成されています。認証設定ファイルは、 `config / auth.php`にあります。これには、認証サービスの振る舞いを微調整するためのいくつかの文書化されたオプションが含まれています。

ラーベールの認証施設は、その核心に「警備員」と「提供者」で構成されています。ガードは、各要求に対してユーザーがどのように認証されるかを定義します。たとえば、Laravelには、セッションストレージとCookieを使用して状態を維持する `session`ガードが付属しています。

プロバイダは、永続ストレージからユーザを取得する方法を定義します。 Laravelには、Eloquentとデータベースクエリビルダーを使用してユーザーを取得するためのサポートが付属しています。ただし、アプリケーションの必要に応じて、追加のプロバイダを自由に定義することができます。

このすべてが今混乱していると心配しないでください！多くのアプリケーションで、デフォルトの認証設定を変更する必要はありません。

<a name="introduction-database-considerations"></a>
### データベースに関する考慮事項

デフォルトでは、Laravelはあなたの `app`ディレクトリに` App \ User` [Eloquent model]（/ docs / {{version}} / eloquent）を含みます。 このモデルはデフォルトのEloquent認証ドライバーで使用できます。 アプリケーションがEloquentを使用していない場合は、Laravelクエリービルダーを使用する `database`認証ドライバを使用することができます。

`App \ User`モデル用のデータベーススキーマを構築するときは、パスワード列の長さが60文字以上であることを確認してください。 255文字のデフォルト文字列の長さを維持することは良い選択です。

また、 `users`（または同等の）テーブルに100文字のヌル入力可能な文字列` remember_token`が含まれていることを確認する必要があります。 この列は、アプリケーションにログインする際に「remember me」オプションを選択したユーザーのトークンを格納するために使用されます。

<a name="authentication-quickstart"></a>
## 認証クイックスタート

Laravelには、 `App \ Http \ Controllers \ Auth`名前空間にあるいくつかのあらかじめ構築された認証コントローラが付属しています。 `RegisterController`は新しいユーザ登録を処理し、` LoginController`は認証を処理し、 `ForgotPasswordController`はパスワードをリセットするための電子メールリンクを処理し、` ResetPasswordController`はパスワードをリセットするロジックを含みます。 これらのコントローラのそれぞれは、必要なメソッドを含めるために特性を使用します。 多くのアプリケーションでは、これらのコントローラをまったく変更する必要はありません。

<a name="included-routing"></a>
### ルーティング

Laravelは、簡単なコマンドを使用して認証に必要なすべてのルートとビューを素早く構築する方法を提供します。

    php artisan make:auth

このコマンドは、新しいアプリケーションで使用する必要があり、レイアウトビュー、登録およびログインビュー、およびすべての認証エンドポイントのルートをインストールします。 アプリケーションのダッシュボードへのログイン後のリクエストを処理するための `HomeController`も生成されます。

<a name="included-views"></a>
### ビュー

前のセクションで述べたように、 `php artisan make：auth`コマンドは認証に必要なすべてのビューを作成し、それらを` resources / views / auth`ディレクトリに置きます。

`make：auth`コマンドはアプリケーションの基本レイアウトを含む` resources / views / layouts`ディレクトリも作成します。 これらのビューはすべてBootstrap CSSフレームワークを使用しますが、自由にカスタマイズすることができます。

<a name="included-authenticating"></a>
### 認証する

付属の認証コントローラ用のルートとビューの設定が完了したので、アプリケーションの新規ユーザーを登録して認証する準備が整いました。 認証コントローラには、既存のユーザーを認証したり、データベースに新しいユーザーを格納したりするためのロジックが既に組み込まれているため、ブラウザでアプリケーションにアクセスできます。

#### パスのカスタマイズ

ユーザーが正常に認証されると、ユーザーは `/ home` URIにリダイレクトされます。 `LoginController`、` RegisterController`、 `ResetPasswordController`に` redirectTo`プロパティを定義することで認証後のリダイレクトの場所をカスタマイズすることができます：

    protected $redirectTo = '/';

次に、 `RedirectIfAuthenticated`ミドルウェアの` handle`メソッドを変更して、ユーザのリダイレクト時に新しいURIを使用する必要があります。

リダイレクトパスにカスタム生成ロジックが必要な場合は、 `redirectTo`プロパティの代わりに` redirectTo`メソッドを定義することができます：

    protected function redirectTo()
    {
        return '/path';
    }

> {tip} The `redirectTo` method will take precedence over the `redirectTo` attribute.

#### ユーザー名のカスタマイズ

By default, Laravel uses the `email` field for authentication. If you would like to customize this, you may define a `username` method on your `LoginController`:

    public function username()
    {
        return 'username';
    }

#### ガードのカスタマイズ

また、ユーザーの認証と登録に使用される「ガード」をカスタマイズすることもできます。 始めに、 `LoginController`、` RegisterController`、 `ResetPasswordController`に` guard`メソッドを定義してください。 このメソッドはガードインスタンスを返す必要があります：

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### 検証/記憶域のカスタマイズ

新しいユーザーがアプリケーションに登録するときに必要なフォームフィールドを変更するには、新しいユーザーをデータベースに格納する方法をカスタマイズするには、 `RegisterController`クラスを変更します。 このクラスは、アプリケーションの新しいユーザーの検証と作成を担当します。

`RegisterController`の` validator`メソッドは、アプリケーションの新しいユーザのためのバリデーションルールを含んでいます。 この方法は自由に変更できます。

`RegisterController`の` create`メソッドは、[Eloquent ORM]（/ docs / {{version}} / eloquent）を使ってあなたのデータベースに新しい `App \ User`レコードを作成する責任があります。 この方法は、データベースのニーズに応じて自由に変更できます。

<a name="retrieving-the-authenticated-user"></a>
### 認証されたユーザーの取得

認証されたユーザには `Auth`ファサードでアクセスすることができます：

    use Illuminate\Support\Facades\Auth;

    // Get the currently authenticated user...
    $user = Auth::user();

    // Get the currently authenticated user's ID...
    $id = Auth::id();

あるいは、ユーザーが認証されると、 `Illuminate \ Http \ Request`インスタンスを介して認証されたユーザーにアクセスすることができます。 タイプヒントクラスは自動的にコントローラメソッドに注入されることを覚えておいてください：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class ProfileController extends Controller
    {
        /**
         * Update the user's profile.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // $request->user() returns an instance of the authenticated user...
        }
    }

#### 現在のユーザーが認証されているかどうかを判断する

ユーザーが既にアプリケーションにログインしているかどうかを判断するには、 `Auth`ファサードで` check`メソッドを使用します。これは、ユーザーが認証されている場合に `true`を返します：

    use Illuminate\Support\Facades\Auth;

    if (Auth::check()) {
        // The user is logged in...
    }

> {tip}ユーザが `check`メソッドを使って認証されているかどうかを判断することは可能ですが、通常はミドルウェアを使って特定のルート/コントローラへのユーザアクセスを許可する前にユーザが認証されていることを確認します。 詳細については、[経路保護]（/ docs / {{version}} / authentication＃protection-routes）のドキュメントを参照してください。

<a name="protecting-routes"> </a>
### 経路を保護する

[Route middleware](/docs/{{version}}/middleware) can be used to only allow authenticated users to access a given route. Laravel ships with an `auth` middleware, which is defined at `Illuminate\Auth\Middleware\Authenticate`. Since this middleware is already registered in your HTTP kernel, all you need to do is attach the middleware to a route definition:

    Route::get('profile', function () {
        // Only authenticated users may enter...
    })->middleware('auth');

もちろん、あなたが[コントローラ]（/ docs / {{version}} / controllers）を使用している場合は、ミドルウェアメソッドをコントローラのコンストラクタから呼び出すことができます。

    public function __construct()
    {
        $this->middleware('auth');
    }

#### 認証されていないユーザーのリダイレクト

`auth`ミドルウェアが権限のないユーザを検出すると、JSONの` 401`レスポンスを返すか、リクエストがAJAXリクエストでなかった場合、ユーザを `login` [named route]（/ docs / { {バージョン}} /ルーティング＃名前付きルート）。

`app / Exceptions / Hander.php`ファイルに` unauthenticated`関数を定義することで、この動作を変更することができます：

    use Illuminate\Auth\AuthenticationException;

    protected function unauthenticated($request, AuthenticationException $exception)
    {
        return $request->expectsJson()
                    ? response()->json(['message' => $exception->getMessage()], 401)
                    : redirect()->guest(route('login'));
    }

#### Guardの指定

`auth`ミドルウェアをルートに接続するときに、ユーザの認証に使用するガードを指定することもできます。 指定されたガードは、 `auth.php`設定ファイルの` guard `配列のキーの1つに対応する必要があります：

    public function __construct()
    {
        $this->middleware('auth:api');
    }

<a name="login-throttling"></a>
### ログイン調整

Laravelの組み込みの `LoginController`クラスを使用している場合、` Illuminate \ Foundation \ Auth \ ThrottlesLogins`特性はすでにあなたのコントローラに含まれています。 デフォルトでは、ユーザーは数回の試行後に正しい資格情報を提示できない場合、1分間ログインすることはできません。 調整は、ユーザーのユーザー名/電子メールアドレスとそのIPアドレスに固有のものです。

<a name="authenticating-users"></a>
## 手動でユーザーを認証する

もちろん、Laravelに付属の認証コントローラを使用する必要はありません。 これらのコントローラを削除する場合は、Laravel認証クラスを直接使用してユーザー認証を管理する必要があります。 心配しないでください、それはシンチです！

`Auth` [facade]（/ docs / {{version}} / facades）を使ってLaravelの認証サービスにアクセスしますので、クラスの先頭に` Auth`ファサードをインポートする必要があります。 次に、 `試み`メソッドを調べてみましょう：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Auth;

    class LoginController extends Controller
    {
        /**
         * Handle an authentication attempt.
         *
         * @return Response
         */
        public function authenticate()
        {
            if (Auth::attempt(['email' => $email, 'password' => $password])) {
                // Authentication passed...
                return redirect()->intended('dashboard');
            }
        }
    }

`試み`メソッドは、最初の引数としてキー/値のペアの配列を受け入れます。配列の値は、データベーステーブル内のユーザーを見つけるために使用されます。したがって、上記の例では、ユーザは `email`カラムの値で検索されます。ユーザーが見つかった場合、データベースに格納されているハッシュされたパスワードは、配列を介してメソッドに渡された `password`値と比較されます。フレームワークは自動的に値をハッシュしてからデータベースのハッシュされたパスワードと比較するので、 `password`値として指定されたパスワードをハッシュしないでください。 2つのハッシュされたパスワードが一致すると、認証されたセッションがユーザーに対して開始されます。

認証が成功した場合、 `試行`メソッドは `true`を返します。それ以外の場合は、 `false`が返されます。

リダイレクタの `意図された`メソッドは、認証ミドルウェアによって傍受される前にアクセスしようとしていたURLにユーザをリダイレクトします。意図した宛先が利用できない場合に備えて、このメソッドにフォールバックURIを渡すことができます。

#### 追加条件の指定

必要に応じて、ユーザーの電子メールとパスワードに加えて、追加の条件を認証クエリに追加することもできます。 たとえば、ユーザーが「アクティブ」とマークされていることを確認することができます。

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // The user is active, not suspended, and exists.
    }

> {note}これらの例では、 `email`は必須オプションではなく、単なる例として使用されています。 データベース内の「ユーザー名」に対応する列名を使用する必要があります。

#### 特定のGuardインスタンスへのアクセス

`Auth`ファサードで` guard`メソッドを使って、利用したいガードインスタンスを指定することができます。 これにより、完全に別個の認証可能なモデルまたはユーザー・テーブルを使用して、アプリケーションの別々の部分に対する認証を管理することができます。

`guard`メソッドに渡されるガード名は` auth.php`設定ファイルで設定されているガードの一つに対応しなければなりません：

    if (Auth::guard('admin')->attempt($credentials)) {
        //
    }

#### ログアウト

アプリケーションからユーザをログアウトするには、 `Auth`ファサードで` logout`メソッドを使用します。これにより、ユーザーのセッション内の認証情報が消去されます。

    Auth::logout();

<a name="remembering-users"></a>
### ユーザーを思い出す

アプリケーションで「remember me」機能を提供したい場合は、 `試行`メソッドの第2引数としてブール値を渡すことができます。これは、ユーザーを無期限に認証したままにするか、手動でログアウトするまで続けます。もちろん、あなたの `users`テーブルには` remember_token`という文字列が含まれていなければなりません。これは "remember me"トークンを格納するために使われます。

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // The user is being remembered...
    }

> {tip} Laravelに同梱されている組み込みの `LoginController`を使用している場合、ユーザを「覚えている」ための適切なロジックは、すでにコントローラが使用している特性によって実装されています。

ユーザーを「覚えている」場合は、 `rememberRemember`メソッドを使用して、ユーザーが「remember me」Cookieを使用して認証されたかどうかを判断できます。

    if (Auth::viaRemember()) {
        //
    }

<a name="other-authentication-methods"></a>
### その他の認証方法

#### ユーザーインスタンスを認証する

既存のユーザインスタンスをアプリケーションにログインさせる必要がある場合は、ユーザインスタンスで `login`メソッドを呼び出すことができます。与えられたオブジェクトは `Illuminate \ Contracts \ Auth \ Authenticatable` [契約]（/ docs / {{バージョン}} /コントラクト）の実装でなければなりません。もちろん、Laravelに含まれている `App \ User`モデルは既にこのインターフェースを実装しています：

    Auth::login($user);

    // Login and "remember" the given user...
    Auth::login($user, true);

もちろん、使用したいガードインスタンスを指定することもできます：

    Auth::guard('admin')->login($user);

#### ユーザーをIDで認証する

自分のIDでユーザをアプリケーションにログインさせるには、 `loginUsingId`メソッドを使用します。このメソッドは、認証するユーザーの主キーを受け入れます。

    Auth::loginUsingId(1);

    // Login and "remember" the given user...
    Auth::loginUsingId(1, true);

#### ユーザーを一度認証する

`once`メソッドを使用して、単一のリクエストのためにユーザをアプリケーションにログインさせることができます。セッションやクッキーは使用されません。つまり、このメソッドは、ステートレスAPIを構築する際に役立ちます。

    if (Auth::once($credentials)) {
        //
    }

<a name="http-basic-authentication"></a>
## HTTP基本認証

（https://en.wikipedia.org/wiki/Basic_access_authentication）では、専用の「ログイン」ページを設定せずに、アプリケーションのユーザーを簡単に認証する方法を提供しています。開始するには、 `auth.basic` [ミドルウェア]（/ docs / {{version}} /ミドルウェア）をあなたのルートに添付してください。 `auth.basic`ミドルウェアはLaravelフレームワークに含まれているので、定義する必要はありません：

    Route::get('profile', function () {
        // Only authenticated users may enter...
    })->middleware('auth.basic');

ミドルウェアがルートに接続されると、ブラウザのルートにアクセスする際に資格情報が自動的に入力されます。 デフォルトでは、 `auth.basic`ミドルウェアはユーザレコードの` email`カラムを `username 'として使用します。

#### FastCGIに関する注意

PHP FastCGIを使用している場合、HTTP基本認証が正しく機能しないことがあります。 あなたの `.htaccess`ファイルに次の行を追加する必要があります：

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="stateless-http-basic-authentication"></a>
### ステートレスHTTP基本認証

セッションでユーザー識別子Cookieを設定せずにHTTP基本認証を使用することもできます。これはAPI認証に特に役立ちます。 これを行うには、 `onceBasic`メソッドを呼び出す[ミドルウェアを定義する]（/ docs / {{version}} /ミドルウェア）。 `onceBasic`メソッドによって応答が返されない場合、リクエストはさらにアプリケーションに渡されます：

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Support\Facades\Auth;

    class AuthenticateOnceWithBasicAuth
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, $next)
        {
            return Auth::onceBasic() ?: $next($request);
        }

    }

次に、[ルートミドルウェアを登録する]（/ docs / {{version}} /ミドルウェア＃登録ミドルウェア）をルートに添付します。

    Route::get('api/user', function () {
        // Only authenticated users may enter...
    })->middleware('auth.basic.once');

<a name="adding-custom-guards"></a>
## カスタムガードを追加する

`Auth`ファサードで` extend`メソッドを使って独自の認証ガードを定義することができます。 この呼び出しを[サービスプロバイダ]（/ docs / {{version}} / providers）内の `extend`に配置する必要があります。 Laravelはすでに `AuthServiceProvider`を持っているので、そのプロバイダにコードを置くことができます：

    <?php

    namespace App\Providers;

    use App\Services\Auth\JwtGuard;
    use Illuminate\Support\Facades\Auth;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Auth::extend('jwt', function ($app, $name, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\Guard...

                return new JwtGuard(Auth::createUserProvider($config['provider']));
            });
        }
    }

上の例で分かるように、 `extend`メソッドに渡されたコールバックは` Illuminate \ Contracts \ Auth \ Guard`の実装を返すべきです。 このインターフェイスには、カスタムガードを定義するために実装する必要があるいくつかのメソッドが含まれています。 カスタムガードが定義されたら、あなたの `auth.php`設定ファイルの` guards`設定でこのガードを使うことができます：

    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],

<a name="adding-custom-user-providers"></a>
## カスタムユーザプロバイダの追加

従来のリレーショナルデータベースを使用してユーザーを保管していない場合は、独自の認証ユーザープロバイダーでLaravelを拡張する必要があります。 `Auth`ファサードで` provider`メソッドを使用してカスタムユーザプロバイダを定義します：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Auth;
    use App\Extensions\RiakUserProvider;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Auth::provider('riak', function ($app, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\UserProvider...

                return new RiakUserProvider($app->make('riak.connection'));
            });
        }
    }

`provider`メソッドを使ってプロバイダを登録したら、` auth.php`設定ファイルで新しいユーザプロバイダに切り替えることができます。まず、新しいドライバを使用する `provider`を定義します：

    'providers' => [
        'users' => [
            'driver' => 'riak',
        ],
    ],

最後に、あなたの `guard '設定でこのプロバイダを使うことができます：

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
    ],

<a name="the-user-provider-contract"></a>
### ユーザプロバイダ契約

`Illuminate \ Contracts \ Auth \ UserProvider`実装は、MySQL、Riakなどの永続的ストレージシステムから` Illuminate \ Contracts \ Auth \ Authenticatable`実装をフェッチすることのみを担当します。これらの2つのインタフェースは、Laravel認証メカニズムユーザデータの格納方法やその表現に使用されるクラスの種類にかかわらず、機能を継続することができます。

`Illuminate \ Contracts \ Auth \ UserProvider`契約を見てみましょう：

    <?php

    namespace Illuminate\Contracts\Auth;

    interface UserProvider {

        public function retrieveById($identifier);
        public function retrieveByToken($identifier, $token);
        public function updateRememberToken(Authenticatable $user, $token);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(Authenticatable $user, array $credentials);

    }

`retrieveById`関数は通常、MySQLデータベースからの自動インクリメントIDのようなユーザを表すキーを受け取ります。 IDと一致する `Authenticatable`実装は、メソッドによって取得され、返されなければなりません。

`retrieveByToken`関数は、` remember_token`フィールドに格納された一意の `$ identifier`と` remember me "` $ token`によってユーザを取得します。前のメソッドと同様に、 `Authenticatable`実装が返されるべきです。

`updateRememberToken`メソッドは` $ user`フィールド `remember_token`を新しい` $ token`で更新します。新しいトークンは、新しいトークン、成功した「覚えている」ログイン試行に割り当てられたもの、またはユーザーがログアウトしたときのいずれかになります。

`retrieveByCredentials`メソッドは、アプリケーションにサインインしようとしたときに` Auth :: attempt`メソッドに渡された認証情報の配列を受け取ります。メソッドは、これらの資格情報に一致するユーザーのために、基になる永続ストレージを「照会」する必要があります。通常、このメソッドは `$ credentials ['username']`の "where"条件でクエリを実行します。このメソッドは、次に、 `Authenticatable`の実装を返します。 **このメソッドは、パスワードの検証や認証を試みるべきではありません。**

`validateCredentials`メソッドは与えられた` $ user`を `$ credentials`と比較してユーザを認証する必要があります。例えば、このメソッドは `$ user-> getAuthPassword（）`の値を `$ credentials ['password']`の値と比較するのに `Hash :: check`を使うべきでしょう。このメソッドは、パスワードが有効かどうかを示す `true`または` false`を返します。

<a name="the-authenticatable-contract"></a>
### 認証可能な契約

`UserProvider`の各メソッドを探ったので、` Authenticatable`契約を見てみましょう。プロバイダーは、このインタフェースの実装を `retrieveById`メソッドと` retrieveByCredentials`メソッドから返すべきであることを忘れないでください：

    <?php

    namespace Illuminate\Contracts\Auth;

    interface Authenticatable {

        public function getAuthIdentifierName();
        public function getAuthIdentifier();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();

    }

This interface is simple. The `getAuthIdentifierName` method should return the name of the "primary key" field of the user and the `getAuthIdentifier` method should return the "primary key" of the user. In a MySQL back-end, again, this would be the auto-incrementing primary key. The `getAuthPassword` should return the user's hashed password. This interface allows the authentication system to work with any User class, regardless of what ORM or storage abstraction layer you are using. By default, Laravel includes a `User` class in the `app` directory which implements this interface, so you may consult this class for an implementation example.

<a name="events"></a>
## イベント

Laravelは、認証処理中に様々な[events]（/ docs / {{version}} / events）を生成します。 `EventServiceProvider`でこれらのイベントにリスナーを添付することができます：

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Auth\Events\Registered' => [
            'App\Listeners\LogRegisteredUser',
        ],

        'Illuminate\Auth\Events\Attempting' => [
            'App\Listeners\LogAuthenticationAttempt',
        ],

        'Illuminate\Auth\Events\Authenticated' => [
            'App\Listeners\LogAuthenticated',
        ],

        'Illuminate\Auth\Events\Login' => [
            'App\Listeners\LogSuccessfulLogin',
        ],

        'Illuminate\Auth\Events\Failed' => [
            'App\Listeners\LogFailedLogin',
        ],

        'Illuminate\Auth\Events\Logout' => [
            'App\Listeners\LogSuccessfulLogout',
        ],

        'Illuminate\Auth\Events\Lockout' => [
            'App\Listeners\LogLockout',
        ],

        'Illuminate\Auth\Events\PasswordReset' => [
            'App\Listeners\LogPasswordReset',
        ],
    ];
