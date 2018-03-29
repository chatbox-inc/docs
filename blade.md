# Blade Templates

- [はじめに]（＃序論）
- [テンプレートの継承]（＃テンプレート継承）
     - [レイアウトの定義]（#define-a-layout）
     - [レイアウトを拡張する]（＃extend-a-layout）
- [コンポーネントとスロット]（＃コンポーネントとスロット）
- [データ表示]（＃表示データ）
     - [Blade＆JavaScript Frameworks]（＃blade-and-javascript-frameworks）
- [制御構造]（＃制御構造）
     - [Ifステートメント]（＃ifステートメント）
     - [スイッチ文]（＃switch文）
     - [ループ]（＃ループ）
     - [ループ変数]（＃ループ変数）
     - [コメント]（＃コメント）
     - [PHP]（＃php）
- [サブビューを含む]（サブビューを含む＃）
     - [コレクションのレンダリングビュー]（＃rendering-views-for-collections）
- [スタック]（＃スタック）
- [サービスインジェクション]（＃サービスインジェクション）
- [エクステンションブレード]（エクステンションブレード）
     - [カスタムIf文]（＃custom-if-statements）

<a name="introduction"></a>
## 前書き

BladeはLaravelと共に提供されるシンプルで強力なテンプレートエンジンです。 他の一般的なPHPテンプレートエンジンとは異なり、BladeはあなたのビューでプレーンなPHPコードを使用することを制限しません。 実際、Bladeのすべてのビューは通常のPHPコードにコンパイルされ、修正されるまでキャッシュされます。つまり、Bladeは本質的にアプリケーションにオーバーヘッドをゼロにします。 ブレードビューファイルは `.blade.php`ファイル拡張子を使用し、通常は` resources / views`ディレクトリに保存されます。

<a name="template-inheritance"></a>
## テンプレートの継承

<a name="defining-a-layout"></a>
### レイアウトの定義

Bladeを使用する主な利点の2つは_テンプレート継承_と_セクション_です。 まず、簡単な例を見てみましょう。 まず、「マスター」ページレイアウトを調べます。 ほとんどのWebアプリケーションはさまざまなページに同じ一般レイアウトを維持するので、このレイアウトを単一のBladeビューとして定義すると便利です。

    <!-- Stored in resources/views/layouts/app.blade.php -->

    <html>
        <head>
            <title>App Name - @yield('title')</title>
        </head>
        <body>
            @section('sidebar')
                This is the master sidebar.
            @show

            <div class="container">
                @yield('content')
            </div>
        </body>
    </html>

このファイルには、典型的なHTMLマークアップが含まれています。 しかし、 `@ section`と` @ yield`の指示に注意してください。 `@ section`ディレクティブは、名前が示すように、コンテンツのセクションを定義し、` @ yield`ディレクティブは、指定されたセクションの内容を表示するために使用されます。

アプリケーションのレイアウトを定義したので、レイアウトを継承する子ページを定義しましょう。

<a name="extending-a-layout"></a>
### レイアウトを拡張する

子ビューを定義するときは、Bladeの `@ extends`ディレクティブを使用して、子ビューが"継承する "レイアウトを指定します。 ブレードレイアウトを拡張するビューは、 `@ section`ディレクティブを使用してレイアウトのセクションにコンテンツを挿入することができます。 上記の例のように、これらのセクションの内容は `@ yield`を使ってレイアウトに表示されます：

    <!-- Stored in resources/views/child.blade.php -->

    @extends('layouts.app')

    @section('title', 'Page Title')

    @section('sidebar')
        @@parent

        <p>This is appended to the master sidebar.</p>
    @endsection

    @section('content')
        <p>This is my body content.</p>
    @endsection

この例では、 `sidebar`セクションはレイアウトのサイドバーにコンテンツを（上書きするのではなく）追加するために` @@ parent`指示文を利用しています。 @@ parentディレクティブは、ビューがレンダリングされるときのレイアウトの内容に置き換えられます。

> {tip}前の例とは異なり、この `sidebar`セクションは` @show`ではなく `@ endsection`で終わります。 `@ endsection`ディレクティブはセクションを定義するだけですが、` @ show`は定義し**即座に**セクションを生成します。

