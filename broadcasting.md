# 放送

- [はじめに]（＃序論）
     - [設定]（＃設定）
     - [ドライバの前提条件]（＃ドライバの前提条件）
- [コンセプト概要]（＃コンセプト概要）
     - [サンプルアプリケーションを使用する]（＃using-example-application）
- [ブロードキャストイベントの定義]（#define-broadcast-events）
     - [ブロードキャスト名]（＃ブロードキャスト名）
     - [ブロードキャストデータ]（＃ブロードキャストデータ）
     - [ブロードキャストキュー]（＃ブロードキャストキュー）
     - [ブロードキャスト条件]（＃ブロードキャスト条件）
- [認証チャネル]（＃認証チャネル）
     - [承認ルートの定義]（#define-authorization-routes）
     - [許可コールバックの定義]（#define-authorization-callbacks）
     - [チャンネルクラスの定義]（#define-channel-classes）
- [放送イベント]（放送イベント数）
     - [他人にのみ]（他人にのみ）
- [受信ブロードキャスト]（＃受信ブロードキャスト）
     - [Laravel Echoのインストール]（＃installing-laravel-echo）
     - [リスニング・フォー・イベント]（リスニング・フォー・イベント）
     - [チャンネルを離れる]（＃脱退するチャンネル）
     - [名前空間]（＃名前空間）
- [Presence Channels]（プレゼンスチャネル数）
     - [プレゼンスチャネルの認可]（＃認可 - プレゼンスチャネル）
     - [参加チャンネルに参加する]（参加するプレゼンスチャンネル数）
     - [プレゼンス・チャンネルへのブロードキャスト]（プレゼンス・チャンネルへのブロードキャスト）
- [クライアントイベント]（＃クライアントイベント）
- [通知]（＃通知）

<a name="introduction"></a>
## 前書き

現代の多くのWebアプリケーションでは、WebSocketを使用してリアルタイムのリアルタイム更新ユーザーインターフェイスを実装しています。サーバー上の一部のデータが更新されると、通常、メッセージはWebSocket接続を介して送信され、クライアントによって処理されます。これにより、継続的に変更をアプリケーションにポーリングするより堅牢で効率的な代替手段が提供されます。

これらのタイプのアプリケーションを構築するのに役立つように、LaravelはWebSocket接続を介してあなたの[events]（/ docs / {{version}} / events）を簡単に "ブロードキャスト"できます。 Laravelイベントをブロードキャストすると、サーバー側コードとクライアント側JavaScriptアプリケーションの間で同じイベント名を共有できます。

> {tip}イベント放送に参加する前に、Laravel [events and listeners]（/ docs / {{version}} / events）に関するすべてのドキュメントをお読みください。

<a name="configuration"></a>
### 設定

アプリケーションのすべてのイベントブロードキャスト設定は `config / broadcast.php`設定ファイルに保存されています。 Laravelは、[プッシャー]（https://pusher.com）、[Redis]（/ docs / {{version}} / redis）、ローカル開発とデバッグのための `log`ドライバとして、 。さらに、放送を完全に無効にすることができる `ヌル`ドライバが含まれています。設定例は `config / broadcast.php`設定ファイルに含まれています。

#### ブロードキャストサービスプロバイダ

Before broadcasting any events, you will first need to register the `App\Providers\BroadcastServiceProvider`. In fresh Laravel applications, you only need to uncomment this provider in the `providers` array of your `config/app.php` configuration file. This provider will allow you to register the broadcast authorization routes and callbacks.

#### CSRFトークン

[Laravel Echo]（＃installing-laravel-echo）は、現在のセッションのCSRFトークンにアクセスする必要があります。アプリケーションの `head` HTML要素がCSRFトークンを含む` meta`タグを定義していることを確認する必要があります：

    <meta name="csrf-token" content="{{ csrf_token() }}">

<a name="driver-prerequisites"></a>
### ドライバーの前提条件

