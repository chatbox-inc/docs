# コレクション

- [はじめに]（＃序論）
     - [コレクションの作成]（＃creating-collections）
     - [コレクションを拡張する]（＃extended-collections）
- [利用可能なメソッド]（利用可能なメソッド数）
- [上位メッセージ]（上位メッセージ）

<a name="introduction"></a>
## 前書き

`Illuminate \ Support \ Collection`クラスは、データの配列を操作するための流暢で便利なラッパーを提供します。 たとえば、次のコードをチェックアウトします。 `collect`ヘルパーを使って、配列から新しいコレクションインスタンスを作成し、各要素で` strtoupper`関数を実行し、空の要素をすべて削除します：

    $collection = collect(['taylor', 'abigail', null])->map(function ($name) {
        return strtoupper($name);
    })
    ->reject(function ($name) {
        return empty($name);
    });

ご覧のように、 `Collection`クラスを使うと、そのメソッドを連鎖させて、流暢なマッピングを実行し、基礎となる配列を減らすことができます。 一般的に、コレクションは不変です。すべての `Collection`メソッドがまったく新しい` Collection`インスタンスを返すことを意味します。

<a name="creating-collections"></a>
### コレクションを作成する

前述のように、 `collect`ヘルパーは、指定された配列の新しい` Illuminate \ Support \ Collection`インスタンスを返します。 したがって、コレクションを作成するのは簡単です：

    $collection = collect([1, 2, 3]);

> {tip} [Eloquent]（/ docs / {{version}} / eloquent）クエリの結果は常に `Collection`インスタンスとして返されます。

<a name="extending-collections"></a>
### コレクションの拡張

コレクションは「マクロ可能」なので、実行時に `Collection`クラスにメソッドを追加することができます。 たとえば、次のコードは `toUpper`メソッドを` Collection`クラスに追加しています：

    use Illuminate\Support\Str;

    Collection::macro('toUpper', function () {
        return $this->map(function ($value) {
            return Str::upper($value);
        });
    });

    $collection = collect(['first', 'second']);

    $upper = $collection->toUpper();

    // ['FIRST', 'SECOND']

通常、[サービスプロバイダ]（/ docs / {{version}} / providers）にコレクションマクロを宣言する必要があります。

<a name="available-methods"></a>
## 利用可能なメソッド

このドキュメントの残りの部分では、 `Collection`クラスで利用できる各メソッドについて説明します。 これらすべてのメソッドは、基本配列を流暢に操作するために連鎖されることがあることを忘れないでください。 さらに、ほとんどすべてのメソッドは、新しい `Collection`インスタンスを返します。必要に応じてコレクションの元のコピーを保存することができます。

<style>
    #collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    #collection-method-list a {
        display: block;
    }
</style>

<div id="collection-method-list" markdown="1">