ブレードビューは、グローバルな `view`ヘルパーを使ってルートから返すことができます：

    Route::get('blade', function () {
        return view('child');
    });

<a name="components-and-slots"></a>
## コンポーネントとスロット

コンポーネントとスロットは、セクションやレイアウトにも同様の利点があります。しかし、コンポーネントやスロットのメンタルモデルが理解しやすくなることがあります。まず、アプリケーション全体で再利用したい、再利用可能な「警告」コンポーネントを想像してみましょう。

    <!-- /resources/views/alert.blade.php -->

    <div class="alert alert-danger">
        {{ $slot }}
    </div>

`{{$ slot}}`変数には、コンポーネントに注入したい内容が入ります。今、このコンポーネントを構築するために、 `@ component` Bladeディレクティブを使うことができます：

    @component('alert')
        <strong>Whoops!</strong> Something went wrong!
    @endcomponent

コンポーネントの複数のスロットを定義すると便利な場合があります。アラートコンポーネントを修正して、「タイトル」の挿入を許可しましょう。名前付きスロットは、名前に一致する変数を「エコーする」ことで表示されます：

    <!-- /resources/views/alert.blade.php -->

    <div class="alert alert-danger">
        <div class="alert-title">{{ $title }}</div>

        {{ $slot }}
    </div>

これで、 `@ slot`指示文を使って名前付きスロットにコンテンツを挿入することができます。 `@ slot`ディレクティブにないコンテンツはすべて、` $ slot`変数の中のコンポーネントに渡されます：

    @component('alert')
        @slot('title')
            Forbidden
        @endslot

        You are not allowed to access this resource!
    @endcomponent

#### 追加のデータをコンポーネントに渡す

場合によっては、追加のデータをコンポーネントに渡す必要が生じることがあります。このため、データの配列を `@ component`ディレクティブの第2引数として渡すことができます。すべてのデータは、コンポーネントテンプレートで変数として使用できるようになります。

    @component('alert', ['foo' => 'bar'])
        ...
    @endcomponent

#### エイリアシングコンポーネント

Bladeコンポーネントがサブディレクトリに格納されている場合は、それらのエイリアスを使用して簡単にアクセスできます。たとえば、 `resources / views / components / alert.blade.php`に格納されているBladeコンポーネントを想像してみてください。 `component`メソッドを使ってコンポーネントを` components.alert`から `alert`にエイリアスすることができます。通常これは `AppServiceProvider`の` boot`メソッドで行います：

    use Illuminate\Support\Facades\Blade;

    Blade::component('components.alert', 'alert');

コンポーネントにエイリアスが設定されたら、次のディレクティブを使用してレンダリングできます。

    @alert(['type' => 'danger'])
        You are not allowed to access this resource!
    @endalert

追加のスロットがない場合は、コンポーネントパラメータを省略することができます。

    @alert
        You are not allowed to access this resource!
    @endalert

<a name="displaying-data"></a>
## データの表示

変数を中括弧で囲み、Bladeビューに渡されるデータを表示することができます。たとえば、次の経路を指定します。

    Route::get('greeting', function () {
        return view('welcome', ['name' => 'Samantha']);
    });

`name`変数の内容を次のように表示することができます：

    Hello, {{ $name }}.

もちろん、ビューに渡される変数の内容の表示に限定されません。 PHP関数の結果をエコーすることもできます。実際、Bladeのecho文の中に、あなたが望む任意のPHPコードを置くことができます：

    The current UNIX timestamp is {{ time() }}.

> {tip} Blade `{{}}`文は、XSS攻撃を防ぐためにPHPの `htmlspecialchars`関数を介して自動的に送られます。

#### エスケープされていないデータを表示する

デフォルトでは、Bladeの `{{}}`文はPHPの `htmlspecialchars`関数を介して自動的に送られ、XSS攻撃を防ぎます。データをエスケープしないようにするには、次の構文を使用します。

    Hello, {!! $name !!}.

> {note}アプリケーションのユーザーから提供されたコンテンツをエコーするときは、非常に注意してください。ユーザーが指定したデータを表示するときは、常にXSS攻撃を防ぐために、エスケープされた二重中括弧構文を使用してください。

#### JSONのレンダリング