#### プッシャー

[Pusher]（https://pusher.com）でイベントをブロードキャストする場合は、Composerパッケージマネージャを使用してPusher PHP SDKをインストールする必要があります。

    composer require pusher/pusher-php-server "~3.0"

次に、 `config / broadcast.php`設定ファイルでPusher認証情報を設定する必要があります。 Pusherの設定例はすでにこのファイルに含まれており、Pusherのキー、シークレット、およびアプリケーションIDをすばやく指定することができます。 `config / broadcast.php`ファイルの` pusher`設定では、Pusherがサポートしている、クラスタなどの追加の `options`を指定することもできます：

    'options' => [
        'cluster' => 'eu',
        'encrypted' => true
    ],

Pusherと[Laravel Echo]（＃installing-laravel-echo）を使うときは、あなたの `resources / assets / js / bootstrap.js`ファイルでEchoインスタンスをインスタンス化するときに、あなたの希望するブロードキャスタとして` pusher`を指定する必要があります：

    import Echo from "laravel-echo"

    window.Pusher = require('pusher-js');

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key'
    });

#### レディス

Redisのブロードキャスタを使用している場合は、Predisライブラリをインストールする必要があります。

    composer require predis/predis

Redisのブロードキャスターは、R​​edisのpub / sub機能を使用してメッセージをブロードキャストします。ただし、これをRedisからのメッセージを受信して​​WebSocketチャネルにブロードキャストできるWebSocketサーバとペアにする必要があります。

Redisの放送者がイベントをパブリッシュすると、イベントの指定されたチャンネル名にパブリッシュされ、ペイロードはイベント名、 `data`ペイロード、イベントのソケットIDを生成したユーザーを含むJSONエンコードされた文字列になります）。

#### Socket.IO

RedisのブロードキャスタとSocket.IOサーバをペアにする場合は、アプリケーションの `head` HTML要素にSocket.IO JavaScriptクライアントライブラリを含める必要があります。 Socket.IOサーバーが起動すると、自動的にクライアントJavaScriptライブラリが標準URLに公開されます。たとえば、Webアプリケーションと同じドメイン上でSocket.IOサーバーを実行している場合は、次のようにクライアントライブラリにアクセスできます。

    <script src="//{{ Request::getHost() }}:6001/socket.io/socket.io.js"></script>

次に、 `socket.io`コネクタと` host`を使ってEchoをインスタンス化する必要があります。

    import Echo from "laravel-echo"

    window.Echo = new Echo({
        broadcaster: 'socket.io',
        host: window.location.hostname + ':6001'
    });

最後に、互換性のあるSocket.IOサーバーを実行する必要があります。 LaravelにはSocket.IOサーバーの実装は含まれていません。しかし、コミュニティ駆動のSocket.IOサーバは現在、[tlaverdure / laravel-echo-server]（https://github.com/tlaverdure/laravel-echo-server）のGitHubリポジトリで管理されています。

#### キューの前提条件

イベントをブロードキャストする前に、[キューリスナー]（/ docs / {{バージョン}} /キュー）を設定して実行する必要があります。すべてのイベントブロードキャストは、アプリケーションの応答時間に重大な影響を与えないように、キューに入れられたジョブを介して行われます。

<a name="concept-overview"></a>
## コンセプトの概要

Laravelのイベントブロードキャストでは、WebSocketへのドライバベースのアプローチを使用して、クライアント側のJavaScriptアプリケーションにサーバー側のLaravelイベントをブロードキャストできます。現在、Laravelには[Pusher]（https://pusher.com）とRedisドライバが同梱されています。イベントは、[Laravel Echo]（＃installation-laravel-echo）Javascriptパッケージを使用してクライアント側で簡単に消費される可能性があります。

