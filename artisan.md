＃アーティザンコンソール

- [はじめに]（ #序論）
- [コマンドの記述]（＃writing-commands）
     - [コマンドの生成]（＃生成コマンド）
     - [コマンド構造体]（＃コマンド構造体）
     - [クローズコマンド]（＃closure-commands）
- [入力期待値の定義]（#define-input-expectations）
     - [引数]（＃引数）
     - [オプション]（＃オプション）
     - [入力配列]（＃input-arrays）
     - [入力の説明]（＃入力の説明）
- [コマンドI /O]（＃command-io）
     - [入力の取得]（＃取得入力）
     - [プロンプト入力]（＃プロンプト入力）
     - [書き込み出力]（＃書き込み出力）
- [コマンドの登録]（＃registers-commands）
- [プログラムで実行するコマンド]（＃programmatically-executing-commands）
     - [他のコマンドからのコマンドの呼び出し]（＃calling-commands-from-other-commands）

<a name="introduction"></a>
## 前書き

ArtisanはLaravelに含まれるコマンドラインインターフェイスです。アプリケーションの構築中に役立つ多くの便利なコマンドが用意されています。使用可能なすべてのArtisanコマンドのリストを表示するには、 `list`コマンドを使用します：

    PHP職人リスト

すべてのコマンドには、コマンドの使用可能な引数とオプションを表示し、説明する「ヘルプ」画面も含まれています。ヘルプ画面を表示するには、コマンド名の前に `help`を付けます：

    PHPの職人の移行を支援

#### Laravel REPL

すべてのLaravelアプリケーションには、[PsySH]（https://github.com/bobthecow/psysh）パッケージを搭載したREPLのTinkerが含まれています。 Tinkerを使用すると、Eloquent ORM、ジョブ、イベントなど、コマンドラインでLaravelアプリケーション全体と対話することができます。 Tinker環境に入るには、 `tinker` Artisanコマンドを実行します：

    PHPの職人の汚れ

<a name="writing-commands"> </a>
##コマンドを書く

Artisanで提供されるコマンドに加えて、独自のカスタムコマンドを作成することもできます。コマンドは通常、 `app / Console / Commands`ディレクトリに格納されます。ただし、Composerでコマンドをロードできるのであれば、自分の記憶場所を自由に選択することができます。

<a name="generating-commands"> </a>
###コマンドの生成

新しいコマンドを作成するには、 `make：command` Artisanコマンドを使います。このコマンドは `app / Console / Commands`ディレクトリに新しいコマンドクラスを作成します。 `make：command` Artisanコマンドを初めて実行するときに作成されるため、このディレクトリがアプリケーションに存在しない場合は心配しないでください。生成されたコマンドには、すべてのコマンドに存在するプロパティとメソッドのデフォルトセットが含まれます。

    php artisan make：コマンドSendEmails

<a name="command-structure"> </a>
###コマンド構造

コマンドを生成した後、クラスの `signature`と` description`プロパティを記入してください。これは `list`スクリーンにコマンドを表示する際に使用されます。 `handle`メソッドは、あなたのコマンドが実行されるときに呼び出されます。この方法では、コマンドロジックを配置することができます。

> {tip}コードの再利用を増やすには、コンソールコマンドを軽くしておき、アプリケーションサービスに任せることでタスクを完了させることをお勧めします。以下の例では、電子メールを送信するための "重労働"を行うサービスクラスを導入しています。

コマンドの例を見てみましょう。必要な依存関係をコマンドのコンストラクタに注入できることに注意してください。 Laravel [サービスコンテナ]（/ docs / {{version}} / container）は、コンストラクタでタイプヒントされたすべての依存関係を自動的に挿入します：

    <?php

    namespace App\Console\Commands;

    use App\User;
    use App\DripEmailer;
    use Illuminate\Console\Command;

    class SendEmails extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'email:send {user}';

        / **
         *コンソールコマンドの説明。
         *
         * @var文字列
         * /
        protected $ description = 'ドリップ電子メールをユーザーに送信する';

        /**
         * The drip e-mail service.
         *
         * @var DripEmailer
         */
        protected $drip;

        /**
         * Create a new command instance.
         *
         * @param  DripEmailer  $drip
         * @return void
         */
        public function __construct(DripEmailer $drip)
        {
            parent::__construct();

            $this->drip = $drip;
        }

        /**
         * Execute the console command.
         *
         * @return mixed
         */
        public function handle()
        {
            $this->drip->send(User::find($this->argument('user')));
        }
    }

