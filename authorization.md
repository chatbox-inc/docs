# 認証

- [はじめに]（＃序論）
- [ゲート]（＃ゲート）
     - [書き込みゲート]（＃書き込みゲート）
     - [承認アクション]（＃権限付与アクション - 経由ゲート）
- [ポリシーの作成]（＃creating-policies）
     - [生成ポリシー]（＃生成ポリシー）
     - [ポリシーの登録]（＃登録ポリシー）
- [書き込み方針]（＃writing-policies）
     - [ポリシーメソッド]（#page-methods）
     - [モデルのないメソッド]（＃methods-without-models）
     - [ポリシーフィルタ]（＃ポリシーフィルタ）
- [ポリシーを使用したアクションの承認]（＃authorizing-actions-using-policies）
     - [ユーザーモデルを介して]（＃ユーザーモデル経由）
     - [ミドルウェア経由]（＃ミドルウェア経由）
     - [コントローラヘルパ経由で]（コントローラ経由で＃）
     - [Via Blade Templates]（＃via-blade-templates）

<a name="introduction"></a>
## 前書き
   
   [認証]（/ docs / {{バージョン}} /認証）サービスをそのまま利用できるほか、Laravelは与えられたリソースに対するユーザーアクションを簡単に承認する方法を提供します。認証と同様に、Laravelの承認方法は簡単で、アクションを承認する主な方法は2つあります.1つはゲートとポリシーです。
   
   ゲートやルートやコントローラなどのポリシーを考えてみましょう。ゲイツ氏は、単純なクロージャベースの認可手法を提供しますが、コントローラなどのポリシーは、特定のモデルやリソースのロジックをグループ化します。私たちは最初に門を掘り下げ、政策を調べます。
   
   ゲートを排他的に使用するか、アプリケーションを構築するときにポリシーを排他的に使用するかを選択する必要はありません。ほとんどのアプリケーションでは、ゲートとポリシーが混在している可能性が高くなります。ゲイツは、管理者のダッシュボードの表示など、モデルやリソースに関係のないアクションに最も適しています。対照的に、ポリシーは、特定のモデルまたはリソースのアクションを承認する場合に使用する必要があります。

<a name="gates"></a>
## ゲイツ