イベントは「チャネル」を介してブロードキャストされ、パブリックまたはプライベートとして指定することができます。アプリケーションへの訪問者は、認証や承認なしにパブリックチャネルにサブスクライブすることができます。ただし、プライベートチャネルに加入するには、そのチャネルでリッスンするユーザーを認証して認証する必要があります。

<a name="using-example-application"></a>
### サンプルアプリケーションの使用

イベント放送の各コンポーネントに入る前に、eコマースストアを例にして概要を説明しましょう。 [Pusher]（https://pusher.com）や[Laravel Echo]（＃installing-laravel-echo）の設定の詳細については、このドキュメントの他のセクションで詳しく説明しますので、ここでは説明しません。

アプリケーションでは、ユーザーが注文の出荷状況を表示できるページがあるとします。配送状況の更新がアプリケーションによって処理されたときに `ShippingStatusUpdated`イベントが発生したと仮定しましょう：

    event(new ShippingStatusUpdated($update));

#### `ShouldBroadcast`インターフェース

ユーザーが注文の1つを表示しているときに、ステータス更新を表示するためにページを更新する必要はありません。代わりに、アプリケーションが作成されたときに更新をブロードキャストしたいと考えています。だから、 `ShouldBroadcast`インターフェースで` ShippingStatusUpdated`イベントをマークする必要があります。これにより、Laravelはイベントが発生したときにそのイベントをブロードキャストします。

    <?php

    namespace App\Events;

    use Illuminate\Broadcasting\Channel;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

    class ShippingStatusUpdated implements ShouldBroadcast
    {
        /**
         * Information about the shipping status update.
         *
         * @var string
         */
        public $update;
    }

`ShouldBroadcast`インターフェースは私たちのイベントが` broadcastOn`メソッドを定義することを要求します。このメソッドは、イベントがブロードキャストするチャネルを返す責任があります。このメソッドの空のスタブは、すでに生成されたイベントクラスで定義されているため、その詳細を入力するだけです。オーダーの作成者がステータスの更新を表示できるようにするため、オーダーに関連付けられたプライベートチャンネルでイベントをブロードキャストします：

    /**
     * Get the channels the event should broadcast on.
     *
     * @return array
     */
    public function broadcastOn()
    {
        return new PrivateChannel('order.'.$this->update->order_id);
    }

#### 認証チャネル