JavaScript変数を初期化するためにJSONとしてレンダリングする意図で、配列をビューに渡すことがあります。例えば：

    <script>
        var app = <?php echo json_encode($array); ?>;
    </script>

しかし、手動で `json_encode`を呼び出す代わりに、` @ json` Bladeディレクティブを使うことができます：

    <script>
        var app = @json($array);
    </script>

#### HTMLエンティティエンコーディング

デフォルトでは、Blade（およびLaravel `e 'ヘルパー）はHTMLエンティティを二重エンコードします。ダブルエンコーディングを無効にしたい場合は、 `AppServiceProvider`の` boot`メソッドから `Blade :: withoutDoubleEncoding`メソッドを呼び出してください：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Blade::withoutDoubleEncoding();
        }
    }

<a name="blade-and-javascript-frameworks"></a>
### ブレード＆JavaScriptフレームワーク

多くのJavaScriptフレームワークでは、中括弧を使用してブラウザに表示する必要があることを示すため、 `@ '記号を使用して、表現を変更しないようにすることができます。例えば：

    <h1>Laravel</h1>

    Hello, @{{ name }}.

この例では、 `@ 'シンボルはBladeによって削除されます。しかし、 `{{name}}`式はBladeエンジンの影響を受けず、JavaScriptフレームワークによってレンダリングされます。

#### The `@verbatim` Directive

テンプレートの大部分にJavaScript変数を表示している場合、HTMLを `@ verbatim`ディレクティブで囲むことで、各Bladeエコー文の先頭に` @ `記号を付ける必要はありません。

    @verbatim
        <div class="container">
            Hello, {{ name }}.
        </div>
    @endverbatim

<a name="control-structures"></a>
## 制御構造

テンプレートの継承とデータの表示に加えて、Bladeは、条件文やループなどの一般的なPHPコントロール構造に便利なショートカットを提供します。これらのショートカットは、PHP制御構造で作業する非常にクリーンで簡潔な方法を提供します。

<a name="if-statements"></a>
### Ifステートメント

`if`文を` @ if`、 `@ elseif`、` @ else`、および `@endif`ディレクティブを使って構築することができます。これらのディレクティブは、PHPの対応するディレクティブと同じように機能します。

    @if (count($records) === 1)
        I have one record!
    @elseif (count($records) > 1)
        I have multiple records!
    @else
        I don't have any records!
    @endif

便宜上、Bladeは `@ unless`指示文も提供しています：

    @unless (Auth::check())
        You are not signed in.
    @endunless

既に説明した条件付きディレクティブに加えて、 `@ isset`および` @ empty`ディレクティブは、それぞれのPHP関数の便利なショートカットとして使用できます。

    @isset($records)
        // $records is defined and is not null...
    @endisset

    @empty($records)
        // $records is "empty"...
    @endempty

#### 認証ディレクティブ

`@ auth`と` @ guest`ディレクティブを使って、現在のユーザが認証されているかゲストであるかを素早く判断できます：

    @auth
        // The user is authenticated...
    @endauth

    @guest
        // The user is not authenticated...
    @endguest

必要に応じて、 `@ auth`と` @ guest`ディレクティブを使うときにチェックすべき[authentication guard]（/ docs / {{version}} / authentication）を指定することができます：

    @auth('admin')
        // The user is authenticated...
    @endauth

    @guest('admin')
        // The user is not authenticated...
    @endguest

#### セクション指示

セクションに `@ hasSection`ディレクティブを使ってコンテンツがあるかどうかを調べることができます：

    @hasSection('navigation')
        <div class="pull-right">
            @yield('navigation')
        </div>

        <div class="clearfix"></div>
    @endif

<a name="switch-statements"></a>
### スイッチ文

switch文は、 `@ switch`、` @ case`、 `@ break`、` @ default`、 `@endswitch`ディレクティブを使って構築することができます：

    @switch($i)
        @case(1)
            First case...
            @break

        @case(2)
            Second case...
            @break

        @default
            Default case...
    @endswitch

<a name="loops"></a>
### ループ

条件文に加えて、BladeはPHPのループ構造を操作するための簡単なディレクティブを提供します。 ここでも、これらのディレクティブのそれぞれは、PHPの対応するディレクティブと同じように機能します。

    @for ($i = 0; $i < 10; $i++)
        The current value is {{ $i }}
    @endfor

    @foreach ($users as $user)
        <p>This is user {{ $user->id }}</p>
    @endforeach

    @forelse ($users as $user)
        <li>{{ $user->name }}</li>
    @empty
        <p>No users</p>
    @endforelse

    @while (true)
        <p>I'm looping forever.</p>
    @endwhile

> {tip}ループするときに、[ループ変数]（＃the-loop-variable）を使用して、ループを通した最初の反復か最後の反復かなど、ループに関する重要な情報を得ることができます。

ループを使用する場合は、ループを終了するか、現在の反復をスキップすることもできます。

    @foreach ($users as $user)
        @if ($user->type == 1)
            @continue
        @endif

        <li>{{ $user->name }}</li>

        @if ($user->number == 5)
            @break
        @endif
    @endforeach

また、ディレクティブ宣言付きの条件を1行に含めることもできます。

    @foreach ($users as $user)
        @continue($user->type == 1)

        <li>{{ $user->name }}</li>

        @break($user->number == 5)
    @endforeach

<a name="the-loop-variable"></a>
### ループ変数

ループするとき、 `$ loop`変数がループの内部で利用可能になります。この変数は、現在のループインデックスや、これがループを通した最初の反復か最後の反復かなどの有用な情報ビットへのアクセスを提供します。

    @foreach ($users as $user)
        @if ($loop->first)
            This is the first iteration.
        @endif

        @if ($loop->last)
            This is the last iteration.
        @endif

        <p>This is user {{ $user->id }}</p>
    @endforeach

ネストされたループの場合は、親ループの `$ loop`変数に` parent`プロパティでアクセスすることができます：

    @foreach ($users as $user)
        @foreach ($user->posts as $post)
            @if ($loop->parent->first)
                This is first iteration of the parent loop.
            @endif
        @endforeach
    @endforeach

`$ loop`変数には、他にもさまざまな有用なプロパティがあります：

プロパティ  | 説明
------------- | -------------
`$loop->index`  |  現在のループ反復のインデックス（0から始まる）。
`$loop->iteration`  |  現在のループ反復（1から開始）。
`$loop->remaining`  |  ループ内に残っている反復。
`$loop->count`  |  反復される配列内のアイテムの総数。
`$loop->first`  |  これがループを通る最初の反復であるかどうか。
`$loop->last`  |  これがループを通る最後の反復であるかどうか。
`$loop->depth`  |  現在のループのネストレベル
`$loop->parent`  |  ネストされたループの場合、親のループ変数。

<a name="comments"></a>
### コメント

Bladeでは、ビュー内にコメントを定義することもできます。 ただし、HTMLコメントとは異なり、Bladeコメントはアプリケーションから返されるHTMLには含まれません。

    {{-- This comment will not be present in the rendered HTML --}}

<a name="php"></a>
### PHP

状況によっては、ビューにPHPコードを埋め込むのが便利です。 Bladeの `@ php`ディレクティブを使用して、テンプレート内でプレーンなPHPのブロックを実行することができます：

    @php
        //
    @endphp

> {tip} Bladeはこの機能を提供していますが、頻繁に使用すると、テンプレート内にロジックが多すぎるというシグナルが出る可能性があります。

<a name="including-sub-views"></a>
## サブビューを含む

Bladeの `@include`ディレクティブを使うと、別のビューからブレードビューを含めることができます。親ビューで使用できるすべての変数は、含まれているビューで使用できるようになります。

    <div>
        @include('shared.errors')

        <form>
            <!-- Form Contents -->
        </form>
    </div>

インクルードされたビューは親ビューで使用可能なすべてのデータを継承しますが、追加のデータの配列をインクルードされたビューに渡すこともできます。

    @include('view.name', ['some' => 'data'])

もちろん、存在しないビューを `@include 'しようとすると、Laravelはエラーを投げます。もし存在していてもいなくてもよいビューを含めるには、 `@includeIf`指示文を使うべきです：

    @includeIf('view.name', ['some' => 'data'])