<a name="closure-commands"></a>
###クロージャコマンド

クロージャベースのコマンドは、コンソールコマンドをクラスとして定義する代わりに使用できます。 ルートクロージャーがコントローラーの代替品と同じように、コマンドクラスの代わりにコマンドクロージャーを考えてみましょう。 あなたの `app / Console / Kernel.php`ファイルの` commands`メソッドの中で、Laravelは `routes / console.php`ファイルをロードします：

    /**
     * Register the Closure based commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        require base_path('routes/console.php');
    }

このファイルではHTTPルートは定義されていませんが、アプリケーションにコンソールベースのエントリポイント（ルート）が定義されています。 このファイルの中で、 `Artisan :: command`メソッドを使ってClosureベースのルートをすべて定義することができます。 `command`メソッドは2つの引数を受け取ります：[コマンドシグネチャ]（#define-input-expectations）とコマンド引数とオプションを受け取るクロージャ：

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    });

Closureは基本となるコマンドインスタンスにバインドされているため、完全なコマンドクラスで一般的にアクセスできるヘルパーメソッドすべてにフルアクセスできます。

#### タイプヒントの依存関係

コマンドの引数とオプションを受け取ることに加えて、Command Closuresは、[サービスコンテナ]（/ docs / {{version}} / container）から解決したい追加の依存関係をタイプヒントすることもできます：

    use App\User;
    use App\DripEmailer;

    Artisan::command('email:send {user}', function (DripEmailer $drip, $user) {
        $drip->send(User::find($user));
    });

#### 終了コマンドの説明

クロージャベースのコマンドを定義するときは、 `describe`メソッドを使用してコマンドに説明を追加することができます。 この説明は `php artisan list`や` php artisan help`コマンドを実行すると表示されます：

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    })->describe('Build the project');

<a name="defining-input-expectations"></a>
## 入力期待値の定義

コンソールコマンドを書くときは、引数やオプションでユーザからの入力を収集するのが一般的です。 Laravelは、あなたのコマンドの `signature`プロパティを使って、ユーザから期待される入力を定義することを非常に便利にします。 `signature`プロパティは、コマンドのための名前、引数、オプションを、表現力豊かなルートに似た単一の構文で定義することを可能にします。

<a name="arguments"></a>
### 引数

ユーザーが指定した引数とオプションはすべて中括弧で囲みます。 次の例では、このコマンドは**必須**引数を定義しています： `user`：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

引数をオプションにして、引数のデフォルト値を定義することもできます。

    // Optional argument...
    email:send {user?}

    // Optional argument with default value...
    email:send {user=foo}

<a name="options"></a>
### オプション

引数のようなオプションは、ユーザ入力の別の形式です。 オプションは、コマンドラインで指定されたときに2つのハイフン（ ` - `）が付加されています。 オプションには、値を受け取るオプションと値を受け取るオプションの2種類があります。 値を受け取らないオプションはブール型の "スイッチ"として機能します。 このタイプのオプションの例を見てみましょう：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue}';

In this example, the `--queue` switch may be specified when calling the Artisan command. If the `--queue` switch is passed, the value of the option will be `true`. Otherwise, the value will be `false`:

    php artisan email:send 1 --queue

<a name="options-with-values"></a>
#### 値のあるオプション

次に、値が必要なオプションを見てみましょう。 ユーザがオプションの値を指定しなければならない場合は、オプション名に `= '記号をつけてください：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue=}';

この例では、ユーザは次のようにオプションの値を渡すことができます：

    php artisan email:send 1 --queue=default

オプション名の後にデフォルト値を指定することによって、オプションにデフォルト値を割り当てることができます。 オプション値がユーザーによって渡されない場合、デフォルト値が使用されます。

    email:send {user} {--queue=default}

<a name="option-shortcuts"></a>
#### オプションのショートカット

オプションを定義するときにショートカットを割り当てるには、オプション名の前に指定し、| 完全なオプション名からショートカットを区切る区切り文字：

    email:send {user} {--Q|queue}

<a name="input-arrays"></a>
### 入力配列

配列の入力を期待する引数やオプションを定義したい場合は、 `*`文字を使うことができます。 まず、配列引数を指定する例を見てみましょう：

    email:send {user*}

このメソッドを呼び出すときには、 `user`引数をコマンドラインに順番に渡すことができます。 たとえば、次のコマンドは `user`の値を` ['foo'、 'bar'] `に設定します：

    php artisan email:send foo bar

配列入力が必要なオプションを定義する場合、コマンドに渡される各オプション値の前にオプション名を付ける必要があります。

    email:send {user} {--id=*}

    php artisan email:send --id=1 --id=2

<a name="input-descriptions"></a>
### 入力の説明

コロンを使用して説明からパラメーターを分離することによって、入力引数およびオプションに説明を割り当てることができます。 あなたのコマンドを定義するために余分なスペースが必要な場合は、定義を複数の行に自由に広げてください。

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send
                            {user : The ID of the user}
                            {--queue= : Whether the job should be queued}';

<a name="command-io"></a>
## コマンドI / O

<a name="retrieving-input"></a>
### 入力の取得

あなたのコマンドが実行されている間は、明らかにあなたのコマンドで受け入れられた引数とオプションの値にアクセスする必要があります。 そうするために、あなたは `argument`と` option`メソッドを使うことができます：

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $userId = $this->argument('user');

        //
    }

すべての引数を `array`として取り出す必要がある場合は、` arguments`メソッドを呼び出します：

    $arguments = $this->arguments();

オプションは、 `option`メソッドを使って引数と同じように簡単に取り出すことができます。 すべてのオプションを配列として取得するには、 `options`メソッドを呼び出します。

    // Retrieve a specific option...
    $queueName = $this->option('queue');

    // Retrieve all options...
    $options = $this->options();

引数またはオプションが存在しない場合、 `null`が返されます。

<a name="prompting-for-input"></a>
### 入力を求めるプロンプト

出力を表示することに加えて、コマンドの実行中に入力を提供するようにユーザーに依頼することもできます。 `ask`メソッドは、ユーザに質問を促し、入力を受け入れ、ユーザの入力をコマンドに返す。

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->ask('What is your name?');
    }

`secret`メソッドは` ask`と似ていますが、コンソールに入力する際にユーザーの入力は表示されません。 このメソッドは、パスワードなどの機密情報を要求するときに便利です。

    $password = $this->secret('What is the password?');

#### 確認を求める

ユーザに簡単な確認を求める必要がある場合は、 `confirm`メソッドを使うことができます。 デフォルトでは、このメソッドは `false`を返します。 しかし、ユーザがプロンプトに応じて「y」または「yes」を入力すると、メソッドは「true」を返す。

    if ($this->confirm('Do you wish to continue?')) {
        //
    }

#### オートコンプリート

`予想する`メソッドは、可能な選択肢の自動補完を提供するために使用することができます。 ユーザーは、自動補完のヒントに関係なく、任意の回答を選択できます。

    $name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

#### 複数の選択肢の質問

ユーザーに定義済みの選択肢を与える必要がある場合は、 `choice`メソッドを使用することができます。 オプションが選択されていない場合、返されるデフォルト値の配列インデックスを設定することができます：

    $name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $defaultIndex);

<a name="writing-output"></a>
### 出力を書く

出力をコンソールに送るには `line`、` info`、 `comment`、` question`と `error`メソッドを使います。 これらのメソッドのそれぞれは、目的に応じて適切なANSIカラーを使用します。 たとえば、一般的な情報をユーザーに表示してみましょう。 通常、 `info`メソッドはコンソールに緑色のテキストとして表示されます：

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->info('Display this on the screen');
    }

Tエラーメッセージを表示するには、 `error`メソッドを使います。 エラーメッセージのテキストは、通常、赤色で表示されます。

    $this->error('Something went wrong!');

単純な無色のコンソール出力を表示したい場合は、 `line`メソッドを使います：

    $this->line('Display this on the screen');

#### テーブルのレイアウト

`table`メソッドは、複数の行/列のデータを正しくフォーマットすることを容易にします。 ヘッダーと行をメソッドに渡すだけです。 幅と高さは、与えられたデータに基づいて動的に計算されます：

    $headers = ['Name', 'Email'];

    $users = App\User::all(['name', 'email'])->toArray();

    $this->table($headers, $users);

#### プログレスバー

長時間実行されるタスクの場合は、進行状況インジケータを表示すると便利です。 出力オブジェクトを使用して、プログレスバーを開始、進行、停止することができます。 まず、プロセスが反復処理するステップの総数を定義します。 次に、各項目を処理した後にプログレスバーを進めます。

    $users = App\User::all();

    $bar = $this->output->createProgressBar(count($users));

    foreach ($users as $user) {
        $this->performTask($user);

        $bar->advance();
    }

    $bar->finish();

より高度なオプションについては、[Symfony Progress Barのコンポーネントのドキュメント]（https://symfony.com/doc/2.7/components/console/helpers/progressbar.html）を参照してください。

<a name="registering-commands"></a>
## コマンドの登録

コンソールカーネルの `commands`メソッドの` load`メソッド呼び出しのため、 `app / Console / Commands`ディレクトリ内のすべてのコマンドは自動的にArtisanに登録されます。 実際に、 `load`メソッドを呼び出してArtisanコマンドのために他のディレクトリを自由に呼び出すことができます：

    /**
     * Register the commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');
        $this->load(__DIR__.'/MoreCommands');

        // ...
    }

`app / Console / Kernel.php`ファイルの` $ commands`プロパティにクラス名を追加することで、コマンドを手動で登録することもできます。 Artisanが起動すると、このプロパティにリストされているすべてのコマンドは、[サービスコンテナ]（/ docs / {{version}} / container）によって解決され、Artisanに登録されます：

    protected $commands = [
        Commands\SendEmails::class
    ];

<a name="programmatically-executing-commands"></a>
## プログラムでプログラムを実行する

場合によっては、CLIの外でArtisanコマンドを実行することもできます。 たとえば、ルートまたはコントローラからアーティザンコマンドを発射することができます。 これを達成するために `Artisan`ファサードで` call`メソッドを使うことができます。 `call`メソッドは、コマンドの名前を第1引数として受け付け、コマンドパラメータの配列を第2引数として受け付けます。 終了コードが返されます：

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

Using the `queue` method on the `Artisan` facade, you may even queue Artisan commands so they are processed in the background by your [queue workers](/docs/{{version}}/queues). Before using this method, make sure you have configured your queue and are running a queue listener:

    Route::get('/foo', function () {
        Artisan::queue('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

`Artisan`ファサードで` queue`メソッドを使うことで、Artisanコマンドを待ち行列に入れさえすることができます。それらはあなたの[キューワーカー]（/ docs / {{version}} / queues）によってバックグラウンドで処理されます。 この方法を使用する前に、キューを構成し、キュー・リスナーを実行していることを確認してください。

    Artisan::queue('email:send', [
        'user' => 1, '--queue' => 'default'
    ])->onConnection('redis')->onQueue('commands');

#### 配列値の受け渡し

あなたのコマンドが配列を受け入れるオプションを定義している場合、その配列に値の配列を渡すことができます：

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--id' => [5, 13]
        ]);
    });

#### ブール値の受け渡し

`migrate：refresh`コマンドの` --force`フラグのように、文字列値を受け入れないオプションの値を指定する必要がある場合は、 `true`または` false`を渡すべきです：

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

<a name="calling-commands-from-other-commands"></a>
### 他のコマンドからのコマンドの呼び出し

場合によっては、既存のArtisanコマンドから他のコマンドを呼び出すこともできます。 あなたは `call`メソッドを使ってそうすることができます。 この `call`メソッドは、コマンド名とコマンドパラメータの配列を受け取ります：

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    }

別のコンソールコマンドを呼び出してその出力をすべて抑制するには、 `callSilent`メソッドを使用します。 `callSilent`メソッドは` call`メソッドと同じシグネチャを持っています：

    $this->callSilent('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