覚えておいて、ユーザーはプライベートチャンネルで聞く権限を持っている必要があります。私たちは `routes / channels.php`ファイルにチャンネル認可ルールを定義しています。この例では、プライベート `order.1`チャンネルをリッスンしようとするユーザーが実際に注文の作成者であることを確認する必要があります。

    Broadcast::channel('order.{orderId}', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

`channel`メソッドは、チャンネルの名前と、` true`を返すコールバックと、チャンネルでリッスンする権限があるかどうかを示す `false`の2つの引数を受け取ります。

すべての認可コールバックは、現在認証されているユーザーを最初の引数として受け取り、追加のワイルドカードパラメータを後続の引数として受け取ります。この例では、チャネル名の「ID」部分がワイルドカードであることを示すために `{orderId}`プレースホルダを使用しています。

#### イベントブロードキャストのリッスン

次に、残っていることは、JavaScriptアプリケーションでイベントをリスンすることだけです。私たちはLaravel Echoを使ってこれを行うことができます。まずプライベートチャンネルを購読するために `private`メソッドを使います。次に、 `listen`メソッドを使用して` ShippingStatusUpdated`イベントを待ち受けることができます。デフォルトでは、イベントのすべてのパブリックプロパティがブロードキャストイベントに含まれます。

    Echo.private(`order.${orderId}`)
        .listen('ShippingStatusUpdated', (e) => {
            console.log(e.update);
        });

<a name="defining-broadcast-events"></a>
## ブロードキャストイベントの定義

特定のイベントをブロードキャストする必要があることをLaravelに通知するには、イベントクラスに「Illuminate \ Contracts \ Broadcasting \ ShouldBroadcast」インターフェイスを実装します。 このインターフェイスは、フレームワークによって生成されたすべてのイベントクラスに既にインポートされているため、イベントに簡単に追加できます。

`ShouldBroadcast`インターフェースでは、` broadcastOn`メソッドを実装する必要があります。 `broadcastOn`メソッドは、イベントが放送するチャンネルまたはチャンネルの配列を返します。 チャネルは `Channel`、` PrivateChannel`、または `PresenceChannel`のインスタンスでなければなりません。 `Channel`のインスタンスは、任意のユーザが加入することができるパブリックチャネルを表し、` PrivateChannels`と `PresenceChannels`は[チャネル許可]を必要とするプライベートチャネル（＃authorization-channels）を表します。

    <?php

    namespace App\Events;

    use Illuminate\Broadcasting\Channel;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

    class ServerCreated implements ShouldBroadcast
    {
        use SerializesModels;

        public $user;

        /**
         * Create a new event instance.
         *
         * @return void
         */
        public function __construct(User $user)
        {
            $this->user = $user;
        }

        /**
         * Get the channels the event should broadcast on.
         *
         * @return Channel|array
         */
        public function broadcastOn()
        {
            return new PrivateChannel('user.'.$this->user->id);
        }
    }

その後、通常どおりに[イベントを起動する]（/ docs / {{version}} / events）だけで済みます。 イベントが発生すると、[queued job]（/ docs / {{version}} / queues）は指定したブロードキャストドライバでイベントを自動的にブロードキャストします。

<a name="broadcast-name"></a>
### 放送名

デフォルトでは、Laravelはイベントのクラス名を使用してイベントをブロードキャストします。 ただし、イベントに対して `broadcastAs`メソッドを定義することで、ブロードキャスト名をカスタマイズすることができます：

    /**
     * The event's broadcast name.
     *
     * @return string
     */
    public function broadcastAs()
    {
        return 'server.created';
    }

`broadcastAs`メソッドを使ってブロードキャスト名をカスタマイズする場合は、リスナーを先行する` .`文字で登録するようにしてください。これは、イベントにアプリケーションのネームスペースを付加しないようにEchoに指示します：

    .listen('.server.created', function (e) {
        ....
    });

<a name="broadcast-data"></a>
### ブロードキャストデータ

イベントがブロードキャストされると、そのすべての `public`プロパティが自動的にシリアライズされ、イベントのペイロードとしてブロードキャストされ、JavaScriptアプリケーションからパブリックデータにアクセスできます。たとえば、あなたのイベントにEloquentモデルを含む単一のパブリック `$ user`プロパティがある場合、イベントのブロードキャストペイロードは次のようになります：

    {
        "user": {
            "id": 1,
            "name": "Patrick Stewart"
            ...
        }
    }

しかし、あなたのブロードキャストペイロードをもっときめ細かくコントロールしたいならば、イベントに `broadcastWith`メソッドを追加することができます。このメソッドは、イベントペイロードとしてブロードキャストするデータの配列を返す必要があります。

    /**
     * Get the data to broadcast.
     *
     * @return array
     */
    public function broadcastWith()
    {
        return ['id' => $this->user->id];
    }

<a name="broadcast-queue"></a>
### ブロードキャストキュー

デフォルトでは、各ブロードキャストイベントは、 `queue.php`設定ファイルで指定されたデフォルトのキュー接続のデフォルトキューに置かれます。 イベントクラスに `broadcastQueue`プロパティを定義することで、ブロードキャスタが使用するキューをカスタマイズすることができます。 このプロパティでは、ブロードキャスト時に使用するキューの名前を指定する必要があります。

    /**
     * The name of the queue on which to place the event.
     *
     * @var string
     */
    public $broadcastQueue = 'your-queue-name';

If you want to broadcast your event using the `sync` queue instead of the default queue driver, you can implement the `ShouldBroadcastNow` interface instead of `ShouldBroadcast`:

    <?php

    use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

    class ShippingStatusUpdated implements ShouldBroadcastNow
    {
        //
    }
<a name="broadcast-conditions"></a>
### 放送条件

特定の条件が満たされている場合にのみ、イベントをブロードキャストしたい場合もあります。 これらの条件はイベントクラスに `broadcastWhen`メソッドを追加することで定義できます：

    /**
     * Determine if this event should broadcast.
     *
     * @return bool
     */
    public function broadcastWhen()
    {
        return $this->value > 100;
    }

<a name="authorizing-channels"></a>
## 認証チャネル

プライベートチャネルでは、現在認証されているユーザーが実際にチャネルでリッスンできることを承認する必要があります。これは、チャネル名でLaravelアプリケーションへのHTTP要求を行い、アプリケーションがそのチャネルでリッスンできるかどうかをアプリケーションが判断できるようにすることで実現します。 [Laravel Echo]（＃installing-laravel-echo）を使用すると、プライベートチャネルへのサブスクリプションを承認するHTTP要求が自動的に行われます。ただし、これらの要求に応答するための適切なルートを定義する必要があります。

<a name="defining-authorization-routes"></a>
### 認可ルートの定義

ありがたいことに、Laravelはチャネル承認要求に応答するルートを簡単に定義できます。 Laravelアプリケーションに含まれている `BroadcastServiceProvider`に` Broadcast :: routes`メソッドの呼び出しがあります。このメソッドは `/ broadcast / auth`ルートを登録して承認リクエストを処理します：

    Broadcast::routes();

`Broadcast :: routes`メソッドは自動的にそのルートを` web`ミドルウェアグループ内に置きます。ただし、割り当てられた属性をカスタマイズする場合は、ルート属性の配列をメソッドに渡すことができます。

    Broadcast::routes($attributes);

<a name="defining-authorization-callbacks"></a>
### 認可コールバックの定義

次に、チャネル認可を実際に実行するロジックを定義する必要があります。これはあなたのアプリケーションに含まれている `routes / channels.php`ファイルで行われます。このファイルでは、 `Broadcast :: channel`メソッドを使用してチャネル認証コールバックを登録することができます：

    Broadcast::channel('order.{orderId}', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

`channel`メソッドは、チャンネルの名前と、` true`を返すコールバックと、チャンネルでリッスンする権限があるかどうかを示す `false`の2つの引数を受け取ります。

すべての認可コールバックは、現在認証されているユーザーを最初の引数として受け取り、追加のワイルドカードパラメータを後続の引数として受け取ります。この例では、チャネル名の「ID」部分がワイルドカードであることを示すために `{orderId}`プレースホルダを使用しています。

#### 認可コールバックモデルバインディング

HTTPルートと同様に、チャネルルートは暗黙的で明示的な[ルートモデルのバインディング]（/ docs / {{バージョン}} /ルーティング＃ルートモデルバインディング）を利用することもできます。たとえば、文字列または数値の注文IDを受け取る代わりに、実際の `Order`モデルインスタンスをリクエストすることができます：

    use App\Order;

    Broadcast::channel('order.{order}', function ($user, Order $order) {
        return $user->id === $order->user_id;
    });

<a name="defining-channel-classes"></a>
### チャネルクラスの定義

アプリケーションが多くの異なるチャネルを使用している場合、 `routes / channels.php`ファイルが大きくなる可能性があります。したがって、Closuresを使用してチャネルを承認する代わりに、チャネルクラスを使用することができます。チャネルクラスを生成するには、 `make：channel` Artisanコマンドを使用します。このコマンドは新しいチャンネルクラスを `App / Broadcasting`ディレクトリに置きます。

    php artisan make:channel OrderChannel

次に、チャンネルを `routes / channels.php`ファイルに登録します：

    use App\Broadcasting\OrderChannel;

    Broadcast::channel('order.{order}', OrderChannel::class);

最後に、チャンネルの認可ロジックをチャンネルクラスの `join`メソッドに置くことができます。この `join`メソッドは、通常、あなたのチャンネル認可閉鎖に置いたのと同じロジックを格納します。もちろん、チャネルモデルのバインディングを利用することもできます。

    <?php

    namespace App\Broadcasting;

    use App\User;
    use App\Order;

    class OrderChannel
    {
        /**
         * Create a new channel instance.
         *
         * @return void
         */
        public function __construct()
        {
            //
        }

        /**
         * Authenticate the user's access to the channel.
         *
         * @param  \App\User  $user
         * @param  \App\Order  $order
         * @return array|bool
         */
        public function join(User $user, Order $order)
        {
            return $user->id === $order->user_id;
        }
    }

> {tip} Like many other classes in Laravel, channel classes will automatically be resolved by the [service container](/docs/{{version}}/container). So, you may type-hint any dependencies required by your channel in its constructor.

<a name="broadcasting-events"></a>
## イベントを放送する

イベントを定義し、 `ShouldBroadcast`インターフェースでそれをマークしたら、` event`関数を使ってイベントを起動するだけです。イベントディスパッチャは、イベントが `ShouldBroadcast`インタフェースでマークされていることに気づき、ブロードキャストのためにイベントをキューに入れます：

    event(new ShippingStatusUpdated($update));

<a name="only-to-others"></a>
### 他人にのみ

イベント放送を利用するアプリケーションを構築する場合、 `event`関数を` broadcast`関数に置き換えることができます。 `event`関数と同様に、` broadcast`関数はあなたのサーバー側のリスナーにイベントを送ります：

    broadcast(new ShippingStatusUpdated($update));

しかし、 `broadcast`関数は` toOthers`メソッドも公開しています。これにより、現在のユーザを放送の受信者から除外することができます：

    broadcast(new ShippingStatusUpdated($update))->toOthers();

`toOthers`メソッドを使いたいときをよりよく理解するために、タスク名を入力して新しいタスクを作成するタスクリストアプリケーションを想像してみましょう。タスクを作成するために、アプリケーションはタスクの作成をブロードキャストし、新しいタスクのJSON表現を返す `/ task`エンドポイントにリクエストを行うかもしれません。 JavaScriptアプリケーションがエンドポイントからの応答を受け取ると、次のように新しいタスクをタスクリストに直接挿入することがあります。

    axios.post('/task', task)
        .then((response) => {
            this.tasks.push(response.data);
        });

イベントを定義し、 `ShouldBroadcast`インターフェースでそれをマークしたら、` event`関数を使ってイベントを起動するだけです。イベントディスパッチャは、イベントが `ShouldBroadcast`インタフェースでマークされていることに気づき、ブロードキャストのためにイベントをキューに入れます：

> {note}あなたのイベントは `toOthers`メソッドを呼び出すために` Illuminate \ Broadcasting \ InteractsWithSockets`特性を使わなければなりません。

#### 構成

Laravel Echoインスタンスを初期化すると、接続にソケットIDが割り当てられます。 [Vue]（https://vuejs.org）と[Axios]（https://github.com/mzabriskie/axios）を使用している場合、ソケットIDは自動的にすべての発信リクエストに `X- 「ソケットID」ヘッダ。次に、 `toOthers`メソッドを呼び出すと、LaravelはヘッダからソケットIDを抽出し、そのソケットIDを持つ接続にブロードキャストしないようにブロードキャスタに指示します。

VueとAxiosを使用していない場合は、 `X-Socket-ID`ヘッダを送信するようにJavaScriptアプリケーションを手動で設定する必要があります。 `Echo.socketId`メソッドを使ってソケットIDを取得することができます：

    var socketId = Echo.socketId();

<a name="receiving-broadcasts"></a>
## ブロードキャストの受信

<a name="installing-laravel-echo"></a>
### Laravelエコーのインストール

Laravel Echoは、チャンネルを購読してLaravelで放送されたイベントを聞くのに苦労するJavaScriptライブラリです。 Echoは、NPMパッケージマネージャを使用してインストールできます。 この例では、Pusherブロードキャスタを使用するので、 `pusher-js`パッケージもインストールします：

    npm install --save laravel-echo pusher-js

Echoがインストールされると、アプリケーションのJavaScriptで新しいEchoインスタンスを作成する準備が整います。 これを行うのに最適な場所は、Laravelフレームワークに含まれている `resources / assets / js / bootstrap.js`ファイルの一番下にあります。

    import Echo from "laravel-echo"

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key'
    });

