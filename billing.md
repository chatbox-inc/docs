# ラーベールキャッシャー

- [はじめに]（＃序論）
- [設定]（＃設定）
     - [Stripe]（＃ストライプ構成）
     - [Braintree]（＃braintree-configuration）
     - [通貨構成]（＃通貨構成）
- [定期購入]（定期購入数）
     - [サブスクリプションの作成]（＃creating-subscriptions）
     - [購読状況の確認]（＃checking-subscription-status）
     - [プランの変更]（＃変更プラン）
     - [契約数]（＃契約数）
     - [サブスクリプション税]（サブスクリプション税）
     - [定期購入のキャンセル]（＃契約のキャンセル）
     - [定期購読の再開]（＃再開 - 定期購読）
     - [クレジットカードの更新]（＃更新カード）
- [購読試行]（＃購読試行）
     - [クレジットカードを手前に]（クレジットカードで手前に＃）
     - [クレジットカードなし]（＃クレジットカードなし）
- [Stripe Webhooksの取り扱い]（＃handling-stripe-webhooks）
     - [Webhookイベントハンドラの定義]（#define-webhook-event-handler）
     - [失敗したサブスクリプション]（処理に失敗したサブスクリプション数）
- [Braintree Webhooksの取り扱い]（＃handling-braintree-webhooks）
     - [Webhookイベントハンドラの定義]（#define-braintree-webhook-event-handlers）
     - [失敗した定期購読]（＃handling-braintree-failed-subscriptions）
- [シングルチャージ]（シングルチャージ）
- [請求書]（請求書番号）
     - [請求書PDFの生成]（＃generating-invoice-pdfs）

<a name="introduction"></a>
## 前書き

Laravel Cashierは、[Stripe's]（https://stripe.com）および[Braintree's]（https://www.braintreepayments.com）のサブスクリプション課金サービスへの表現力豊かなインターフェイスを提供します。 それはあなたが書くことを恐れている定型募集の請求コードのほとんどを処理します。 キャッシャーは、基本的なサブスクリプション管理に加えて、クーポンの処理、サブスクリプションの交換、サブスクリプションの「数量」、キャンセル猶予期間、さらには請求書PDFの生成も可能です。

> {note}「一回限りの」請求のみを行っており、サブスクリプションを提供していない場合は、キャッシャーを使用しないでください。 代わりに、Stripe SDKとBraintree SDKを直接使用してください。

<a name="configuration"></a>
## 構成

<a name="stripe-configuration"></a>
### ストライプ

#### Composer

まず、Stripe用のCashierパッケージを依存関係に追加します。

    composer require "laravel/cashier":"~7.0"

#### データベースの移行

キャッシャーを使用する前に、[データベースの準備]（/ docs / {{バージョン}} /移行）も行う必要があります。 あなたの `users`テーブルにいくつかのカラムを追加し、私たちの顧客のサブスクリプションをすべて保持するための新しい` subscriptions`テーブルを作成する必要があります：

    Schema::table('users', function ($table) {
        $table->string('stripe_id')->nullable();
        $table->string('card_brand')->nullable();
        $table->string('card_last_four')->nullable();
        $table->timestamp('trial_ends_at')->nullable();
    });

    Schema::create('subscriptions', function ($table) {
        $table->increments('id');
        $table->integer('user_id');
        $table->string('name');
        $table->string('stripe_id');
        $table->string('stripe_plan');
        $table->integer('quantity');
        $table->timestamp('trial_ends_at')->nullable();
        $table->timestamp('ends_at')->nullable();
        $table->timestamps();
    });

移行が作成されたら、 `migrate` Artisanコマンドを実行してください。

#### 請求可能なモデル

次に、 `Billable`特性をモデル定義に追加します。 この特性は、サブスクリプションの作成、クーポンの適用、クレジットカード情報の更新など、共通の課金タスクを実行するためのさまざまな方法を提供します。

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

#### APIキー

最後に、 `services.php`設定ファイルでStripeキーを設定する必要があります。 ストライプコントロールパネルからストライプAPIキーを取得することができます：

    'stripe' => [
        'model'  => App\User::class,
        'key' => env('STRIPE_KEY'),
        'secret' => env('STRIPE_SECRET'),
    ],

<a name="braintree-configuration"></a>
### ブレーンツリー