<a name="writing-gates"></a>
### ゲートを書く
    
    ゲートは、ユーザーが特定のアクションを実行する権限を与えられているかどうかを判断するクロージャであり、通常は `Gate \ 'ファサードを使用して` App \ Providers \ AuthServiceProvider`クラスで定義されます。 ゲイツは常に最初の引数としてユーザーインスタンスを受け取り、必要に応じて関連するEloquentモデルなどの追加の引数を受け取ることができます。

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', function ($user, $post) {
            return $user->id == $post->user_id;
        });
    }

ゲイツは、コントローラのような `Class @ method`スタイルのコールバック文字列を使って定義することもできます：

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', 'PostPolicy@update');
    }

#### リソースゲート

また、 `resource`メソッドを使って複数のGate能力を一度に定義することもできます：

    Gate::resource('posts', 'PostPolicy');

これは以下のゲート定義を手動で定義するのと同じです：

    Gate::define('posts.view', 'PostPolicy@view');
    Gate::define('posts.create', 'PostPolicy@create');
    Gate::define('posts.update', 'PostPolicy@update');
    Gate::define('posts.delete', 'PostPolicy@delete');

デフォルトでは、 `view`、` create`、 `update`、` delete`の各能力が定義されます。 `resource`メソッドの第3引数として配列を渡すことで、デフォルトの能力を上書きしたり追加したりすることができます。 配列のキーは能力の名前を定義し、値はメソッド名を定義します。 例えば、次のコードは `posts.image`と` posts.photo`の2つの新しいGate定義を作成します：

    Gate::resource('posts', 'PostPolicy', [
        'image' => 'updateImage',
        'photo' => 'updatePhoto',
    ]);

<a name="authorizing-actions-via-gates"></a>
### 承認アクション

ゲートを使ってアクションを認可するには、 `allows`メソッドまたは` denies`メソッドを使うべきです。 現在認証されているユーザーをこれらのメソッドに渡す必要はありません。 Laravelは自動的にユーザーをゲートに通す処理を行います閉鎖：

    if (Gate::allows('update-post', $post)) {
        // The current user can update the post...
    }

    if (Gate::denies('update-post', $post)) {
        // The current user can't update the post...
    }

特定のユーザがアクションを実行する権限を持っているかどうかを確認したい場合は、 `Gate`ファサードで` forUser`メソッドを使うことができます：

    if (Gate::forUser($user)->allows('update-post', $post)) {
        // The user can update the post...
    }

    if (Gate::forUser($user)->denies('update-post', $post)) {
        // The user can't update the post...
    }

<a name="creating-policies"></a>
## ポリシーの生成

<a name="generating-policies"></a>
### ポリシーの生成

ポリシーとは、特定のモデルやリソースを対象とする認可ロジックを構成するクラスです。 例えば、あなたのアプリケーションがブログの場合、 `Post`モデルとそれに対応する` PostPolicy`があり、投稿の作成や更新などのユーザアクションを承認することができます。

`make：policy` [artisanコマンド]（/ docs / {{version}} / artisan）を使ってポリシーを生成することができます。 生成されたポリシーは `app / Policies`ディレクトリに置かれます。 このディレクトリがアプリケーションに存在しない場合、Laravelはそれを作成します：

    php artisan make:policy PostPolicy

`make：policy`コマンドは、空のポリシークラスを生成します。 すでにクラスに含まれている基本的な "CRUD"ポリシーメソッドを持つクラスを生成したい場合は、コマンドを実行するときに `--model`を指定することができます：

    php artisan make:policy PostPolicy --model=Post

> {tip}すべてのポリシーはLaravel [サービスコンテナ]（/ docs / {{version}} / container）で解決され、ポリシーコンストラクタに必要な依存関係を入力して自動的に注入することができます。

<a name="registering-policies"></a>
### ポリシーの登録

ポリシーが存在したら、それを登録する必要があります。 最新のLaravelアプリケーションに含まれている `AuthServiceProvider`には、Eloquentモデルを対応するポリシーにマップする` policies`プロパティが含まれています。 ポリシーを登録すると、特定のモデルに対するアクションを承認する際にどのポリシーを利用するかをLaravelに指示します。

    <?php

    namespace App\Providers;

    use App\Post;
    use App\Policies\PostPolicy;
    use Illuminate\Support\Facades\Gate;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * The policy mappings for the application.
         *
         * @var array
         */
        protected $policies = [
            Post::class => PostPolicy::class,
        ];

        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            //
        }
    }

<a name="writing-policies"></a>
## ポリシーの作成

<a name="policy-methods"></a>
### ポリシーの作成

ポリシーが登録されると、承認されたアクションごとにメソッドを追加できます。 例えば、指定された `User`が与えられた` Post`インスタンスを更新できるかどうかを判断する `PostPolicy`で` update`メソッドを定義しましょう。

`update`メソッドは引数として` User`と `Post`インスタンスを受け取り、与えられた` Post`を更新する権限があるかどうかを示す `true`か` false`を返します。 したがって、この例では、ユーザーの `id`が投稿の` user_id`と一致することを確認しましょう：

    <?php

    namespace App\Policies;

    use App\User;
    use App\Post;

    class PostPolicy
    {
        /**
         * Determine if the given post can be updated by the user.
         *
         * @param  \App\User  $user
         * @param  \App\Post  $post
         * @return bool
         */
        public function update(User $user, Post $post)
        {
            return $user->id === $post->user_id;
        }
    }

権限を与えられたさまざまなアクションの必要に応じて、ポリシー上で追加のメソッドを定義し続けることができます。たとえば `view`メソッドや` delete`メソッドを定義して、 `Post`アクションを許可することもできますが、ポリシーメソッドに任意の名前を付けることができます。

> {tip} Artisanコンソールからポリシーを生成するときに `--model`オプションを使用した場合、` view`、 `create`、` update`、 `delete`アクションのメソッドがすでに含まれています。

<a name="methods-without-models"></a>
### モデルのないメソッド

一部のポリシーメソッドは、現在認証されているユーザーのみを受け取り、許可したモデルのインスタンスは受け取りません。この状況は `create`アクションを承認するときに最も一般的です。たとえば、ブログを作成している場合、ユーザーが投稿を作成する権限を持っているかどうかを確認することができます。

`create`メソッドのようにモデルインスタンスを受け取らないポリシーメソッドを定義するとき、モデルインスタンスを受け取ることはありません。代わりに、メソッドを認証されたユーザーのみが必要とするものとして定義する必要があります。

    /**
     * Determine if the given user can create posts.
     *
     * @param  \App\User  $user
     * @return bool
     */
    public function create(User $user)
    {
        //
    }

<a name="policy-filters"></a>
### ポリシーフィルタ

特定のユーザーの場合、特定のポリシー内のすべてのアクションを許可することができます。 これを達成するには、ポリシーに対して `before`メソッドを定義します。 `before`メソッドは、ポリシー上の他のメソッドの前に実行され、意図したポリシーメソッドが実際に呼び出される前にアクションを承認する機会を与えます。 この機能は、アプリケーション管理者に何らかのアクションを実行する権限を与えるために最も一般的に使用されます。

    public function before($user, $ability)
    {
        if ($user->isSuperAdmin()) {
            return true;
        }
    }

ユーザのすべての権限を拒否したい場合は、 `before`メソッドから` false`を返すべきです。 `null`が返された場合、権限はポリシーメソッドに渡されます。

> {note}ポリシークラスの `before`メソッドは、クラスにチェックされている能力の名前と一致する名前のメソッドが含まれていない場合は呼び出されません。

<a name="authorizing-actions-using-policies"></a>
## ポリシーを使用した承認アクション

<a name="via-the-user-model"></a>
### ユーザモデルを介して

Laravelアプリケーションに含まれる `User`モデルには、` can`と `cant`の2つのアクションを許可する便利なメソッドがあります。 `can`メソッドは、あなたが承認したいアクションと関連するモデルを受け取ります。 例えば、ユーザーが特定の `Post`モデルを更新する権限を持っているかどうかを判断しましょう。

    if ($user->can('update', $post)) {
        //
    }

指定されたモデルに対して[policyが登録されている]場合（＃登録ポリシー）、 `can`メソッドは自動的に適切なポリシーを呼び出してブール結果を返します。 モデルに登録されているポリシーがない場合、 `can`メソッドは、与えられたアクション名と一致するClosureベースのGateを呼び出そうとします。

#### モデルを必要としないアクション

`create`のようないくつかのアクションはモデルインスタンスを必要としないかもしれないことを忘れないでください。 このような状況では、クラス名を `can`メソッドに渡すことができます。 クラス名は、アクションを承認するときにどのポリシーを使用するかを決定するために使用されます。

    use App\Post;

    if ($user->can('create', Post::class)) {
        // Executes the "create" method on the relevant policy...
    }

<a name="via-middleware"></a>
### ミドルウェア経由

Laravelには、到着したリクエストが自分のルートやコントローラに到達する前にアクションを承認できるミドルウェアが含まれています。 デフォルトでは、 `Illuminate \ Auth \ Middleware \ Authorize`ミドルウェアは` App \ Http \ Kernel`クラスの `can`キーに割り当てられています。 `can`ミドルウェアを使ってユーザーがブログ投稿を更新できるようにする例を探そう。

    use App\Post;

    Route::put('/post/{post}', function (Post $post) {
        // The current user may update the post...
    })->middleware('can:update,post');

この例では、 `can`ミドルウェアに2つの引数を渡しています。 最初のものは許可するアクションの名前で、もう1つはポリシーメソッドに渡すルートパラメータです。 この場合、[暗黙のモデルバインディング]（/ docs / {{version}} / routing＃暗黙的バインディング）を使用しているので、 `Post`モデルがポリシーメソッドに渡されます。 ユーザーが与えられたアクションを実行する権限を与えられていない場合、ミドルウェアはステータスコード「403」のHTTP応答を生成します。

#### モデルを必要としないアクション

やはり、 `create`のようないくつかのアクションはモデルインスタンスを必要としないかもしれません。 このような状況では、クラス名をミドルウェアに渡すことができます。 クラス名は、アクションを承認するときにどのポリシーを使用するかを決定するために使用されます。

    Route::post('/post', function () {
        // The current user may create posts...
    })->middleware('can:create,App\Post');

<a name="via-controller-helpers"></a>
### コントローラヘルパーを介して

`User`モデルに提供された便利なメソッドに加えて、Laravelは、` App \ Http \ Controllers \ Controller`ベースクラスを拡張する任意のコントローラに有用な `authorize`メソッドを提供します。 `can`メソッドと同様に、このメソッドは、承認するアクションの名前と関連するモデルを受け入れます。 アクションが許可されていない場合、 `authorize`メソッドは` Illuminate \ Auth \ Access \ AuthorizationException`をスローします。デフォルトのLaravel例外ハンドラは `403`ステータスコードのHTTPレスポンスに変換されます：

    <?php

    namespace App\Http\Controllers;

    use App\Post;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given blog post.
         *
         * @param  Request  $request
         * @param  Post  $post
         * @return Response
         */
        public function update(Request $request, Post $post)
        {
            $this->authorize('update', $post);

            // The current user can update the blog post...
        }
    }

#### モデルを必要としないアクション

前述のように、 `create`のようないくつかのアクションはモデルインスタンスを必要としないかもしれません。 このような状況では、クラス名を `authorize`メソッドに渡すことができます。 クラス名は、アクションを承認するときにどのポリシーを使用するかを決定するために使用されます。

    /**
     * Create a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function create(Request $request)
    {
        $this->authorize('create', Post::class);

        // The current user can create blog posts...
    }

<a name="via-blade-templates"></a>
### ビアブレードテンプレート

ブレードテンプレートを書き込むときに、ユーザーが特定の操作を実行する権限を与えられている場合にのみ、ページの一部を表示することができます。 たとえば、ユーザーが実際に投稿を更新できる場合のみ、ブログ投稿の更新フォームを表示することができます。 このような状況では、 `@ can`と` @ cannot`のディレクティブファミリーを使うことができます：

    @can('update', $post)
        <!-- The Current User Can Update The Post -->
    @elsecan('create', App\Post::class)
        <!-- The Current User Can Create New Post -->
    @endcan

    @cannot('update', $post)
        <!-- The Current User Can't Update The Post -->
    @elsecannot('create', App\Post::class)
        <!-- The Current User Can't Create New Post -->
    @endcannot

これらのディレクティブは `@ if`文と` @ unless`文を書くための便利なショートカットです。 上記の `@ can`と` @ cannot`は、それぞれ以下のステートメントに翻訳されます：

    @if (Auth::user()->can('update', $post))
        <!-- The Current User Can Update The Post -->
    @endif

    @unless (Auth::user()->can('update', $post))
        <!-- The Current User Can't Update The Post -->
    @endunless

#### モデルを必要としないアクション

他の認可メソッドの大部分と同様に、アクションがモデルインスタンスを必要としない場合、 `@ can`および` @ cannot`ディレクティブにクラス名を渡すことができます：

    @can('create', App\Post::class)
        <!-- The Current User Can Create Posts -->
    @endcan

    @cannot('create', App\Post::class)
        <!-- The Current User Can't Create Posts -->
    @endcannot