`pusher`コネクタを使用するEchoインスタンスを作成するときには、` cluster`と接続を暗号化するかどうかを指定することもできます：

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key',
        cluster: 'eu',
        encrypted: true
    });

<a name="listening-for-events"></a>
### イベントのリスニング

Echoをインストールしてインスタンス化すると、イベントブロードキャストの受信を開始する準備が整います。 まず、 `channel`メソッドを使用してチャンネルのインスタンスを取得し、` listen`メソッドを呼び出して指定されたイベントを待ち受けます：

    Echo.channel('orders')
        .listen('OrderShipped', (e) => {
            console.log(e.order.name);
        });

プライベートチャンネルでイベントを聞きたいのであれば、代わりに `private`メソッドを使用してください。 `listen`メソッドへの呼び出しを連鎖し続け、単一のチャンネルで複数のイベントを待ち受けることができます：

    Echo.private('orders')
        .listen(...)
        .listen(...)
        .listen(...);

<a name="leaving-a-channel"></a>
### チャンネルを離れる

チャンネルを終了するには、あなたのEchoインスタンスで `leave`メソッドを呼び出すことができます：

    Echo.leave('orders');

<a name="namespaces"></a>
### 名前空間

上の例では、イベントクラスの完全な名前空間を指定していないことに気付いたかもしれません。 これは、Echoがイベントが `App \ Events`名前空間にあると自動的に判断するためです。 ただし、 `namespace`設定オプションを渡してEchoをインスタンス化するときに、ルート名前空間を設定することができます：

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key',
        namespace: 'App.Other.Namespace'
    });