#### ブレーントリーの警告

多くの操作で、キャッシャーのストライプとブレーントリーの実装は同じように機能します。 どちらのサービスもクレジットカードによる定期購入請求を提供していますが、BraintreeはPayPalによる支払いもサポートしています。 しかし、BraintreeにはStripeがサポートしている機能もいくつかあります。 StripeまたはBraintreeを使用することを決定するときは、次の点に注意してください。

<div class="content-list" markdown="1">
- BraintreeはPayPalをサポートしていますが、StripeはPayPalをサポートしていません。
- Braintreeはサブスクリプションの `increment`と` decrement`メソッドをサポートしていません。 これはブレーンツリーの制限であり、キャッシャーの制限ではありません。
- Braintreeはパーセンテージベースの割引をサポートしていません。 これはブレーンツリーの制限であり、キャッシャーの制限ではありません。
</div>

#### Composer

まず、Braintree用のキャッシャーパッケージをあなたの依存関係に追加してください：

    composer require "laravel/cashier-braintree":"~2.0"

#### サービスプロバイダ

次に、あなたの `config / app.php`設定ファイルに` Laravel \ Cashier \ CashierServiceProvider` [サービスプロバイダ]（/ docs / {{version}} / providers）を登録します：

    Laravel\Cashier\CashierServiceProvider::class

#### クレジットクーポンの計画

ブレーンツリーでキャッシャーを使用する前に、ブレーンツリーのコントロールパネルで「プランクレジット」割引を定義する必要があります。 この割引は、毎年から毎月の請求、または毎月から毎年の請求に変更される購読を適切に比例配分するために使用されます。

Braintreeのコントロールパネルで設定された割引額は、クーポンを適用するたびにキャッシャーが定義した金額を独自の金額で上書きするため、任意の値を設定できます。 このクーポンは、Braintreeがサブスクリプション頻度の間に比例配分をサポートしていないため、必要です。

#### データベースの移行

キャッシャーを使用する前に、[データベースを準備する]（/ docs / {{バージョン}} /移行）が必要です。 あなたの `users`テーブルにいくつかのカラムを追加し、私たちの顧客のサブスクリプションをすべて保持するための新しい` subscriptions`テーブルを作成する必要があります：

    Schema::table('users', function ($table) {
        $table->string('braintree_id')->nullable();
        $table->string('paypal_email')->nullable();
        $table->string('card_brand')->nullable();
        $table->string('card_last_four')->nullable();
        $table->timestamp('trial_ends_at')->nullable();
    });

    Schema::create('subscriptions', function ($table) {
        $table->increments('id');
        $table->integer('user_id');
        $table->string('name');
        $table->string('braintree_id');
        $table->string('braintree_plan');
        $table->integer('quantity');
        $table->timestamp('trial_ends_at')->nullable();
        $table->timestamp('ends_at')->nullable();
        $table->timestamps();
    });

移行が作成されたら、 `migrate` Artisanコマンドを実行してください。

#### 請求可能なモデル

次に、あなたのモデル定義に `Billable`特性を追加します：

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

#### API キー

次に、 `services.php`ファイルに以下のオプションを設定する必要があります：

    'braintree' => [
        'model'  => App\User::class,
        'environment' => env('BRAINTREE_ENV'),
        'merchant_id' => env('BRAINTREE_MERCHANT_ID'),
        'public_key' => env('BRAINTREE_PUBLIC_KEY'),
        'private_key' => env('BRAINTREE_PRIVATE_KEY'),
    ],

次に、次のBraintree SDK呼び出しを `AppServiceProvider`サービスプロバイダの` boot`メソッドに追加する必要があります：

    \Braintree_Configuration::environment(config('services.braintree.environment'));
    \Braintree_Configuration::merchantId(config('services.braintree.merchant_id'));
    \Braintree_Configuration::publicKey(config('services.braintree.public_key'));
    \Braintree_Configuration::privateKey(config('services.braintree.private_key'));

<a name="currency-configuration"></a>
### 通貨構成

キャッシャーのデフォルト通貨は、米ドル（USD）です。 デフォルトの通貨を変更するには、サービスプロバイダの `boot`メソッドの中から` Cashier :: useCurrency`メソッドを呼び出します。 `useCurrency`メソッドは、通貨と通貨記号の2つの文字列パラメータを受け取ります：

    use Laravel\Cashier\Cashier;

    Cashier::useCurrency('eur', '€');