与えられたブール条件に応じてビューを `@include 'したい場合、` @include`ディレクティブを使うことができます：

    @includeWhen($boolean, 'view.name', ['some' => 'data'])

与えられたビューの配列から存在する最初のビューをインクルードするには、 `includeFirst`ディレクティブを使用します：

    @includeFirst(['custom.admin', 'admin'], ['some' => 'data'])

> {note}あなたのブレードビューでは、キャッシュされコンパイルされたビューの位置を参照するため、 `__DIR__`と` __FILE__`定数を使わないでください。

<a name="rendering-views-for-collections"></a>
### コレクションのレンダリングビュー

ループを組み合せて、Bladeの `@ each`指令で1行に含めることができます：

    @each('view.name', $jobs, 'job')

最初の引数は、配列またはコレクション内の各要素に対してレンダリングするビューの一部です。 2番目の引数は、反復処理する配列またはコレクションです.3番目の引数は、ビュー内の現在の反復に割り当てられる変数名です。したがって、たとえば、 `jobs`の配列を反復処理している場合、通常、ビューのpartial内の` job`変数として各ジョブにアクセスしたいと思うでしょう。現在の反復のキーは、ビューのpartial内の `key`変数として利用できます。

第4引数を `@ each`ディレクティブに渡すこともできます。この引数は、指定された配列が空の場合にレンダリングされるビューを決定します。

    @each('view.name', $jobs, 'job', 'view.empty')

> {note} `@ each`を介してレンダリングされたビューは、親ビューから変数を継承しません。子ビューにこれらの変数が必要な場合は、代わりに `@ foreach`と` @include`を使用してください。

<a name="stacks"></a>
## スタック

Bladeを使用すると、別のビューやレイアウトの別の場所にレンダリングできる名前付きスタックにプッシュすることができます。これは、子ビューで必要なJavaScriptライブラリを指定する場合に特に便利です。

    @push('scripts')
        <script src="/example.js"></script>
    @endpush

必要に応じて何度もスタックにプッシュすることができます。完全なスタック内容をレンダリングするには、スタックの名前を `@ stack`ディレクティブに渡します：

    <head>
        <!-- Head Contents -->

        @stack('scripts')
    </head>

<a name="service-injection"></a>
## サービスインジェクション

@Iject`ディレクティブは、Laravel [サービスコンテナ]（/ docs / {{version}} / container）からサービスを取得するために使用できます。 `@ inject`に渡される最初の引数は、サービスが配置される変数の名前です.2番目の引数は、解決したいサービスのクラスまたはインタフェース名です。

    @inject('metrics', 'App\Services\MetricsService')

    <div>
        Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
    </div>