あるいは、エコーを使ってイベントクラスに登録するときに、イベントクラスの前に `.`を付けることもできます。 これにより、完全修飾クラス名を常に指定することができます：

    Echo.channel('orders')
        .listen('.Namespace.Event.Class', (e) => {
            //
        });

<a name="presence-channels"></a>
## 在席チャンネル

プレゼンスチャネルは、プライベートチャネルのセキュリティを基盤として構築され、チャネルに加入しているユーザーの認知度を向上させます。 これにより、別のユーザーが同じページを表示しているときにユーザーに通知するなど、強力なコラボレーションアプリケーション機能を簡単に構築できます。

<a name="authorizing-presence-channels"></a>
### プレゼンス・チャネルの認可

すべてのプレゼンス・チャネルもプライベート・チャネルです。 したがって、ユーザーは[アクセス権がある]（＃authorization-channels）する必要があります。 ただし、プレゼンス・チャネルの認可コールバックを定義するときに、ユーザーがチャネルに参加する権限を持っている場合はtrueを返しません。 代わりに、ユーザーに関するデータの配列を返す必要があります。

The data returned by the authorization callback will be made available to the presence channel event listeners in your JavaScript application. If the user is not authorized to join the presence channel, you should return `false` or `null`:

    Broadcast::channel('chat.{roomId}', function ($user, $roomId) {
        if ($user->canJoinRoom($roomId)) {
            return ['id' => $user->id, 'name' => $user->name];
        }
    });