<a name="subscriptions"></a>
## 定期購読

<a name="creating-subscriptions"></a>
### 購読の作成

サブスクリプションを作成するには、まず請求可能なモデルのインスタンスを取得します。このインスタンスは、通常はApp \ User`のインスタンスになります。 モデルインスタンスを取得したら、 `newSubscription`メソッドを使ってモデルのサブスクリプションを作成することができます：

    $user = User::find(1);

    $user->newSubscription('main', 'premium')->create($stripeToken);

`newSubscription`メソッドに渡される最初の引数は、サブスクリプションの名前でなければなりません。 アプリケーションが単一のサブスクリプションしか提供しない場合は、これを `main`または` primary`と呼ぶかもしれません。 2番目の引数は、ユーザーが加入している特定のStripe / Braintreeプランです。 この値は、StripeまたはBraintreeのプランの識別子に対応する必要があります。

Stripeクレジットカード/ソーストークンを受け取る `create`メソッドは、サブスクリプションを開始するだけでなく、顧客IDと他の関連する課金情報でデータベースを更新します。

#### 追加のユーザーの詳細

追加の顧客の詳細を指定したい場合は、 `create`メソッドの第2引数として渡すことで可能です。

    $user->newSubscription('main', 'monthly')->create($stripeToken, [
        'email' => $email,
    ]);

StripeまたはBraintreeがサポートする追加フィールドの詳細については、Stripeの[顧客作成に関するドキュメント]（https://stripe.com/docs/api#create_customer）または対応する[Braintreeのドキュメント]（https：// developers .braintreepayments.com / reference / request / customer / create / php）.

#### クーポン

サブスクリプションの作成時にクーポンを適用する場合は、 `withCoupon`メソッドを使用します：

    $user->newSubscription('main', 'monthly')
         ->withCoupon('code')
         ->create($stripeToken);

<a name="checking-subscription-status"></a>
### サブスクリプションステータスの確認

ユーザーがアプリケーションに加入すると、さまざまな便利な方法を使用して簡単にそのサブスクリプションのステータスを確認できます。 まず、サブスクリプションが試用期間内であっても、ユーザがアクティブなサブスクリプションを持っている場合、 `subscribed`メソッドは` true`を返します：

    if ($user->subscribed('main')) {
        //
    }

`subscribed`メソッドは、[ルートミドルウェア]（/ docs / {{バージョン}} /ミドルウェア）の候補にもなり、ユーザの購読ステータスに基づいてルートとコントローラへのアクセスをフィルタリングすることができます：

    public function handle($request, Closure $next)
    {
        if ($request->user() && ! $request->user()->subscribed('main')) {
            // This user is not a paying customer...
            return redirect('billing');
        }

        return $next($request);
    }

ユーザーが試用期間内にいるかどうかを確認したい場合は、 `onTrial`メソッドを使用することができます。 このメソッドは、試用期間中であることをユーザーに警告するために役立ちます。

    if ($user->subscription('main')->onTrial()) {
        //
    }

`subscribedToPlan`メソッドは、ユーザーが与えられたStripe / Braintree計画IDに基づいて与えられた計画に加入しているかどうかを決定するために使用されます。 この例では、ユーザーの `main`サブスクリプションが` monthly`プランに積極的に加入しているかどうかを判断します：

    if ($user->subscribedToPlan('monthly', 'main')) {
        //
    }

#### サブスクリプションステータスのキャンセル

ユーザーがアクティブなサブスクライバだったのか、サブスクリプションをキャンセルしたのかを判断するには、 `cancelled`メソッドを使用します：

    if ($user->subscription('main')->cancelled()) {
        //
    }

ユーザーがサブスクリプションをキャンセルしたかどうかを判断することもできますが、サブスクリプションが完全に期限切れになるまで "猶予期間"に入っています。 たとえば、ユーザーが当初3月10日に有効期限が切れる予定だった3月5日のサブスクリプションをキャンセルした場合、ユーザーは3月10日まで「猶予期間」に入っています。 このとき、 `subscribed`メソッドは` true`を返すことに注意してください：

    if ($user->subscription('main')->onGracePeriod()) {
        //
    }

<a name="changing-plans"></a>
### プランの変更

ユーザーがアプリケーションに加入した後、新しいサブスクリプションプランに変更したいことがあります。 ユーザーを新しいサブスクリプションにスワップするには、プランの識別子を `swap`メソッドに渡します。

    $user = App\User::find(1);

    $user->subscription('main')->swap('provider-plan-id');

ユーザーが試用中の場合、試用期間は維持されます。 また、購読に「数量」が存在する場合、その数量も維持されます。

プランを入れ替え、試用期間をキャンセルしたい場合は、 `skipTrial`メソッドを使うことができます：

    $user->subscription('main')
            ->skipTrial()
            ->swap('provider-plan-id');

<a name="subscription-quantity"></a>
### 購読数量

> {note}購読数量は、キャッシャーのストライプ版でのみサポートされています。 Braintreeには、Stripeの「数量」に対応する機能はありません。

サブスクリプションは「数量」の影響を受けることがあります。 たとえば、アプリケーションでは、1アカウントにつき1ユーザーあたり** 10ドルを請求する場合があります。 購読数量を簡単に増減するには、 `incrementQuantity`と` decrementQuantity`メソッドを使います：

    $user = User::find(1);

    $user->subscription('main')->incrementQuantity();

    // Add five to the subscription's current quantity...
    $user->subscription('main')->incrementQuantity(5);

    $user->subscription('main')->decrementQuantity();

    // Subtract five to the subscription's current quantity...
    $user->subscription('main')->decrementQuantity(5);

あるいは、 `updateQuantity`メソッドを使って特定の数量を設定することもできます：

    $user->subscription('main')->updateQuantity(10);

`noProrate`メソッドは、料金をプロに格付けせずにサブスクリプションの数量を更新するために使用することができます：

    $user->subscription('main')->noProrate()->updateQuantity(10);

定期購読数の詳細については、[Stripe documentation]（https://stripe.com/docs/subscriptions/quantities）を参照してください。

<a name="subscription-taxes"></a>
### サブスクリプション税

ユーザーがサブスクリプションに支払う税率を指定するには、請求可能なモデルに `taxPercentage`メソッドを実装し、小数点以下2桁以内で0〜100の数値を返します。

    public function taxPercentage() {
        return 20;
    }

`taxPercentage`メソッドは、モデルごとに税率を適用することができます。これは、複数の国と税率にまたがるユーザー基盤に役立つかもしれません。

> {note} `taxPercentage`メソッドは、サブスクリプション料金にのみ適用されます。 キャッシャーを使用して「一回限りの」請求を行う場合、その時点で税率を手動で指定する必要があります。

<a name="cancelling-subscriptions"></a>
### 定期購入のキャンセル

サブスクリプションをキャンセルするには、ユーザのサブスクリプションで `cancel`メソッドを呼び出します。

    $user->subscription('main')->cancel();

サブスクリプションがキャンセルされると、キャッシャーは自動的にデータベースの `ends_at`列を設定します。 この列は `subscribed`メソッドがいつ` false`を返すべきかを知るために使われます。 たとえば、顧客が3月1日にサブスクリプションをキャンセルしたが、3月5日までサブスクリプションが終了する予定がない場合、3月5日まで `subscribed`メソッドは` true`を返し続けます。

ユーザーがサブスクリプションをキャンセルしても、 `onGracePeriod`メソッドを使って"猶予期間 "に入っているかどうかを判断することができます：

    if ($user->subscription('main')->onGracePeriod()) {
        //
    }

直ちにサブスクリプションをキャンセルする場合は、ユーザのサブスクリプションで `cancelNow`メソッドを呼び出します。

    $user->subscription('main')->cancelNow();

<a name="resuming-subscriptions"></a>
### 定期購読の再開

ユーザーがサブスクリプションをキャンセルして再開したい場合は、 `resume`メソッドを使用します。定期購入を再開するには、ユーザー**は引き続き猶予期間が必要です。

    $user->subscription('main')->resume();

ユーザーがサブスクリプションをキャンセルし、サブスクリプションが完全に期限切れになる前にそのサブスクリプションを再開すると、すぐに請求されることはありません。代わりに、そのサブスクリプションが再度アクティブ化され、元の請求期間に請求されます。

<a name="updating-credit-cards"></a>
### クレジットカードの更新

`updateCard`メソッドを使用して、顧客のクレジットカード情報を更新することができます。このメソッドはStripeトークンを受け取り、新しい請求書をデフォルトの請求元に割り当てます。

    $user->updateCard($stripeToken);

<a name="subscription-trials"></a>
## サブスクリプショントライアル

<a name="with-credit-card-up-front"></a>
### クレジットカードの前に

支払方法情報を収集しながら試用期間を顧客に提供したい場合は、購読を作成するときに `trialDays`メソッドを使用する必要があります。

    $user = User::find(1);

    $user->newSubscription('main', 'monthly')
                ->trialDays(10)
                ->create($stripeToken);

このメソッドは、データベース内のサブスクリプションレコードの試用期間の終了日を設定し、Stripe / Braintreeにこの日以降に請求を開始しないように指示します。

> {note}お客様の定期購入が試験終了日前にキャンセルされない場合、試用期間が終了するとすぐに請求されるため、試験終了日をユーザーに必ず通知してください。

ユーザインスタンスの `onTrial`メソッド、またはサブスクリプションインスタンスの` onTrial`メソッドのいずれかを使って、ユーザが試用期間内にいるかどうかを判断できます。以下の2つの例は同じです。

    if ($user->onTrial('main')) {
        //
    }

    if ($user->subscription('main')->onTrial()) {
        //
    }

<a name="without-credit-card-up-front"></a>
### クレジットカードなし前

ユーザーの支払い方法情報を収集せずに試用期間を提供する場合は、ユーザーレコードの `trial_ends_at`列を希望の試行終了日に設定することができます。これは通常、ユーザー登録時に実行されます。

    $user = User::create([
        // Populate other user properties...
        'trial_ends_at' => now()->addDays(10),
    ]);

> {note}あなたのモデル定義に `trial_ends_at`の[date mutator]（/ docs / {{version}} / eloquent-mutators＃date-mutators）を必ず追加してください。

キャッシャーは、既存のサブスクリプションには添付されていないため、このタイプのトライアルを「汎用トライアル」と呼びます。現在の日付が `trial_ends_at`の値を超えていない場合、` User`インスタンスの `onTrial`メソッドは` true`を返します：

    if ($user->onTrial()) {
        // User is within their trial period...
    }

また、ユーザが "一般的な"試用期間内にあり、実際のサブスクリプションをまだ作成していないことを特に知りたい場合は、 `onGenericTrial`メソッドを使用することもできます：

    if ($user->onGenericTrial()) {
        // User is within their "generic" trial period...
    }

ユーザの実際のサブスクリプションを作成する準備ができたら、通常どおり `newSubscription`メソッドを使用します：

    $user = User::find(1);

    $user->newSubscription('main', 'monthly')->create($stripeToken);

<a name="handling-stripe-webhooks"></a>
## Stripe Webhooksの取り扱い

StripeとBraintreeの両方は、webhooksを介してさまざまなイベントをアプリケーションに通知できます。 Stripe webhooksを処理するには、キャッシャーのwebhookコントローラーを指すルートを定義します。このコントローラは、すべての着信webhook要求を処理し、適切なコントローラメソッドにそれらをディスパッチします。

    Route::post(
        'stripe/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

> {note}経路を登録したら、ストライプコントロールパネルの設定でwebhook URLを設定してください。

デフォルトでは、このコントローラは、（ストライプ設定で定義されているように）失敗した料金が多すぎる契約のキャンセルを自動的に処理します。しかし、すぐにわかるように、このコントローラーを拡張して、好きなWebhookイベントを処理できます。

#### WebhooksとCSRFの保護

Stripe webhooksはLaravelの[CSRF保護]（/ docs / {{version}} / csrf）をバイパスする必要があるため、 `VerifyCsrfToken`ミドルウェアでURIを例外としてリストアップするか、` web`ミドルウェア グループ：

    protected $except = [
        'stripe/*',
    ];

<a name="defining-webhook-event-handlers"></a>
### Webhookイベントハンドラの定義

キャッシャーは不合格料金で自動的に購読キャンセルを処理しますが、処理したいストライプwebhookイベントがある場合は、Webhookコントローラーを延長してください。 あなたのメソッド名はキャッシャーの予定されている規約に対応していなければなりません。具体的には、メソッドの先頭に `handle`を付け、` camel case 'の名前に処理するストライプwebhookを付けます。 たとえば、 `invoice.payment_succeeded` Webhookを処理したい場合は、` handleInvoicePaymentSucceeded`メソッドをコントローラに追加する必要があります：

    <?php

    namespace App\Http\Controllers;

    use Laravel\Cashier\Http\Controllers\WebhookController as CashierController;

    class WebhookController extends CashierController
    {
        /**
         * Handle a Stripe webhook.
         *
         * @param  array  $payload
         * @return Response
         */
        public function handleInvoicePaymentSucceeded($payload)
        {
            // Handle The Event
        }
    }