<a name="extending-blade"></a>
## ブレードを伸ばす

Bladeでは、 `directive`メソッドを使用して独自のカスタムディレクティブを定義することができます。 Bladeコンパイラがカスタム・ディレクティブに遭遇すると、ディレクティブに含まれている式を指定してコールバックを呼び出します。

次の例は `$ var`をフォーマットする` @datetime（$ var） `指示文を作成します。これは` DateTime`のインスタンスでなければなりません：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Blade::directive('datetime', function ($expression) {
                return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
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

ご覧のとおり、 `format`メソッドをディレクティブに渡す式に連鎖させます。したがって、この例では、このディレクティブによって生成される最終的なPHPは次のよ​​うになります。

    <?php echo ($var)->format('m/d/Y H:i'); ?>

> {note} Bladeディレクティブのロジックを更新したら、キャッシュされたBladeビューをすべて削除する必要があります。キャッシュされたブレードビューは、 `view：clear` Artisanコマンドを使用して削除することができます。

<a name="custom-if-statements"></a>
### カスタムIf文

カスタムディレクティブのプログラミングは、シンプルなカスタム条件ステートメントを定義するときに必要以上に複雑な場合があります。そのため、Bladeは `Blade :: if`メソッドを提供しています。これにより、Closuresを使用してカスタムの条件付きディレクティブを素早く定義することができます。たとえば、現在のアプリケーション環境をチェックするカスタム条件を定義しましょう。 `AppServiceProvider`の` boot`メソッドでこれを行うかもしれません：

    use Illuminate\Support\Facades\Blade;

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::if('env', function ($environment) {
            return app()->environment($environment);
        });
    }

カスタム条件式が定義されると、テンプレートで簡単に使用できます。

    @env('local')
        // The application is in the local environment...
    @elseenv('testing')
        // The application is in the testing environment...
    @else
        // The application is not in the local or testing environment...
    @endenv