<a name="joining-presence-channels"></a>
### 在席チャンネルへの参加

プレゼンスチャンネルに参加するには、Echoの `join`メソッドを使用します。 `join`メソッドは、` listen`メソッドを公開するとともに、 `here`、` joined`、 `leave`イベントにサブスクライブする` PresenceChannel`実装を返します。

    Echo.join(`chat.${roomId}`)
        .here((users) => {
            //
        })
        .joining((user) => {
            console.log(user.name);
        })
        .leaving((user) => {
            console.log(user.name);
        });

`here`コールバックは、チャンネルが正常に結合されると即座に実行され、チャンネルに現在加入している他のすべてのユーザーのユーザー情報を含む配列を受け取ります。 `join`メソッドは、新しいユーザーがチャンネルに参加するときに実行され、` leave`メソッドはユーザーがチャンネルを離れるときに実行されます。

<a name="broadcasting-to-presence-channels"></a>
### 存在チャネルへのブロードキャスト

プレゼンスチャネルは、パブリックチャネルまたはプライベートチャネルのようなイベントを受信することがあります。 チャットルームの例を使用して、ルームのプレゼンスチャネルに `NewMessage`イベントをブロードキャストしたい場合があります。 これを行うために、イベントの `broadcastOn`メソッドから` PresenceChannel`のインスタンスを返します：

    /**
     * Get the channels the event should broadcast on.
     *
     * @return Channel|array
     */
    public function broadcastOn()
    {
        return new PresenceChannel('room.'.$this->message->room_id);
    }