次に、 `routes / web.php`ファイル内のキャッシャー・コントローラーへのルートを定義します。

    Route::post(
        'stripe/webhook',
        '\App\Http\Controllers\WebhookController@handleWebhook'
    );

<a name="handling-failed-subscriptions"></a>
### 失敗した購読

顧客のクレジットカードの有効期限が切れるとどうなりますか？心配する必要はありません - キャッシャーには、顧客の購読を簡単にキャンセルできるWebhookコントローラーが含まれています。上記のように、コントローラへのルートを指示するだけです。

    Route::post(
        'stripe/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

それでおしまい！失敗した支払いは、コントローラによって取得され、処理されます。ストライプがサブスクリプションが失敗したと判断した場合（通常3回の支払いが失敗した後）、コントローラーは顧客のサブスクリプションをキャンセルします。

<a name="handling-braintree-webhooks"></a>
## Braintree Webhooksの取り扱い

StripeとBraintreeの両方は、webhooksを介してさまざまなイベントをアプリケーションに通知できます。 Braintreeウェブフックを処理するには、キャッシャーのウェブフックコントローラを指すルートを定義します。このコントローラは、すべての着信webhook要求を処理し、適切なコントローラメソッドにそれらをディスパッチします。

    Route::post(
        'braintree/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

> {note}あなたのルートを登録したら、Braintreeのコントロールパネルの設定でwebhook URLを設定してください。

デフォルトでは、このコントローラーは、（Braintreeの設定で定義されているように）失敗した料金が多すぎる契約のキャンセルを自動的に処理します。しかし、すぐにわかるように、このコントローラーを拡張して、好きなWebhookイベントを処理できます。

#### WebhooksとCSRFの保護

Braintree webhooksはLaravelの[CSRF保護]（/ docs / {{version}} / csrf）をバイパスする必要があるので、 `VerifyCsrfToken`ミドルウェアにURIを例外としてリストアップするか、` web`ミドルウェアグループ：

    protected $except = [
        'braintree/*',
    ];

<a name="defining-braintree-webhook-event-handlers"></a>
### Webhookイベントハンドラの定義

Cashierは失敗した料金で自動的に契約キャンセルを処理しますが、Braintree webhookイベントを追加で処理したい場合は、Webhookコントローラを延長してください。あなたのメソッド名は、キャッシャーの予定されている規約に対応していなければなりません。具体的には、メソッドの前に `handle`と` Braint case 'というハンドルネームの名前を付ける必要があります。例えば、 `dispute_opened` webhookを処理したい場合は、` handleDisputeOpened`メソッドをコントローラに追加する必要があります：

    <?php

    namespace App\Http\Controllers;

    use Braintree\WebhookNotification;
    use Laravel\Cashier\Http\Controllers\WebhookController as CashierController;

    class WebhookController extends CashierController
    {
        /**
         * Handle a Braintree webhook.
         *
         * @param  WebhookNotification  $webhook
         * @return Response
         */
        public function handleDisputeOpened(WebhookNotification $notification)
        {
            // Handle The Event
        }
    }

<a name="handling-braintree-failed-subscriptions"></a>
### 失敗した購読

顧客のクレジットカードの有効期限が切れるとどうなりますか？ 心配する必要はありません - キャッシャーには、顧客の購読を簡単にキャンセルできるWebhookコントローラーが含まれています。 コントローラへのルートを指示するだけです：

    Route::post(
        'braintree/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

それでおしまい！ 失敗した支払いは、コントローラによって取得され、処理されます。 Braintreeがサブスクリプションが失敗したと判断した場合（通常3回の支払いが失敗した後）、コントローラーはお客様のサブスクリプションをキャンセルします。 忘れないでください：Braintreeコントロールパネルの設定でwebhook URIを設定する必要があります。

<a name="single-charges"></a>
## シングルチャージ

### シンプルチャージ

> {note} When using Stripe, the `charge` method accepts the amount you would like to charge in the **lowest denominator of the currency used by your application**. However, when using Braintree, you should pass the full dollar amount to the `charge` method:

購読している顧客のクレジットカードに対して「一回限りの」請求をしたい場合は、請求可能なモデルインスタンスで `charge`メソッドを使用することができます。

    // Stripe Accepts Charges In Cents...
    $user->charge(100);

    // Braintree Accepts Charges In Dollars...
    $user->charge(1);

`charge`メソッドは配列を2番目の引数として受け取ります。これにより、基本となるStripe / Braintreeの課金作成に任意のオプションを渡すことができます。料金を作成する際に利用できるオプションについては、StripeまたはBraintreeのマニュアルを参照してください。

    $user->charge(100, [
        'custom_option' => $value,
    ]);

チャージが失敗した場合、 `charge`メソッドは例外をスローします。チャージが成功すると、ストライプ/ブレーンツリーの完全な応答がメソッドから返されます。

    try {
        $response = $user->charge(100);
    } catch (Exception $e) {
        //
    }

### 請求書による請求

場合によっては、ワンタイムチャージを行う必要がある場合もありますが、請求書を生成して顧客にPDF領収書を提供する場合もあります。 `invoiceFor`メソッドはそれを行うことができます。たとえば、お客様に$ 5.00の「ワンタイムフィー」を請求するとします。

    // Stripe Accepts Charges In Cents...
    $user->invoiceFor('One Time Fee', 500);

    // Braintree Accepts Charges In Dollars...
    $user->invoiceFor('One Time Fee', 5);

請求書は、ユーザーのクレジットカードに対して直ちに請求されます。 `invoiceFor`メソッドはまた、配列を3番目の引数として受け取り、基になるStripe / Braintree課金作成に必要なオプションを渡すことができます：

    $user->invoiceFor('One Time Fee', 500, [
        'custom-option' => $value,
    ]);

Braintreeを請求提供者として使用している場合は、 `invoiceFor`メソッドを呼び出すときに` description`オプションを含める必要があります：

    $user->invoiceFor('One Time Fee', 500, [
        'description' => 'your invoice description here',
    ]);


> {note} `invoiceFor`メソッドは、失敗した請求の試みを再試行するStripe請求書を作成します。請求書に失敗した料金を再試行しないようにするには、最初に失敗した料金の請求後にStripe APIを使用して請求書を閉じる必要があります。

<a name="invoices"></a>
## 請求書

`invoices`メソッドを使用して請求可能なモデルの請求書の配列を簡単に取り出すことができます：

    $invoices = $user->invoices();

    // Include pending invoices in the results...
    $invoices = $user->invoicesIncludingPending();

得意先の請求書を一覧表示する際には、請求書のヘルパーメソッドを使用して関連する請求書情報を照会することができます。 たとえば、すべての請求書をテーブルに一覧表示して、ユーザーがそれらのいずれかを簡単にダウンロードできるようにすることができます。

    <table>
        @foreach ($invoices as $invoice)
            <tr>
                <td>{{ $invoice->date()->toFormattedDateString() }}</td>
                <td>{{ $invoice->total() }}</td>
                <td><a href="/user/invoice/{{ $invoice->id }}">Download</a></td>
            </tr>
        @endforeach
    </table>

<a name="generating-invoice-pdfs"></a>
### 請求書のPDFを生成する

ルートまたはコントローラ内から `downloadInvoice`メソッドを使用して、請求書のPDFダウンロードを生成します。 このメソッドは自動的に適切なHTTPレスポンスを生成し、ダウンロードをブラウザに送信します：

    use Illuminate\Http\Request;

    Route::get('user/invoice/{invoice}', function (Request $request, $invoiceId) {
        return $request->user()->downloadInvoice($invoiceId, [
            'vendor'  => 'Your Company',
            'product' => 'Your Product',
        ]);
    });