[all](#method-all)
[average](#method-average)
[avg](#method-avg)
[chunk](#method-chunk)
[collapse](#method-collapse)
[combine](#method-combine)
[concat](#method-concat)
[contains](#method-contains)
[containsStrict](#method-containsstrict)
[count](#method-count)
[crossJoin](#method-crossjoin)
[dd](#method-dd)
[diff](#method-diff)
[diffAssoc](#method-diffassoc)
[diffKeys](#method-diffkeys)
[dump](#method-dump)
[each](#method-each)
[eachSpread](#method-eachspread)
[every](#method-every)
[except](#method-except)
[filter](#method-filter)
[first](#method-first)
[firstWhere](#method-first-where)
[flatMap](#method-flatmap)
[flatten](#method-flatten)
[flip](#method-flip)
[forget](#method-forget)
[forPage](#method-forpage)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[implode](#method-implode)
[intersect](#method-intersect)
[intersectByKeys](#method-intersectbykeys)
[isEmpty](#method-isempty)
[isNotEmpty](#method-isnotempty)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[macro](#method-macro)
[make](#method-make)
[map](#method-map)
[mapInto](#method-mapinto)
[mapSpread](#method-mapspread)
[mapToGroups](#method-maptogroups)
[mapWithKeys](#method-mapwithkeys)
[max](#method-max)
[median](#method-median)
[merge](#method-merge)
[min](#method-min)
[mode](#method-mode)
[nth](#method-nth)
[only](#method-only)
[pad](#method-pad)
[partition](#method-partition)
[pipe](#method-pipe)
[pluck](#method-pluck)
[pop](#method-pop)
[prepend](#method-prepend)
[pull](#method-pull)
[push](#method-push)
[put](#method-put)
[random](#method-random)
[reduce](#method-reduce)
[reject](#method-reject)
[reverse](#method-reverse)
[search](#method-search)
[shift](#method-shift)
[shuffle](#method-shuffle)
[slice](#method-slice)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[splice](#method-splice)
[split](#method-split)
[sum](#method-sum)
[take](#method-take)
[tap](#method-tap)
[times](#method-times)
[toArray](#method-toarray)
[toJson](#method-tojson)
[transform](#method-transform)
[union](#method-union)
[unique](#method-unique)
[uniqueStrict](#method-uniquestrict)
[unless](#method-unless)
[unwrap](#method-unwrap)
[values](#method-values)
[when](#method-when)
[where](#method-where)
[whereStrict](#method-wherestrict)
[whereIn](#method-wherein)
[whereInStrict](#method-whereinstrict)
[whereNotIn](#method-wherenotin)
[whereNotInStrict](#method-wherenotinstrict)
[wrap](#method-wrap)
[zip](#method-zip)

</div>

<a name="method-listing"></a>
## 方法リスト

<style>
    #collection-method code {
        font-size: 14px;
    }

    #collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="method-all"></a>
#### `all()` {#collection-method .first-collection-method}

`all`メソッドは、コレクションによって表される基本的な配列を返します。

    collect([1, 2, 3])->all();

    // [1, 2, 3]

<a name="method-average"></a>
#### `average()` {#collection-method}

Alias for the [`avg`](#method-avg) method.

<a name="method-avg"></a>
#### `avg()` {#collection-method}

`avg`メソッドは、与えられたキーの[average value]（https://en.wikipedia.org/wiki/Average）を返します：

    $average = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->avg('foo');

    // 20

    $average = collect([1, 1, 2, 4])->avg();

    // 2

<a name="method-chunk"></a>
#### `chunk()` {#collection-method}

`chunk`メソッドは、コレクションを指定されたサイズの複数の小さなコレクションに分割します。

    $collection = collect([1, 2, 3, 4, 5, 6, 7]);

    $chunks = $collection->chunk(4);

    $chunks->toArray();

    // [[1, 2, 3, 4], [5, 6, 7]]

このメソッドは、[ブートストラップ]（https://getbootstrap.com/css/#grid）などのグリッドシステムで作業する場合、[ビュー]（/ docs / {{バージョン}} /ビュー）で特に便利です。 グリッドに表示する[Eloquent]（/ docs / {{version}} / eloquent）モデルのコレクションがあるとします。

    @foreach ($products->chunk(3) as $chunk)
        <div class="row">
            @foreach ($chunk as $product)
                <div class="col-xs-4">{{ $product->name }}</div>
            @endforeach
        </div>
    @endforeach

<a name="method-collapse"></a>
#### `collapse()` {#collection-method}

`collapse`メソッドは、配列のコレクションを単一のフラットなコレクションにまとめます：

    $collection = collect([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    $collapsed = $collection->collapse();

    $collapsed->all();

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-combine"></a>
#### `combine()` {#collection-method}

`combine`メソッドは、コレクションのキーと別の配列またはコレクションの値を組み合わせます：

    $collection = collect(['name', 'age']);

    $combined = $collection->combine(['George', 29]);

    $combined->all();

    // ['name' => 'George', 'age' => 29]

<a name="method-concat"></a>
#### `concat()` {#collection-method}

`concat`メソッドは与えられた`配列 `またはコレクションの値をコレクションの最後に追加します：

    $collection = collect(['John Doe']);

    $concatenated = $collection->concat(['Jane Doe'])->concat(['name' => 'Johnny Doe']);

    $concatenated->all();

    // ['John Doe', 'Jane Doe', 'Johnny Doe']

<a name="method-contains"></a>
#### `contains()` {#collection-method}

`contains`メソッドは、コレクションに与えられたアイテムが含まれているかどうかを判定します：

    $collection = collect(['name' => 'Desk', 'price' => 100]);

    $collection->contains('Desk');

    // true

    $collection->contains('New York');

    // false

指定したペアがコレクションに存在するかどうかを判断する `contains`メソッドにキーと値のペアを渡すこともできます：

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
    ]);

    $collection->contains('product', 'Bookcase');

    // false

最後に、コールバックを `contains`メソッドに渡して、独自の真理テストを実行することもできます：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->contains(function ($value, $key) {
        return $value > 5;
    });

    // false

`contains`メソッドは、項目値をチェックするときに「緩い」比較を使用します。つまり、整数値の文字列は同じ値の整数とみなされます。 厳密な比較を使用してフィルタリングするには、[`containsStrict`]（＃method-containsstrict）メソッドを使用します。

<a name="method-containsstrict"></a>
#### `containsStrict()` {#collection-method}

This method has the same signature as the [`contains`](#method-contains) method; however, all values are compared using "strict" comparisons.

<a name="method-count"></a>
#### `count()` {#collection-method}

`count`メソッドはコレクション内のアイテムの総数を返します：

    $collection = collect([1, 2, 3, 4]);

    $collection->count();

    // 4

<a name="method-crossjoin"></a>
#### `crossJoin()` {#collection-method}

`crossJoin`メソッドは、指定された配列またはコレクションの間でコレクションの値をクロス結合し、すべての可能な順列を持つデカルト積を返します。

    $collection = collect([1, 2]);

    $matrix = $collection->crossJoin(['a', 'b']);

    $matrix->all();

    /*
        [
            [1, 'a'],
            [1, 'b'],
            [2, 'a'],
            [2, 'b'],
        ]
    */

    $collection = collect([1, 2]);

    $matrix = $collection->crossJoin(['a', 'b'], ['I', 'II']);

    $matrix->all();

    /*
        [
            [1, 'a', 'I'],
            [1, 'a', 'II'],
            [1, 'b', 'I'],
            [1, 'b', 'II'],
            [2, 'a', 'I'],
            [2, 'a', 'II'],
            [2, 'b', 'I'],
            [2, 'b', 'II'],
        ]
    */

<a name="method-dd"></a>
#### `dd()` {#collection-method}

`dd`メソッドはコレクションのアイテムをダンプし、スクリプトの実行を終了します：

    $collection = collect(['John Doe', 'Jane Doe']);

    $collection->dd();

    /*
        Collection {
            #items: array:2 [
                0 => "John Doe"
                1 => "Jane Doe"
            ]
        }
    */

スクリプトの実行をやめたくない場合は、代わりに[`dump`]（＃method-dump）メソッドを使用してください。

<a name="method-diff"></a>
#### `diff()` {#collection-method}

`diff`メソッドは、その値に基づいてコレクションを別のコレクションまたは単純なPHPの`配列 `と比較します。 このメソッドは、指定されたコレクションに存在しない元のコレクションの値を返します。

    $collection = collect([1, 2, 3, 4, 5]);

    $diff = $collection->diff([2, 4, 6, 8]);

    $diff->all();

    // [1, 3, 5]

<a name="method-diffassoc"></a>
#### `diffAssoc()` {#collection-method}

`diffAssoc`メソッドは、キーと値に基づいて、コレクションを別のコレクションまたはプレーンなPHPの`配列 `と比較します。 このメソッドは、指定されたコレクションに存在しない元のコレクションのキーと値のペアを返します。

    $collection = collect([
        'color' => 'orange',
        'type' => 'fruit',
        'remain' => 6
    ]);

    $diff = $collection->diffAssoc([
        'color' => 'yellow',
        'type' => 'fruit',
        'remain' => 3,
        'used' => 6
    ]);

    $diff->all();

    // ['color' => 'orange', 'remain' => 6]

<a name="method-diffkeys"></a>
#### `diffKeys()` {#collection-method}

`diffKeys`メソッドは、そのキーに基づいて、コレクションを別のコレクションまたは普通のPHPの`配列 `と比較します。 このメソッドは、指定されたコレクションに存在しない元のコレクションのキーと値のペアを返します。

    $collection = collect([
        'one' => 10,
        'two' => 20,
        'three' => 30,
        'four' => 40,
        'five' => 50,
    ]);

    $diff = $collection->diffKeys([
        'two' => 2,
        'four' => 4,
        'six' => 6,
        'eight' => 8,
    ]);

    $diff->all();

    // ['one' => 10, 'three' => 30, 'five' => 50]

<a name="method-dump"></a>
#### `dump()` {#collection-method}

`dump`メソッドはコレクションのアイテムをダンプします：

    $collection = collect(['John Doe', 'Jane Doe']);

    $collection->dump();

    /*
        Collection {
            #items: array:2 [
                0 => "John Doe"
                1 => "Jane Doe"
            ]
        }
    */

コレクションをダンプした後にスクリプトの実行を停止したい場合は、代わりに[`dd`]（＃method-dd）メソッドを使用してください。

<a name="method-each"></a>
#### `each()` {#collection-method}

`each`メソッドは、コレクション内の項目を繰り返し処理し、各項目をコールバックに渡します。

    $collection = $collection->each(function ($item, $key) {
        //
    });

アイテムの反復処理をやめたい場合は、コールバックから `false`を返すことができます：

    $collection = $collection->each(function ($item, $key) {
        if (/* some condition */) {
            return false;
        }
    });

<a name="method-eachspread"></a>
#### `eachSpread()` {#collection-method}

`eachSpread`メソッドは、コレクションの項目を繰り返し処理し、それぞれのネストされた項目値を指定のコールバックに渡します。

    $collection = collect([['John Doe', 35], ['Jane Doe', 33]]);

    $collection->eachSpread(function ($name, $age) {
        //
    });

あなたは、コールバックから `false`を返すことによって項目を反復することを止めることができます：

    $collection->eachSpread(function ($name, $age) {
        return false;
    });

<a name="method-every"></a>
#### `every()` {#collection-method}

`every`メソッドは、コレクションのすべての要素が与えられた真理テストに合格したことを検証するために使用できます：

    collect([1, 2, 3, 4])->every(function ($value, $key) {
        return $value > 2;
    });

    // false

<a name="method-except"></a>
#### `except()` {#collection-method}

`except`メソッドは、指定されたキーを持つものを除いて、コレクション内のすべてのアイテムを返します。

    $collection = collect(['product_id' => 1, 'price' => 100, 'discount' => false]);

    $filtered = $collection->except(['price', 'discount']);

    $filtered->all();

    // ['product_id' => 1]

`except`の逆の場合は、[only]（＃メソッドのみ）メソッドを参照してください。

<a name="method-filter"></a>
#### `filter()` {#collection-method}

`filter`メソッドは与えられたコールバックを使ってコレクションをフィルタリングし、与えられた真理テストに合格したアイテムだけを保持します：

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->filter(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [3, 4]

コールバックが指定されていない場合、 `false`に相当するコレクションのすべてのエントリが削除されます：

    $collection = collect([1, 2, 3, null, false, '', 0, []]);

    $collection->filter()->all();

    // [1, 2, 3]

`filter`の逆の場合は、[reject]（＃method-reject）メソッドを参照してください。

<a name="method-first"></a>
#### `first()` {#collection-method}

`first`メソッドは与えられた真理テストに合格するコレクションの最初の要素を返します：

    collect([1, 2, 3, 4])->first(function ($value, $key) {
        return $value > 2;
    });

    // 3

引数のない `first`メソッドを呼び出して、コレクションの最初の要素を取得することもできます。 コレクションが空の場合、 `null`が返されます。

    collect([1, 2, 3, 4])->first();

    // 1

<a name="method-first-where"></a>
#### `firstWhere()` {#collection-method}

`firstWhere`メソッドは、与えられたキーと値のペアを持つコレクションの最初の要素を返します。

    $collection = collect([
        ['name' => 'Regena', 'age' => 12],
        ['name' => 'Linda', 'age' => 14],
        ['name' => 'Diego', 'age' => 23],
        ['name' => 'Linda', 'age' => 84],
    ]);

    $collection->firstWhere('name', 'Linda');

    // ['name' => 'Linda', 'age' => 14]

演算子を使って `firstWhere`メソッドを呼び出すこともできます：

    $collection->firstWhere('age', '>=', 18);

    // ['name' => 'Diego', 'age' => 23]

<a name="method-flatmap"></a>
#### `flatMap()` {#collection-method}

`flatMap`メソッドは、コレクションを反復し、各値を指定のコールバックに渡します。 コールバックは、アイテムを修正してそれを返すことが自由であるため、変更されたアイテムの新しいコレクションを形成します。 次に、配列はレベルによって平坦化されます。

    $collection = collect([
        ['name' => 'Sally'],
        ['school' => 'Arkansas'],
        ['age' => 28]
    ]);

    $flattened = $collection->flatMap(function ($values) {
        return array_map('strtoupper', $values);
    });

    $flattened->all();

    // ['name' => 'SALLY', 'school' => 'ARKANSAS', 'age' => '28'];

<a name="method-flatten"></a>
#### `flatten()` {#collection-method}

`flatten`メソッドは、多次元コレクションを単一次元に平坦化します：

    $collection = collect(['name' => 'taylor', 'languages' => ['php', 'javascript']]);

    $flattened = $collection->flatten();

    $flattened->all();

    // ['taylor', 'php', 'javascript'];

オプションで関数に "depth"引数を渡すことができます：

    $collection = collect([
        'Apple' => [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
        ],
        'Samsung' => [
            ['name' => 'Galaxy S7', 'brand' => 'Samsung']
        ],
    ]);

    $products = $collection->flatten(1);

    $products->values()->all();

    /*
        [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
            ['name' => 'Galaxy S7', 'brand' => 'Samsung'],
        ]
    */

この例では、深度を指定せずに `flatten`を呼び出すとネストされた配列が平坦化され、` `'iPhone 6S' '、' Apple '、' Galaxy S7 '、' Samsung ']`という結果になります。 深さを指定すると、フラット化されるネストされた配列のレベルを制限できます。

<a name="method-flip"></a>
#### `flip()` {#collection-method}

`flip`メソッドは、コレクションのキーと対応する値を入れ替えます。

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $flipped = $collection->flip();

    $flipped->all();

    // ['taylor' => 'name', 'laravel' => 'framework']

<a name="method-forget"></a>
#### `forget()` {#collection-method}

`forget`メソッドは、そのキーによってコレクションから項目を削除します：

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $collection->forget('name');

    $collection->all();

    // ['framework' => 'laravel']

> {note}他のほとんどのコレクションメソッドとは異なり、 `forget`は新しい変更コレクションを返しません。 呼び出されたコレクションを変更します。

<a name="method-forpage"></a>
#### `forPage()` {#collection-method}

`forPage`メソッドは、指定されたページ番号に存在するアイテムを含む新しいコレクションを返します。 このメソッドは、最初の引数としてページ番号を受け取り、2番目の引数としてページごとに表示する項目の数を受け取ります。

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunk = $collection->forPage(2, 3);

    $chunk->all();

    // [4, 5, 6]

<a name="method-get"></a>
#### `get()` {#collection-method}

`get`メソッドは、指定されたキーの項目を返します。 キーが存在しない場合、 `null`が返されます：

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('name');

    // taylor

オプションで、デフォルト値を2番目の引数として渡すこともできます。

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('foo', 'default-value');

    // default-value

コールバックをデフォルト値として渡すことさえできます。 指定されたキーが存在しない場合、コールバックの結果が返されます。

    $collection->get('email', function () {
        return 'default-value';
    });

    // default-value

<a name="method-groupby"></a>
#### `groupBy()` {#collection-method}

`groupBy`メソッドはコレクションのアイテムを与えられたキーでグループ化します：

    $collection = collect([
        ['account_id' => 'account-x10', 'product' => 'Chair'],
        ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ['account_id' => 'account-x11', 'product' => 'Desk'],
    ]);

    $grouped = $collection->groupBy('account_id');

    $grouped->toArray();

    /*
        [
            'account-x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'account-x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

文字列 `key`を渡すことに加えて、コールバックを渡すこともできます。 コールバックはグループをキーする値を返します：

    $grouped = $collection->groupBy(function ($item, $key) {
        return substr($item['account_id'], -3);
    });

    $grouped->toArray();

    /*
        [
            'x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

複数のグループ化基準を配列として渡すことができます。 各配列要素は、多次元配列内の対応するレベルに適用されます。

    $data = new Collection([
        10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
        20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
        30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
        40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
    ]);

    $result = $data->groupBy([
        'skill',
        function ($item) {
            return $item['roles'];
        },
    ], $preserveKeys = true);

    /*
    [
        1 => [
            'Role_1' => [
                10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
                20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
            ],
            'Role_2' => [
                20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
            ],
            'Role_3' => [
                10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
            ],
        ],
        2 => [
            'Role_1' => [
                30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
            ],
            'Role_2' => [
                40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
            ],
        ],
    ];
    */

<a name="method-has"></a>
#### `has()` {#collection-method}

`has`メソッドは、指定されたキーがコレクションに存在するかどうかを判定します：

    $collection = collect(['account_id' => 1, 'product' => 'Desk']);

    $collection->has('product');

    // true

<a name="method-implode"></a>
#### `implode()` {#collection-method}

`implode`メソッドは、コレクション内の項目を結合します。 その引数は、コレクション内のアイテムのタイプによって異なります。 コレクションに配列やオブジェクトが含まれている場合は、結合する属性のキーと値の間に配置する "グルー"文字列を渡す必要があります。

    $collection = collect([
        ['account_id' => 1, 'product' => 'Desk'],
        ['account_id' => 2, 'product' => 'Chair'],
    ]);

    $collection->implode('product', ', ');

    // Desk, Chair

コレクションに単純な文字列または数値が含まれている場合は、メソッドに唯一の引数として "グルー"を渡します。

    collect([1, 2, 3, 4, 5])->implode('-');

    // '1-2-3-4-5'

<a name="method-intersect"></a>
#### `intersect()` {#collection-method}

`intersect`メソッドは、指定された配列またはコレクションに存在しない元のコレクションの値をすべて削除します。 結果のコレクションは元のコレクションのキーを保持します：

    $collection = collect(['Desk', 'Sofa', 'Chair']);

    $intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

    $intersect->all();

    // [0 => 'Desk', 2 => 'Chair']

<a name="method-intersectbykeys"></a>
#### `intersectByKeys()` {#collection-method}

`intersectByKeys`メソッドは、指定された配列またはコレクションに存在しないキーを元のコレクションから削除します：

    $collection = collect([
        'serial' => 'UX301', 'type' => 'screen', 'year' => 2009
    ]);

    $intersect = $collection->intersectByKeys([
        'reference' => 'UX404', 'type' => 'tab', 'year' => 2011
    ]);

    $intersect->all();

    // ['type' => 'screen', 'year' => 2009]

<a name="method-isempty"></a>
#### `isEmpty()` {#collection-method}

コレクションが空の場合、 `isEmpty`メソッドは` true`を返します。 それ以外の場合は、 `false`が返されます。

    collect([])->isEmpty();

    // true

<a name="method-isnotempty"></a>
#### `isNotEmpty()` {#collection-method}

コレクションが空でない場合、 `isNotEmpty`メソッドは` true`を返します。 それ以外の場合は、 `false`が返されます。

    collect([])->isNotEmpty();

    // false

<a name="method-keyby"></a>
#### `keyBy()` {#collection-method}

`keyBy`メソッドは、指定されたキーでコレクションをキーします。 複数のアイテムが同じキーを持つ場合、最後のコレクションのみが新しいコレクションに表示されます。

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $keyed = $collection->keyBy('product_id');

    $keyed->all();

    /*
        [
            'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

メソッドにコールバックを渡すこともできます。 コールバックは、コレクションをキーするための値を返します。

    $keyed = $collection->keyBy(function ($item) {
        return strtoupper($item['product_id']);
    });

    $keyed->all();

    /*
        [
            'PROD-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'PROD-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

<a name="method-keys"></a>
#### `keys()` {#collection-method}

`keys`メソッドはすべてのコレクションのキーを返します：

    $collection = collect([
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $keys = $collection->keys();

    $keys->all();

    // ['prod-100', 'prod-200']

<a name="method-last"></a>
#### `last()` {#collection-method}

`last`メソッドは、与えられた真理テストに合格するコレクションの最後の要素を返します：

    collect([1, 2, 3, 4])->last(function ($value, $key) {
        return $value < 3;
    });

    // 2

引数のない `last`メソッドを呼び出して、コレクションの最後の要素を取得することもできます。 コレクションが空の場合、 `null`が返されます。

    collect([1, 2, 3, 4])->last();

    // 4

<a name="method-macro"></a>
#### `macro()` {#collection-method}

静的 `macro`メソッドを使うと、実行時に` Collection`クラスにメソッドを追加することができます。 詳細については、[コレクションの拡張]（＃extended-collections）のドキュメントを参照してください。

<a name="method-make"></a>
#### `make()` {#collection-method}

静的 `make`メソッドは、新しいコレクションインスタンスを作成します。 [コレクションの作成]（＃creating-collections）セクションを参照してください。

<a name="method-map"></a>
#### `map()` {#collection-method}

`map`メソッドは、コレクションを反復し、各値を指定のコールバックに渡します。 コールバックは、項目を変更してそれを返すことが自由であるため、変更された項目の新しいコレクションを形成します。

    $collection = collect([1, 2, 3, 4, 5]);

    $multiplied = $collection->map(function ($item, $key) {
        return $item * 2;
    });

    $multiplied->all();

    // [2, 4, 6, 8, 10]

> {note}他のほとんどのコレクションメソッドと同様に、 `map`は新しいコレクションインスタンスを返します。 それが呼び出されたコレクションは変更されません。 元のコレクションを変換する場合は、[`transform`]（＃method-transform）メソッドを使用します。

<a name="method-mapinto"></a>
#### `mapInto()` {#collection-method}

`mapInto（）`メソッドは、コレクションを繰り返し処理し、その値をコンストラクタに渡すことで、指定されたクラスの新しいインスタンスを作成します。

    class Currency
    {
        /**
         * Create a new currency instance.
         *
         * @param  string  $code
         * @return void
         */
        function __construct(string $code)
        {
            $this->code = $code;
        }
    }

    $collection = collect(['USD', 'EUR', 'GBP']);

    $currencies = $collection->mapInto(Currency::class);

    $currencies->all();

    // [Currency('USD'), Currency('EUR'), Currency('GBP')]

<a name="method-mapspread"></a>
#### `mapSpread()` {#collection-method}

`mapSpread`メソッドはコレクションのアイテムを繰り返し処理し、それぞれのネストされたアイテムの値を指定のコールバックに渡します。 コールバックは、項目を変更してそれを返すことが自由であるため、変更された項目の新しいコレクションを形成します。

    $collection = collect([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunks = $collection->chunk(2);

    $sequence = $chunks->mapSpread(function ($odd, $even) {
        return $odd + $even;
    });

    $sequence->all();

    // [1, 5, 9, 13, 17]

<a name="method-maptogroups"></a>
#### `mapToGroups()` {#collection-method}

`mapToGroups`メソッドは、指定されたコールバックによってコレクションのアイテムをグループ化します。 コールバックは、単一のキーと値のペアを含む連想配列を返し、グループ化された値の新しいコレクションを形成します。

    $collection = collect([
        [
            'name' => 'John Doe',
            'department' => 'Sales',
        ],
        [
            'name' => 'Jane Doe',
            'department' => 'Sales',
        ],
        [
            'name' => 'Johnny Doe',
            'department' => 'Marketing',
        ]
    ]);

    $grouped = $collection->mapToGroups(function ($item, $key) {
        return [$item['department'] => $item['name']];
    });

    $grouped->toArray();

    /*
        [
            'Sales' => ['John Doe', 'Jane Doe'],
            'Marketing' => ['Johhny Doe'],
        ]
    */

    $grouped->get('Sales')->all();

    // ['John Doe', 'Jane Doe']

<a name="method-mapwithkeys"></a>
#### `mapWithKeys()` {#collection-method}

`mapWithKeys`メソッドは、コレクションを反復し、各値を指定のコールバックに渡します。 コールバックは、単一のキーと値のペアを含む連想配列を返します。

    $collection = collect([
        [
            'name' => 'John',
            'department' => 'Sales',
            'email' => 'john@example.com'
        ],
        [
            'name' => 'Jane',
            'department' => 'Marketing',
            'email' => 'jane@example.com'
        ]
    ]);

    $keyed = $collection->mapWithKeys(function ($item) {
        return [$item['email'] => $item['name']];
    });

    $keyed->all();

    /*
        [
            'john@example.com' => 'John',
            'jane@example.com' => 'Jane',
        ]
    */

<a name="method-max"></a>
#### `max()` {#collection-method}

`max`メソッドは与えられたキーの最大値を返します：

    $max = collect([['foo' => 10], ['foo' => 20]])->max('foo');

    // 20

    $max = collect([1, 2, 3, 4, 5])->max();

    // 5

<a name="method-median"></a>
#### `median()` {#collection-method}

`median`メソッドは、与えられたキーの[median value]（https://en.wikipedia.org/wiki/Median）を返します：

    $median = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->median('foo');

    // 15

    $median = collect([1, 1, 2, 4])->median();

    // 1.5

<a name="method-merge"></a>
#### `merge()` {#collection-method}

`merge`メソッドは、指定された配列またはコレクションを元のコレクションとマージします。 指定されたアイテムの文字列キーが元のコレクションの文字列キーと一致する場合、指定されたアイテムの値は元のコレクションの値を上書きします。

    $collection = collect(['product_id' => 1, 'price' => 100]);

    $merged = $collection->merge(['price' => 200, 'discount' => false]);

    $merged->all();

    // ['product_id' => 1, 'price' => 200, 'discount' => false]

指定された項目のキーが数値の場合、値はコレクションの末尾に追加されます。

    $collection = collect(['Desk', 'Chair']);

    $merged = $collection->merge(['Bookcase', 'Door']);

    $merged->all();

    // ['Desk', 'Chair', 'Bookcase', 'Door']

<a name="method-min"></a>
#### `min()` {#collection-method}

`min`メソッドは与えられたキーの最小値を返します：

    $min = collect([['foo' => 10], ['foo' => 20]])->min('foo');

    // 10

    $min = collect([1, 2, 3, 4, 5])->min();

    // 1

<a name="method-mode"></a>
#### `mode()` {#collection-method}

`mode`メソッドは、与えられたキーの[モード値]（https://en.wikipedia.org/wiki/Mode_（statistics））を返します：

    $mode = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->mode('foo');

    // [10]

    $mode = collect([1, 1, 2, 4])->mode();

    // [1]

<a name="method-nth"></a>
#### `nth()` {#collection-method}

`nth`メソッドはn番目の要素からなる新しいコレクションを作成します：

    $collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);

    $collection->nth(4);

    // ['a', 'e']

オプションでオフセットを2番目の引数として渡すこともできます：

    $collection->nth(4, 1);

    // ['b', 'f']

<a name="method-only"></a>
#### `only()` {#collection-method}

`only`メソッドは、指定されたキーを持つコレクション内の項目を返します：

    $collection = collect(['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]);

    $filtered = $collection->only(['product_id', 'name']);

    $filtered->all();

    // ['product_id' => 1, 'name' => 'Desk']

`only`の逆の場合は、[except]（＃method-except）メソッドを参照してください。

<a name="method-pad"></a>
#### `pad()` {#collection-method}

`pad`メソッドは、配列が指定されたサイズに達するまで、指定された値で配列を埋めます。 このメソッドは[array_pad]（https://secure.php.net/manual/en/function.array-pad.php）PHP関数のように動作します。

左に貼り付けるには、負のサイズを指定する必要があります。 指定されたサイズの絶対値が配列の長さ以下の場合、パディングは行われません。

    $collection = collect(['A', 'B', 'C']);

    $filtered = $collection->pad(5, 0);

    $filtered->all();

    // ['A', 'B', 'C', 0, 0]

    $filtered = $collection->pad(-5, 0);

    $filtered->all();

    // [0, 0, 'A', 'B', 'C']

<a name="method-partition"></a>
#### `partition()` {#collection-method}

`partition`メソッドは` list` PHP関数と組み合わせて、与えられた真理テストをパスする要素とそうでない要素を分離することができます：

    $collection = collect([1, 2, 3, 4, 5, 6]);

    list($underThree, $aboveThree) = $collection->partition(function ($i) {
        return $i < 3;
    });

<a name="method-pipe"></a>
#### `pipe()` {#collection-method}

`pipe`メソッドはコレクションを与えられたコールバックに渡し、結果を返します：

    $collection = collect([1, 2, 3]);

    $piped = $collection->pipe(function ($collection) {
        return $collection->sum();
    });

    // 6

<a name="method-pluck"></a>
#### `pluck()` {#collection-method}

`pluck`メソッドは、与えられたキーのすべての値を取得します：

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $plucked = $collection->pluck('name');

    $plucked->all();

    // ['Desk', 'Chair']

結果として得られるコレクションのキーイング方法を指定することもできます。

    $plucked = $collection->pluck('name', 'product_id');

    $plucked->all();

    // ['prod-100' => 'Desk', 'prod-200' => 'Chair']

<a name="method-pop"></a>
#### `pop()` {#collection-method}

`pop`メソッドはコレクションの最後のアイテムを削除して返します：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->pop();

    // 5

    $collection->all();

    // [1, 2, 3, 4]

<a name="method-prepend"></a>
#### `prepend()` {#collection-method}

`prepend`メソッドはコレクションの先頭にアイテムを追加します：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->prepend(0);

    $collection->all();

    // [0, 1, 2, 3, 4, 5]

また、前に付いた項目のキーを設定するために2番目の引数を渡すこともできます：

    $collection = collect(['one' => 1, 'two' => 2]);

    $collection->prepend(0, 'zero');

    $collection->all();

    // ['zero' => 0, 'one' => 1, 'two' => 2]

<a name="method-pull"></a>
#### `pull()` {#collection-method}

`pull`メソッドは、そのキーによってコレクションから項目を削除して返します：

    $collection = collect(['product_id' => 'prod-100', 'name' => 'Desk']);

    $collection->pull('name');

    // 'Desk'

    $collection->all();

    // ['product_id' => 'prod-100']

<a name="method-push"></a>
#### `push()` {#collection-method}

`push`メソッドはコレクションの最後にアイテムを追加します：

    $collection = collect([1, 2, 3, 4]);

    $collection->push(5);

    $collection->all();

    // [1, 2, 3, 4, 5]

<a name="method-put"></a>
#### `put()` {#collection-method}

`put`メソッドは、コレクション内の指定されたキーと値を設定します：

    $collection = collect(['product_id' => 1, 'name' => 'Desk']);

    $collection->put('price', 100);

    $collection->all();

    // ['product_id' => 1, 'name' => 'Desk', 'price' => 100]

<a name="method-random"></a>
#### `random()` {#collection-method}

`random`メソッドはコレクションからランダムな項目を返します：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->random();

    // 4 - (retrieved randomly)

オプションで、ランダムに検索するアイテムの数を指定するために、整数に `random`を渡すことができます。 受け取るアイテムの数を明示的に渡すと、アイテムのコレクションが常に返されます。

    $random = $collection->random(3);

    $random->all();

    // [2, 4, 5] - (retrieved randomly)

<a name="method-reduce"></a>
#### `reduce()` {#collection-method}

`reduce`メソッドはコレクションを単一の値に減らし、各反復の結果を次の反復に渡します：

    $collection = collect([1, 2, 3]);

    $total = $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    });

    // 6

最初の反復での `$ carry`の値は` null`です。 ただし、第2引数を `reduce`に渡すことで初期値を指定することができます：

    $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    }, 4);

    // 10

<a name="method-reject"></a>
#### `reject()` {#collection-method}

`reject`メソッドは、指定されたコールバックを使ってコレクションをフィルタリングします。 結果のコレクションから項目を削除する必要がある場合、コールバックは `true`を返します。

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->reject(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [1, 2]

`reject`メソッドの逆については、[` filter`]（＃method-filter）メソッドを参照してください。

<a name="method-reverse"></a>
#### `reverse()` {#collection-method}

`reject`メソッドの逆については、[` filter`]（＃method-filter）メソッドを参照してください。

    $collection = collect(['a', 'b', 'c', 'd', 'e']);

    $reversed = $collection->reverse();

    $reversed->all();

    /*
        [
            4 => 'e',
            3 => 'd',
            2 => 'c',
            1 => 'b',
            0 => 'a',
        ]
    */

<a name="method-search"></a>
#### `search()` {#collection-method}

`search`メソッドは、指定された値をコレクション内で検索し、見つかった場合はそのキーを返します。 項目が見つからない場合、 `false`が返されます。

    $collection = collect([2, 4, 6, 8]);

    $collection->search(4);

    // 1

検索は「ルーズ」比較を使用して行われます。つまり、整数値の文字列は同じ値の整数とみなされます。 厳密な比較を使用するには、メソッドの第2引数として `true`を渡します：

    $collection->search('4', true);

    // false

あるいは、自分のコールバックを渡して、真理テストに合格した最初のアイテムを検索することもできます。

    $collection->search(function ($item, $key) {
        return $item > 5;
    });

    // 2

<a name="method-shift"></a>
#### `shift()` {#collection-method}

`shift`メソッドは、コレクションから最初のアイテムを削除して返します：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->shift();

    // 1

    $collection->all();

    // [2, 3, 4, 5]

<a name="method-shuffle"></a>
#### `shuffle()` {#collection-method}

`shuffle`メソッドはコレクション内のアイテムをランダムにシャッフルします：

    $collection = collect([1, 2, 3, 4, 5]);

    $shuffled = $collection->shuffle();

    $shuffled->all();

    // [3, 2, 5, 1, 4] - (generated randomly)

<a name="method-slice"></a>
#### `slice()` {#collection-method}

`slice`メソッドは、指定されたインデックスから始まるコレクションのスライスを返します。

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $slice = $collection->slice(4);

    $slice->all();

    // [5, 6, 7, 8, 9, 10]

返されるスライスのサイズを制限する場合は、メソッドの2番目の引数として必要なサイズを渡します。

    $slice = $collection->slice(4, 2);

    $slice->all();

    // [5, 6]

The returned slice will preserve keys by default. If you do not wish to preserve the original keys, you can use the [`values`](#method-values) method to reindex them.

<a name="method-sort"></a>
#### `sort()` {#collection-method}

`sort`メソッドはコレクションをソートします。 ソートされたコレクションは元の配列キーを保持するので、この例では、[`values`]（＃method-values）メソッドを使用して、キーを連続した番号のインデックスにリセットします。

    $collection = collect([5, 3, 1, 2, 4]);

    $sorted = $collection->sort();

    $sorted->values()->all();

    // [1, 2, 3, 4, 5]

ソートの必要性がさらに進んだ場合は、独自のアルゴリズムでコールバックを `sort`に渡すことができます。 [`uasort`]（https://secure.php.net/manual/en/function.uasort.php#refsect1-function.uasort-parameters）のPHPドキュメントを参照してください。コレクションの` sort`メソッド ボンネットの下で電話する。

> {tip}ネストされた配列やオブジェクトのコレクションをソートする必要がある場合は、[`sortBy`]（＃method-sortby）メソッドと[` sortByDesc`]（＃method-sortbydesc）メソッドを参照してください。

<a name="method-sortby"></a>
#### `sortBy()` {#collection-method}

`sortBy`メソッドは、指定されたキーでコレクションをソートします。 ソートされたコレクションは元の配列キーを保持するので、この例では、[`values`]（＃method-values）メソッドを使用して、キーを連続した番号のインデックスにリセットします。

    $collection = collect([
        ['name' => 'Desk', 'price' => 200],
        ['name' => 'Chair', 'price' => 100],
        ['name' => 'Bookcase', 'price' => 150],
    ]);

    $sorted = $collection->sortBy('price');

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'price' => 100],
            ['name' => 'Bookcase', 'price' => 150],
            ['name' => 'Desk', 'price' => 200],
        ]
    */

独自のコールバックを渡してコレクション値をソートする方法を決定することもできます。

    $collection = collect([
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $sorted = $collection->sortBy(function ($product, $key) {
        return count($product['colors']);
    });

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'colors' => ['Black']],
            ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
            ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
        ]
    */

<a name="method-sortbydesc"></a>
#### `sortByDesc()` {#collection-method}

このメソッドは[`sortBy`]（＃method-sortby）メソッドと同じシグネチャを持ちますが、逆の順序でコレクションをソートします。

<a name="method-splice"></a>
#### `splice()` {#collection-method}

`splice`メソッドは、指定されたインデックスから始まる項目のスライスを削除して返します：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2);

    $chunk->all();

    // [3, 4, 5]

    $collection->all();

    // [1, 2]

結果のチャンクのサイズを制限するために2番目の引数を渡すことができます：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 4, 5]

さらに、新しい項目を含む3番目の引数を渡して、コレクションから削除された項目を置き換えることもできます。

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1, [10, 11]);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 10, 11, 4, 5]

<a name="method-split"></a>
#### `split()` {#collection-method}

`split`メソッドはコレクションを与えられた数のグループに分割します：

    $collection = collect([1, 2, 3, 4, 5]);

    $groups = $collection->split(3);

    $groups->toArray();

    // [[1, 2], [3, 4], [5]]

<a name="method-sum"></a>
#### `sum()` {#collection-method}

`sum`メソッドは、コレクション内のすべてのアイテムの合計を返します。

    collect([1, 2, 3, 4, 5])->sum();

    // 15

コレクションにネストされた配列やオブジェクトが含まれている場合は、どの値を合計するかを決定するためにキーを渡す必要があります。

    $collection = collect([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);

    $collection->sum('pages');

    // 1272

さらに、自分のコールバックを渡して、合計するコレクションの値を決めることもできます。

    $collection = collect([
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $collection->sum(function ($product) {
        return count($product['colors']);
    });

    // 6

<a name="method-take"></a>
#### `take()` {#collection-method}

`take`メソッドは指定された数の項目を持つ新しいコレクションを返します：

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(3);

    $chunk->all();

    // [0, 1, 2]

負の整数を渡して、指定した量のアイテムをコレクションの最後から取得することもできます。

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(-2);

    $chunk->all();

    // [4, 5]

<a name="method-tap"></a>
#### `tap()` {#collection-method}

`tap`メソッドはコレクションを指定のコールバックに渡します。これにより、特定のポイントでコレクションに「タップ」し、アイテム自体に影響を与えずに何かを行うことができます：

    collect([2, 4, 3, 1, 5])
        ->sort()
        ->tap(function ($collection) {
            Log::debug('Values after sorting', $collection->values()->toArray());
        })
        ->shift();

    // 1

<a name="method-times"></a>
#### `times()` {#collection-method}

静的 `times`メソッドは、コールバックを一定の時間だけ呼び出すことによって新しいコレクションを作成します：

    $collection = Collection::times(10, function ($number) {
        return $number * 9;
    });

    $collection->all();

    // [9, 18, 27, 36, 45, 54, 63, 72, 81, 90]

このメソッドは、[Eloquent]（/ docs / {{version}} / eloquent）モデルを作成するファクトリと組み合わせて使用すると便利です。

    $categories = Collection::times(3, function ($number) {
        return factory(Category::class)->create(['name' => 'Category #'.$number]);
    });

    $categories->all();

    /*
        [
            ['id' => 1, 'name' => 'Category #1'],
            ['id' => 2, 'name' => 'Category #2'],
            ['id' => 3, 'name' => 'Category #3'],
        ]
    */

<a name="method-toarray"></a>
#### `toArray()` {#collection-method}

`toArray`メソッドは、コレクションを単純なPHPの`配列 `に変換します。 コレクションの値が[Eloquent]（/ docs / {{version}} / eloquent）モデルの場合、モデルも配列に変換されます。

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toArray();

    /*
        [
            ['name' => 'Desk', 'price' => 200],
        ]
    */

> {note} `toArray`はコレクションのすべてのネストされたオブジェクトも配列に変換します。 生の配列を取得する場合は、代わりに[`all`]（＃method-all）メソッドを使用してください。

<a name="method-tojson"></a>
#### `toJson()` {#collection-method}

`toJson`メソッドは、コレクションをJSON直列化文字列に変換します：

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toJson();

    // '{"name":"Desk", "price":200}'

<a name="method-transform"></a>
#### `transform()` {#collection-method}

`transform`メソッドは、コレクションを反復し、コレクション内の各アイテムで指定のコールバックを呼び出します。 コレクション内の項目は、コールバックから返された値に置き換えられます。

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->transform(function ($item, $key) {
        return $item * 2;
    });

    $collection->all();

    // [2, 4, 6, 8, 10]

> {note}他のほとんどのコレクションメソッドとは異なり、 `transform`はコレクション自体を変更します。 代わりに新しいコレクションを作成する場合は、[`map`]（＃method-map）メソッドを使用してください。

<a name="method-union"></a>
#### `union()` {#collection-method}

`union`メソッドは、指定された配列をコレクションに追加します。 指定された配列に、すでに元のコレクションにあるキーが含まれている場合、元のコレクションの値が優先されます。

    $collection = collect([1 => ['a'], 2 => ['b']]);

    $union = $collection->union([3 => ['c'], 1 => ['b']]);

    $union->all();

    // [1 => ['a'], 2 => ['b'], 3 => ['c']]

<a name="method-unique"></a>
#### `unique()` {#collection-method}

`unique`メソッドは、コレクション内のすべてのユニークな項目を返します。 返されたコレクションは元の配列キーを保持するので、この例では[`values`]（＃method-values）メソッドを使用してキーを連続した番号のインデックスにリセットします：

    $collection = collect([1, 1, 2, 2, 3, 4, 2]);

    $unique = $collection->unique();

    $unique->values()->all();

    // [1, 2, 3, 4]

ネストされた配列やオブジェクトを扱うときは、一意性を判断するために使用するキーを指定することができます。

    $collection = collect([
        ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'iPhone 5', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
        ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
    ]);

    $unique = $collection->unique('brand');

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ]
    */

独自のコールバックを渡してアイテムの一意性を判断することもできます。

    $unique = $collection->unique(function ($item) {
        return $item['brand'].$item['type'];
    });

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
            ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
        ]
    */

`unique`メソッドは項目値をチェックするときに"緩やかな "比較を使います。つまり、整数値の文字列は同じ値の整数とみなされます。 [厳密な比較を使用してフィルタリングするには、[uniqueStrict`]（＃method-uniquestrict）メソッドを使用します。

<a name="method-uniquestrict"></a>
#### `uniqueStrict()` {#collection-method}

このメソッドは[`unique`]（＃method-unique）メソッドと同じシグネチャを持ちます。 ただし、すべての値は「厳密な」比較を使用して比較されます。

<a name="method-unless"></a>
#### `unless()` {#collection-method}

`unless`メソッドは、メソッドに与えられた最初の引数が` true`と評価されない限り、与えられたコールバックを実行します：

    $collection = collect([1, 2, 3]);

    $collection->unless(true, function ($collection) {
        return $collection->push(4);
    });

    $collection->unless(false, function ($collection) {
        return $collection->push(5);
    });

    $collection->all();

    // [1, 2, 3, 5]

`unless`の逆については、` `when`（＃method-when）メソッドを参照してください。

<a name="method-unwrap"></a>
#### `unwrap()` {#collection-method}

静的な `unwrap`メソッドは、適用可能であれば、与えられた値からコレクションの根底にある項目を返します。

    Collection::unwrap(collect('John Doe'));

    // ['John Doe']

    Collection::unwrap(['John Doe']);

    // ['John Doe']

    Collection::unwrap('John Doe');

    // 'John Doe'

<a name="method-values"></a>
#### `values()` {#collection-method}

`values`メソッドはキーが連続した整数にリセットされた新しいコレクションを返します：

    $collection = collect([
        10 => ['product' => 'Desk', 'price' => 200],
        11 => ['product' => 'Desk', 'price' => 200]
    ]);

    $values = $collection->values();

    $values->all();

    /*
        [
            0 => ['product' => 'Desk', 'price' => 200],
            1 => ['product' => 'Desk', 'price' => 200],
        ]
    */

<a name="method-when"></a>
#### `when()` {#collection-method}

`when`メソッドは、メソッドに与えられた最初の引数が` true`と評価されたときに、与えられたコールバックを実行します：

    $collection = collect([1, 2, 3]);

    $collection->when(true, function ($collection) {
        return $collection->push(4);
    });

    $collection->when(false, function ($collection) {
        return $collection->push(5);
    });

    $collection->all();

    // [1, 2, 3, 4]

For the inverse of `when`, see the [`unless`](#method-unless) method.

<a name="method-where"></a>
#### `where()` {#collection-method}

`where`メソッドは与えられたキーと値のペアでコレクションをフィルタリングします：

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->where('price', 100);

    $filtered->all();

    /*
        [
            ['product' => 'Chair', 'price' => 100],
            ['product' => 'Door', 'price' => 100],
        ]
    */

`where`メソッドは、項目値をチェックするときに「緩い」比較を使用します。つまり、整数値の文字列は同じ値の整数とみなされます。 [厳密な比較を使用してフィルタリングするには、[`whereStrict`]（＃method-wherestrict）メソッドを使用します。

<a name="method-wherestrict"></a>
#### `whereStrict()` {#collection-method}

このメソッドは、[`where`]（＃method-where）メソッドと同じシグネチャを持ちます。 ただし、すべての値は「厳密な」比較を使用して比較されます。

<a name="method-wherein"></a>
#### `whereIn()` {#collection-method}

`whereIn`メソッドは、指定された配列に含まれるキー/値によってコレクションをフィルタリングします：

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereIn('price', [150, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Bookcase', 'price' => 150],
            ['product' => 'Desk', 'price' => 200],
        ]
    */

`whereIn`メソッドは項目値をチェックするときに"緩やかな "比較を使用します。つまり、整数値の文字列は同じ値の整数とみなされます。 [厳密な比較を使用してフィルタリングするには、[`whereInStrict`]（＃method-whomstrict）メソッドを使用します。

<a name="method-whereinstrict"></a>
#### `whereInStrict()` {#collection-method}

このメソッドは[`whereIn`]（＃method-whom）メソッドと同じシグネチャを持ちます。 ただし、すべての値は「厳密な」比較を使用して比較されます。

<a name="method-wherenotin"></a>
#### `whereNotIn()` {#collection-method}

`whereNotIn`メソッドは、指定された配列に含まれていないキー/値によってコレクションをフィルタリングします：

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereNotIn('price', [150, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Chair', 'price' => 100],
            ['product' => 'Door', 'price' => 100],
        ]
    */

whereNotIn`メソッドは項目値をチェックするときに"緩やかな "比較を使います。つまり、整数値の文字列は同じ値の整数とみなされます。 [厳密な比較を使用してフィルタリングするには、[`whereNotInStrict`]（＃method-wherenotinstrict）メソッドを使用します。

<a name="method-wherenotinstrict"></a>
#### `whereNotInStrict()` {#collection-method}

このメソッドは[`whereNotIn`]（＃method-wherenotin）メソッドと同じシグネチャを持ちます。 ただし、すべての値は「厳密な」比較を使用して比較されます。

<a name="method-wrap"></a>
#### `wrap()` {#collection-method}

静的な `wrap`メソッドは、指定された値を該当する場合はコレクションにラップします：

    $collection = Collection::wrap('John Doe');

    $collection->all();

    // ['John Doe']

    $collection = Collection::wrap(['John Doe']);

    $collection->all();

    // ['John Doe']

    $collection = Collection::wrap(collect('John Doe'));

    $collection->all();

    // ['John Doe']

<a name="method-zip"></a>
#### `zip()` {#collection-method}

`zip`メソッドは、指定された配列の値を、対応するインデックスにある元のコレクションの値と一緒にマージします：

    $collection = collect(['Chair', 'Desk']);

    $zipped = $collection->zip([100, 200]);

    $zipped->all();

    // [['Chair', 100], ['Desk', 200]]

<a name="higher-order-messages"></a>
## 高次メッセージ

コレクションはまた、コレクションの共通の操作を実行するためのショートカットである「上位メッセージ」のサポートも提供します。 より高次のメッセージを提供する収集方法は、「平均」、「平均」、「包含」、「各」、「すべて」、「フィルタ」、「最初」、「フラットマップ」、「マップ」、 `reject`、` sortBy`、 `sortByDesc`、` sum`、および `unique`です。

上位の各メッセージは、コレクションインスタンスの動的プロパティとしてアクセスできます。 例えば、コレクション内の各オブジェクトのメソッドを呼び出すために、 `each`という上位メッセージを使用しましょう。

    $users = User::where('votes', '>', 500)->get();

    $users->each->markAsVip();

同様に、「総和」上位メッセージを使用して、ユーザーの集まりに対する「投票」の総数を集めることができます。

    $users = User::where('group', 'Development')->get();

    return $users->sum->votes;