公開イベントまたは非公開イベントと同様に、プレゼンスチャネルイベントは、「ブロードキャスト」機能を使用してブロードキャストされてもよい。 他のイベントと同様に、 `toOthers`メソッドを使用して、現在のユーザがブロードキャストを受信するのを除外することができます：

    broadcast(new NewMessage($message));

    broadcast(new NewMessage($message))->toOthers();

あなたはEchoの `listen`メソッドを使ってjoinイベントを聞くことができます：

    Echo.join(`chat.${roomId}`)
        .here(...)
        .joining(...)
        .leaving(...)
        .listen('NewMessage', (e) => {
            //
        });

<a name="client-events"></a>
## クライアントイベント

場合によっては、Laravelアプリケーションをまったく使用せずに、接続されている他のクライアントにイベントをブロードキャストすることもできます。 これは、特定の画面に別のユーザーがメッセージを入力しているアプリケーションのユーザーに警告する、「入力」通知などの場合に特に便利です。 クライアントイベントをブロードキャストするには、Echoの `whisper`メソッドを使用します：

    Echo.private('chat')
        .whisper('typing', {
            name: this.user.name
        });

クライアントイベントをリッスンするには、 `listenForWhisper`メソッドを使用します：

    Echo.private('chat')
        .listenForWhisper('typing', (e) => {
            console.log(e.name);
        });

<a name="notifications"></a>
## 通知

イベントのブロードキャストと[通知]（/ docs / {{バージョン}} /通知）を組み合わせることで、JavaScriptアプリケーションはページを更新することなく新しい通知を受け取ることがあります。 まず、ブロードキャスト通知チャネル（/ docs / {{バージョン}} /通知＃ブロードキャスト通知）の使用に関するドキュメントを必ず読んでください。

ブロードキャストチャンネルを使用するように通知を設定したら、Echoの `notification`メソッドを使用してブロードキャストイベントを聞くことができます。 チャンネル名は、通知を受け取るエンティティのクラス名と一致する必要があります。

    Echo.private(`App.User.${userId}`)
        .notification((notification) => {
            console.log(notification.type);
        });

この例では、 `App \ User`インスタンスに` broadcast`チャンネル経由で送信されたすべての通知がコールバックによって受信されます。 `App.User。{id}`チャンネルのチャンネル許可コールバックは、Laravelフレームワークに同梱されているデフォルトの `BroadcastServiceProvider`に含まれています。
